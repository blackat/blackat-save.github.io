---
layout: post
title: "Spring Framework Testing"
date: 2013-05-08 16:48
comments: false
categories: [spring, test, testing, junit, integration test, unit test]
published: false
---
Enterprise application are hosted in a server environement such as a servlet container or a full JEE application server.
To test the application, the developer can use only the _public interface_ of the application like a web service, a messaging service, RMI or a web browser.

Architecture recommendations for Spring allow to produce clean layering and componentization of the codebase that will facilitate unit testing. It will be possible to test a service layer without accessing persistent data while running unit tests.
<!-- more -->

##Scenario
The implementation of the service layer involve different changes in the data base schema and corresponding testing to check the correctness of queries and schema. Quite often during the prototyping phase is normal to change the schema of the database so an agile and light way of testing is required.

__System Under Test (SUT).__ It is the _service layer_.
{% codeblock lang:java %}
public class BulbProductServiceImplTest {
    @Autowired
    BulbProductService bulbProductService;

    @Test
    public void testFindAll() throws Exception {
        BulbProducts bulbProducts = bulbProductService.findAll();
    }
}
{% endcodeblock %}

It would be great being able to test the method outside the application server without any external enterprise infrastructure that is _in isolation_.

To ensure that BulbProductServiceImpl class performs its business correctly using H2-in-memory database to host data model and testing data, JPA providers (Hibernate and Spring Data JPA's repository abstraction).

Consider the following steps:

1. __Spring profiles definition.__ Define profiles for development, testing and production to have databases with different characteristics according to the specific environment.
2. __Infrastructure classes.__ To form a base class for service layer testing with a specific testing configuration for the database.
3. __Service layer testing.__ Implementation of the service layer tests in isolation from the application container.

###Spring profiles definition
Starting from Spring 3.1 M1 the new _bean definition profile_ feature has been introduced to configure the container when configuration classes are used.

`@Configuration` classes, introduced in Spring 3.1, are the pure Java equivalent to Spring `<beans/>` XML files. Each XML component has its counterpart class, for instance the XML component `<jdbc:embedded-database/>` is represented by the class `EmbeddedDatabaseBuilder`.

Thus it is possible to write a class which represents the `@Configuration` of the `ApplicationContext` specific to test the service layer. This way of configuring the container is _string-free_ and _type-safe_.

[1]: http://blog.springsource.org/2011/02/14/spring-3-1-m1-introducing-profile/


Components.

* __Entity.__ A thing having attributes and partecipating in relationships to other entities. The OO paradigm adds behaviors to an entity calling it object.
    * An entity is a fine-grained business domain object which can be created, updated and deleted in any context.
        * _In-memory entity._ Simple Java object managed by the JVM which can be easily changed without being persisted.
        * _Database entity._ A transaction context is required to commit changes in the database.
    * Metadata expressed through annotations defined in the `javax.persistence` package enables the persistence layer to recognize, interpret and properly manage the entity.
* __Entity object.__ In-memory instances of entity classes, the instance is a an object state which, when made persistent, represents a physical row in the database.
* __Entity manager.__ Interface used to interact with a persistent context, each `EntityManager` manages its own persistence context.
* __Persistent context.__ Collection of all the managed objects of an Entity Manager. If an entity object has to be retrieved but exists in the persistent context it is returned without actually accessing the database.
    * _Main role._ A database entity object is represented by no more than one in-memory entity object within the same EntityManager.
    * _Local cache._ A persistent context can be seen as a local cache for a given an `EntityManager`.

###Key points

* Unit testing focus on the components of the application in _isolation_.
* IoC principle adds a value to unit testing and Spring Framework can simplify the integration testing.
* Spring Framework provides mock objects and testing support classes to facilitate testing in certain scenario.
* Spring module `spring-test` is the first class support for integration testing.

###Mock objects

###Unit testing support classes

##Integration Testing
* __Focus on.__
    * testing outside any container
    * testing without relying on any specific application server or deployment environment
    * avoid connection and set up of any external enterprise infrastructures
    * able to test components in isolation
    * test correct wiring and data access
    * simplify testing unit with Spring container
    * dependency injection can remove direct dependencies such as the ones on the entity manager making the unit testing difficult ?????
        * the advantage of the DI???
* __In practice.__
    * `spring-test` module and `org.springframework.test` package
    * `org.springframework.test.context` package, annotation driven framework to provide unit and integration testing support
        * agnostic of the actual testing framework in use allowing instrumentation of JUnit, TestNG, etc
* __Annotations.__
    * `@ContextConfiguration` to configure an `ApplicationContext` for the test.
        * Dependencies of the test instances are _injected_ from beans defined in the application context.
        * If no locations are specified, the configured `ContextLoader` loads a context from a default set of location.
            * `GenericXmlContextLoader` is the default context loader and generates the file name and the location based on the name of the test class adding the suffix `-context.xml`.
        * Provide an array of _resource locations_ of XML configuration metadata such as the one provided in the `web.xml` deployment descriptor.
        * `ApplicationContext` is reused for each test so the setup cost happens only once per test fixture.
    * `@DirtiesContext` can be used to annotate a method causing the rebuild of the application context before executing the next test.

Testing outside the server facilitates the way to get an enterprise application bug free but also reduces the time and makes the unit testing more complete and automatic.

The advantage is to test without deployment and any connection to an external enterprise infrastructure allowing the developer to test
  * correct wiring of the Spring IoC container contexts,
  * data access using JDBC or an ORM tool being able to also test SQL statements, queries, JPA entity mappings and strategies and so on.

Spring support for integration testing is expressed by `spring-test` module which encapsulate the  `org.springframework.test` package, a set of classes useful to perform integration testing with a Spring container.

###Spring TestContext Framework
Framework located in the `org.springframework.test.context` package put a lot of importance on _convention over configuration_.

* __Core of the framework.__
    * `TestContext` class encapsulates the context in which a test is executed, agnostic of the testing framework in use.
    * `TestContextManager` class is the main entry point of the framework, manages a _single_ `TestContext` and signals events to all registered `TestExecutionListener`.
    * `TestExecutionListener` is an listener interface that, if implemented, let to react to test execution events published by `TestContextManager`. The listener API is made of well-define exectuion point methods to be implemented in order to provide specific behavior during test execution.
        * `DependencyInjectionTestExecutionListener`, `DirtiesContextTestExecutionListener`, and `TransactionalTestExecutionListener`.
* __Dependency injection.__
    * Dependencies of the test instances are injected from beans in the application context configured through `@ContextConfiguration`.
* __JUnit 4.5+ support classes.__
    * `org.springframework.test.context.junit4` package
    * `AbstractJUnit4SpringContextTests` integrates the context framework with `ApplicationContext` testing support in a JUnit environment.
    * `AbstractTransactionalJUnit4SpringContextTests` transactional extension which expects in the application context
        * `javax.sql.DataSource` bean
        * `PlatformTransactionManager` bean
    * `@RunWith(SpringJUnit4ClassRunner.class)` allows developer to implement standard JUnit 4.5+ unit and integration tests and get the benefits of the TestContext framework such as dependency injection of test instances, transactional test method execution, etc.
        * Transaction management will roll back the transaction after the execution of each _test method_.

Notes

* `@Configuration` classes are pure-java configuration, the equivalent of the Spring XML metadata files.
