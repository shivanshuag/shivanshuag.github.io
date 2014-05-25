---
layout: post
title: Porting a module to Durpal 8 step by step
description: ""
category: articles
tags: [drupal8, module, gsoc]
comments: true
share: true
---

When

On a first look at a drupal 8 module, it all looks so scary and so different. So many questions like - why are there so many files and directories in this module? Why the hell are namespaces used in so many places? Where are all the familiar hooks and what are these .yml files for? come to mind. Adding to this is the constant metion of the big time changes in the drupal architecture with the inclusion of Symfiny and Twig everywhere on the web. With this series of posts, I aim to simplify this process of porting a module to drupal 8 for everyone - new to drupal of already experienced in drupal 7. I will also clarify some of the less documented things about drupal 8 which perplexed me while porting.

First step is to set up the development environment for the project. My preferred IDE is phpstorm which has excellent Drupal support. The following link explains how to set up phpstorm for drupal module development -https://drupal.org/node/1962108. Currently, drupal 8 support is avaliable in phpstorm-eap.

The concepts of PHP namespaces and PSR-0 standards are very lucidly explained by [effulgentsia](https://drupal.org/user/78040) in
http://effulgentsia.drupalgardens.com/content/drupal-8-hello-oop-hello-world.
Reading above removes a lot of confusion about so many directories and files in the module and use of namespaces.

Some more points to be noted from the above blog post are
- .info file has changed to .info.yml file. You can find more about it on https://drupal.org/node/2000204
- use of Controllers and routes which was earlier implemented by hook_menu. More on routing at https://drupal.org/developing/api/8/routing
- use of PSR0 file structure, classes inside lib directory. 

Configuration Management

- http://drupal8cmi.org/drupal-8-hello-configuration-management
- https://drupal.org/node/1809490   simple configuration api
- Most useful - https://drupal.org/node/1667896
- Use config inspector module to find the variables in core
- state vs config - https://drupal.org/developing/api/8/configuration
- Many variables have been moved to state . To upgrade, https://drupal.org/node/1787318
- There are also configuration entities. More about them on https://drupal.org/node/2120523

What I understood 
- config keys are just like variables
- .settings file is to define default values for the varibales
- you can set a key anywhere in code even if its default value does not exist in the settings file
- can also set default values in hook_install()
- If key was not set earlier and its default value is not defined in hook_install() or .settings file, returns NULL

Schema
- https://drupal.org/node/1905070

.install file

- hook_enable and hook_disable have been deprecated https://drupal.org/node/2193013, 
- Porting this involves perforing the tasks that were being performed in hook_enable in hook_install. Same for hook_disable and hook_uninstall
