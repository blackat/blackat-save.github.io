---
layout: post
title: "Fullstack Javascript Web Application"
date: 2014-09-09 22:13:06 +0200
comments: true
categories:
---

## The application
The application will allow the user to manage a list of movies, show them, update them, store new ones and delete existing ones. Basically a normal `CRUD` based application but totally written in Javascript.

__// todo: better specify the actors into play, the name of the resource that will be introduced later on__
The first flow will be a request from the `UI` to get the list of movies already stored in the database. The `REST` request will use the _HTTP verb_ `GET`, then `Express.js`, according to a list of configured routes, will call a method on the controller managing the movie resource. The controller will call the database to fetch data.

How to structure a full stack Javascript web application. I will consider these ingredients

## Base web application to scale
Let's start implementing a basic web application base on `node` and `expressjs`.

- Prerequisites
`node.js` must be installed, check [node] for your platform.
- Create `package.json`
This file is used by `npm` to store metadata of projects published as npm module.

~~~
{
    "name": "web-app-101",
    "version": "0.0.1",
    "dependencies": {
        "express": "4.0.x"
    },
    "devDependencies": {

    }
}
~~~
{:.language-json}

[express] is web application framework for node able to manage requests and responses, defining routes and much more.

- Create `server.js`

~~~
var express = require('express'),
    app = express();

app.get('/', function(req, res) {
    res.send("Hello World")
});

var server = app.listen(9001, function() {
    console.log('Express server is listening on port %d', server.address().port);
});
~~~
{:.language-javascript}

Express framework module is loaded by `require('express')` node method, then a new application is created invoking `express()`. Express provides _high level_ methods to defines routes and dealing with request and response.

A route is defined via `app.VERB` method, where _VERB_ stands for _HTTP VERB_, in this case `GET/`, the _URL_ the route is mapped to and the response to be sent.

Finally _bind_ and _listen_ for connection invoking method `app.listen()`.

- Run the application
Download modules running `npm install` from the command line, then `node server.js`, open a browser and hit `http://localhost:9001`.

[node]: (http://nodejs.org/download/) [node_installation]
[express]: (http://expressjs.com/) "express-web-framework"

### Add Express routes and structure the application
Express 4.0 has introduced the `app.route()` [method](http://expressjs.com/4x/api.html#app.route "Express route method") which is the recommended approach to handle HTTP verbs. We can start to structure a little bit the application creating a `routes.js` file under `lib` folder:

~~~
module.exports = function(app) {

    app.route('/').get(function(req, res, next){
        res.send('Hello World');
    });
};
~~~
{:.language-javascript}

then change `server.js` as follows:

~~~
var express = require('express'),
    app = express();

require('./lib/routes')(app);

var server = app.listen(9001, function(){
    console.log('Express server is listening on port %d', server.address().port);
});
~~~
{:.language-javascript}

now routes are in a separated files which is loaded by `server.js`.

### Add Single Page Web Application
Returning a message as a response to a `GET/` is fine, but usually it is better to have a page, for instance `index.html`, which serves us a __single page application__.

- Modify `routes.js` in order to call a function able to return our application page.

~~~
var index = require('./controllers');

module.exports = function(app) {

    app.route('/').get(index.index);
};
~~~
{:.language-javascript}


- Now add the _function_ able to return as a response the render of the `index.html` page.
~~~
exports.index = function(req, res) {
    res.render('index');
}
~~~
{:.language-javascript}

- Create the _html page_ which will support the single page web application.

~~~
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
~~~
{:.language-html}

- The `index.html` page cannot still be found by the __web framework__. Everything related to routing, serving resources and then handling requests and responses is managed by `expressjs`, the web framework. The page `index.html` is a resources. Configure the web framework to point to a folder containing the views which will be rendered to the user and the __render engine__: `ejs`.

~~~
var path = require('path');

module.exports = function(app) {

    var rootpath = path.normalize(__dirname + '/../..');

    app.set('views', rootpath + '/app');

    app.engine('html', require('ejs').renderFile);
    app.set('view engine', 'html');
};
~~~
{:.language-javascript}

- Add `"ejs": "~0.8.4"` to `package.json` file and run `npm install` to have the render engine available in `node_modules` folder.


Bla

~~~
var express = require('express'),
    app = express();

require('./lib/routes')(app);
require('./lib/config/express')(app);

var server = app.listen(9001, function(){
    console.log('Express server is listening on port %d', server.address().port);
});
~~~
{:.language-javascript}


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


### Dependencies

```javascript package.json
{
    "name": "simple-webapp",
    "description": "a simple fullstack web application",
    "version": "0.0.1",
    "private": true,
    "dependencies": {
        "express": "4.x"
    }
}
```

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

### Express

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

### NodeJS
I need something quite simple, when a request is routed some data should be fetched from the database and then sent back in the response to the client.

What I should implement, given a route, is the access to the data source to retrieve the data.
