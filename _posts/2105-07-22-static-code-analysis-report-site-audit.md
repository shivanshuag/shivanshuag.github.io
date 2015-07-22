---
layout: post
title: Static Code Analysis Report in Site Audit
description: 
category: hacking
tags: [GSoC 2015, drush, drupal 8, PHP Mess Detector]
comments: true
share: true
---

As a part of GSoC 2015, I am working on extending the functionality of [Site Audit](https://drupal.org/project/site_audit)  -- a Drush extension that provides drush commands for static analysis of a Drupal site under the mentorship of [Jon Peck](https://drupal.org/u/fluxsauce). To gauge the requirements of community using Site Audit, and set goals for its future development, I had conducted interviews on the subject with some agencies that use it regularly. The transcript of the interview for different agencies can be found in the following links - [Pantheon](https://docs.google.com/document/d/1DQqlE5yh76MZvnfT8cENy6r2hgbp5c1bStmcymS-tuM/edit), [Hooks 42](https://docs.google.com/document/d/1uZavKAKWySNpoFndF23UHwIb627UF-I0q-mpdECCMyA/edit), [Four Kitchens](https://docs.google.com/document/d/1oLS2E5gssyJ4-tuP7ysUIS0hsSJjfhlZSdI-5j7LhH8/edit#heading=h.72mlvxlu8i8r) and [Kalamuna](https://docs.google.com/document/d/1pj3OrX68bh3FMw-JQnLDEyh1dI9t7lRmSldTAiSMQK8/edit#heading=h.joibnnbtsex5).

One of the common requirement of all four was static analysis of custom code in the drupal site. The custom code written is often of poor quality and a source of several bugs and performance issues. Reviewing custom code is the part of a site audit that takes maximum time and pains. There are several PHP tools already available which review the quality of PHP code. [PHP Mess Detector](http://phpmd.org), [PHP Copy/Paste Detector](https://github.com/sebastianbergmann/phpcpd), [PHP Dead Code Detector](https://github.com/sebastianbergmann/phpdcd) are a few which I and my mentor agreed upon to be integrated during the GSoC period. The github issue tracking the integration of these tools with site audit this is present [here](https://github.com/fluxsauce/site_audit/issues/45).

## Major Concerns while Integration

Integrating a third party tool with site audit required several decisions to be taken. Sould the third party tool be packaged along with site audit, or should it be a dependency to be installed separately. There are arguments in favour of both. Packaging dependencies along with site audit will make it easier to set up. In fact, just running `drush dl site-audit` will be enough to install it. But it will require a new release of site audit whenever the third party tool changes. Finally we decided on not packaging the dependencies with site audit but including them as a dependency in composer. Composer lets you maintian a very fine-grained control on the versions of dependencies. This makes composer as the only way of installing site audit.

Another question was how to integrate the tool with site audit? There are two possible ways - parse the output given by the tool or use the specific functions in the codebase of the tool to get the required output. Using the specific function inside the codebase requires the dependencies to be present in a specific place from where they can be autoloaded(possible if the dependencies have been installed using composer) while parsing the output can be difficult if it not structured consistently. This is not that big an issue and after looking at the output of the tools that are to be integrated, we decided to go with parsing the output.

Other concerns were regrading which rules to use with Php Mess Detector and how to specify custom code in the site.

## Running Php Mess Detector check in site audit

As of today, I have successfully integrated PHP Mess Detector with Site Audit as a check in `Static Code Analysis Report` for drupal 8 version of site audit. To run this report, run

{% highlight bash %}
drush asca --custom_code=[custom code paths separated by comma]
{% endhighlight %}

PHP Mess Detector rules to be used can also be specified by using `phpmd_rules` option. By default, it uses codesize, naming, design and unusedcode rules.

The custom code paths can also be specified in the conf array inside settings.php file of a drupal site in the following manner

{% highlight php %}
$conf['site_audit']['custom_code'] = array(
	'./modules/custom',
	'./modules/features',
);
{% endhighlight %}

## Conclusion

I will love to hear feedback on the way this check has been implemented ways on how to improve it. In the next few days I will integrate the other two tools with site audit and write about them next week.






