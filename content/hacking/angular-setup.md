+++
Categories = ["Hacking"]
Description = ""
Tags = ["Angular 2", "Typescript", "Webpack", "Gulp"]
date = "2015-12-22T13:35:54+05:30"
menu = "main"
title = "Production and Development set-up for Angular 2 with TypeScript using Webpack and Gulp"
+++

Building modular, better structured web applications is now easier with Angular 2, but this comes with a new set of challenges on developing, packaging and deploying these applications. When developing in typescript, it has to be transpiled to javascript in a way such that its modules can be loaded by the browser. For debugging, source maps need to be enabled and for production, javascript needs to be minified. Along with this, the HTML and CSS of each individual component in the application should be present at the right place. There are different choices available for each of the step and there is no fixed way to do anything.

All this can get very confusing when trying to start a new angular project. I explored various solutions and found the following set up using webpack to bundle javascript, gulp for running tasks, npm for package management to be the best. The full code for this setup is present at [https://github.com/shivanshuag/angular2-seed] . This setup is built upon the [angular2-seed](https://github.com/angular/angular2-seed) app.


## Directory Structure
This is the directory structure I have. `app.ts` bootstraps the angular app. `app/seed-app.ts` has the main app component. Other components are inside the components directory with a separate directory for each component which contains its `html` and `css` files along with the typescript file. `vendor.ts` imports all the third-party library the the app uses, which includes angular 2 in this case.
```
maya
├── CONTRIBUTE.md
├── gulpfile.js
├── maya.conf.nginx
├── node_modules
├── package.json
├── README.md
├── tsconfig.json
├── webpack.config.js
└── src
    ├── app
    │   ├── components
    │   │   ├── about
    │   │   │   ├── about.css
    │   │   │   ├── about.html
    │   │   │   └── about.ts
    │   │   ├── home
    │   │   │    ├── home.css
    │   │   │    ├── home.html
    │   │   │    └── home.ts
    │   │   │
    │   │   
    │   ├── seed-app.html
    │   ├── seed-app.ts
    │   └── services
    │       └── service.ts
    ├── app.ts
    ├── favicon.ico
    ├── index.html
    └── vendor.ts
```

## Installing Packages

NPM is used for package mamagement. [`package.json`](https://github.com/shivanshuag/angular2-seed/blob/master/package.json) contains all the required packages which include angular 2 and its dependecies, webpack , gulp, typescript etc.

## Typescript and Webpack

Typescript compiler configuration is present in `tsconfig.json`. It has `source maps` enabled which helps in debugging. Webpack configuration is present in `webpack.config.json`. It bundles all the third-party dependencies import in `vendor.ts` into a file `vendor.bundle.js`. All other code is bundles into a files named `bundle.js`. `index.html` just imports these two files.

For debugging, source maps are enabled in the bundles created for the development mode using gulp.

## Gulp Tasks

For the application to work, along with the transpiled javascript, the html and css files for each of the components should be placed at the path given as the `templateUrl` or `styleUrl` in the typescript file for the component. Gulp tasks are defined in `gulpfile.js` for these things, and for running webpack.

To create production package of the application, run
```
❯ gulp deploy

# This will create minified javascript bundles and copy all the html, css file in the 'dist' directory.
# This directory can then be served by a web server
```
To create development package of the application, run
```
❯ gulp dev

# This will create javascript bundles with source maps and copy html, css file in the 'dist' directory
```

To run webpack-dev-server which serves the app for development purposes and supports live-reload, run
```
❯ gulp dev-server

# Starts webpack-dev-server at port 8080 serving the app
```

## Issues

Currently, all the CSS and HTML files of the application are served separately, there is no way to bundle them in a single file. This was possible in Angularjs as there were no components, and whole CSS of the app could be placed in a single file. In angular 2, doing this can cause css selector conflicts. For HTML in Angularjs, there was template-cache, which could pick all the templates from a single html file.

Angular Dart had components, but it also had [transformers](https://github.com/angular/angular.dart/wiki/CSS-Shim) which namespaced the CSS selectors according to components and also transformed the url of css files in each component. For html, it had template-cache.

Still waiting for angular2-templatecache and transformers.
