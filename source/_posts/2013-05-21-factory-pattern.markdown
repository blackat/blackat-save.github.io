---
layout: post
title: "Factory Pattern"
date: 2013-05-21 22:17
comments: false
categories: [design pattern, factory pattern]
---
###Introduction
* An abstract factory method isolates the client from knowing what class is actually created.
* __New__ operator means _directly instantiating_ an object so _direct dependency on a concrete class_.
    * Code is not close to modification.
* __Interface and abstract class__ imply decoupling code from the actual object. If code is written to an interface it will work with any new class implementing that interface through polymorphism.
* __Design principle.__ [Dependency Inversion Principle.](/oo-design-principles/index.html#inversion_principle)

<!-- more -->

###In a Nutshell
* __All factory pattern.__
    * Encapsulate object creation.
    * Promote _coding to abstraction_ reducing dependency on concrete classes and improving loose coupling.
    * Client depends on _interfaces_ removing direct object instantiation.
    * Hide object construction details from the client.
* __Simple factory.__
    * Not a proper design pattern, but more a programming idiom.
    * A simple way to _decouple_ a client from concrete classes.
    * Usually a factory class provides a static method to construct an object.
* __Factory method.__
    * Is _abstract and protected_ so defers the instantiation to its subclasses.
    * __Relies on inheritance__ that is object creation is delegated to the subclasses which implement the _abstract factory method_ able to create objects.
    * __Design principle.__ [Dependency Inversion Principle.](/oo-design-principles/index.html#inversion_principle)
* __Abstract factory.__
    * Class whose interface is mostly made of abstract methods.
    * Creates a family of related objects without depending on _concrete classes_.
    * __relies on object composition__ that is object creation is implemented in methods exposed in the abstract factory class interface.

##1. Simple Factory
* __Definition.__ It is not a proper design pattern, but more a programming idiom.

* __Class diagram.__

* __Simple factory in action.__
    * The factory class is responsible to create different _concrete products_ hiding construction details from the client.
    * Construction details are in a centralized place, easy to maintain.
    * The client is aware only about the _product interface_ and not anymore tightly coupled to its concrete instantiation.
    * The factory class _encapsulate object creation,_ not flexible and the creation of new products oblige to modify the class. A class should be closed to modification but open for extensions.

###Example
Suppose a car manufacturer which has to build different models of cars but should not be aware of all the production details for each model. Better to use a factory that knows all the details for each model.
{% codeblock lang:java %}
public class CarManufacturer {

    private CarFactory carFactory;

    public CarManufacturer(CarFactory factory) {
        this.carFactory = factory;
    }

    public AbstractCar buildCar(String model) {
        AbstractCar car = this.carFactory.createCarInstance(model);

        Validate.notNull(car, "Model " + model + " is not available for the build.");

        car.assemble();
        car.paint();
        car.mountWheels();
        car.test();

        return car;
    }
}
{% endcodeblock %}

The factory knows, according to the model, which _concrete class_ should be instantiated and returned. `CarManufacturer` HAS-A factory instance and the `buildCar` method returns an `abstract` car, so that programming to interfaces allow to return multiple object types.
{% codeblock lang:java %}
public class CarFactory {

    public static final String A_CLASS = "a-class";
    public static final String B_CLASS = "b-class";

    public AbstractCar createCarInstance(String model) {
        if (model.equalsIgnoreCase(A_CLASS)) {
            return new ClassA();
        } else if (model.equalsIgnoreCase(B_CLASS)) {
            return new ClassB();
        }
        return null;
    }
}
{% endcodeblock %}

Finally the test class shows how

* implementation details of each model are hidden to the client,
* different factory instances could be passed to the car manufacturer at runtime,
* client is not aware of the concrete object type returned by the factory, but just knows the interface implemented.
{% codeblock lang:java %}
public class CarManufacturerTest {

    private CarManufacturer carManufacturer;

    @Before
    public void setUp() {
        carManufacturer = new CarManufacturer(new CarFactory());
    }

    @Test
    public void testBuildCarExistingModelClassA() throws Exception {
        AbstractCar car = carManufacturer.buildCar("a-class");
        assertCarBuilt(car);
    }

    @Test
    public void testBuildCarExistingModelClassB() throws Exception {
        AbstractCar car = carManufacturer.buildCar("b-class");
        assertCarBuilt(car);
    }

    private void assertCarBuilt(AbstractCar car) throws Exception {
        assertTrue(car.isAssembled());
        assertTrue(car.isPainted());
        assertTrue(car.isWheelsMounted());
        assertTrue(car.isTested());
    }

    @Rule
    public ExpectedException thrown = ExpectedException.none();

    @Test
    public void testBuildCarNotExistingModel() throws Exception {
        thrown.expect(IllegalArgumentException.class);
        thrown.expectMessage("Model c-class is not available for the build.");
        AbstractCar car = carManufacturer.buildCar("c-class");
    }
}
{% endcodeblock %}


##2. Factory Method
* __Definition.__ Defines a _method interface_ to create an object and lets subclasses decide which class should be instantiated. Thus the instantiation of the concrete class is deferred to subclasses.

* __Design principle.__ The dependency inversion principle.

* __Class diagram.__

{% img /images/posts/design-patterns/factory_method.png %}

* __Factory method in action.__
    * With respect to the simple factory, the factory method gets rid of the external factory class and localize the making activity in the client class which is then abstract.
    * The creation activity is concentrated into a method which acts as a _factory._
    * Parallel class hierarchies, both the creator and the product hierarchy start from an abstract class.
        * Each class extending the abstract one, the client, is called __creator class__ and many of them could be defined.
        * The product class is abstract as well, so many product classes could be available.
    * Subclasses decide which implementation will be used, so more flexibility and products can vary.

###Example
Get rid of the external factory and defines _within the client class_ an _abstract method_ which acts as a factory. The advantage of this design is infinite extension of the class to add new models without changing the factory.
{% codeblock lang:java %}
public abstract class AbstractCarManufacturer {

    public AbstractCarManufacturer() { }

    public final AbstractCar buildCar(final String model) {
        AbstractCar car = createCar(model);

        Validate.notNull(car, "Model " + model + " is not available for the build.");

        car.assemble();
        car.paint();
        car.mountWheels();
        car.test();

        return car;
    }

    protected abstract AbstractCar createCar(final String model);
}
{% endcodeblock %}

Now the concrete client class implements the factory method and many clients can be defined to create different models.
{% codeblock lang:java %}
public class FamilyCarManufacturer extends AbstractCarManufacturer {

    public static final String C_CLASS = "c-class";
    public static final String E_CLASS = "e-class";

    @Override
    public AbstractCar createCar(final String model) {
        if (model.equalsIgnoreCase(C_CLASS)) {
            return new ClassC();
        } else if (model.equalsIgnoreCase(E_CLASS)) {
            return new ClassE();
        }
        return null;
    }
}
{% endcodeblock %}

Another manufacturer able to created different kinds of models.
{% codeblock lang:java %}
public class SportCarManufacturer extends AbstractCarManufacturer {

    public static final String SLK_CLASS = "slk-class";
    public static final String SLR_CLASS = "slr-class";

    @Override
    public AbstractCar createCar(final String model) {
        if (model.equalsIgnoreCase(SLK_CLASS)) {
            return new ClassA();
        } else if (model.equalsIgnoreCase(SLR_CLASS)) {
            return new ClassB();
        }
        return null;
    }
}
{% endcodeblock %}

##3. Abstract Factory Method
* __Definition.__ Provide and interface for creating families of related or dependent objects without specifying their concrete classes.
* __Class diagram.__

{% img /images/posts/design-patterns/abstract_factory.png %}
* __Class diagram explained.__
    * The `Client` defines an `AbstractFactory` variable and the actual factory will be resolved at runtime.
    * The `AbstractFactory` defines an _interface_.
    * Each `ConcreteFactory` represents a _family of products_ and implements the method defined in the abstract interface. `ConcreteFactory2` for instance produces the set of products represented by `ProductA2` and `ProductB2`.
    * The _product family_ is represented by the two interfaces `AbstractProductA` and `AbstractProductB`.
* __Factory method in action.__
    * The job of an Abstract Factory is to define an interface for creating a set of products through a set of methods.
    * Each product method in an interface is responsible to create a product.
    * Subclasses of the Abstract Factory provides the implementations.
    * Factory Methods are a natural way to implement the product methods in the abstract factories.
* __Comparison.__
    * __Abstract factory__ creates a _family or set of products_.
    * __Method factory__ creates a _single product_.


