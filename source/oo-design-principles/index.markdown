---
layout: page
title: "Object Oriented Principles"
date: 2013-05-27 21:14
comments: true
sharing: true
footer: true
---
##<a id="encapsulates_what_varies_principle">Encapsulates What Varies</a>
{% blockquote %}
Encapsulates what varies so it won't affect the rest of the code if it changes
{% endblockquote %}
* __Design Pattern.__ Strategy pattern.
* __Meaning.__ If there are some aspects of the code that could change with every new requirements then it is a behavior that need to be pulled out and separated from stuff that doesn't change.
* __Key points.__
    * Identify the aspects of the application that vary and separate them from what stays the same.
    * Take parts that vary and encapsulate them, so that later you can later or extend the parts that vary without affecting those that don't.
    * _Base of all design patterns._ All patterns provides a way to let some part of a system vary independently of all other parts.

##<a id="program_to_interface_principle">Program to an Interface not to an Implementation</a>
{% blockquote %}
Encapsulates what varies so it won't affect the rest of the code if it changes
{% endblockquote %}
* __Design Pattern.__ Strategy pattern.
* __Meaning.__ Program to an interface means _program to a supertype_ usually an abstract class or an interface so that the class declaring them doesn't have to know about the actual object types.
* __Key points.__
    * A _behavior_, for instance, can be represented by an _interface_ so that the _actual implementation_ of the behavior wont be locked by a class or subclass.
    * _Behavior are no longer hidden into a class_ but can _be reused_ and new behaviors can _be added_ without modifying any of the existing classes.
    * Behavior are no longer hidden into a class but can be reused and new behaviors can be added without modifying any of the existing classes.

##<a id="composition_over_inheritance_principle">Favor Composition Over Inheritance</a>
{% blockquote %}
Getting behavior by being composed is better than inherit them.
{% endblockquote %}
* __Design Pattern.__ Strategy pattern.
* __Key points.__
    * Composition gives more flexibility.
    * Allow to encapsulate family of algorithms into their own set of classes.
    * Allow to change behavior at runtime.



##<a id="open_close_principle">The Open-Closed Principle</a>
{% blockquote %}
Classes should be opened for extension but closed for modification.
{% endblockquote %}
* __Design Pattern.__ Decorator pattern.
* __Meaning.__ It allows code to be extended without direct modification.
* __Key points.__
    * introduce new levels of abstraction which adds complexity to the code
    * concentrate on those area are likely to change in your design
    * do not apply the open-close principle everywhere, it is unnecessary, leads to complexity and hard to understand code

##<a id="loosely_coupled_principle">Loosely Coupled Design Principle</a>
{% blockquote %}
Strive for loosely coupled designs between objects that interact.
{% endblockquote %}
* __Design Pattern.__ Obsever pattern.
* __Meaning.__ Loosely coupled designs allow to build flexible systems minimizing the interdependency between objects.
* __Loosely coupling power.__
    * An observer has to implement an interface, can be added or removed at any time.
    * The `Subject` just deliver notifications to any objects implementing the `Observer` interface.
    * Any object can get notifications, it has just to implement the `Observer` interface.
    * A `Subject` and an `Observer` can change independenlty as long as they respect the respective interfaces.

##<a id="inversion_principle">The Dependency Inversion Principle</a>
{% blockquote %}
Depend upon abstractions, do not depend upon concrete classes.
{% endblockquote %}
* __Design Pattern.__ Factory Method pattern is one of the possible way to respect this principle.
* __Meaning.__
    * Start from the top, high-level components and go down to the bottom, the concrete classes.
    * Look at how classes are linked and the direction of the arrows when the code depends on concrete implementation using _new_ and compare with arrows when factory method is involved.
    * The factory method returns a concrete type implementing a given interface and the client is just waiting for something respecting that interface without considering a specific type.
    * The _return_ in the factory method reverts the direction of the creation arrows.
    * Now both _low-level_ (concrete cars) and _high-level_ (car manufacturers) components depend on the abstraction (the abstract car).
    * _Invertion._ Start at Pizzas level and see what you can abstract and then go back to PizzaStore.
* __Key points.__
    * Similar to "Program to an interface, not an implementation" but stronger about abstraction.
    * High-level and low-level components depend only on the abstraction, that is Pizza.
    * PizzaStore is the high level component and depends on concrete pizza classes.
    * Pizzas are the low-level components.
* __Guideline to follow the principle.__
    * _No variable should hold a reference to a concrete class._ Do not use _new_ but a factory to get around that.
    * _No class should derive from a concrete class._ Deriving from a concrete class implies a dependency on that concrete class.
        * Derive instead from _abstraction_ or _interface_.
    * _No method should override an implemented method of any of its base classes._ Overriding a method means that the base class was not really an abstraction.
        * Methods in the base class are meant to be _shared_ among subclasses.

##<a id="hollywood_principle">The Hollywood Principle</a>
* __Definition.__ Don't call us, we'll call you.
* __Design Pattern.__ Template Method pattern.
* __Meaning.__ This principle allows low-level components to hook themselves into a system, but the high-level components determine when they are needed and how. The high-level components give the low-level components a "don't call us, se'll call you" treatment.
    * The abstract class is the _high-level_ component, it has the control over the algorithm.
	    * Abstract class calls the subclasses only when an implementation of a method is needed.
	    * Client will depend on the class abstraction rather than on concrete class reducing the dependencies in the system.
    * Subclasses are the _low-level_ components used to _provide implementation details_.
    * A low-level component _never calls directly_ a high-level component, it can be hooked into the computation without creating dependencies between low-level and high-level layers.
    * High-level components reserve the low-level components a _"don't call us, we'll call you"_ treatment.
* __Key points.__
    * Technique for creating designs that allow low-level structure to interoperate while preventing other classes from becoming too dependent on them
    * Avoid explicit circular dependencies between the low-level components and the high-level ones
* __Advantages.__ Decoupling, lower-level components can be hooked into the computation without creating dependencies between the lower-level components and the higher-level layers.

##Principle of Least Knowledge (aka Law of Demeter)
* __Definition.__ Talk only to your immediate friends.
* __Design Pattern.__
* __Meaning.__ It reduces the interaction between objects to just a few close friends.
* __Key points.__ Invoke only methods that belong to
    * the object itself
    * objects passed in as a parameter to the method
    * any object the method creates or instantiates
    * any components of the object (HAS-A relationship)
* __Advantages.__
    * For any object be careful of the number of classes it interacts with and also how it interacts.
    * Prevent the creation of a large number of classes coupled together (changes in one part cascade to the others).
    * Many dependencies between many classes the system will be fragile:
        * difficult to maintain
        * complex for others to understand.
* __Disadvantages.__
    * More wrapper classes being written to handle method calls to other components:
        * increase complexity, development time and runtime performance





