---
layout: post
title: "Facade Pattern"
date: 2013-03-27 08:02
comments: false
categories: [java, design pattern]
---
{% blockquote Bates and Sierra, http://shop.oreilly.com/product/9780596007126.do Head First Design Patterns %}
The Facade Pattern provides a unified interface to a set of interfaces in a subsystem. Facade defines a higher-level interface that makes the subsystem easier to use.
{% endblockquote %}
<!-- more -->

##Class diagram
{% img /images/posts/design-patterns/facade_pattern.png %}

The `Facade` is a _unified interface_ easily usable by the client which doesn't dialog anymore with the complex subsytem.

##Design Principle
Principle of _Least Knowledge_, talk only to you immediate friends.

* __Loosely coupled system.__ Prevent from creating designs that have a large number of classes coupled together so that changes in one part of the system cascade on the other parts.
* __Fragile system.__ It happens when there are a lot of dependencies between many classes.

##Key Points

* __Only one friend.__ The _client_ has only one friend, the facade.
* __Decoupling.__ Allow to decouple client implementation from any subsystem. Coding to the facade, rather than to the subystem, allows the client code to not change every time the subsystem changes, just the facade has to update.
* __Delegation.__ Implementing the facade requires to compose the facade with its subsystem and use _delegation_ to perform the work.
* __Subsytem update.__ The update doesn't affect the client.
* __Additional facade.__ If the subsystem gets too complex _additional facade_ could be introduced to _form layers of subsystems_.
* __Not encapsulate.__ The Facade doesn't encapsulate the classes but provides a simpler interface to the client. The subsystem classes can be still used by the client to achieve some low levels functionality.
* __Many facades.__ Given a subsystem many facades can be created.

##Comparison

* __Wrap multiple classes.__ Both can wrap multiple classes.
* __Facade__
	* Simplify an interface.
	* Decouple a client from a subsystem of components
	* Subsystem classes are still available to the client for low level functionality.
* __Adapter__
	* Convert an interface into something different, something the client is expecting.
	* Encapsulate the subsystem in order to hide it to the client which will use just the adapter interface.
