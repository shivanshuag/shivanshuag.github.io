First of all didn't understand the directory structure and namespaced classes. Therefore read - 
http://effulgentsia.drupalgardens.com/content/drupal-8-hello-oop-hello-world

Points to be notes above are 
- .info file changed to .info.yml file. More about it on https://drupal.org/node/2000204
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
