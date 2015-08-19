+++
Categories = ["Hacking"]
Description = ""
Tags = []
date = "2015-08-12T13:35:54+05:30"
menu = "main"
title = "Site Audit - WebPageTest and Google PageSpeed Insights integration"
+++

In last week, I have been working on integrating GooglePageSpeed Insights v2 api and [webpagetest.org](http://www.webpagetest.org/) with site_audit.

Google PageSpeed Ingihts measures how the page can improve its performance on:

* time to above-the-fold load: Elapsed time from the moment a user requests a new page and to the moment the above-the-fold content is rendered by the browser.
* time to full page load: Elapsed time from the moment a user requests a new page to the moment the page is fully rendered by the browser.

I then generates actionable reports based on the above tests on how to make the webaite load faster. PageSpeed Insights gives recommanedation for both - for mobile and desktop devices. An API key is needed to run the PageSpeed Insights test. Follow the instructions given [here](https://developers.google.com/+/web/api/rest/oauth) to obtain an API key.

Here is a sample report I generated for [https://drupal.org](https://drupal.org) using site_audit

{{< highlight bash >}}
 ❯ drush @d8 afe --detail https://drupal.org [API-Key]                                                                             [19:34:16]
Front End: 72%
  Google PageSpeed Insights: Analysis by the Google PageSpeed Insights service
    Desktop:
      Page stats
        - Number Resources: 54
        - Number Hosts: 20
        - Total Request Bytes: 9.74kB
        - Number Static Resources: 23
        - Html Response Bytes: 27.27kB
        - Css Response Bytes: 132.35kB
        - Image Response Bytes: 377.99kB
        - Javascript Response Bytes: 323.1kB
        - Other Response Bytes: 10.97kB
        - Number Js Resources: 12
        - Number Css Resources: 2
        Detailed results:
          Avoid landing page redirects: 
            Your page has no redirects. Learn more about avoiding landing page redirects.
          Enable compression: (low impact: 0.5096)
            Compressing resources with gzip or deflate can reduce the number of bytes sent over the network.
            Enable compression for the following resources to reduce their transfer size by https://developers.google.com/speed/docs/insights/EnableCompression (4.6KiB reduction).
              Compressing https://www.drupal.org/sites/all/modules/drupalorg/drupalorg/images/d8.svg could save 2.8KiB (60% reduction).
              Compressing https://www.drupal.org/sites/all/themes/bluecheese/images/community-users.svg could save 1.1KiB (51% reduction).
              Compressing https://www.drupal.org/sites/all/themes/bluecheese/images/community-commits.svg could save 683B (48% reduction).
          Leverage browser caching: (low impact: 2.483630952381)
            Setting an expiry date or a maximum age in the HTTP headers for static resources instructs the browser to load previously downloaded resources from local disk rather than over the network.
            Leverage browser caching for the following cacheable resources:
              https://tag.perfectaudience.com/serve/54419274f6c96ed073000003.js (30 minutes)
              https://pagead2.googlesyndication.com/pagead/osd.js (60 minutes)
              https://partner.googleadservices.com/gampad/google_service.js (60 minutes)
              https://securepubads.g.doubleclick.net/gampad/google_ads.js (60 minutes)
              https://www.google-analytics.com/analytics.js (2 hours)
          Reduce server response time: (low impact: 0.67)

            In our test, your server responded in 0.27 seconds. There are many factors that can slow down your server response time. Please read our recommendations to learn how you can monitor and measure where your server is spending the most time.
          Minify CSS: 
            Your CSS is minified. Learn more about minifying CSS.
          Minify HTML: 
            Your HTML is minified. Learn more about minifying HTML.
          Minify JavaScript: (low impact: 0.9004)
            Compacting JavaScript code can save many bytes of data and speed up downloading, parsing, and execution time.
            Minify JavaScript for the following resources to reduce their size by https://developers.google.com/speed/docs/insights/MinifyResources (8.3KiB reduction).
              Minifying https://www.drupal.org/files/advagg_js/js__UiO29A8qiU-liB_96gCC8E9JKDzRn317nHrd6s0stps__AwigsEBkeDuYYEcio1o-kRLmSuICds4-G7yNidtd27A__GKPXQ36vHDFbrOKQtK_IEMxZ1RUffPGNLuaADE9Tq48.js could save 4.3KiB (14% reduction) after compression.
              Minifying https://www.drupal.org/files/advagg_js/js__5OZAYFxGTZOYdP8S9RW-QXHGBJVbXz4b02eO_gjMy78__HWGhrFIv37TjqCCJrDkkX6L1WUDisR3SpJLbn2PZpps__GKPXQ36vHDFbrOKQtK_IEMxZ1RUffPGNLuaADE9Tq48.js could save 1.6KiB (35% reduction) after compression.
              Minifying https://www.drupal.org/files/advagg_js/js__F-D1Wr4s7lvQ_5eZ1uikRth42rCUHezP7S--qIwe4BE__BENqbZRxUBWhW4xtQpgAaYXNTGo08uOCQ9RnS4xs6mE__GKPXQ36vHDFbrOKQtK_IEMxZ1RUffPGNLuaADE9Tq48.js could save 1.6KiB (39% reduction) after compression.
              Minifying https://www.drupal.org/files/advagg_js/js__EMCfKGudr6YbSRPLPtEJCm6hBbDPYG9OmcnCkXac-lw___4lW3RAogR_Dg7YQ0GivoywrPe3IWr3Aivpj4b4lFKQ__GKPXQ36vHDFbrOKQtK_IEMxZ1RUffPGNLuaADE9Tq48.js could save 772B (11% reduction) after compression.
          Eliminate render-blocking JavaScript and CSS in above-the-fold content: (HIGH impact: 18)
            Your page has 2 blocking script resources and 2 blocking CSS resources. This causes a delay in rendering your page.
            None of the above-the-fold content on your page could be rendered without waiting for the following resources to load. Try to defer or asynchronously load blocking resources, or inline the critical portions of those resources directly in the HTML.
            Remove render-blocking JavaScript:
              https://partner.googleadservices.com/gampad/google_service.js
              https://securepubads.g.doubleclick.net/gampad/google_ads.js
            Optimize CSS Delivery of the following:
              https://www.drupal.org/files/advagg_css/css__sOq59WKNiWOgbu4pGFLqc02-UxptzM-43WLTi07EISQ___n4Hsqnm5n4Q3o61O6Nwbi2uyaDvjIjy3P1ub9akHgM__GKPXQ36vHDFbrOKQtK_IEMxZ1RUffPGNLuaADE9Tq48.css
              https://www.drupal.org/files/advagg_css/css__FGk3wEy-4xUHLoLxx47psMf0CNXbYM7sWoYboFOgmrQ__3nTsvNkh65TIbU2ER0cjRnslRv1W9gKUmayT_E04YcY__GKPXQ36vHDFbrOKQtK_IEMxZ1RUffPGNLuaADE9Tq48.css
          Optimize images: (HIGH impact: 21.849)
            Properly formatting and compressing images can save many bytes of data.
            Optimize the following images to reduce their size by https://developers.google.com/speed/docs/insights/OptimizeImages (213.4KiB reduction).
              Compressing and resizing https://www.drupal.org/files/styles/case-4-2x/public/Drupal-case-study-1.png?itok=QeT_-GJq could save 144.5KiB (70% reduction).
              Compressing and resizing https://www.drupal.org/files/styles/case-4-2x/public/first4numbers-by-website-express.jpg?itok=hllwuASQ could save 30.6KiB (70% reduction).
              Compressing and resizing https://www.drupal.org/files/styles/case-4-2x/public/ABOUT.jpg?itok=SqZStV_H could save 29.3KiB (71% reduction).
              Losslessly compressing https://tpc.googlesyndication.com/simgad/13661195180415780042 could save 7.6KiB (17% reduction).
              Losslessly compressing https://www.drupal.org/sites/all/themes/bluecheese/images/sprites.png could save 1.3KiB (12% reduction).
          Prioritize visible content: 
            You have the above-the-fold content properly prioritized. Learn more about prioritizing visible content.
    Mobile
      Page stats
        - Number Resources: 55
        - Number Hosts: 21
        - Total Request Bytes: 9.84kB
        - Number Static Resources: 23
        - Html Response Bytes: 28.25kB
        - Css Response Bytes: 132.35kB
        - Image Response Bytes: 377.28kB
        - Javascript Response Bytes: 319.4kB
        - Other Response Bytes: 10.96kB
        - Number Js Resources: 12
        - Number Css Resources: 2
        Detailed results:
          Avoid landing page redirects: 
            Your page has no redirects. Learn more about avoiding landing page redirects.
          Avoid plugins: 
            Your page does not appear to use plugins, which would prevent content from being usable on many platforms. Learn more about the importance of avoiding plugins.
          Configure the viewport: 
            Your page specifies a viewport matching the device's size, which allows it to render properly on all devices. Learn more about configuring viewports.
          Enable compression: (low impact: 0.5096)
            Compressing resources with gzip or deflate can reduce the number of bytes sent over the network.
            Enable compression for the following resources to reduce their transfer size by https://developers.google.com/speed/docs/insights/EnableCompression (4.6KiB reduction).
              Compressing https://www.drupal.org/sites/all/modules/drupalorg/drupalorg/images/d8.svg could save 2.8KiB (60% reduction).
              Compressing https://www.drupal.org/sites/all/themes/bluecheese/images/community-users.svg could save 1.1KiB (51% reduction).
              Compressing https://www.drupal.org/sites/all/themes/bluecheese/images/community-commits.svg could save 683B (48% reduction).
          Leverage browser caching: (HIGH impact: 3.7254464285714)
            Setting an expiry date or a maximum age in the HTTP headers for static resources instructs the browser to load previously downloaded resources from local disk rather than over the network.
            Leverage browser caching for the following cacheable resources:
              https://tag.perfectaudience.com/serve/54419274f6c96ed073000003.js (30 minutes)
              https://pagead2.googlesyndication.com/pagead/osd.js (60 minutes)
              https://partner.googleadservices.com/gampad/google_service.js (60 minutes)
              https://securepubads.g.doubleclick.net/gampad/google_ads.js (60 minutes)
              https://www.google-analytics.com/analytics.js (2 hours)
          Reduce server response time: 
            Your server responded quickly. Learn more about server response time optimization.
          Minify CSS: 
            Your CSS is minified. Learn more about minifying CSS.
          Minify HTML: 
            Your HTML is minified. Learn more about minifying HTML.
          Minify JavaScript: (low impact: 0.9004)
            Compacting JavaScript code can save many bytes of data and speed up downloading, parsing, and execution time.
            Minify JavaScript for the following resources to reduce their size by https://developers.google.com/speed/docs/insights/MinifyResources (8.3KiB reduction).
              Minifying https://www.drupal.org/files/advagg_js/js__UiO29A8qiU-liB_96gCC8E9JKDzRn317nHrd6s0stps__AwigsEBkeDuYYEcio1o-kRLmSuICds4-G7yNidtd27A__GKPXQ36vHDFbrOKQtK_IEMxZ1RUffPGNLuaADE9Tq48.js could save 4.3KiB (14% reduction) after compression.
              Minifying https://www.drupal.org/files/advagg_js/js__5OZAYFxGTZOYdP8S9RW-QXHGBJVbXz4b02eO_gjMy78__HWGhrFIv37TjqCCJrDkkX6L1WUDisR3SpJLbn2PZpps__GKPXQ36vHDFbrOKQtK_IEMxZ1RUffPGNLuaADE9Tq48.js could save 1.6KiB (35% reduction) after compression.
              Minifying https://www.drupal.org/files/advagg_js/js__F-D1Wr4s7lvQ_5eZ1uikRth42rCUHezP7S--qIwe4BE__BENqbZRxUBWhW4xtQpgAaYXNTGo08uOCQ9RnS4xs6mE__GKPXQ36vHDFbrOKQtK_IEMxZ1RUffPGNLuaADE9Tq48.js could save 1.6KiB (39% reduction) after compression.
              Minifying https://www.drupal.org/files/advagg_js/js__EMCfKGudr6YbSRPLPtEJCm6hBbDPYG9OmcnCkXac-lw___4lW3RAogR_Dg7YQ0GivoywrPe3IWr3Aivpj4b4lFKQ__GKPXQ36vHDFbrOKQtK_IEMxZ1RUffPGNLuaADE9Tq48.js could save 772B (11% reduction) after compression.
          Eliminate render-blocking JavaScript and CSS in above-the-fold content: (HIGH impact: 72)
            Your page has 2 blocking script resources and 2 blocking CSS resources. This causes a delay in rendering your page.
            None of the above-the-fold content on your page could be rendered without waiting for the following resources to load. Try to defer or asynchronously load blocking resources, or inline the critical portions of those resources directly in the HTML.
            Remove render-blocking JavaScript:
              https://partner.googleadservices.com/gampad/google_service.js
              https://securepubads.g.doubleclick.net/gampad/google_ads.js
            Optimize CSS Delivery of the following:
              https://www.drupal.org/files/advagg_css/css__sOq59WKNiWOgbu4pGFLqc02-UxptzM-43WLTi07EISQ___n4Hsqnm5n4Q3o61O6Nwbi2uyaDvjIjy3P1ub9akHgM__GKPXQ36vHDFbrOKQtK_IEMxZ1RUffPGNLuaADE9Tq48.css
              https://www.drupal.org/files/advagg_css/css__FGk3wEy-4xUHLoLxx47psMf0CNXbYM7sWoYboFOgmrQ__3nTsvNkh65TIbU2ER0cjRnslRv1W9gKUmayT_E04YcY__GKPXQ36vHDFbrOKQtK_IEMxZ1RUffPGNLuaADE9Tq48.css
          Optimize images: (low impact: 1.2324)
            Properly formatting and compressing images can save many bytes of data.
            Optimize the following images to reduce their size by https://developers.google.com/speed/docs/insights/OptimizeImages (12KiB reduction).
              Losslessly compressing https://tpc.googlesyndication.com/simgad/13661195180415780042 could save 7.6KiB (17% reduction).
              Losslessly compressing https://www.drupal.org/files/styles/case-4-2x/public/first4numbers-by-website-express.jpg?itok=hllwuASQ could save 2.4KiB (6% reduction).
              Losslessly compressing https://www.drupal.org/sites/all/themes/bluecheese/images/sprites.png could save 1.3KiB (12% reduction).
              Losslessly compressing https://www.drupal.org/files/styles/case-4-2x/public/ABOUT.jpg?itok=SqZStV_H could save 661B (2% reduction).
          Prioritize visible content: 
            You have the above-the-fold content properly prioritized. Learn more about prioritizing visible content.
          Size content to viewport: 
            The contents of your page fit within the viewport. Learn more about sizing content to the viewport.
          Size tap targets appropriately: (HIGH impact: 10.320821005917)
            Some of the links/buttons on your webpage may be too small for a user to easily tap on a touchscreen. Consider making these tap targets larger to provide a better user experience.
            The following tap targets are close to other nearby tap targets and may need additional spacing around them.
              The tap target <a href="#content" class="element-invisi…ment-focusable">Skip to main content</a> and 1 others are close to other tap targets final.
              The tap target <a href="/start">Get Started</a> and 3 others are close to other tap targets final.
              The tap target <input id="edit-meta-type-" type="radio" name="meta_type" class="form-radio"> and 5 others are close to other tap targets final.
              The tap target <label for="edit-meta-type-" class="option">All</label> and 5 others are close to other tap targets final.
              The tap target <a id="tab-news-link" href="#tab-news">News</a> is close to 1 other tap targets.
              The tap target <a id="tab-commits-link" href="#tab-commits">Commits</a> is close to 1 other tap targets.
              The tap target <a href="/drupal-8.0">Drupal 8 is in beta</a> is close to 1 other tap targets.
              The tap target <a href="https://groups…g/core/updates">regular updates</a> and 7 others are close to other tap targets.
              The tap target <a href="https://www.drupal.org/news">Drupal News</a> and 4 others are close to other tap targets.
              The tap target <a href="https://www.drupal.org/planet">Planet Drupal</a> and 26 others are close to other tap targets.
          Use legible font sizes: 
            The text on your page is legible. Learn more about using legible font sizes.
      Full report at https://developers.google.com/speed/pagespeed/insights
{{< /highlight >}}


