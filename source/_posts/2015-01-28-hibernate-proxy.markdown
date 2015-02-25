---
layout: post
title: "Hibernate proxy"
date: 2015-01-28 18:26:28 +0100
comments: true
categories:
published: false
---
Sometimes certain portions of an entity are rarely accessed so retrieving performance can be optimized fetching _only_ the data frequently accessed. The defer loading of the state of an attribute happens only when it is referenced. Some attribute states are loaded and some others are deferred.

The Hibernate persistent provider uses a _proxy_ object to manage deferred loading.

<!-- more -->

### The domain model
JPA annotations allow to map a _domain model_ to a _data model_ specifying even the fetch policy to be adopted for each attribute.

``` java
@Entity
@Table(name = "HOUSE")
public class House implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "HOUSE_ID")
    private long id;

    @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    @JoinColumn(name = "ANIMAL_ID")
    private Animal animal;

    public Animal getAnimal() {
        return animal;
    }

    public void setAnimal(Animal animal) {
        this.animal = animal;
    }
}
```

_Enumerated type_ `FetchType` allows to specify the value for the `fetch` element: `LAZY` or `EAGER`, the default value. For the attribute `animal` has been set the _fetch type_ to LAZY instructing the persistence provider to defer the load of the status only when the attribute will be referenced.

It is interesting to see how the persistent provider, Hibernate in this case, manages the lazy loading.

###### Tuplizer and EntityMode
A piece of data or instance can be represented by a simple structure or by a complex one. For instance it can be a _POJO_, a _MAP_ but even by a _DOM_ structure. The `org.hibernate.EntityMode` defines the representation mode for an entity as `POJO` or `MAP`. The _DOM4J_ entity-mode was an experimental feature removed in Hibernate 4.

`org.hibernate.tuple.Tuplizer` is an interface defining a contract to manage a particular representation of a piece of data. The representation is defined by the `EntityMode`. For instance a piece of data can be represented as a _POJO_ so its representation is `EntityMode.POJO`. A tuplizer for such piece of data will be able to create an instance calling the POJO's constructor, extract and inject value using getters/setters or through direct access to fields.

Hibernate already provides some implementation of the `Tuplizer` interface such as the `org.hibernate.tuple.entity.PojoEntityTuplizer` specific to the pojo entity-mode.

Why a custom tuplizer?
- Use a different implementation of a given interface such as `java.util.Map`.
- Specify a different proxy generation strategy wrt the one used by deafult.
The custom tuplizer will be then attached to the entity it is meant to manage.

###### Proxy Object
When Hibernate initializes a field annotated with `Fetch.LAZY` creates a _reference_ to the data object without loading the real data. Hibernate will intrecept all the method call to this reference and loads real data from the data source.

This reference is called _Proxy Object_. The creation of the reference and its initialization happens through two components: `org.hibernate.proxy.LazyInitializer` and `org.hibernate.tuple.Tuplizer`.

###### Lazy Initializer
`org.hibernate.proxy.LazyInitializer` handles fetching of the underlying entity for a proxy. Hibernate provides two abstract implementation to lazy load data from inside a Hibernate session:
- `AbstractLazyInitializer`
- `BasicLazyInitializer`

###### Create
- CustomLazyInitializer
- CustomTuplizer
- CustomProxyFactory
- CustomHibernateProxy object
- Custom Proxy Replacement object


###### Step 1
Extend `BasicLazyInitializer` to create a custom initiliazer component to fetch data.

###### Step 2
Extend `PojoEntityTuplizer`

###### Step 3
Build a proxy factory to create a proxy object representing the _entity_ to be lazy loaded. The proxy object has to be of type `HibernateProxy`

##### Hibernate loading
When Hibernate initilizes the application configuration it creates the `CustomTuplizer` and the LazyProxyFactory for the entity to be lazy loaded.

###### Step A - Goal: build the proxy factory for the entity
Create `CustomTuplizer` class extending `PojoEntityTuplizer` and overriding `buildProxyFactory()` method to create and return a custom `ProxyFactory` implementation.

###### Step B - Goal: build the LazyProxyFactory
The `getProxy()` method is dynamically called by Hibernate to get the proxy representation of the _entity_ to be loaded. The _proxy object_ will represent the _entity_.

_Javassist_ is used to create the proxy class, then a Javassist based Class Object's instance will be returned as a _proxy object_.

###### Step C - Goal: build CustomLazyInitializer
Return a replacement object via its super implementation. This call will be added as callback handler. The callback handler is able to return a proxy replacement object shared with the remote client.

Add `MethodHandler` interface to have the `invoke()` method.

###### Step D - Goal: add a callback handler
Then an interceptor has to be created to intercept the calls to this _proxy object_.

When a proxy object methods are invoked, the method calls are intercepted and the invocation is passed to the `invoke()` method.



## Lazy Loading
Lazy loading improves the performance when data is fetched from a data source reducing the memory footprint. Hibernate, the persistent manager, loads the data from the data source, initializes the data object but, for the fields annotated as `FetchType.LAZY`, it creates a _reference object_ without loading any data from the data source as shown in the picture below.
{% img center /images/posts/hibernate-proxy/object-reference.png %}

Hibernate doesn't load the object until a method call requiring the reference happens, then it _intercepts_ the call and load data from the data source. To intercept the method call it is mandatory that the object belongs to an Hibernate session.

Hibernate uses `LazyInitializer` and `Tuplizer` to create and initialize reference objects called _proxy objects_.

### Proxy pattern
The __Proxy__ is a structural pattern useful to save up the memory used as to control the access to an object. It has been defined as follows

{% blockquote [Gang of Four] %}
Allows for object level access control by acting as a pass through entity or a placeholder object.
{% endblockquote %}

{% img center /images/posts/hibernate-proxy/proxy-pattern.png %}

The `Caller` client may not have directly access to the `RealImage` so it has to pass through the `ProxyImage` which will handle the creation of the real object delegating to it any method call from the client.

#### When use proxy pattern
- Control the access to the original object.
- Create an object on demand.
- The real object comes from an external source.

### Hibernate, Javassist and Entities
Javassit as Cglib are frameworks offering proxy implementation bytecode. Proxy pattern offers access control forwarding some method calls and intercepting others. Both Javassits and Cglib offers proxy implementation as JDK does. JDK supports only interfaces and no subclassing.

The client messages the proxy which, in turn, delegates the method call to a real object.

#### Hibernate components

- `LazyInitializer` hibernate component which helps to generate proxy objects for entities and used to fetch the underlying object of these proxies.
    - `getImplementation()` the method fetch the entity and returns a persistent object;
    - `getSerializableObject` returns a serializable proxy replacement object for the _entity_.
- `AbstractLazyInitializer` and `BasicLazyInitializer` _abstract_ implementations provided by Hibernate to lazy load or fetch data from inside an _Hibernate session_.
- `BasicLazyInitializer` will be extended to create a custom initializer component to fetch data from the database.

Hibernate provides implementations for _Tuplizer_ components which can be entities or mapped component, respectively `EntityTuplizer` and `ComponentTuplizer`. Tuplizers requires the creation of a `ProxyFactory` to generate proxy objects for entities.

- `POJOEntityTuplizer` POJO based representation of the domain model in memory.

Todo: extend `POJOEntityTuplizer` and build proxy factory able to create a proxy object of type `HibernateProxy`, as expected by Hibernate, representing the entity to be lazy initialized.

``` java
    package org.hibernate.proxy;

    import java.io.Serializable;
    import org.hibernate.proxy.LazyInitializer;

    public interface HibernateProxy extends Serializable {
        Object writeReplace();

        LazyInitializer getHibernateLazyInitializer();
    }
```

The object returned is the lazy entity reference, is the proxy.

Create
- CustomLazyInitializer
- CustomTuplizer extends POJOEntityTuplizer used to create a proxy factory
- CustomProxyFactory
- CustomHibernateProxy
- CustomProxyReplacement

###### Step 1
When Hibernate starts, initializes the configuration for the application and creates `CustomTuplizer` and build `LazyProxyFactory` for the _entity_.

###### Step 2
Override the method `buildProxyFactory` of the `CustomTuplizer` class to return the `CustomProxyFactory` implementation.

### The scaffolding

### The test suite
A test suite can be useful to investigate how a persistent provider manages lazy initialization. The first test just retrieves all the `House` objects stored in the database then check for the only existing one the type of the `animal` reference.

``` java
    @Test
    public void testInstanceOf() throws Exception {

        String query = "select h from House h";
        List<House> houses = em.createQuery(query).getResultList();
        assertNotNull(houses);
        assertEquals(1, houses.size());

        House house = houses.get(0);
        assertNotNull(house);

        assertTrue(house.animal instanceof Animal);

        assertFalse(house.animal instanceof Cat);
    }
```
The first assertion is quite obvious, the type of the attribute is `Animal`, so any instance of a class extending `Animal` can be assigned. An instance of `Cat` is stored in the database but it is not loaded and at its place there is a wierd reference.

{% img center /images/posts/evaluate.png %}


### Serialization
Lazy loaded attribute needs to have a session attached, making an access in the presentation layer to the attribute via a getter method will produce a LazyInitializationException
