---
layout: post
title: "Decorator Pattern"
date: 2013-03-12 20:34
comments: false
categories: [java, design patterns]
---
{% blockquote Bates and Sierra, http://shop.oreilly.com/product/9780596007126.do Head First Design Patterns %}
The Decorator Pattern attaches additional responsibilities to an object dynamically.
Decorators provide a flexible alternative to subclassing for extending functionality.
{% endblockquote %}
<!-- more -->

##Class Diagram Explained
{% img /images/posts/design-patterns/decorator.png %}

`ConcreteComponent` object dynamically gets new behaviors, that is new methods are added to the component.

`Decorator` _HAS-A_ or _wraps_ a component having an instance variable referencing to a component.

`ConcreteDecorators` can extends the state of a component adding new properties, but also add new methods. A new method means a new behavior which is added by doing computation before or after an existing method in the component.

##Key Points
* __Open-Closed Principle.__ Classes should be open for extension but closed for modification.

* __Design.__ It should make possible to extend the _<u>behavior</u>_ without modifying the existing code. _<u>Composition</u>_ and _<u>delegation</u>_ allow to add new behaviors at runtime [Bates, Sierra].

* __Adding Behavior.__ _Subclassing_ is not the only solution and is not always flexible, decorator classes wrap, decorate, the concrete components as many times you want adding functionality dynamically.

* __Core.__ Decorators _<u>change the behavior</u>_ of their _<u>components</u>_ by adding new functionality before and/or after method calls to the component [Bates, Sierra].

* __Jargon.__
	* _Composition_ means HAS-A, wrapping, holding a reference variable.
	* _Behavior_ is synonym of method.
	* _Delegation_ is a chain of method calls.

##Scenario
Image to be the owner of a coffee shop. A customer can choose a _coffee_ among many kinds and then add to it some __condiments__. There are basic condiments and sets of condiments ready to transform a coffee into a mocha, cappuccino and so on.

* __Goal.__ A bill should be produced considering the coffee and all the extra ingredients added.

* __Adding Behavior.__ The customer start from a normal coffee and adding condiments (new functionalities) changes the _behavior_ of the coffee into something else such as mocha or cappuccino.

##Implementation
* Create a `Coffee` class and then different subclasses, one for each possible combination of condiments. What happens if a new condiment is added or removed from the list? if a recipe changes? Class explosion and it could be necessary to modified the code.

* Use _decorator pattern_ to dynamically add new behaviors.

{% codeblock lang:java %}
public abstract class Coffee {

    private String description = "Simple coffe";

    private double cost;

    public Coffee() {
    }

    public Coffee(final String coffeeDescription) {
        description = coffeeDescription;
    }

    public String getDescription() {
        return description;
    }

    public abstract double getCost();
}
{% endcodeblock %}

{% codeblock lang:java %}
public abstract class CondimentDecorator extends Coffee {

    public abstract String getDescription();
}
{% endcodeblock %}

{% codeblock lang:java %}
public class Espresso extends Coffee {

    public Espresso() {
        super("Espresso");
    }

    @Override
    public final double getCost() {
        return 0.95;
    }
}
{% endcodeblock %}

{% codeblock lang:java %}
public class MilkDecorator extends CondimentDecorator {

    public static final double MILK_COST = .50;

    private Coffee coffee;

    public MilkDecorator(final Coffee coffeeBeverage) {
        coffee = coffeeBeverage;
    }

    @Override
    public final String getDescription() {
        return coffee.getDescription() + ", Milk";
    }

    @Override
    public final double getCost() {
        return coffee.getCost() + MILK_COST;
    }
}
{% endcodeblock %}

{% codeblock lang:java %}
public class DecoratorTest {

    private Coffee coffee;

    private CondimentDecorator condiment;

    @Before
    public void setUp() {
        coffee = new Espresso();
        condiment = new MilkDecorator(coffee);
    }

    @Test
    public void testDescription() throws Exception {
        assertEquals(condiment.getDescription(), "Espresso, Milk");
    }

    @Test
    public void testCost() throws Exception {
        assertEquals(condiment.getCost(), 1.45, 0.0);
    }
}
{% endcodeblock %}
