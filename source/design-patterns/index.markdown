---
layout: page
title: "Design Patterns"
date: 2013-03-13 22:45
comments: false
sharing: true
footer: true
---
##Design Patterns Categories
Design patterns can be divided in some categories according to what they achieve.

###Creational Patterns
They abstract the instantiation process, helping to make the system independent with respect to how the objects are created. 
	
The instantiation is _delegated_ to another object. The system will depends on _object composition_ rather than _class inheritance_. Two themes recur in the creational patterns [Gamma et all.]:

* they encapsulate knowledge about which concrete class the system uses;
* they hide how instances of these classes are created and put together.

What a system knows about an object is only is _interface_ so the _programming at the interfaces_ using creational patterns gives a lot of flexibility about _what_ is created, _who_ creates the object, _how_ and _when_ the instance is created.

The configuration of _what_ is created can be decided _dynamically_, at _run-time_ or _statically_, at _compile time_.

* __[Factory Pattern](/blog/2013/05/21/factory-pattern).__
* __[Singleton](/blog/2013/03/14/singleton-pattern).__
* Prototype
* Builder

###Structural Patterns
They are related to how classes and objects are composed to from large structures, using inheritance it is possible to compose interfaces or implementations.

* __[Adapter](/blog/2013/03/27/adapter-pattern).__ Converts one interface to another.
* __[Facade](/blog/2013/03/27/facade-pattern).__ Makes an interface simpler.
* __[Decorator Pattern](/blog/2013/03/12/decorator-pattern).__ Doesn't alter an interface, but adds responsibilities.
* __[Composite](/blog/2013/03/15/composite-pattern).__
* __[Proxy](/blog/2013/03/20/proxy-pattern).__
    * __[Protection Proxy](/blog/2013/03/22/protection-proxy-pattern).__
    * __[Remote Proxy](/blog/2013/03/22/remote-proxy-pattern).__
    * __[Virtual Proxy](/blog/2013/03/22/virtual-proxy-pattern).__
* Flyweight
* Bridge

###Behavioral Patterns
They are related to algorithms, assignment of responsibilities among objects and communication among them.

* __[Template Method](/blog/2013/03/26/template-pattern)__.
* __[Strategy](/blog/2013/03/25/strategy-pattern)__.
* __[Observer](/blog/2013/03/25/observer-pattern).__
* __[Iterator](/blog/2013/03/15/iterator-pattern).__
* __[Command](/blog/2013/03/28/command-pattern).__
* State
* Mediator
* Visitor
* Memento
* Interpreter

##Comparisons

##References

* E. Gamma, R. Helm, R. Johnson, J. Vlissides. Design Patterns: Elements of Reusable Object-Oriented Software. Addison-Wesley, 1998.
