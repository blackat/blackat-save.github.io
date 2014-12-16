---
layout: post
title: "Authentication Manager"
date: 2013-05-10 21:25
comments: true
categories: [spring, test, security]
published: false
---
Authentication manager is a module implemented to be plugged in a web application to manage the authorization of registred user. It based on Spring and offers different persistent mechanisms, others could be added.
<!-- more -->

{% codeblock lang:java %}
@MappedSuperclass
public abstract class AbstractUser {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column(name = "email", unique = true, nullable = false)
    private EmailAddress emailAddress;

    public final Long getId() {
        return id;
    }
}
{% endcodeblock %}

The `EmailAddress` is a value object, that is an object containing attributes but does not have a conceptual identity [[1]]. A value object is used to express domain concepts implemented as a primitive type but allowing to implement domain constraints inside the value object so when it is implemented it is valid and immutable.

In the following value object implementation when the email object is instantiated means that the wrapped email address is valid.

{% codeblock lang:java %}
@Embeddable
public class EmailAddress {

    private static final String EMAIL_PATTERN = "^[_A-Za-z0-9-\\+]+(\\.[_A-Za-z0-9-]+)*@"
            + "[A-Za-z0-9-]+(\\" + ""
            + ".[A-Za-z0-9]+)*(\\.[A-Za-z]{2,})$";

    @Column(name = "email")
    private String emailAddress;

    protected EmailAddress() {

    }

    public EmailAddress(final String email) {
        Assert.isTrue(isValid(email), "The email address is not correct: " + email);
        this.emailAddress = email;
    }

    private boolean isValid(final String email) {
        Pattern pattern = Pattern.compile(EMAIL_PATTERN);
        Matcher matcher = pattern.matcher(email);
        return matcher.matches();
    }

    // omitted methods
}
{% endcodeblock %}

##Testing
Once the data model has been defined a testing infrastructure has to be built in order to test what has been done so far and to prevent possible future regressions. Testing is a _continuos activity_ during software developing and can be done at different levels:

* __Unit tests.__ Written by developers and focus on single application components tested in isolation.
* __Integration tests.__ Written by developers and focus on use cases within the application such as the login or the reservation of a ticket.
    * Make full use of external resources such as a database
    * Take a component from an application and run in isolation as if they were inside the application server.
    * Slower than a unit test but ensure there are no regressions.
* __Functional tests.__ Are black box tests written and automated by quality engineers instead of develpers.
    * based on functional specification and user interface of a product
    * verify the product behavior without understanding the implementation.
    * Acceptance tests. Are customer-driven.
        * conducted manually
        * carried out directly by the costumers to verify the requirements are fulfilled in the user interface and behavior of the application.

A unti test involve, for instance, the `EmailAddress` class to ensure that only valid email address are store. Integration test, instead, could involve more than one class and should verify the persistence capabilities before the deployment into production environment so _outside the application server_.

The code _should not_ be tightly couple to the container, _Dependency Injection_ should help reducing this container dependency. A clean layering and componentization of the codebase could facilitate easier unit testing. Spring provides mock objects and testing support classes to help building unit and integration tests.

##Integration Tests Infrastructure

* __Goal.__ Define a testing infrastructure for integration tests in order to check the correctness of the authentication service layer without requiring deployment into a server or connection to other enterprise infrastructures.
* __What to test.__ Defore deploying into a production environment test:
    * correct wiring of the Spring IoC container contexts,
    * data access using JDBC or any ORM tool. This involve testing the correctness of SQL statements, Hibernate queries and JPA entity mapping.
* __What Spring offers.__ Spring provides first-class (object may be named by variable, passed as argument, returned as result, included in data structure) support for integration testing
    * _module:_ `spring-test` name of the artifact,
    * _package:_ `org.springframework.test` class for integration testing with a Spring container.
* __Unit and integration Spring test support.__ Annotation-driven `TestContext` framework allows to _instrument tests_ in various environment such as Junit and TestNG.
* __Spring testing primary goals.__
    * _Managing Spring IoC container caching between test executions:_ `ApplicationContext` is loaded and cached once for each test suite to improve the startup time. Spring container could take time to instantiate some objects (i.e. project with a lot of Hibernate mapping files). Once loaded the `ApplicationContext` is resused for each test. Test class provides an array of XML configuration metadata to configure the application, if needed the `ApplicationContext` can be rebuild before executing the next test.
    * _Providing Dependency Injection of test fixture instances:_ A _test fixture_ or _test context_ refers to a fixed state used as a baseline for running tests ensuring there will be a fixed and well know environment to run repeatable tests. For instance a fixture is created by `setUp()` and `tearDown()` methods to create and destroy an expected state _separating_ them from the actual testing. `TestContext` framework loads `ApplicationContext` and configure, through _Dependency Injection_, instances of the test classes with preconfigured beans setting up a test fixture. The same application context can be reused in different testing scenarios.
    * _Providing transaction management appropriate to integration testing._ Operations involving a persistent store affect its state and must be performed inside of a transaction. `TestContext` framework, by default, craetes and roll back a transaction for _each test_.
    * _Supplying Spring-specific base classes._ Base `abstract` classes providing _well-defines hooks_ into the testing framework enabling access to `ApplicationContext` and `SimpleJdbcTemplate` to execute SQL statements to query a database.

##Testing Scenario
The focus in on testing authentication methods provided by `AuthenticationService` and its JPA implementation `AuthenticationServiceImpl` using H2 in-memory database to host the data model, DBUnit, a JUnit extension to put the database in a known state when the tests run, JPA providers such as Hibernate and Spring Data JPA's repository abstraction.


###Application context and profile definition
Defining an environment in Spring means configuring an `ApplicationContext` which allows the definition of a set of beans that can be injected at runtime orchestrating an object. Dependency injection is essential to keep classes clean and without any direct dependencies which usually render very difficult the test of a class.

An `ApplicationContext` for testing purpose can be configured for a given `@Profile`. Different profiles can be defined to set up a development and a testing environment respectively having, for instance, the same in memory database but with or without prepopulated tables.

Two profiles can be defined: `dev` and `test`. Thus a selected datasource and sql scripts will be applied only to a specific environment.

{% codeblock lang:xml %}
<!-- Data source applicable only for development -->
<beans profile="dev">
    <jdbc:embedded-database id="dataSource" type="H2">
        <jdbc:script location="classpath:schema.sql"/>
        <jdbc:script location="classpath:test-data.sql"/>
    </jdbc:embedded-database>
</beans>
{% endcodeblock %}

Now set the `dev` profile as _default profile_ for the current application development adding a `<context-param>` to the the `web.xml` descriptor. This tells Spring to bootstrap the `WebApplicationContext` with beans defined in the `dev` profile.

{% codeblock lang:xml %}
<!-- The definition of the Root Spring Container shared by all Servlets and Filters -->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/spring/root-context.xml</param-value>
</context-param>
<!-- Spring boostrap WebApplicationContext with the beans defined for the dev profile -->
<context-param>
    <param-name>spring.profile.active</param-name>
    <param-value>dev</param-value>
</context-param>
{% endcodeblock %}


###Infrastructure Classes
First of all an interface is define in order to decouple the data source configuration from the integration testing environment where the data source configuration will be injected.
{% codeblock lang:java %}
public interface DataConfig {

    DataSource dataSource();
}
{% endcodeblock %}

The implementation of the interface defines a `@Configuration` which is a Java-based configuration of the `ApplicationContext` substituting the metadata bean XML file. Each XML element has its own counterpart pure Java component. `@Configuration` approach is a _string-free_ and _type-safe_ way to configure the Spring container.

Different data source configurations can be defined with respect to the environment of the application such as production, development and test. A `@Profile`-annotated `@Configuration` makes a pure Java configuration specific for a profile. According to which profile is activated, the container will  _process_ or _skip_ a given the configuration.

The `StandaloneH2DataConfig` defines the `ApplicationContext` for service layer testing and it will be processed by the container when the _test_ profile is active.
{% codeblock lang:java %}
@Configuration
@Profile("test")
public class StandaloneH2DataConfig implements DataConfig {

    @Override
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .addScript("classpath:schema.sql")
            .addScript("classpath:test-data.sql")
            .build();
    }
}
{% endcodeblock %}

Another pure Java configuration file is defined and the specific data source is injected using the `@Autowired` annotation.

The `datasource-tx-jpa.xml` is imported and define the transaction and JPA configuration reusable for testing. `@ComponentScan` annotation instructs Spring to scan the JPA implementation of service layer beans which are going ot be tested. (TO BE CHECKED: the scan is used to inject testing db into the layer)
{% codeblock lang:java %}
@Configuration
@ImportResource("classpath:datasource-tx-jpa.xml")
@ComponentScan(basePackages = {"com.contrastofbeauty.authenticationmanager.service.jpa"})
public class ServiceTestConfig {

    @Autowired
    DataConfig dataConfig;

    @Bean
    public DataSourceDatabaseTester dataSourceDatabaseTester() {
        return new DataSourceDatabaseTester(dataConfig.dataSource());
    }
}
{% endcodeblock %}

The abstract base test class can be extended by all the other test classes in order to share the same `ApplicationContext` configuration for a test environment.

`@RunWith` belongs to JUnit library and allows to specify a custom runner instead of the one built into JUnit. Then JUnit will invoke the Spring's JUnit runner class `SpringJUnit4ClassRunner.class` to execute the test case wihtin the Spring's `ApplicationContext` environment. It `SpringJUnit4ClassRunner.class` extends `BlockJUnit4ClassRunner` providing Spring testing functionality to standard JUnit tests.

`@ContextConfiguration` annotation specify to the Spring JUnit runner which configuration should be loaded (XML and class files can be passed). Finally `@ActiveProfile` annotation indicates to Spring container to process and load _beans_ belonging to the _test_ profile and specified in `ServiceTestConfig` class configuration. The data source defined in `StandaloneH2DataConfig.class` will be loaded and not the one defined in `datasource-tx-jpa.xml` because maked with _"dev"_.

Moreover the class extends `AbstractTransactionalJUnit4SpringContextTests` class which is Spring's support for JUnit with direct injection and transaction management mechanism. In this way Spring is able to control and rollback the transaction after each test method execution, so all the database operation are rolled back ripristinating the orignal state of the database.

`EntityManager` variable is define and autowired to be used within the test cases.
{% codeblock lang:java %}
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {ServiceTestConfig.class})
@ActiveProfiles("test")
public abstract class AbstractServiceImplTest extends AbstractTransactionalJUnit4SpringContextTests {

    @PersistenceContext
    protected EntityManager em;
}
{% endcodeblock %}

##Repository
The implementation of the data access layer involves a lot of boiler plate code. Spring Data provides _repository abstraction_ concept and construct to easily implement data access layers for various _persistence store_.
{% codeblock lang:java %}
public interface AuthenticationRepository extends CrudRepository<User, Long> {
}
{% endcodeblock %}

Standard CRUD functionality for the entity being managed are provided by `CrudRepository<T, ID extends Serializable>`. It extends `Repository<T, ID>` interface which is a _marker interface_ to capture the types to deal with and discovery the interfaces that extends this one.

Other queries on the uderlying datastore are declared through the following interface
{% codeblock lang:java %}
public interface AuthenticationRepository extends Repository<User, Long> {

    User findUserByEmail(String email);
}
{% endcodeblock %}

Now using a custom namespace from Spring Data JPA module it is possible to define JPA repository beans. For all the beans annotated with `@Repository` exception translation from JPA persistence provider into Spring's `DataAccessException` hierarchy is activated.

{% codeblock lang:xml %}
<jpa:repositories base-package="com.acme.repositories" />
{% endcodeblock %}


##Web Interface



[1]: http://en.wikipedia.org/wiki/Domain-driven_design
