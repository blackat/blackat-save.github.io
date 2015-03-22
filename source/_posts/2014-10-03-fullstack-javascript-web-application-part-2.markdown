---
layout: post
title: "Fullstack Javascript Web Application Part 2"
date: 2014-10-03 23:52:35 +0200
comments: true
categories:
---

The single page web application will be enriched with a database support, a build system with dependencies management and a testing framework.

{% img center /images/posts/popcorn-time-screenshot.png %}

[Grunt] task runner and [Bower] package management system will be configured. Then [Mongodb] with a persistent model will be added, finally [Jasmine] and [Karma] will support tests and their execution.
<!-- more -->


## 1. Continuos build integration
[Grunt] is a _task runner_, means it runs pre-defined tasks and custom tasks defined by the user in order to achieve a certain goal such as the deployment in production, running unit and end-to-end (e2e) tests and so on.

[Bower] is a package management system quite widespread in the javascript community added to manage packages for client-side programming. It depends on _Node.js_ and _npm_.


### 1.1 Grunt Task Runner
``` json package.json https://github.com/blackat/popcorn-time-express-angular-mongodb-grunt/blob/master/package.json
{
    "name": "popcorn-time-backend",
    "description": "a web application based on node, express, mongodb and grunt task runner with bower dependencies manager",
    "version": "0.0.1",
    "private": true,
    "dependencies": {
        "express": "4.0.x",
        "ejs": "~0.8.4"
    },
    "devDependencies": {
        "grunt": "~0.4.5",
        "grunt-contrib-jshint": "~0.10.0",
        "grunt-contrib-uglify": "~0.5.0",
        "load-grunt-tasks": "~0.2.0",
        "grunt-express-server": "~0.4.19",
        "grunt-open": "~0.2.0",
        "time-grunt": "~0.2.1",
        "load-grunt-tasks": "~0.2.0"
    }
}
```

Modify the list of modules in order to include the dependencies to Grunt and Grunt plugins in `devDependencies`.

Previous dependencies have been removed and will be managed by Bower later on. `package.json` manages just Grunt and Bower dependencies necessary to manage the all project lifecycle.

### 1.2 Grunt file configuration
``` javascript Gruntfile.js
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

Add `Gruntfile.js` to the project used to define tasks and load plugins. Grunt configuration file is made of four parts

1. Wrapper function: when Grunt is run from the CLI, it looks for the configuration file in order to load plugins and run the required task. Grunt
2. Project and tasks configuration: here plugin's tasks are defined in order to be reused later on. For instance `express` Grunt plugin has been declared in the `package.json`, along with the plugin come some tasks which could be configured and used. `express` plugin task is declared and configured as the `open` plugin's task.
3. Load grunt plugins and tasks: for instance `express` plugin must be declared under `devDependencies` in `package.json`, configured in `Gruntfile.js` but even loaded. This section is done for loading such plugins which define tasks such as  `grunt.loadNpmTasks('grunt-express-server');`.

    When the project grows there could be many lines each one to load a plugin, this can become quite annoying and even result in a good amount of boiler-plate code. As alternative solution, use [load-grunt-task] plugin in order to automatically load plugins from `dependencies` and `devDependencies` in `package.json`, so just _load plugins in one line_.

4. Custom tasks: here define your task as aggregation of tasks defined and configured in section number 2.

[load-grunt-task]: (https://github.com/sindresorhus/load-grunt-tasks) "Load grunt task plugin"

### 1.3 First run
Run `npm install` to install modules locally or `npm install -g` to install modules globally then run `grunt serve` task.

### 1.4 Bower Package Management System
``` javascript bower.json https://github.com/blackat/popcorn-time-express-angular-mongodb-grunt/blob/master/bower.json
{
    "name": "popcorn-time-backend-bower",
    "version": "0.0.1",
    "homepage": "https://github.com/blackat/popcorn-time-express-angular-mongodb-grunt",
    "authors": [
        "blackat <blackat@somewhere.world>"
    ],
    "description": "A web application based on node, express, mongodb and grunt task runner with bower dependencies manager",
    "moduleType": [
        "node"
    ],
    "license": "MIT",
    "ignore": [
        "**/.*",
        "node_modules",
        "bower_components",
        "app/bower_components",
        "test",
        "tests"
    ]
}
```

Type `npm install -g bower` to install __Bower__ globally, then `bower init` in the root folder of the current project to have `bower.json` _manifest_ file created and initialized with some basic information.

#### 1.4.1 Bower in a Nutshell
_Bower_ manages front-end components (css, html, js), _npm_ manages Javascript modules called packages. Some components can be installed by _npm_ as well but step by step a cleaner difference between backend and frontend elements is arising. _Ember_ updates for instance are currently published only under _Bower_. More on the difference  [here](http://tech.pro/tutorial/1190/package-managers-an-introductory-guide-for-the-uninitiated-front-end-developer).

Thus `package.json` is used to install _Node_ modules such as _Grunt_, _Express_ and so on, instead `bower.json` for all the client-side libraries such as _Angular_, _Bootstrap_ and all the components could be required by `index.html` page.

#### 1.4.2 Configure Grunt Wiredep Task
``` json package.json https://github.com/blackat/popcorn-time-express-angular-mongodb-grunt/blob/master/package.json
},
    "devDependencies": {
        ...
        "grunt-wiredep": "~1.9.x",
        ...
    }
```

Add `grunt-wiredep` to `package.json`. It is the _grunt task_ will inject UI dependencies. Then add and configure the grunt task in `Gruntfile.js` as follows:

``` json Gruntfile.js https://github.com/blackat/popcorn-time-express-angular-mongodb-grunt/blob/master/Gruntfile.js
    ...
    grunt.initConfig({
        ...
        // Download and inject Bower components into the app
        wiredep: {
            target: {
                src: 'app/views/**/*.html',
                options: {
                    cwd: '',
                    dependencies: true,
                    devDependencies: false,
                    exclude: ['app/bower_components/bootstrap/dist/js/bootstrap.js']
                }
            }
        }
    });
        ...
    grunt.registerTask('serve', 'default is dev environment', function (target) {
        grunt.task.run([
            'wiredep',
            'express:dev',
            'open',
            'express-keepalive'
        ]);
    });
```


#### 1.4.3 Inject Bower Components
``` json bower.json https://github.com/blackat/popcorn-time-express-angular-mongodb-grunt/blob/master/bower.json
    ...
    "dependencies": {
        "angular": ">=1.2.*",
        "angular-resource": ">=1.2.*",
        "angular-route": ">=1.2.*",
        "bootstrap": ">=3.2.*",
        "less.js": ">=1.7.*",
        "fontawesome": ">=4.2.x",
        "angular-xeditable": ">=0.1.x",
        "angular-mocks": ">=1.2.*"

    },
    "devDependencies": {

    }
```

Insert the list of dependencies Bower has to inject into `index.html` and remove them from `package.json`. Now frontend dependencies are managed and injected by _Bower_ and not anymore by _npm_

``` html app/views/index.html https://github.com/blackat/popcorn-time-express-angular-mongodb-grunt/blob/master/app/views/index.html
<!DOCTYPE html>
<html lang="en" ng-app="movieApp">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Movie page</title>
    <!-- bower:css -->
    <!-- endbower -->
    <link href="/styles/style.less" rel="stylesheet/less">
</head>
<body ng-controller="MainController">
...
</body>
<!-- bower:js -->
<!-- endbower -->
<script src="scripts/services/services.js"></script>
<script src="scripts/controllers/controllers.js"></script>
</html>
```

Specify in `index.html` where _Bower_ should insert css and javascript dependencies that have been specified in `bower.json`.

Once done run

 - `bower install` to download _Bower_ components;
 - `grunt serve` to run all the defined tasks and have the server up and running.

 Just to test the injection task run `grunt wiredep` and check that dependencies have been added to the `index.html` page as follows:
``` html app/views/index.html https://github.com/blackat/popcorn-time-express-angular-mongodb-grunt/blob/master/app/views/index.html
<!DOCTYPE html>
<html lang="en" ng-app="movieApp">
<head>
     <meta charset="utf-8">
     <meta name="viewport" content="width=device-width, initial-scale=1">
     <title>Movie page</title>
     <!-- bower:css -->
     <link rel="stylesheet" href="../bower_components/bootstrap/dist/css/bootstrap.css" />
     <link rel="stylesheet" href="../bower_components/fontawesome/css/font-awesome.css" />
     <link rel="stylesheet" href="../bower_components/angular-xeditable/dist/css/xeditable.css" />
     <!-- endbower -->
     <link href="/styles/style.less" rel="stylesheet/less">
</head>
<body ng-controller="MainController">
 ...
</body>
<!-- bower:js -->
<script src="../bower_components/jquery/dist/jquery.js"></script>
<script src="../bower_components/angular/angular.js"></script>
<script src="../bower_components/angular-resource/angular-resource.js"></script>
<script src="../bower_components/angular-route/angular-route.js"></script>
<script src="../bower_components/less.js/dist/less-1.7.5.js"></script>
<script src="../bower_components/angular-xeditable/dist/js/xeditable.js"></script>
<script src="../bower_components/angular-mocks/angular-mocks.js"></script>
<!-- endbower -->
<script src="scripts/services/services.js"></script>
<script src="scripts/controllers/controllers.js"></script>
</html>
```

## 2. MongoDB

### 2.1 Model
As first thing I want to define the object model should be persisted, retrieved and finally show off in the client user interface. So in '../lib/models' define

``` javascript /lib/models/product.js
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


#### RESTful Backend
In order to better interact with the frontend, a RESTful backend has to be provided by means of _REST endpoints_ as follows:


|Url            | HTTP Verb | POST Body   | Result
|---------------|-----------|-------------|--------------------------|
|api/movies     | GET       | empty       | return all the movies    |
|api/movies     | POST      | JSON String | create a new movie       |
|api/movies/:id | GET       | empty       | return a single movie    |
|api/movies/:id | PUT       | JSON String | update an existing movie |
|api/movies/:id | DELETE    | empty       | delete an existing entry |



## 3. Morgan the Logger


## 4. Angular
A nice way to implement _CRUD operations_ in [Angular] is using `$resource` factory which helps to interact with a standard _REST endpoints_. In `bower.json` the module `angular-resource` is already present, then it will be injected into 'index.html' by [Bower].


### 4.1 Consuming a RESTful service
``` javascript /app/scripts/services/movieServices.js
'use strict';

var  movieServices = angular.module('movieApp.movieServices', ['ngResource']);

movieServices.factory('Movie', function($resource){
    return $resource('movies/movies/:id', {id:'@_id'});
});
```

Module `ngResource` needs to be installed and declared in order to use its __service__ `$resource`. The returned resources has __action methods__ providing higher-level behaviors than the low level `$http`.

According to Angular  [api](https://docs.angularjs.org/api/ngResource/service/$resource) a parametrized URL has to be specified, it is the _full endpoint_ address which covers all the URLs for basic CRUD operations.

    $resource(url, [paramDefaults], [actions], options);

`:id` is the parameter and `{id:@_id}` is the name of the parameter in the `data` object provided when the method will be called, so it means extract the value of parameter `data._id` and assign it to `id` parameter.

The returned object has a _default set_ of __resource actions__ which can be extended with custom `actions`.

    {
        'get':    {method:'GET'},
        'save':   {method:'POST'},
        'query':  {method:'GET', isArray:true},
        'remove': {method:'DELETE'},
        'delete': {method:'DELETE'}
    };



## 5. Testing

 - unit test the RESTful api, user [SuperTest](https://github.com/visionmedia/supertest)
  - Jasmine
  - Karma
  - Mongodb? integration test?

<!-- Links -->
[Grunt]:    http://gruntjs.com/ "Grunt homepage"
[Bower]:    bower.io    "Bower homepage"
[Mongodb]:  http://www.mongodb.org/ "MongoDB homepage"
[Jasmine]:  http://jasmine.github.io/   "Jasmine homepage"
[Karma]:    http://karma-runner.github.io/0.12/index.html "Karma homepage"
[mongoose]: http://mongoosejs.com/ "Mongoose home page"
[mongoose_schema]: http://mongoosejs.com/docs/guide.html "Mongoose Schemas"
[route]: http://expressjs.com/4x/api.html#app.route "express route api"
[Angular]: https://angularjs.org/
