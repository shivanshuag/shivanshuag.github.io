---
layout: post
title: Setting Up PHP_CodeSniffer for Drupal 7 and 8
description: Setting up CodeSniffer for Drupal 7 and 8 developement in the same environment
category: hacking
tags: [GSoC 2015, drupal, PHP_CodeSniffer]
comments: true
share: true
---

[PHP CodeSniffer](https://github.com/squizlabs/PHP_CodeSniffer) is a library which tokenises PHP, Javascript and CSS. [Coder](https://www.drupal.org/project/coder) is a Drupal project that provides sniffs for PHP CodeSniffer which tell whether a piece of code follows [Drupal coding standards](https://www.drupal.org/coding-standards).

My GSoC project requires me to work on both - Drupal 7 and 8 version of [site_audit](https://drupal.org/project/site_audit). So, I had to set up PHP_CodeSniffer such that it can work with both the versions of Drupal. The Coder standards are different for Drupal 7 and Drupal 8. This post deals with how to set up CodeSniffer in such a suituation. This will require installing coder module twice, once for Drupal 7 standards and other for Drupal 8 standards.

## Installing CodeSniffer
CodeSniffer can be installed using composer by the following command
{% highlight bash %}
composer global require squizlabs/PHP_CodeSniffer:\>=2

#make phpcs command globally availabe
sudo ln -s ~/.composer/vendor/bin/phpcs /usr/local/bin
{% endhighlight %}


## Installing Coder for Drupal 8
Coder provides the set of sniffs for CodeSniffer. We need two copies of this, one for Drupal 7 and other for Drupal 8.
Install the first copy using Composer
{% highlight bash %}
composer global require drupal/coder
{% endhighlight %}
This will install 8.x-2.x version of coder which provides standards for Drupal 8. Composer puts the code module inside `$HOME/.composer/vendor/drupal/` directory.

## Installing Coder for Drupal 7
Install the second copy using drush
{% highlight bash %}
drush dl coder-7.x --destination=$HOME/.drush
{% endhighlight %}
This will download 7.x2.x version of coder to `.drush` directory inside your home folder.

## Creating Aliases for usage
Different aliases can be created for CodeSniffer with Drupal 7 standards and CodeSniffer with Drupal 8 standards. To create aliases, add the following into your `.bashrc` or `.zshrc` file
{% highlight bash %}
#drupalcs8 alias for testing Drupal 8 coding standards
alias drupalcs8="$HOME/.composer/vendor/squizlabs/php_codesniffer/scripts/phpcs --standard=~/.composer/vendor/drupal/coder/coder_sniffer/Drupal --extensions='php,module,inc,install,test,profile,theme,js,css,info,txt'"

#drupalcs7 for testing Drupal 7 coding standards
alias drupalcs7="phpcs --standard=~/.drush/coder/coder_sniffer/Drupal --extensions='php,module,inc,install,test,profile,theme,js,css,info,txt'"
{% endhighlight %}

To test some code for Drupal 8 coding standards, run `drupalcs8 [path to file/folder to check]`. Similarly, `drupalcs7` can be used to test for Drupal 7 coding standards.
