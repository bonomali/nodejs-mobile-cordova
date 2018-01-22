# Node.js for Mobile Apps Cordova plugin

## Installation

```bash
$ cordova plugin add nodejs-mobile-cordova
```

## Requirements

 - Cordova 7.x (Cordova 8.x is currently not supported)
 - iOS 11 or higher
 - Android API 21 or higher

When building an application for the Android platform, make sure you have the [Android NDK](https://developer.android.com/ndk/index.html) installed and the environment variable `ANDROID_NDK_HOME` set, for example:
```bash
$ export ANDROID_NDK_HOME=/Users/username/Library/Android/sdk/ndk-bundle
```

## Supported Platforms

- Android (ARMv7a, x86)
- iOS (ARM64)

## Reporting Issues

We have a [central repo](https://github.com/janeasystems/nodejs-mobile/issues) where we manage all the issues related to Node.js for Mobile Apps, including specific issues of the Node.js for Mobile Cordova plugin.
So please, open the issue [there](https://github.com/janeasystems/nodejs-mobile/issues).

## Cordova Methods

- `nodejs.start`
- `nodejs.startWithScript`
- `nodejs.channel.setListener`
- `nodejs.channel.send`


### nodejs.start

```js
  nodejs.start(scriptFileName, callback);
```

### nodejs.startWithScript

```js
  nodejs.startWithScript(scriptBody, callback);
```

### nodejs.channel.setListener

```js
  nodejs.channel.setListener(listenerCallback);
```

### nodejs.channel.send

```js
  nodejs.channel.send(message);
```

## Node.js Methods

```js
  var cordova = require('cordova-bridge');
```

- `cordova.channel.send`
- `cordova.channel.on`

### cordova.channel.send

```js
 cordova.channel.send(message);
```

### cordova.channel.on

```js
 cordova.channel.on('message', listnerCallback);
```


## Usage

This shows how to build an iOS app that exchanges text messages between the Cordova layer and the Node.js layer.  
In macOS, using `Terminal`:
```bash
$ cordova create HelloCordova
$ cd HelloCordova
$ cordova platform add ios
$ cordova plugin add nodejs-mobile-cordova
```
You can either manually create the `./www/nodejs-project/` folder, the `./www/nodejs-project/main.js` script file and edit `./www/js/index.js` or use a provided helper script to do it automatically.

---
#### Set up project files using the helper script
If you choose to use the helper script, you will be asked to overwrite the existing `./www/js/index.js` file:
```bash
$ ./plugins/nodejs-mobile-cordova/install/sample-project/copy-sample-project.sh
$ overwrite www/js/index.js? (y/n [n]) y
```
The script creates the `./www/nodejs-project/` folder and adds two files:
 - `./www/nodejs-project/main.js`
 - `./www/nodejs-project/package.json`

It also changes `./www/js/index.js` to add code to invoke Node.js for Mobile from Cordova.

---
#### Set up project files using the manual steps
If you want to do those steps manually, first create the project folder for the Node.js files:
```bash
$ mkdir www/nodejs-project
```
Then, with your editor fo choice (we use VS Code in this example) create the `main.js` script file:
```bash
$ code www/nodejs-project/main.js
```
Add the following code to `main.js` and save the file:
```js
const cordova = require('cordova-bridge');

cordova.channel.on('message', function (msg) { 
  console.log('[node] received:', msg); 
  cordova.channel.send('Replying to this message: ' + msg);
});
```
Edit the cordova script file `www/js/index.js`:
```
$ code `./www/js/index.js`
```
Append the following code at the end of the file:
```js
function channelListener(msg) {
    console.log('[cordova] received:' + msg);
}
  
function startNodeProject() {
nodejs.channel.setListener(channelListener);
nodejs.start('main.js',
             function(err) {
               if (err) {
                 console.log(err);
               } else {
                 console.log ('NodeJs Engine Started');
                 nodejs.channel.send('Hello from Cordova!');
               }
             });
}
```

Search for the `onDeviceReady` event and in the event handler add a call to `startNodeProject()`:
```js
  onDeviceReady: function() {
      this.receivedEvent('deviceready');
      startNodeProject();
  },
```
Save the changes to the `www/js/index.js` file to complete the manual steps of setting up the project files.

---

After the project files have been created, either manually or using the helper script, open the Cordova app project in Xcode:
```bash
$ open platforms/ios/HelloCordova.xcodeproj
```
Switch to Xcode:  
 * select HelloCordova to view the project settings  
 * in the `General` settings:  
    * in the `Signing` section select a team to sign the app  
    * in `Deployment Info` section select `Deployment Targert` `11.0` or higher  
 
Go back to `Terminal` to build the Cordova app 
```bash
$ cordova build ios --device
```
Switch to Xcode:
 * select a target device for the project
 * run the project
 * enlarge the `Console` area, at the end of the console log it should show:

```bash
2017-10-02 18:49:18.606100+0200 HelloCordova[2182:1463518] NodeJs Engine Started
[node] received: Hello from Cordova!
2017-10-02 18:49:18.690132+0200 HelloCordova[2182:1463518] [cordova] received: Replying to this message: Hello from Cordova!
```

## NPM Modules
NPM modules can be added to the project under the `./www/nodejs-project/` folder. You will need a `package.json` file in the `./www/nodejs-project` folder.  
If you used the helper script to install the sample project, the `package.json` file is already present and you can proceed adding the desired NPM modules.  
If you don't know how to create one, just copy the sample one from `./plugins/nodejs-mobile-cordova/install/sample-project/www/nodejs-project/package.json`.
Then proceed as you would do for a regular Node.js project:
```
$ cd www/nodejs-project/
$ npm install module-name
```

Rebuild your Cordova project so that the newly added NPM modules are added to the Cordova application.

At present, only pure javascript modules are supported. Native modules, that require to be crossed compiled for the target platform, are not supported yet.

On Android, the project files and the NPM modules need to be extracted from the APK assets in order to make them available to the Node.js for Mobile engine. They are extracted from the APK and copied to a working folder when the application is launched for the first time or a new version of the application has been installed.
To expedite the process of extracting the files, a list of files (file.list) and a list of folders (dir.list) are created when the application is compiled and then added to the application assets.
