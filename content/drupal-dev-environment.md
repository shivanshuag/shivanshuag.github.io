+++
Categories = ["Hacking"]
Description = "Setting up development environment for durpal module development"
Tags = ["Drupal 8", "GSoC 2015", "Site Audit", "Drush"]
date = "2015-06-24T13:35:54+05:30"
menu = "main"
title = "GSoC 2015 Extending Site Audit - Setting Up Dev Environment"
+++

I was selected for [Google Summer Of Code 2015](https://www.google-melange.com/gsoc/homepage/google/gsoc2015) program under [Drupal](https://drupal.org). My project is to extend [Site Audit](https://drupal.org/project/site_audit) under the mentorship of [Jon Peck](https://www.drupal.org/u/fluxsauce) and my proposal for the project can be found [here](https://docs.google.com/document/d/1VFZHZvPb-hulxN9jTF6gKIkpnWA7xCRZzADqqWGPLdI/edit?usp=sharing).

Weeks before the coding period started, the first challenge was to set up a developemnt environment such that both me and my mentor can test the work I do in a consistent way. Following are some of the things I did and learned in the process.

## Installing Composer
Composer is a package manager for PHP. Pcakage manager for most of the linux distros have `composer` in their official repositories. So, intalling it is simple. In my case, I used
`sudo pacman -S php-composer`

## Installing drush
Drush is a command line shell for Drupal. Drupal 8 requires version 7 of Drush. To install drush 7 from the latest HEAD using composer, run the following command
`composer global require drush/drush:dev-master`

## Install drupal 7.36 via drush
* Download drupal 7.36 using `drush dl drupal --drupal-project-rename=drupal7`

* Install it by running the following command inside the `drupal7` director
  {{< highlight bash >}}
  drush site-install standard \
  --db-url='mysql://[username]:[password]@localhost/[database-name]'\
  --site-name='[site name]' --account-name=[admin-username]\
  --account-pass=[admin-password]
  {{< /highlight >}}

* Use `drush en [module-name]` command to install all the required modules

* Alternatively, use drush make with the following make file and then do a drush site-install

    {{< highlight bash >}}
  ;full documentation for drush make file at http://www.drush.org/en/master/make/
  core = 7.x
  api = 2
  projects[drupal][version] = 7.36
  projects[] = views
  projects[] = coder
  {{< /highlight >}}

## Install drupal-8.0.0-beta10 via drush
* Download the specified version of drupal using `drush dl drupal-8.0.0-beta10 --drupal-project-rename=drupal8`

* Install it in a manner similar to drupal 7

## Setting up drush aliases
Drush aliases provide a way to run drush commands on a drupal site from any directory.

* In the .drush directory in the home folder, put a file named `[sitename].aliases.drushrc.php` with the following content

    {{< highlight php >}}
    #Example aliases file at https://github.com/drush-ops/drush/blob/master/examples/example.aliases.drushrc.php
    <?php
    $aliases["d8"] = array (
      'root' => '/home/shivanshu/webapps/drupal8',
      'uri' => 'http://drupal8.localhost',
    );

    $aliases["d7"] = array (
      'root' => '/home/shivanshu/webapps/drupal7',
      'uri' => 'http://drupal7.localhost',
    );
    ?>
    {{< /highlight >}}

* Now, the drush commands can be run from any directory in the following manner
`drush @d7 en [module-name]`

* To list all enabled modules, use `drush @d7 pm-list --type=Module --status=enabled`

## Backing up and Restoring site using Drush
* `drush archive-dump` will backup the website in an archive

* `drush archive-restore [path to archive]` will restore the backed up archive in the folder where the command is run 
