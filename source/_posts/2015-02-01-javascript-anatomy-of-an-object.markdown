---
layout: post
title: "JavaScript: anatomy of an object"
date: 2015-02-01 11:27:38 +0100
comments: true
categories:
---
__JavaScript__ is a _prototypal inheritance language_ where objects inherit properties directly from other objects. Differently a _class-based languages_ such as Java defines an object as instance of a class and inheritance hierarchy is defined through classes. Packages then define the layout of files and folders in a project.

Objects in JavaScript are _class-free_ and _dynamic_ or _mutable_, they can change at any time during code execution. In Java objects are not dynamic, their layout is lock down by class definition.

JavaScript defines a _prototype linkage feature_ to allow an object to inherit properties from another object.

<!-- more -->

## 1. Object representation
The JavaScript world is divided in two big chunks:

1. _simple types:_ strings, booleans, numbers, `null` and `undefined`;
2. _objects:_ rest of the world such as functions, arrays, regular expressions and, naturally, objects.

{% blockquote Douglas Crockford http://shop.oreilly.com/product/9780596517748.do JavaScript: The Good Parts %}
Objects in JavaScripts are mutable keyed collections.
{% endblockquote %}

An object is represented as a collection of properties where each property has a _name_ and a _value_ as shown in the example below:

```javascript
'use strict';

var catFelix = {
    name: 'felix',
    house: 'house',
    domain: 'cat'
}

console.log(catFelix.name + '@' + catFelix.house + '.' + catFelix.domain);
//output: felix@house.cat
```

## 2. How to create an object

### 2.1 Object literal
The _object literal notation_ is the easiest way to create an object. It is just a pair of curly brackets:
```javascript
'use strict';

var cat = {};
```
The statement just creates an empty object, optionally it can be then populated as following:
```javascript
'use strict';

var cat = {};
cat.name = 'just a cat';
cat.house = 'none';
```
Another way is to directly populate the object optionally enclosing a collection of _key/value_ pairs between curly brackets:
```javascript
'use strict';

var catFelix = {
    name: 'felix',
    house: 'house',
    domain: 'cat'
}
```
Whenever is required a new property/value pair can be added to the object. Moreover an object might contain one or more objects as following:
```javascript
'use strict';

var catFelix = {
    name: 'felix',
    house: 'house',
    domain: 'cat',
    features: {
        language: 'meow',
        body: 'black',
        eyes: 'white',
        grin: 'giant',
        photo: 'felix-awesome.png'
    }
}
```
{% blockquote Douglas Crockford http://shop.oreilly.com/product/9780596517748.do JavaScript: The Good Parts %}
Objects are passed by reference, they are never copied.
{% endblockquote %}


#### 2.1.1 Property access
Once an object has been created, there are two ways to get access to _property values_, _[property_name]_ and _dot notation_
```javascript
    //Object declaration.

    console.log('I am ' + catFelix['name']); // I am felix

    console.log('I speak ' + catFelix['features']['language']); // I speak meow

    console.log('I have a ' + catFelix.features.grin + ' grin'); // I have a giant grin

    console.log('My shortcoming is ' + catFelix.features.shortcoming || "none"); // My shortcoming is undefined
```

### <a name="function_constructor"></a>2.2 Function Constructor
A class based language such as Java uses classes to obtain encapsulation and to structure a hierarchy. The constructor will build the object hierarchy and bring an object to an initial and well known state.

A constructor avoid to initialize properties one by one everytime a new object has to be created. JavaScript offers a such mechanism through the `new` operator.

{% blockquote Douglas Crockford http://shop.oreilly.com/product/9780596517748.do The Principles of Object-Oriented JavaScript %}
A Constructor is simply a function that is used with `new` to create an object.
{% endblockquote %}

```javascript
'use strict';

function Cat() {};

var cat = Cat();
console.assert(cat === undefined, "cat is not an object");

var cat = new Cat();
console.assert(cat === undefined, "cat is not anymore undefined");
```
The function `Cat()` is defined and it is called in two different ways:

1. not produce any new object and the `console.assert` method passes;
2. generate an exception, the `new` operator has allocated a new object and has automatically returned the reference to it stored in the variable `cat`.

To share properties (methods and variables) among objects of a _given type_ put them on the _prototype_ as follows:
```javascript
'use strict';

function Cat() {};

Cat.prototype.language = 'meow';

Cat.prototype.legs = 4;

Cat.prototype.getLegs = function() {
    return this.legs;
}

var felix = new Cat();
console.log("I speak " + felix.language + " language and I have " + felix.getLegs() + " legs");
```
Three properties have been attached to `prototype` having values of type `String`, `int` and `function` respectively. Notice that properties has been _attached_ and not _added_ [[2]]. For this reason JavaScript has been defined as _prototypal inheritance language_, objects can inherit properties directly from other objects.

Constructors, by convention, are stored in variable with a capitalized name to avoid the risk to be called without `new` operator.
```javascript
'use strict';

// Capitalized function name.
function Cat() {};

// New instance.
var justACat = new Cat();

// Capitalized function variable name.
var BlackCat = function() {};

// New instance.
var felix = new BlackCat();
```


## 3. Prototype Property
The property `prototype` is an _internal property_ and all internal properties are not accessible via code, they defines behavior of code. ECRMAScript defines multiple internal properties indicating them by _double square-bracket notation_ as `[[internal_property]]`.

{% blockquote Nicholas C. Zakas http://shop.oreilly.com/product/9781593275402.do The Principles of Object-Oriented JavaScript %}
You can think of a <span style="color:#B90504">prototype</span> as a recipe for an object.
{% endblockquote %}

##### Prototype property using object literal
Create an object via _object literal_ notation in a browser console (Safari in this case):
```javascript
> var cat = {};
> cat
< Object
    __proto__: Object
```
Typing the name of the variable will show type of the object which is `Object`, and the property `__proto__`. This _property_ is supported by WebKit based browser and allows to both read and write to the `[[Prototype]]` property.

The property `[[Prototype]]` is a pointer back to the prototype object used by the instance. In the example the property `__proto__` points to the _prototype object_ `Object` because it has been created via object literal. Object `cat` inherits properties directly from `Object.prototype`.

##### Prototype property using function constructor
Now let's define a _constructor function_ named `Cat` and create an object via the `new` operator:
```javascript
> function Cat(){};
> Cat.prototype.name = 'just a cat';
> var cat = new Cat();
> cat
< Cat
    __proto__: Cat
        constructor: function Cat() {}
        name: "just a cat"
        __proto__: Object
```
Typing `cat` something different happens: the variable `cat` contains a reference to an instance of type `Cat`, the property `__proto__` is of type `Cat` and, in turn, it has a property `constructor`, a property `name` (which is the one define on the `prototype` property of the constructor function) and, in the end, another `__proto__` prototype of type `Object`.

#### Prototype property in a nutshell
In JavaScript an object is defined by a set of pairs (property,value), prototype is an _inner hidden property_ whose value is an object representing the _prototype chain_. The _prototype_ property is a way to share common properties among objects in a _prototype chain_ in order to build a hierarchy.

The `__proto__` property reference the prototype chain used to return property values. The prototype chain determines from where an object inherits properties, if an object does not have a property, it is searched in the prototype chain, this is called __delegation__ until the property is reached otherwise the return value is `undefined`.

### 3.1 Keep track of the prototype
An instance uses the _internal property_ `[[Prototype]]` to keep track of its prototype, that is _the object it descends from_. Everything an object wants to share must be defined inside the prototype property.

#### Prototype property and constructors
The paragraph [function constructor](#function_constructor) has introduced a way to share common properties among objects created by the same constructor function. For instance:
```javascript
'use strict';

function Cat() {};

Cat.prototype.language = 'meow';

Cat.prototype.legs = 4;

Cat.prototype.toString = function() {
	return 'language: ' + this.language + ' and ' + this.legs + ' legs.';
};

var felix = new Cat();
felix.toString();
```
the function constructor `Cat` has been defined via _object literal_ notation and then its _prototype_ is augmented with some properties in order to not be repeated every time an object of type `Cat` is created. In the example the function `toString()` is overridden so, when invoked, the one from `Cat.prototype` will be used instead of the one from `Object.prototype`.

Typing `felix` in the console the referenced object is shown.
```javascript
felix
Cat
    __proto__: Cat
        constructor: function Cat() {}
        language: "meow"
        legs: 4
        toString: function () {
        __proto__: Object
```
The instance referenced by `felix` is of type `Cat` as expected, along with the properties _attached_ to `Cat.prototype`.

### References
[1]: https://developer.mozilla.org/en-US/docs/Web/JavaScript "Mozilla Developer Network"
[2]: http://www.manning.com/resig/ "Secrets of the JavaScript Ninja"
[3]: http://shop.oreilly.com/product/9780596517748.do "JavaScript: The Good Parts"
[4]: http://shop.oreilly.com/product/9781593275402.do "The Principles of Object-Oriented JavaScript"
