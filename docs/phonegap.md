
Some thoughts about phonegap and possible near and longer term changes.

# nowgap

To set the stage for the **futuregap** stuff mentioned below, here are some things that might be useful to do now. I know I would appreciate them in my AMD-based userland phonegap project:

## Phonegap as a function

Instead of having to overwrite addEventListener and such to support 'deviceready' event, I would rather just see Phonegap be a function that called the argument passed to it once the device was ready:

```javascript
    Phonegap(function (PG) {
        //This executes after device ready.
    });
```

then have something like the following for code that needs to do initialization/setup before deviceready callbacks are done:

```javascript
    Phonegap.on('init', function (PG) {
    });
```

I'm not too concerned with the actual naming of the pub/sub stuff, but the basic point is to have three APIs (Phonegap(), Phonegap.on(), Phonegap.exec()) to start working with Phonegap.

## Register as an AMD plugin

This would allow using Phonegap as a plain AMD module and more importantly set the stage to be used as a loader plugin (see next section), but basically doing something like this at the end of the Phonegap file:

```javascript
    if (typeof define === 'function' && define.amd) {
        define(function () { return Phonegap; });
    }
```

## Phonegap as an AMD loader plugin

I would like to use Phonegap as a loader plugin, so that if it is used as a plugin, it would mean "do not resolve the dependency until device ready".

```javascript
    define(['phonegap!', 'my/mod'], function (PG, mymod) {
        //This is not called until device ready.
    });
```

Assuming Phonegap() above is supported, to work as a loader plugin, Phonegap would implement a "load" function.

```javascript
    Phonegap.load = function (name, require, load, config) {
        if (config.isBuild) {
            load();
        } else {
            Phonegap(load);
        }
    }
```

If you prefer to not define that function on Phonegap for when Phonegap is not used in an AMD loader, the define call could look like:

```javascript
    if (typeof define === 'function' && define.amd) {
        define(function () {
            function PG () {
                return Phonegap.apply(null, arguments);
            }
            PG.load = function (name, require, load, config) {
                if (config.isBuild) {
                    load();
                } else {
                    Phonegap(load);
                }
            }
            return PG;
        });
    }
```

# futuregap

This section is much more speculative. It is a sketch on how a shiny component future with phonegap could work. This doc tries to outline how a user creating phonegap apps would do the work, but the same process could be used to inform phonegap.js development.

Names used below are just placeholder names, they are not meant to be final, but they could be used as-is.

## phonegap manager command-line

Installing phonegap would mean downloading a pgm.js (phonegapmanager) file. This file is a scaffolding/npm-type of executable JS, and it runs in Node. So Node would be an install requirement.

Sample commands:

**pgm create name=submarine platform=android,ios** - creates a project called submarine, targeting android and ios.

**pgm add sms platform=android** - installs the sms plugin for android only.

**pgm run android** -- would run the app in android.

**pgm webdebug android** -- maybe just starts up the desktop-browser based ripple emulator, set up to emulate android.

## Userland directory structure

Assume the following command is run:

**pgm create name=submarine platform=android,ios**

This creates a project called **submarine**. Creates a directory for submarine and the native code bits that are needed to run this app in android and ios. The directory structure for the project would look something like this:

* **submarine**
    * **sim**: any files needed for a browser-based simulator? may not be needed.
    * **native**
        * **android**: java source & binary goo in here.
        * **ios**: objective c source & binary goo in here.
    * **www**
        * **[index.html](#index-html)**
        * **css**
            * **index.css**
        * **js**
            * **[index.js](#index-js)**
            * **android**
                * **[plugins.js](#plugins-js)**
                * **exec.js**: phonegap.exec impl for android
                * **[sms.js](#sms-js)** (assuming `pgm add sms` was done)
            * **ios**
                * **plugins.js**
                * **exec.js**
            * **[phonegap.js](#phonegap-js)**
            * **phonegap**
                * **[src.js](#src-js)**
                * **[core.js](#core-js)**
                * **env.js**: the env plugin, [similar to this one](https://github.com/jrburke/submarine/blob/master/phonegap/www/js/env.js), but includes support of all phonegap platforms.
                * **require.js**: the RequireJS implementation, only used in dev
    * **tools**
        * **build.js**

This layout assumes the use of an AMD loader like RequireJS, with support for loader plugins. It is set up so that it can run in a desktop-browser based emulator, like the one ripple has, and all that is needed to debug is to do a reload.

There is a build process that uses tools/build.js similar to how it is done in [gladius](https://github.com/alankligman/gladius/blob/develop/tools/build.js) to use the almond.js AMD API shim after the build.

Since the build tool traces module dependencies, it can be used to inform what native/binary components need to be installed.

The build process can be run as part of the `pgm run` command that packs up the app and delivers it to the test device.

More on specific files below...

## index.html <a name="index-html"></a>

```html
    <!DOCTYPE html>
    <html>
    <head>
      <title>submarine</title>
      <meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0" />
      <meta charset="utf-8">
      <link rel="stylesheet" href="css/index.css" type="text/css" media="screen">
      <script data-main="js/index" src="js/phonegap.js"></script>
    </head>
    <body>
    </body>
    </html>
```

## js/phonegap.js  <a name="phonegap-js"></a>

* document.write's out the require.js tag
* maps the 'phonegap' module ID to the 'phonegap/src.js' file.
* it will load js/index.js async style.
* Only used in dev.
* In the build output, this file will be replaced with all the minified/combined modules for the app, no document.write, and no require.js -- it can be almond.js instead.
* If user wants dynamic loading, for instance for the google maps API, then require.js can be included instead (build system can figure that out, scan for dynamic require([]) calls).

## js/index.js  <a name="index-js"></a>

```javascript
    require(['phonegap!', 'my/mod'], function (phonegap, myMod) {
        //phonegap used as a loader plugin.
        //This function is called when phonegap device ready state is achieved, passes the
        //phonegap function as defined in 'phonegap/core' as the value of the dependency.
        //recall that phonegap.js has mapped the 'phonegap' module ID to
        //be the phonegap/src.js file when in dev/browser emulator.
    });
```

## phonegap/src.js <a name="src-js"></a>

The "dev" version of phonegap. Looks something like:

```javascript
    define(['phonegap/core', 'env!env/plugins'], function (core, plugins) {
        //Plugins have already registered with core, just return core function.
        return core;
    });
```

## phonegap/core.js <a name="core-js"></a>

Defines the main three public APIs. Calls for the environment-specific
exec.js to define phonegap.exec.

```javascript
    define(['env!env/exec'], function (exec) {
        //put in pub/sub stuff in here, do the core registering
        //of init and deviceready
        function on() {} //or whatever

        function phonegap(onDeviceReady) {
            //hold on to onDeviceReady callbacks until ready.
        }

        //Attach pub/sub methods to phonegap here
        phonegap.on = on; //or whatever

        //Attach platform-specific exec function.
        phonegap.exec = exec;

        //Add AMD loader plugin support, to suppor 'phonegap!'
        //dependencies in userland code
        phonegap.load = function (name, require, load, config) {
            if (config.isBuild) {
                load();
            } else {
                Phonegap(load);
            }
        };

        return phonegap;
    });
```

## android/plugins.js <a name="plugins-js"></a>

The file is loaded for 'env!env/plugins' when env === android. As the user does `pgm add sms` that
would end up placing the sms.js file in the **android** directory, then inserting
a `require('./sms');` in this file. Doing an `phonegap remove sms` would remove whatever
was added during the `pgm add sms` step.

```javascript
    define(function (require) {
        require('./sms');
    })
```

## android/sms.js <a name="sms-js"></a>

```javascript
    define(['phonegap/core', function (phonegap) {
        //Use phonegap's pub/sub to list to event that happens
        //after phonegap is basically ready but before the deviceready
        //stage.
        phonegap.on('init', function () {
            navigator.sms = function () {};
        })
    });
```
