# Electrify-updated-test

Easily package your Meteor apps with Electron, and *butter*.

## Updated version notes:
This updated version of Electrify now defaults to Electron 1.4.0. When requiring Electrify, use `require('electrify-updated-test')`
Also the developpement mode now uses 1.4.0 as well.
Other than that there are a few tweaks here and there. Nothing major.

And if anyone is interested, I think ASAR support should be the next thing we should work on for this package. It would require some writing from the ground-up just so you know, given that this package relies on spawning mongo and node, and ASAR does not support that. If anyone's up to it, let me know.

This package **belongs to its original author Anderson Arboleya**. Check [Meteor x Electron integration](https://github.com/arboleya/electrify). The purpose of this fork is simply to keep it up to date and resolve some issues that keep cropping up as Electron and Meteor keep receiving new features.

## Tweaking
Now, Electrify runs a self-contained MongoDB server. Running the app with `electrify` will run fine. However, once you package the app and run it, you might run into a MongoDB driver issue, something like
```
 The default storage engine 'wiredTiger' is not available with this build of mongod. Please specify a different storage engine explicitly, e.g. --storageEngine=mmapv1
```
causing the app not to start; That is because Electrify natively uses an older version of MongoDB than the one it bundles with your app (in fact, it copies the executables directly from your `.meteor` folder).

How to fix
----------

Refer to [this issue](https://github.com/arboleya/electrify/issues/61#issuecomment-238031131). Long story short,  you must have an older version `mongo.exe` and `mongod.exe` (or executables, in the case of Linux) in your .electrify/bin folder before you package. You can find the link to those in the above link. Otherwise, if your app is already packaged, copy them to your/app/path/.../resources/app/bin. The first method just ensures that future packaging will not run into this issue each time.

Some more Tweaking
------------------

This case assumes you want to specify a distant MongoDB for your app (or change entirely to something like CouchDB). Now Electrify does not natively support that, and *will ignore* a `MONGO_URL` environnement variable if you pass one. Now to fix this, you need to modify this line: `'self.env.MONGO_URL = 'mongodb://localhost:'+ self.port +'/meteor';` to: `self.env.MONGO_URL = process.env.MONGO_URL;` in the `lib/plugins/mongodb.js` file

You can either modify this file in your global packages folder for this modification to be effective in all future projets. (see `npm root -g`), or inside your packaged app, `my-electrified-app/resources/app/node_modules/electrify-updated-test/lib/plugins/mongodb.js`


## TL;DR (Old docs, still useful)

````shell
npm install -g electrify
cd /your/meteor/app
electrify
````
## Compatibility

Works on all Meteor's supported [platforms](https://github.com/meteor/meteor/wiki/Supported-Platforms).

## Help

````bash
$ electrify -h

  Usage: electrify [command] [options]

  Commands:

    run       (default) start meteor app within electrify context
    bundle    bundle meteor app at `.electrify` dir
    package   bundle and package app to `--output` dir

  Options:

    -h, --help             output usage information
    -V, --version          output the version number
    -i, --input    <path>  meteor app dir       | default = .
    -o, --output   <path>  output dir           | default = .electrify/.dist
    -s, --settings <path>  meteor settings file | default = null (optional)

  Examples:

    # cd into meteor dir first
    cd /your/meteor/app

    electrify
    electrify run
    electrify package
    electrify package -o /dist/dir
    electrify package -o /dist/dir -s file.json
    electrify package -i /app/dir -o /dist/dir -s dev.json
    electrify package -- <electron-packager-options>

    # more info about electron packager options:
    # ~> https://www.npmjs.com/package/electron-packager
````

## Installation

````shell
npm install -g electrify
````

> For invoking Electron methods from Meteor, you'll also need to install the
> `arboleya:electrify` meteor package.
> For more info check [Meteor x Electron integration](#meteor-x-electron-integration).

## Running app

````shell
cd /your/meteor/app
electrify
````

## Packaging

````shell
cd /your/meteor/app
electrify package
````

The packaging process is done under the hood using `electron-packager`
npm package. The following variables are automatically set:

  * `--out` -- *comes from cli option [-o, --out]*
  * `--arch` -- *comes from system [current arch]*
  * `--platform` -- *comes from system [current platform]*
  * `--version` -- *comes from .electrify/package.json [current app version]*

You can overwrite these default values and also set others by passing custom
arguments directly to `electron-packager` after `--`, i.e:

````shell
cd /your/meteor/app
electrify package -- --icon=/folder/x/img/icon.png --version=x.y.z
````

All the available options for `electron-packager` can be found here:
https://www.npmjs.com/package/electron-packager

### Notes

The output app will match your current operational system and arch type.

  * To get an **OSX** app, run it from a **Osx** machine.
  * To get an **Linux 32bit** app, run it from a **32bit Linux** machine.
  * To get an **Linux 64bit** app, run it from a **64bit Linux** machine.
  * To get an **Windows 32bit** app, run it from a **32bit Windows** machine.
  * To get an **Windows 64bit** app, run it from a **64bit Windows** machine.

Due to NodeJS native bindings of such libraries such as Fibers -- *which are
mandatory for Meteor*, you'll need to have your Meteor app fully working on the
desired platform before installing this plugin and packaging your app.

So, at this time, you cannot package your app in a cross-platform fashion from
one single OS.

Perhaps you can live with it? :)

> **DO NOT** use options to output for multiple arch/platforms at once, such as
`--arch=all`. It won't work, Electrify can bundle Meteor apps only for the
platform you're running on.


## Options

1. `-i, --input` - Meteor app folder, default is current directory (`process.cwd()`).
1. `-o, --output` - Sets output folder for your packaged app, default is
`/your/meteor/app/.dist`
1. `-s, --settings` Sets path for Meteor
[settings](http://docs.meteor.com/#/full/meteor_settings) file, this will be
available inside your Meteor code both in development and after being packaged.

## Structure

You'll notice a new folder called `.electrify` in your meteor app dir, its
structure will be like this:

````
/your/meteor/app
├── .electrify
│   ├── .gitignore
│   ├── electrify.json
│   ├── index.js
│   └── package.json
├── .meteor
└── ...
````

This is a pure Electron project, so you can use the whole Electron API from JS
files in this folder. Also, you can install electron dependencies and store them
in the `package.json` file. Note that the `electrify` package is itself a
dependency.

See this folder as the `desktop layer` for your Meteor app. Remember to check
out the `index.js` file, it constains the electrify start/stop usage.

The `electrify.json` file will hold specific preferences for Electrify, such as
plugins and so on. It's still a WIP, but you can get around it.

### Config (`electrify.json`)

For now there's only one option here: `preserve_db`.

Set it to true to preserve database between installs. It works by saving the
mongo data dir inside user's data folder, instead of being self contained within
the app folder (which gets deleted when new version is installed).

# Customizing

Let's see how one would be able to do a simple SplashScreen:

````javascript
var app       = require('app');
var browser   = require('browser-window');
var electrify = require('electrify')(__dirname);

var window = null;
var splash = null; // splash variable

app.on('ready', function() {
  splash = new browser({ // starts splash window
    // >>> your configs here
  });
  splash.loadUrl('./splash.html'); // create the ".electrify/splash.html" file

  // then move along and start electrify
  electrify.start(function(meteor_root_url) {

    // before opening a new window with your app, destroy the splash window
    splash.close(); // >>> or .destroy(), check what works for you


    // from here on, well, nothing changes..


    window = new browser({
      width: 1200, height: 900,
      'node-integration': false // node integration must to be off
    });
    window.loadUrl(meteor_root_url);
  });
});


// ....

````

## Meteor x Electron integration

You can seamlessly call Electron methods from your Meteor's client/server code.

Define your Electron methods inside the `.electrify` folder:

````javascript
// `.electrify/index.js` file
electrify.methods({
  'hello.world': function(firstname, lastname, done) {
    // do things with electron api, and then call the `done` callback
    // as ~> done(err, res);
    done(null, 'Hello '+ firstname +' '+ lastname +'!');
   }
});
````

Then, in your Meteor code (client and server), you can call this method like:

````javascript
// Electrify.call(method_name, args, done_callback);
Electrify.call('hello.world', ['anderson', 'arboleya'], function(err, msg) {
  console.log(msg); // Hello anderson arboleya!
});
````

> **IMPORTANT**
>
> You can only call methods after the connection is made between Meteor and
> Electron, to make sure it's ready you can wrap your code in a startup block:
>
> ````javascript
> Electrify.startup(function(){
>   Electrify.call(...);
> });
> ````

## Upgrading

When upgrading to newer versions, it's **important** to know that:

### ~> templates

Once these files exists on disk, they *will not* be overwritten.
 * `.electrify/index.js`
 * `.electrify/package.json`
 * `.electrify/electrify.json`
 * `.electrify/.gitignore.json`

### ~> api

As these files above is never overwritten, in case of any API change that needs
adjustments, these will have to be made manually.

### ~> version matching

Always keep the same electrify version in your Meteor, and inside the
`.electrify` folder, *as per specified in `.electrify/package.json` file*.

## Questions?

Do not open issues, use the chat channel instead.

[![Join the chat at https://gitter.im/arboleya/electrify](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/arboleya/electrify)

## Problems?

This is very young and active software, so make sure your are always up to date
before opening an issue. Follow the released fixes through the
[HISTORY.md](HISTORY.md) file.

If you find any problem, please open a meaningful issue describing in detail how
to reproduce the problem, which platform/os/arch type you're using, as well as
the version of Meteor and Electrify, and any other info you may find usefull.

## License

The MIT License (MIT)

Copyright (c) 2015 Anderson Arboleya
