---
layout: post
title: Caching In Drupal 8
description:
category: hacking
tags: [GSoC 2015, drupal, Site Audit, caching]
comments: true
share: true
---

First week of my GSoC 2015 project required me to port the caching report of Site Audit to Drupal 8. To generate caching report, some checks are performed on the drupal website's various cache settings. There have been some major changes in Drupal 8 caching architecture from Drupal 7. Most of them are fairly new with little documentation. This post contains a list of articles that helped me understand how caching works in Drupal 8.

If you go and have a look at the `/admin/config/development/performance` page of a Drupal 8 site, you will find most of the caching options which were there in Drupal 7 missing. There is no option for enabling Page Caching, Block caching etc. on the page. Where did they all go?

## Cache API in Drupal 8
A good place to start is having a look at [Cache API documentation](https://api.drupal.org/api/drupal/core!modules!system!core.api.php/group/cache/8) for Drupal 8. It gives a basic idea of Cache Bins, Cache Tags, Cache Backends and how to use those. The idea of Cache Bins and Cache backends is similar to that of Drupal 7 but Cache Tags is a new concept which helps in cache invalidation (more on this later)

## Render Caching
But what about page caching and block caching? It is enabled by default! And it is faster than Drupal 7 caching. Caching for all entities in Drupal 8 is done using a new technique called render caching. More informaiton on this can be found at - [Drupal 8 performance: render caching](https://www.acquia.com/blog/drupal-8-performance-render-caching). This article also provides a good overview of how cache tags are used in cache invalidation and how it makes Drupal 8 cache faster than Drupal 7 chance.

A detailed explanation how to use render caching (it also makes the concept of render caching more clear) can be found at - [Cacheability of render arrays](https://www.drupal.org/developing/api/8/render/arrays/cacheability).

More details about Cache Contexts can be found at - [Cache Context](https://www.drupal.org/developing/api/8/cache/contexts). I am still exploring how these are useful but I think different contexts are used to assign different Cache Ids to the same entity.

## More Resources to Understand Drupal 8 Caching
  * [Talk on Render Caching by Wim Leers](http://wimleers.com/talk-render-caching-drupal-7-and-8/#/5)
  * [Render Cache in Drupal 8](https://drupalwatchdog.com/volume-4/issue-1/automagic-speed-cache)
  * [Drupal 8 now has page caching enabled by default](http://wimleers.com/blog/drupal-8-page-caching-enabled-by-default)
