+++
Categories = ["Hacking"]
Description = "Testing Drush Extensions"
Tags = ["Drupal 8", "GSoC 2015", "Drush", "PHPUnit", "Drupal Planet"]
date = "2015-07-07T13:35:54+05:30"
menu = "main"
title = "Writing Tests for Drush Commands"
+++

As a part of my GSoC project, I have been recently writing functional tests for [site_audit](https://drupal.org/site_audit) -- a Drush extension that provides drush commands for static analysis of a Drupal site. The process seemed painful at first due to lack of good documentation and gaps in my knowledge of Drush and PHPUnit. But, after a few hiccups intially, I was successfully able to write tests for Site Audit.

## Background

Drush uses [PHPUnit](https://phpunit.de/) testing framework for testing its core commands. All the tests are namespaced under the name `Unish`. Drush test suites provides some base classes which, amongst other functionality, have primitves for setting up a drupal sandbox to provide an environment for testing drush commands.

All the Drush tests are inside the [tests](https://github.com/drush-ops/drush/tree/master/tests) directory inside the drush root folder which also contains a readme file providing some instructions to run the tests. The directory also has configuration settings for  PHPUnit 
in the [phpunit.xml.dist](https://github.com/drush-ops/drush/blob/master/tests/phpunit.xml.dist) directory. If you look inside this file, there are some commented lines that contain environment requried for the tests to run. You can either uncomment these and provide appropriate values or set these variables as bash environment variables where you run the test.

To run the tests on your local machine, first go the the drush root directory (directory where drush is installed) and run 
{{< highlight bash >}}
composer install
{{< /highlight >}}
This command will install all the dependencies (including PHPUnit) required for running the tests inside a directory named `vendor`. Then run the `unish.sh` script from the drush root directory. 

## Creating your own Test

You can create a directory named `tests` inside you drush extension to contain all the tests. PHPUnit looks inside *Test.php files inside a directory for tests. Drush follows Lower Camel Case for the naming of both the file and the class name. So all you file names should be in Lower Camel Case with a `Test` suffix. 

{{< highlight php >}}
<?php
/**
 * @file
 * Contains /site_audit/tests/BestPracticesFast404Case.
 */

namespace Unish;

/**
 * Class BestPracticesFast404Case.
 *
 * @group commands
 */
class BestPracticesFast404Case extends CommandUnishTestCase {

  /**
   * Sets up the environment for this test.
   */
  public function setUp() {
  }
{{< /highlight >}}

First, we specify the namespace `Unish` for the test. The `@group` annotation in the doc comment is optional. You can read about its use [here](https://phpunit.de/manual/current/en/appendixes.annotations.html#appendixes.annotations.group). Our class extends `CommandUnishTestCase` which provides all the common functionality like setting up a drupal sandbox and running drush commands used in functional tests. Finally we have a `setUp()` function which is invoked before a test method is run. In We set up the object we will test against in this method.

## The SetUp Function

{{< highlight php >}}
<?php

  public function setUp() {
    if (UNISH_DRUPAL_MAJOR_VERSION < 8) {
      $this->markTestSkipped('This version of Site Audit is for D8');
    }
    $site = $this->setUpDrupal(1, TRUE, UNISH_DRUPAL_MAJOR_VERSION, 'standard');
    $root = $this->webroot();
    $this->options = array(
      'yes' => NULL,
      'root' => $root,
      'uri' => key($site),
    );
    // Symlink site_audit inside the site being tested, so that it is available
    // as a drush command.
    $target = dirname(__DIR__);
    \mkdir($root . '/drush');
    \symlink($target, $this->webroot() . '/drush/site_audit');
  }
{{< /highlight >}}

In this function, we have used `UNISH_DRUPAL_MAJOR_VERSION` which is provided by the PHPUnit settings variables we had set in phpunit.xml.dist . Since the version of site_audit where this test is written is for Drupal 8, we will skip the test if it is being run for Drupal 7.  The `setUpDrupal()` is a function provided by `CommandUnishTestCase` class and is used to create any number of drupal sanboxes which will be used for testing. Here, we are installing a single drupal 8 site with the standard profile. 
`$root` variable has the path to the root directory of the website. 

Since the drush commands will be run in the context of the sandbox we have just installed, it will not be able to discover the commands defined by the extensions. So, we will have to symlink our extension inside the drush directory inside the webroot using the `symlink()` function. Since the drush directory does not exist by default, it has to be created first using the `mkdir()` function.

## The Test Function

{{< highlight php >}}
<?php

  /**
   * If fast_404 is enabled and fast_404 paths are empty, check should warn.
   */
  public function testFast404() {
    // Enable fast_404 and make fast_404 paths empty.
    $eval1 = "\$config = \\Drupal::configFactory()->getEditable('system.performance'); \$config->set('fast_404.enabled', TRUE); \$config->save();";
    $eval2 = "\$config = \\Drupal::configFactory()->getEditable('system.performance'); \$config->set('fast_404.paths', ''); \$config->save();";
    $this->drush('php-eval', array($eval1), $this->options);
    $this->drush('php-eval', array($eval2), $this->options);

    // Execute the best-practices command and get output.
    $this->drush('audit-best-practices', array(), $this->options + array('detail' => NULL, 'json' => NULL));
    $output = json_decode($this->getOutput());
    $this->assertEquals(\SiteAuditCheckAbstract::AUDIT_CHECK_SCORE_WARN, $output->checks->SiteAuditCheckBestPracticesFast404->score);
  }

{{< /highlight >}}

The test has to edit some config variables of the drupal site. This is accomplished by the `php-eval` command provided by drush. The `php-eval` command evaluate arbitrary php code after bootstrapping Drupal. `CommandUnishTestCase` provides the `drush()` function which can be used to execute drush commands. The second argument of the function takes in the arguments to the drush command and the third argument takes the options. Note that it is necessary to provide the options we had defined in the `setUp()` function so that it executes the drush command on the proper test site (the one which we created in the setUp function). 

`$eval1` and `$eval2` contain the php code to be executed in strings and then we pass these as arguments to the `php-eval` command. The output of the drush command can be obtained by the `getOutput()` function. PHPUnit provides several functions for different types of [assetions](https://phpunit.de/manual/current/en/appendixes.assertions.html) which determine whether the test passed or failed.

After all the tests in a class are finished, the drupal sandbox created for tests is automatically removed. All the tests in a class share the same sandbox. Any test in the middle of the class can remove the existing sandbox and create a new one using the `setUpFreshSandBox()` function. The `setUpDrupal()` will have to be called again to install drupal for testing after this.

## Running the tests
To run the tests, you have to run PHPUnit pointing to the phpunit.xml.dist inside the drush tests directory. So running the following command from **inside the root of you drush extension** should run the tests.

{{< highlight bash >}}
phpunit --configuration [path to drush tests directiory] [path to your tests directory]
{{< /highlight >}}

In our case, the path to our tests directory is tests. If you install drush using composer, it will be inside .composer/vendor/drush directory . In this case, following command will work.
 {{< highlight bash >}}
 phpunit --configuration .composer/vendor/drush/drush/tests tests
 {{< /highlight >}}

## Conclusion

There are other functions provided by the test classes in Unish. You can look them up inside the drush codebase. The best resource to learn about testing drush commands is to look at the existing tests written for the drush core commands. The tests I am writing can be found in the Site Audit github repository [here](https://github.com/shivanshuag/site_audit/tree/unit-tests/tests) and these will eventually be merged to the main github repository of Site Audit [here](https://github.com/fluxsauce/site_audit).

I would be happy to here feedback and suggestions for improvements on the article.