---
layout: post
title: "kii web application"
date: 2014-12-11 19:32:27 +0100
comments: true
categories: [javascript, angularjs, android, genymotion, mobile, ios]
---
The current __tut__ explore how to create a simple mobile application with HTML5, CSS3 and JavaScript based on the new Kii Javascript SDK. Basically Kii Corporation offers a rich cloud mobile backend to help the development of a mobile application providing some services out of the box such as user registration and login. Then the application will be wrapped by Apache Cordova to obtain a deployable app for iOS and Android in a hybrid fashion.

{% img center /images/posts/genymotion-get-your-box.png %}
<!-- more -->

## Preview of the application in Plunker
A preview of the web application that is going to be implemented can be seen and tested in <a href="http://plnkr.co/edit/mSrWyzmmgOeSzCDKRWk8?p=preview" target="_blank">Plunker</a> using ``jon/dohh`` as username/password or just registering a new user. Password must be at least 4 charaters and special ones are not allowed

In order to make it work I had to brutally copy/paste the content of the Kii Javascript SDK in a file because Plunker does not allow file upload.

##Build steps

### 1. Create a Kii cloud application
First of all create a Kii account and then a new Kii application, put the name and select the HTML5 logo. Now download the <a href="https://developer.kii.com/v2/apps/5510/downloads-templates" target="_blank">Kii Javascript SDK</a>.

### 2. Clone Github repository
Clone the repository containing the simple application

    $ git clone https://github.com/blackat/kii-sdk-js-101.git

The application is the same application has been implemented in <a href="http://plnkr.co/edit/mSrWyzmmgOeSzCDKRWk8?p=preview" target="_blank">Plunker</a> with the following structure:

    ├── css
    │   ├── bootstrap.min.css
    │   └── style.css                   # custom style
    ├── fonts
    │   └── CherryCreamSoda.ttf         # a fancy font
    ├── index.html                      # single page of the web application
    ├── js
    │   ├── angular.min.js
    │   ├── angular.min.js.map
    │   ├── kiisdk.js                   # Kii JavaScript SDK
    │   └── script.js                   # custom script for Angular controllers
    ├── package.json
    └── server.js                       # server side configuration


In the ``index.html`` a minimal form has been implemented to allow login and registration of the user. Just to make the UI a bit more fancy and easy to develop, Boostrap and AngularJS frameworks have been added.

#### 2.1 Kii JavaScript SDK
The SDK has been used to perform _login_ and _registration_ action.

```javascript js/script.js https://github.com/blackat/kii-sdk-js-101/blob/master/js/script.js
$scope.login = function () {

    KiiUser.authenticate($scope.user.username, $scope.user.password, {
        // Called on successful registration
        success: function (theUser) {

        },

        // Called on a failed authentication
        failure: function (theUser, errorString) {

        }
    })
};
```
Some details has been remove just to focus on the ``authenticate`` function. Once the credential has been sent to the cloud one of the two functions will be called according to the outcome of the authentication.

The registration function is a bit different, a new ``KiiUser`` object has to be created with the data coming from the ``form`` and then the function ``register`` has to be called. As for the ``login`` function two callbacks are available to manage possible process outcomes.

```javascript js/script.js https://github.com/blackat/kii-sdk-js-101/blob/master/js/script.js
$scope.register = function () {

    // Create the KiiUser object
    var user = KiiUser.userWithUsername($scope.user.username, $scope.user.password);

    // Register the user, defining callbacks for when the process completes
    user.register({
        // Called on successful registration
        success: function (theUser) {

        },
        // Called on a failed registration
        failure: function (theUser, errorString) {

        }
    });
};
```

### 3. Local test with node.js
In order to locally test in a browser the application ``node.js`` can be used. Download and install <a href="http://nodejs.org/download/" target="_blank">node.js</a>. On mac use <a href="http://brew.sh/"target="_blank">homebrew</a>

    $ brew install node

once done use ``npm`` to install modules declared in ``package.json``

    $ npm install -g

where ``-g`` option means glabally, remove it to have all the modules installed locally in a folder named ``node_modules``. Then, in the folder where the repository has been cloned, run the command

    $ node server.js

and type in a browser tab ``http://127.0.0.1:5000/`` to load the web page.

### 4. PhoneGap/Apache Cordova Installation
PhoneGap wraps a native web application written in HTML, CSS and JavaScript in a native application of a given platform such as iOS, Android, Windows8 and more.

[Here](http://www.smashingmagazine.com/2014/02/11/four-ways-to-build-a-mobile-app-part3-phonegap/) it is possible to find a more detailed tutorial about hot to build a mobile application using PhoneGap.

#### PhoneGap vs Cordova
>PhoneGap is a distribution of Apache Cordova. You can think of Apache Cordova as the engine that powers PhoneGap, similar to how WebKit is the engine that powers Chrome or Safari.

More [here](http://phonegap.com/2012/03/19/phonegap-cordova-and-what%E2%80%99s-in-a-name/)

#### Prepare the environment
[Install Node.js](http://nodejs.org/download/) if not yet done.

Install Apache Cordova

    npm install -g cordova

Optional, to debug install [Apache Ripple emulator](http://www.raymondcamden.com/index.cfm/2013/11/5/Ripple-is-Reborn)

    $ npm install -g ripple-emulator

Install Android SDK

    $ brew install android

Install ''actual SDK stuff''

    $ android

[Install Genymotion](https://www.genymotion.com/#!/download) to test the mobile application for Android devices.

#### Genymotion
Genymotion offers a better and more straight forward way to test an Android app than the native emulator. Once done register one or more virtual divices according to your preferences.

{% img center /images/posts/genymotion-devices.png %}

Pay attention about the specified ``API version`` of the virtual devices because it must match the one installed previously when ``android`` command has been run to install ''actual SDK stuff'', both are at version 21.

{% img center /images/posts/android-sdk.png %}

### 5. Create the Apache Cordova application
#### Step A - Create a Cordova project
Run the command

    $ cordova create kii101phonegap com.kii.phonegap Kii101PhoneGap

This command will create a project named _Kii101PhoneGap_ in the folder ``kii101phonegap``.

#### Step B - Add project files
Copy the cloned repository content under the ```www``` into the project just created.

    ├── css
    │   ├── bootstrap.min.css
    │   └── style.css
    ├── fonts
    │       └── CherryCreamSoda.ttf
    ├── index.html
    └── js
        ├── angular.min.js
        ├── angular.min.js.map
        ├── kiisdk.js
        └── script.js

#### Step C - Add platforms
Switch to folder ``kii101phonegap`` and add the platforms against which the test will be done, for instance iOS and Android

    $ cordova platform add ios
    $ cordova platform add android

#### Step D - Build and emulate
Build the application and test it in __iOS__ platform running

    $ cordova build ios
    $ cordova emulate ios

after the project have been built, an emulation window opens showing the developed application.

For __Android__ it is a bit different, before start the Genymotion virtual device created in one of the previous steps, then in the command line type

    $ adb devices

and see how the Genymotion virtual device is seen as a real one

    List of devices attached
    192.168.56.101:5555	device

so run the command

    $ cordova run android

and the application will be deployed into the Genymotion device.

## From the Kii Team
If you're interested in more samples that use __Kii Cloud__ please take a look at the [GitHub page](https://github.com/KiiCorp) from the Kii team. Also, if you're interested in other platforms, please check out their samples section available [here](http://docs.kii.com/en/samples/).

## Trobleshooting

#### Android target
Sometimes could happen that a prject has been created when a previuos ``android-sdk`` was installed, then having created a new device based on a more recent ``API`` the following error arises

    ERROR: Error: Please install Android target 19 (the Android newest SDK).
    Make sure you have the latest Android tools installed as well.
    Run "android" from your command-line to install/update any missing SDKs or tools.

In the folder ``kii101phonegap/platforms/android`` of the Cordova application check the following files and keep android-sdk version consistent.

    ├── AndroidManifest.xml     # <uses-sdk> tag
    ├── local.properties        # location of the SDK. This is only used by Ant
    ├── project.properties      # project target
