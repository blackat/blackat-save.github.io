---
layout: post
title: "Fullstack Javascript Web Application Part 1"
date: 2014-09-09 22:13:06 +0200
comments: true
categories: [javascript, monmgodb, grunt, bower, webapp, fullstack]
---

The web application will allow the user to manage a list of movies, show them, update them, store new ones and delete existing ones.

{% img center /images/posts/popcorn-time-screenshot.png %}

This web application will help to show how to structure a full stack Javascript web application and which components are required to make it work.

The application will start simple and step by step components will be added such as the database, the testing framework, the task runner and the dependency management.

<!-- more -->

## 1. Single Page Web Application Scaffolding
As first step the web application will be composed by

- Node.js the web server
- Express the web framework

### 1.1 Folder structure
The starting point is the folder structure to organize resources and configuration files.

    ├── app
    │   └── views
    │       └── index.html              # => single page of the web application
    ├── lib
    │   ├── config
    │   │   └── express.js      # => configuration file for expressjs framework
    │   ├── controllers
    │   │   └── index.js        # => controller to serve the starting page
    │   └── routes.js           # => router to map REST request with controller
    ├── package.json            # => set of required nodeJs modules
    └── server.js               # => boostrap of the server side

### 1.2 Express Dependency

``` json
{
    "name": "popcorn-time-backend",
    "version": "0.0.1",
    "dependencies": {
        "express": "4.0.x"
    },
    "devDependencies": {

    }
}
```

__Express__ is a web application framework for __Node.js__ used to develop web application totally written in Javascript. The framework provides _high level methods_ way to manage requests mapping a route with the designated controller able to provide data.

In the `package.json` file add the dependency on the web framework. This file is used by `npm` to store metadata of projects published as npm module and to easily installed them locally running the command `npm install`.


### 1.3 Server Configuration

``` javascript
var express = require('express'),
    app = express();

app.get('/', function(req, res) {
    res.send("Hello World")
});

var server = app.listen(9001, function() {
    console.log('Express server is listening on port %d', server.address().port);
});
```

__Express__ module is loaded by `require('express')` then a new application instance is created invoking `express()`.

Defined a route via

- `app.VERB` method, where `VERB` stands for `HTTP VERB`. In the example `GET` is the verb;
- `/` the `URL` the route is mapped to;
- a function implementation to manage request and response.

Finally _bind_ and _listen_ for connection invoking method `app.listen()`.

### 1.4 First Run of the Application
Run `npm install` in the command line to locally install the declared dependencies. Once done type `node server.js`, open a browser and hit `http://localhost:9001`.

[node]: (http://nodejs.org/download/) [node_installation]
[express]: (http://expressjs.com/) "express-web-framework"

### 1.5 Separate Express routes

``` javascript lib/routes.js
module.exports = function(app) {

    app.route('/').get(function(req, res, next){
        res.send('Hello World');
    });
};
```

Express 4.0 has introduced the `app.route()` [method](http://expressjs.com/4x/api.html#app.route "Express route method") which is the recommended approach to handle HTTP verbs. Routes are in a separated files which is loaded by `server.js` as follows.

``` javascript server.js
var express = require('express'),
    app = express();

require('./lib/routes')(app);

var server = app.listen(9001, function(){
    console.log('Express server is listening on port %d', server.address().port);
});
```

### 1.6 Return index.html

``` javascript lib/routes.js
var index = require('./controllers');

module.exports = function(app) {

    app.route('/').get(index.index);
};
```

The `html` page will give the life to the single page web application. The function calls the `index` controller method called `index` which returns the page as follows:

``` javascript lib/controllers/index.js
exports.index = function(req, res) {
    res.render('index');
}
```

Then the `html` page to be returned

``` html app/index.html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>Popcorn Time</title>
</head>
<body>
    <h1>Popcorn Time</h1>
</body>
</html>
```

### 1.7 Express Configuration and the Render Engine
The `index.html` page cannot still be found by express. Configure the web framework to render and serve views from a given folder.

``` javascript lib/config/express.js
var path = require('path');

module.exports = function(app) {

    var rootpath = path.normalize(__dirname + '/../..');

    app.set('views', rootpath + '/app/views');

    app.engine('html', require('ejs').renderFile);
    app.set('view engine', 'html');
};
```

Finally add `ejs` render engine for `html` page as a dependency

``` json package.json
{
    "name": "popcorn-time-backend",
    "version": "0.0.1",
    "dependencies": {
        "express": "4.0.x",
        "ejs": "~0.8.4"
    },
    "devDependencies": {

    }
}
```

### 1.8 Update Server with Express Configuration
Update the server requiring the express configuration

``` javascript server.js
var express = require('express'),
    app = express();

require('./lib/routes')(app);
require('./lib/config/express')(app);

var server = app.listen(9001, function(){
    console.log('Express server is listening on port %d', server.address().port);
});
```

then run `npm install`, `node server.js` and check the result in the browser.

## 2 Web Application features and Look & Feel
In this second step some features and look & feel will be added to the web application through

- Angular the ui framework
- Bootstrap the css framework

### 2.1 Folder Structure

    ├── app
    │   ├── favicon.ico
    │   ├── fonts
    │   │   └── Lobster_1.3.otf
    │   ├── images
    │   │   ├── play.png
    │   │   └── popcorn.png
    │   ├── package.json                # => dependency modules of the frontend
    │   ├── scripts
    │   │   ├── controllers
    │   │   │   ├── controllers.js      # => angular controllers
    │   │   │   └── xeditable.min.js    # => angular xeditable
    │   │   ├── directives
    │   │   └── services
    │   │       └── services.js         # => angular REST services
    │   ├── styles
    │   │   ├── style.less              # => less style sheet
    │   │   └── xeditable.css
    │   └── views
    │       └── index.html              # => single page of the web application
    ├── lib
    │   ├── config
    │   │   └── express.js
    │   ├── controllers
    │   │   └── index.js
    │   └── routes.js
    ├── package.json
    ├── popcorn-time-express-angular.iml
    └── server.js

Some folders have been added in order to better organize the frontend code, images, styles and so on.


### 2.2 Dependencies Configuration
```json app/package.json https://github.com/blackat/popcorn-time-express-angular/blob/master/app/package.json
{
    "name": "popcorn-time-frontend",
    "version": "1.0.0",
    "dependencies": {
        "angular": "1.2.x",
        "angular-mocks": "~1.2.x",
        "angular-route": "~1.2.x",
        "angular-resource": "0.1.0",
        "bootstrap": "3.2.x",
        "less": "1.7.x"
    },
    "devDependencies": {

    }
}
```
The frontend has a separated file to configure the dependencies from the backend one. _Angular_ and _Bootstrap_ dependencies have been added along with _Less_ in order to resolve `*.less` style sheet files on the fly.

### 2.3 Serve Static Resources

``` javascript lib/config/express.js https://github.com/blackat/popcorn-time-express-angular/blob/master/lib/config/express.js
'use strict';

var path = require('path'),
    express = require('express');

module.exports = function(app) {

    var rootpath = path.normalize(__dirname + '/../..');

    app.use(express.static('app'));
    app.set('views', rootpath + '/app/views');

    app.engine('html', require('ejs').renderFile);
    app.set('view engine', 'html');
};
```
Let's add the configuration for the static resources path so they can be referenced from the `index.html` page.
