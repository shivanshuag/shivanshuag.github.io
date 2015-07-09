+++
Categories = ["Hacking"]
Description = "Some basics for neophytes"
Tags = ["Drupal 8", "GSoC"]
date = "2014-06-02T13:35:54+05:30"
menu = "main"
title = "Porting a module to Drupal 8"
+++

I am porting a drupal 7 module named [securesite](https://drupal.org/project/securesite) to drupal 8 under of Google Summer Of Code'14. It has  been two weeks into the coding period and I am thoroughly enjoying it. This is my first port in a series of posts that I plan to write during this period.

One look at a drupal 8 module is enough to make you realize that there have been some major changes in its architecture. If you observe closely, you will find ample use of OOP constructs, YAML files, and a directory structure totally different form its drupal 7 counterpart. There is enough mention about the big changes in drupal 8 on the web to scare anyone. With this series of posts, I aim to simplify this process of porting a module to drupal 8 for everyone - new to drupal or already experienced in drupal 7. I will also clarify some of the less documented things about drupal 8 which perplexed me while porting.

## Development Environment

First step is to set up the development environment for the project. I prefer [phpstorm](http://www.jetbrains.com/phpstorm/) IDE, even though it is not free, due to its excellent drupal support. The following link explains how to set up phpstorm for drupal module development -[https://drupal.org/node/1962108](https://drupal.org/node/1962108). 

Currently, drupal 8 support is avaliable in [phpstorm-eap](http://confluence.jetbrains.com/display/PhpStorm/PhpStorm+Early+Access+Program). The IDE is awesome enough to not only point out the syntax errors, but even mark the functions you are using that have been deprecated or don't exist drupal 8.

## OOP, Namespaces and YAML files
I feel that the best way of learning on how to make a d8 module is by looking at the existing d8 module in drupal core. They make up for the lack of documentation in many places.

On looking at a d8 version of a d7 module, first thing you need to figure out is - where did all the code and hooks in `.module` file go?
The concepts of PHP namespaces and PSR-0 standards are very lucidly explained by [effulgentsia](https://drupal.org/user/78040) in
[http://effulgentsia.drupalgardens.com/content/drupal-8-hello-oop-hello-world](http://effulgentsia.drupalgardens.com/content/drupal-8-hello-oop-hello-world)
Reading above removes a lot of confusion about so many directories and files in the module and the use of namespaces.
Some other useful points to be noted from the above blog post are

* `.info` file has changed to `.info.yml` file. You can find more about it on [https://drupal.org/node/2000204](https://drupal.org/node/2000204). For those who don't know about the `.info` file, look up the drupal documentation for the same at [https://drupal.org/node/542202](https://drupal.org/node/542202)
* use of Controllers and routing by YAML files which was earlier implemented by `hook_menu`. More on routing at [https://drupal.org/developing/api/8/routing](https://drupal.org/developing/api/8/routing). If you don't know about drupal hooks, look it up in the drupal documentation. 

## Configuration Management

One of the bigger changes in Drupal 8 is the Configuration Management Initiative which completely replaces the Drupal 7 variables. One of the reasons for this is to be able to store configuration in files rather than database which helps in easier import-export of the configuration.

* If you don't know what varibales are and why are they used, here is a simple explanation. Many a times in your module, you need to store module settings or state or any other information required by the module. These are stored in the variable table in the database.
* To know more about the Simple Configuration API, look at [https://drupal.org/node/1809490](https://drupal.org/node/1809490)
* The next link explains on how to convert the variables of your d7 module to d8. It it perhaps the most useful link you will find regarding CMI.  [https://drupal.org/node/1667896](https://drupal.org/node/1667896)
* You can use the [Config Inspector](https://drupal.org/project/config_inspector) module to find the variables in core. Once you have converted your variables to configuration keys, you can see their values through this module(you will also have to create a schema file for this module to show your configuration). Also have a look at the config.yml file of core modules to have an idea of how to convert the variables to configuration keys.
* You can define schema for you configuration. [https://drupal.org/node/1905070](https://drupal.org/node/1905070)
* Apart from configuration, there are other type of informations also like state information and configuration entities. More on this at - [https://drupal.org/node/2120523](https://drupal.org/node/2120523). Many of the d7 variables have been moved to state. The next link explains on how to find these varibles and convert them. [https://drupal.org/node/1787318](https://drupal.org/node/2120523).

### Key Takeaways 

* config keys are just like variables
* `.settings.yml` file is to define default values for the varibales
* It is not necessary to define a key in `.settings.yml` file before using it(get or set) anywhere in the code. But make sure to include all your keys in the schema file.
* can also set default values in `hook_install()`
* If a key has not been set earlier in the code and its default value is not defined in `hook_install()` or `.settings.yml` file, get function on the key returns NULL

## .install file

`hook_enable` and `hook_disable` have been deprecated. [https://drupal.org/node/2193013](https://drupal.org/node/2193013). To port to d8, all the tasks that were being performed in  `hook_enable` are to be performed in `hook_install`. Same with `hook_disable` and `hook_uninstall`. 
If you have not developed a drupal 7 module before, the difference between disable-enable and install-uninstall can be confounding. Disabling a module leaves its configuration untouched, along with any data like variables it may have created while uninstalling deletes all the data. Due to removal of disabled state in d8, some of the tasks performed in `hook_enable` may become unecessary.

I hope someone finds this useful. I will write about forms, tests and authentication in my next blog post.
