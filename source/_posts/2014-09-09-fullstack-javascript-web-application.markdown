---
layout: post
title: "Fullstack Javascript Web Application"
date: 2014-09-09 22:13:06 +0200
comments: true
categories:
---

## The Single Page Web Application
The web application will allow the user to manage a list of movies, show them, update them, store new ones and delete existing ones.

{% img center /images/posts/popcorn-time-screenshot.png %}

This web application will help to show how to structure a full stack Javascript web application and which components are required to make it work.

The application will start simple and step by step components will be added such as the database, the testing framework, the task runner and the dependency management.

## Web Application Scaffolding
As first step the web application will be composed by

- Node.js the web server
- Express the web framework

<!-- more -->

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


### 2.2 Serve Static Resources

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


## Continuos build integration

// todo: how to install them
- Grunt: task runner
- Bower: dependency management system

Grunt is a _task runner_, means it runs pre-defined tasks and custom tasks defined by the user in order to achieve a certain goal such as the deployment in production, running unit and end-to-end (e2e) tests and so on.

Add following two files to the root of the project:

-   `package.json`: used by `npm` to store metadata of projects published as npm module, Grunt is one of them as Grunt plugins.
-   `Gruntfile.js`: used to define tasks and load plugins.


##### package.json
List Grunt and all the Grunt plugins in `devDependencies.`

~~~
{
    "name": "movie-app",
    "version": "0.0.1",
    "dependencies": {
        "connect": "3.1.x",
        "serve-static": "1.4.x",
        "express": "4.0.x",
        "mongoose": "3.9.x"
    },
    "devDependencies": {
        "grunt": "~0.4.5",
        "load-grunt-tasks": "~0.2.0",
        "grunt-express-server": "~0.4.19",
        "grunt-open": "~0.2.0"
    }
}
~~~
{:.language-json}

##### Grunt file configuration
Grunt configuration file is made of four parts

1. Wrapper function: when Grunt is run from the CLI, it looks for the configuration file in order to load plugins and run the required task. Grunt
2. Project and tasks configuration: here plugin's tasks are defined in order to be reused later on. For instance `express` Grunt plugin has been declared in the `package.json`, along with the plugin come some tasks which could be configured and used. `express` plugin task is declared and configured as the `open` plugin's task.
3. Load grunt plugins and tasks: for instance `express` plugin must be declared under `devDependencies` in `package.json`, configured in `Gruntfile.js` but even loaded. This section is done for loading such plugins which define tasks such as  `grunt.loadNpmTasks('grunt-express-server');`.

    When the project grows there could be many lines each one to load a plugin, this can become quite annoying and even result in a good amount of boiler-plate code. As alternative solution, use [load-grunt-task] plugin in order to automatically load plugins from `dependencies` and `devDependencies` in `package.json`, so just _load plugins in one line_.

    Do not forget to add `"load-grunt-tasks": "~0.2.0",` to your `package.json`.

4. Custom tasks: here define your task as aggregation of tasks defined and configured in section number 2.

```javascript Gruntfile.js
'use strict';

// 1. Wrapper function
module.exports = function (grunt) {

    // replace step 3
    require('load-grunt-tasks')(grunt);

    // 2. Project and tasks configuration
    grunt.initConfig({
        pkg: grunt.file.readJSON('package.json'),
        express: {
            options: {
                port: process.env.PORT || 9000
            },
            dev: {
                script: 'server.js',
                debug: true
            }
        },
        open: {
            server: {
                url: 'http://localhost:<%= express.options.port %>'
            }
        }
    });


    // 3. Load grunt plugins and tasks

    // 4. Custom tasks
    grunt.registerTask('serve', 'default is dev environment', function(target){

        grunt.task.run([
            'express:dev',
            'open'
        ]);
    });
}

```

[load-grunt-task]: (https://github.com/sindresorhus/load-grunt-tasks) "Load grunt task plugin"

##### First run
Before running the `Grunt serve` task just defined, declared modules have to be downloaded, su run `npm install` to install modules locally or `npm install -g` to install modules globally.

#### User interface
- AngularJS: UI MVC framework

#### Server side
- NodeJS: server
- Express: web application framework for node

#### Database
- MongoDB: NoSQL database

### Project folder structure

### Init Grunt and Bower
just to have a basic project which can run tests


### MongoDB

#### Model
As first thing I want to define the object model should be persisted, retrieved and finally show off in the client user interface. So in '../lib/models' define

```javascript /lib/models/product.js
'use strict';

var mongoose = require('mongoose'),
    Schema   = mongoose.Schema;

var ProductSchema = new Schema({
    name: String,
    description: String
    });

mongoose.model('Product', ProductSchema);
```
I have decided to use [mongoose] [mongoose] in order to reduce boilerplate code for validation, read/write operation and so on.

>With Mongoose, everything is derived from a Schema. Each [schema] [mongoose_schema] maps to a MongoDB collection and defines the shape of the documents within that collection.

I have defined a very simple schema for a product with only two fields `name` and `description`.

>Models are fancy constructors compiled from our Schema definitions.

#### Routes

Once the user has required an action on a resource it has to be served according to its name and the `HTTP verb`. Thus create an instance of [route] with a specific name and then manage all the `HTTP verbs` avoiding duplicates in names.

``` javascript /lib/routes.js
'use strict';

var products = require('./controllers/products');

module.exports = function(app) {

    app.route('api/products').get(products.products);
};
```

[route]: http://expressjs.com/4x/api.html#app.route "express route api"



#### Controller
Once done the object should be retrieved from the data source, create a new module file in '../lib/controllers':

``` javascript /lib/controllers/products.js
'use strict';

var mongoose = require('mongoose'),
    Product = mongoose.model('Product');

exports.products = function (req, res) {
    return Product.find(function (err, prods) {
        if (!err) {
            return res.json(prods);
        } else {
            return res.send(err);
        }
    });
};

```
In order to retrieve all the product documents stored in the data source, use static method `find`. Documents are instances of the model.

The module `products.js` exports the function `products()` adding it to the `exports` object. The function will be added to the root of the module.


<!-- Reference links -->
[mongoose]: http://mongoosejs.com/ "Mongoose home page"
[mongoose_schema]: http://mongoosejs.com/docs/guide.html "Mongoose Schemas"

### NodeJS
I need something quite simple, when a request is routed some data should be fetched from the database and then sent back in the response to the client.

What I should implement, given a route, is the access to the data source to retrieve the data.
