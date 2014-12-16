---
layout: post
title: "Composite Pattern"
date: 2013-03-15 23:45
comments: false
categories: [design patterns, composite, java]
---
{% blockquote Bates and Sierra, http://shop.oreilly.com/product/9780596007126.do Head First Design Patterns %}
The Composite Pattern allows you to compose objects into tree structures to represent part-whole hierarchies. Composite lets client treat individual objects and composition of objects uniformly.
{% endblockquote %}
<!-- more -->

##Class Diagram
{% img /images/posts/design-patterns/composite.jpg %} 

`Client` uses the `Component` to manipulate objects.

`Component` defines an interface both for the composite and for the leaf which are element of the collection. It might implement a default behavior for methods.

`Leaf` has no children and inherits methods and override what make sense for the class itself.

`Composite` has children, the `Components`, which can be `Component` or `Leaf` type and inherits methods and override what make sense for the class itself.

##Key Points
* __New Approach.__ Not necessarily related to iterators.

* __Trees.__ Build object structures in the form of _trees_ containing both nodes and leaves.
	* Same operations are applied over both composite and leaves.
	* Ignore the differences between nodes and leaves.

* __Part-whole Hierarchy.__ Node with children and leaves are in the same structure.
	* Tree of objects made of parts, nodes and leaves, but threaten as a whole.
