---
layout: post
title: "Adapter Pattern"
date: 2013-03-27 08:01
comments: false
categories: [java, design pattern]
---
{% blockquote Bates and Sierra, http://shop.oreilly.com/product/9780596007126.do Head First Design Patterns %}
The Adapter Method Pattern converts the interface of a class into another interface the clients expect. Adapter lets classes work together that couldn't otherwise because of incompatible interfaces.
{% endblockquote %}
<!-- more -->

##Class diagram
{% img /images/posts/design-patterns/adapter_pattern.png %}

* Client can see only the `Target` interface.
* `Adapter` implements the `Target` interface and is composed with the `Adaptee` to which translates or delegates all the requests.
* `Adaptee` gets all the requests delegated by the `Adapter`.

##Scenario

##Key Points

* __Object composition.__ The pattern _wraps_ the _adaptee_ with an _altered interface_ and it can use any _subclass_ of the adaptee.
* __Bind to an interface.__ The pattern binds the client to an interface and not to an implementation.
* __Adapter.__
	* __Decouple__ the client from the implemented interface.
	* __Encapsulate what changes__ so the client doesn't have to modify each time needs to operate against a different interface.
	* __Convert__ one interface to another so the _adapter_ could wrap one or more _adaptee_ but it would be a bit messy (see Facade Pattern).
	* __Implement__ the interface of the type the client is expecting.
	* __Delegate__ all the requests to the object it wraps.
	* __Two ways Adapter.__ A client could expect old and new interfaces, so the adapter _implement both interfaces_ to support the client.

##Adapter vs. Decorator

* __Focus on__
	* _Adapter_ convert the interface of what it wrap, _decorator_ not change the interface.
* __Decorator__
	* Wrapped many other adapters.
	* Add new behaviors.
* __Adapter__
	* Allow clients to use other libraries without changing any code.

##Real World Adapters
* Enumerator. Allow iterate over a collection elements without knowing the collection implementation details.
* Iterator. Like the enumerator but with also the remove method.

`Enumerator` is the old fashion interface to iterate the collection elements so an adapter could _adapt_ the old fashion to the new one.

* __Target interface.__ `Iterator`
* __Adaptee interface.__ `Enumeration`
* The `Adapter` has to implement the `Target` and to compose with the `Adaptee`.
