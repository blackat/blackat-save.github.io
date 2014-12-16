---
layout: post
title: "Strategy Pattern"
date: 2013-03-25 20:21
comments: false
categories: [java, design patterns]
---
{% blockquote Bates and Sierra, http://shop.oreilly.com/product/9780596007126.do Head First Design Patterns %}
The Strategy Pattern defines a family of algorithms, encapsulates each one and makes them interchangeable. Strategy lets algorithm vary independently from clients that use it.
{% endblockquote %}
<!-- more -->

##Class diagram
{% img /images/posts/design-patterns/strategy_pattern.png %}

##Design Principles
Some design principle can help in the definition and explanation of the pattern.

* __Encapsulate what varies.__
	* Encapsulating what varies _don't affect_ other parts of the code when changes.
	* Because of _new requirements_ some _aspects_ of the application could change quite often. _Aspects_ mean _behaviors_ and they have to be separated from _aspects_ that don't change.
	* Base scheme for all design patterns, they allow some part of the system to _vary independently_ of all other parts.
* __Program to an interface, not an implementation.__
	* Decouple the code from a specific implementation.
	* Allow to change the implementation at runtime.
* __Favor composition over inheritance.__
	* Better to get behaviors by _composition_ than by _inheritance_.
	* Composition gives more _flexibility_.
	* Encapsulate family of _algorithms or behaviors_ into their own _set of classes_.
	* Allow to change behavior at runtime.

##Scenario
A product could update quite frequently because of new requirements introducing new behavior. Considering a simple class inheritance is the first solution that comes to mind.

A class defines some variables to define the state of an instance and methods to implement behaviors. Focus on behaviors.

* Subclassing allows to inherit _behaviors_ which could change _across subclasses_.
* Overriding _behaviors_ must be done subclass by subclass.
* Subclass may inherit useless behaviors.

	So how to improve _code reuse_, avoid _code duplication_ and make simple the design?

##Design Principles in Action
To better design the application apply the design principles explained so far.

* __Encapsulate what varies.__
	* Create one or more _set of classes_ to encapsulate the _implementation_ of their respective _behaviors_.
	* Behaviors encapsulated into classes can be _reused_.
	* Encapsulation means put a given implementation in a separated class.
	* Separation and encapsulation allow _composition_, so _behaviors are assigned to instances_.
* __Program to an interface.__
	* Implementation can be defined at runtime, no more specific implementation.
	* Interfaces are a set of behaviors, some classes exist _only to implement a behavior_.
	* __Inheritance.__ Synonym of _concrete implementation_. An _inherited behavior_ means concrete implementation from the superclass or the subclass providing a specialized implementation of the behavior.
	* __Interface.__ Synonym of a _set of behaviors_.
		* __Program to an abstract supertype.__ Program to an interface or abstract class.
		* __Polymorphism.__ The _runtime object_ or _actual implementation of the behavior_ is not locked into the client class.
* __Favorite composition.__
	* __HAS-A relation.__
		* It means _delegation_, a class _delegate its behavior_ to other classes instead of defining them by itself, IS-A.
		* It is better than IS-A because it avoids to lock the client to a specific implementation and change behaviors at runtime using interfaces.
		* Behaviors can be _changed_ without affecting the clients, new behaviors can be added without touching the existing ones.
		* Behaviors can be _reused_ by multiple clients.

##Set Behavior Implementation
The behavior implementation can be set via

* __constructor:__ once the class is created the behavior class is set,
* __setter method:__ call the setter any time to change the class behavior on the fly,
* __mixed:__ use the constructor to set a default behavior and the setter to change it at runtime,
* __subclass:__ can define a new constructor.

_Set of behaviors_ can be seen as a __family of algorithms__.

##Example
{% img /images/posts/design-patterns/behavior_interface.png %}

Composition is realized by defining an interface type variable and then delegating to different set of classes the implementation of the behaviors.

{% img /images/posts/design-patterns/person_class.png %}

Behaviors defined by composition are implemented by external classes which exist only for this purpose, so they can change, extended and added without affecting `Person` class.
