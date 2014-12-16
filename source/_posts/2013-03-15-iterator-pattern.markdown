---
layout: post
title: "Iterator Pattern"
date: 2013-03-15 23:05
comments: false
categories: [design patterns, iterator, java]
---
{% blockquote Bates and Sierra, http://shop.oreilly.com/product/9780596007126.do Head First Design Patterns %}
The Iterator Pattern provides a way to access the elements of an aggregates object sequentially without exposing its underlying representation.
{% endblockquote %}
<!-- more -->

##Class Diagram
{% img /images/posts/design-patterns/iterator.jpg %} 

##Key Points
* __Encapsulate what varies.__ `Iterator`
	* knows how to iterate over a collection of item;
	* hides the collection implementation details;
	* depends on the collection and how to iterate over it.
* __Single Responsibility Principle.__
	* A class should have only one reason to change.
	* Every responsibility of a class is an area of potential change.
* __Cohesion.__
	* General concept related to _Single Responsibility Principle_.
	* Measure how closely a class or a module supports a single responsibility.
	* _High cohesion:_ the class is designed around a set of related functions and is more maintainable.
	* _Low cohesion:_ the class is design around a set of unrelated functions and is less maintainable.
* __Standard Interface.__ The `Iterator` is a well known interface and interact
	* in the same way with all the collections;
	* without knowing collection implementation details, decouples the client from the implementation details.
* __Other Interfaces.__ There are also
	* `ListIterator` allows to get also the previous item
	* `Enumarate` old interface
* __Polymorphic Iteration.__ The `Iterator` code iterate over any collection as long as it supports the interface.
* __for/in__ Statement to iterate over a collection without explicitly creating an iterator.