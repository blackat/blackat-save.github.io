---
layout: post
title: "Template Pattern"
date: 2013-03-26 01:00
comments: false
categories: [java, design patterns]
---
{% blockquote Bates and Sierra, http://shop.oreilly.com/product/9780596007126.do Head First Design Patterns %}
The Template Method Pattern defines the skeleton of an algorithm in an operation, deferring some steps to subclasses. Template Method lets subclasses redefine certain steps of an algorithm without changing the algorithm's structure.
{% endblockquote %}
<!-- more -->

##Class diagram
{% img /images/posts/design-patterns/template_method.png %}

The abstract class contains the _template method_ and _abstract_ versions of the operations used in the template method. Each method, abstract or concrete, is a step of an algorithm which could varies.

The template methods use the `primitiveOperations` to implement an algorithm _decoupling_ from the actual implementation of these operations.

The concrete class implements all the abstract operations called by the template method.

##Design Principle
[The Hollywood Principle.](/oo-design-principles/index.html#hollywood_principle)

##Scenario
An algorithm is made of steps useful to accomplish some tasks. It can be imagined as a recipe which is a set of instruction to prepare a dish. Some recipes could have some instructions in common so it should be better to avoid _code duplication_.

##Key Points
* __Template method.__ It defines the skeleton of an algorithm _deferring_ some steps to subclasses. It lets subclasses _redefine_ some steps without changing the algorithm's structure.
* __Abstract class.___
	* It is a template of methods for an algorithm which could implemented in slightly different ways in some of the steps.
	* It is made to be extended and abstract methods to be implemented. So the abstract class collect all the all the _common_ methods or instructions different algorithms.
	* It reduces the dependencies in the system.
* __Abstract methods.__ They point out that they are just _placeholders_ because they are in common with all the algorithms but their _implementation differ_ from algorithm to algorithm.
* __Concrete methods.__ Concrete meaning that their _implementation is the same and in common_ among algorithms.
* __Inheritance.__ It allows all the subclasses, the _algorithms_, to have the _same behaviors_ of the superclass, if they are _abstract_ they could change across subclasses.
* __Interfaces.__ They don't have code so _no code reuse_.
* __Concrete class.__ Concrete implementation is a working algorithm which implements each _abstract method_ and _could add some other algorithm specific methods_.
* __Hook methods.__
	* They are concrete methods _doing nothing by default_, they are _optional steps of the algorithm_ and the subclasses are not obliged to override them. The subclass can hook its own code into the algorithm, _an optional part of the algorithm_.
	* They could also be used to _conditionally control_, using conditional statements, the flow of the algorithm in the abstract class.

##Template method vs. Strategy

* __Focus on__
	* Strategy and Template both encapsulate algorithms, one by _inheritance_ and one by _composition_.
* __Template__
	* Define the _outline of an algorithm_ and let my _subclasses_ do some of the work.
	* _Keep the control_ over the algorithm's structure and allow be different implementations of individual steps.
	* Provide method for _code reuse_ allowing _subclasses_ to specify behavior.
	* Depend on method implemented in the superclass.
* __Strategy__
	* Define a _family of algorithms_ and make them _interchangeable_.
	* Each algorithm is encapsulated so the client can use different algorithms easily.
	* _Not use inheritance_ for algorithm implementations.
	* Clients use algorithm implementation through _object composition_.
	* Clients can change algorithm at _runtime_ by using different _strategy object_.
	* Not depend on any superclass.

##Template Method in Action
