---
status: publish
layout: post
author:
  display_name: Scott Davis
  email: stdavis@utah.gov
tags:
  - javascript
published: true
date: 2013-04-16 06:43:30 -0600
title: ESRI JSAPI 3.4 and the Dojo Build System
categories:
  - Developer
---
In a <a href="{{site.baseurl}}{% post_url 2012-05-01-speed-up-your-esri-javascript-api-webapp %}">previous post</a>, I outlined how I use the <a href="http:dojotoolkit.org/reference-guide/build/">Dojo Build System</a> to optimize my web app code for production. Specifically I showed how I get around the problem of working with ESRI's <a href="http://help.arcgis.com/en/webapi/javascript/arcgis/">ArcGIS API for JavaScript</a> library which has already been run through the build system. However, with their recent upgrade to AMD-style module loading my handy trick of using:

```js
dojo['require']('esri.map');
```
... to fool the build system into skipping 'esri...' modules imports didn't work anymore. I had no idea how to exclude modules when loading them like this:

```js
define(['esri.map'], function (Map) { /* ... */ });
```

So I headed over to <a href="http://dojotoolkit.org/chat">#dojo</a> to see if the experts had any ideas. Fortunately for me,&nbsp;<a href="https://twitter.com/brianarn">brianarn</a>&nbsp;was there and was aware of the problem. After some brain storming, we came up with the idea to use a custom loader plugin for loading ESRI modules. Since the build system doesn't try to flatten modules that are imported with nested requires, we hoped that importing them through the plugin would solve my problem. The plugin was a relatively simple implementation:

```js
define(function () {
    // summary:
    //      A dojo loader plugin for loading esri modules so that
    //      they get ignored by the build system.
    return {
        load: function (id, require, callback) {
            // id: String
            //      esri module id
            // require: Function
            //      AMD require; usually a context-sensitive require bound to the module making the plugin request
            // callback: Function
            //      Callback function which will be called, when the loading finished.
            require([id], function (mod) {
                callback(mod);
            });
        }
    };
});
```

You can put this module within any package and use it like this:

```js
define(['app/EsriLoader!esri/map'], function(Map) { /* ... */ });
```

Using the plugin to load ESRI module effectively prevents the build system from trying to include them in your layer files thus allowing the build script to complete successfully. Of course, none of this hacking would be needed if ESRI would just <a href="http://ideas.arcgis.com/ideaView?id=087E00000004JOzIAM">release their source code</a>. :)

If you find a better way of getting around this problem or have any other suggestions please let me know at <a href="http://twitter.com/SThomasDavis">@SThomasDavis</a>.
 