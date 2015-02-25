### Object prototype
{% blockquote Douglas Crockford http://shop.oreilly.com/product/9780596517748.do JavaScript: The Good Parts %}
Every object is linked to a prototype object from which it can inherit properties.
{% endblockquote %}

When a new object is created is automatically linked to `Object.prototype` if otherwise specified. For instance the object `cat` is created from object literal there is no specification of a different prototype object.

Each object then has a prototype link following which it is possible to climb the object hierarchy. It is the same process when a property value is retrieved: if the current object does not have the property defined, the _property chain_ is used to climb the hierarchy and look for the property in the parent object. If the parent object lacks the property its parent is checked. Through the _delegation_ all the objects in the property chain are checked, the latest one is `Object.prototype`. If the latest one does not have the property defined, the result will be the `undefined` value.

The prototype chain is a _dynamic relationship_, when a _new property_ is added to a _prototype_, all the objects based on that prototype inherits that property.


The hierarchy concept resides in the `prototype` property which can be exploited to build a new instance.


When a function is called via the `new` operator, its context is the one of the new object instance so it is possible to define and initialize properties via `this` parameter attaching them to the `prototype`.

### Function Constructor
A function constructor is used to create multiple similar objects having same properties and methods.

{% blockquote Nocholas C. Zakas http://shop.oreilly.com/product/9781593275402.do JavaScript: The Good Parts %}
Every object is linked to a prototype object from which it can inherit properties.
{% endblockquote %}

By convention a function constructor name begins with a capital letter in order to distinguish a constructor from a normal function otherwise no syntactic difference could be spotted.

```javascript
'use strict';
function Cat() {
    // function constructor code
}

function cat() {
    // normal function code
}
```

The first function will be called with the `new` operator, the second one as a normal function. A constructor function is invoked as follows

```javascript
'use strict';
function Cat() {
    // function constructor code
}

var cat = new Cat();
console.log(cat instanceof Cat); // true
```
There is not an explicit return statement because the `new` operator creates an instance of type `Cat` and returns the reference to that object. In the JavaScript object world one of the most important word is __property__. Properties are used to store variable values and methods, moreover prototypal inheritance is based on special properties.

An instance of type `Cat` has been created and this can be checked, as written in the example, using the `instanceof` operator or, alternatively using one of the aforementioned special properties: `constructor`.

When an object instance is created, automatically gets a `constructor` property which contains a reference to the function constructor used to created it. The most accurate way to check an object type remains the `instanceof` operator.

    »function Cat(){};
    «undefined

    »var cat = new Cat();
    «undefined

    »cat instanceof Cat
    «true

    »cat.constructor
    «function Cat(){}

    »cat.constructor === Cat
    «true
