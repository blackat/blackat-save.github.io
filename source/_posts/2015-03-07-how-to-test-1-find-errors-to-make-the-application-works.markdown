---
layout: post
title: "How to test 1: find errors to make the application works"
date: 2015-03-07 09:37:33 +0100
comments: true
categories:
---
This is an _experiment tutorial_ to better learn some _101_ practices and how testing can be a better replacement of developing by debugging.

The goal of the _experiment-project_ is to find, to correct bugs and make the application works by writing tests.

<!-- more -->

## A bit of theory and jargon

_SUT:_ the System Under Test, in a unit test it could be a _class_.

_Test class:_ the _client_ of the interface exposed by the SUT.

_test fixture:_ the context under which a test is run, it is a baseline under which running tests. It is a set of operations to bring the _System Under Test (SUT)_ in a given state, run the test and verify the _expected outcome_. Test fixture and the expected outcome are tightly coupled, the test fixture allows the test to be repeatable for ever and ever. Method `setUp()` or `initialization` is used to create fixture while `tearDown()` or `distruction` is invoked to restore the original state.

A test has four phases:

1. __set up__ to _apply_ the test fixture;
2. __exercise__ to _interact_ with the API exposed by the SUT;
3. __verifiy__ to check if the expected outcome has been _matched_;
4. __tear down__ to return to the _original state_ tearing down the test fixture.

## Clone the project
Clone the project from [Github](https://github.com/blackat/tutorial-howtotest-1-collectors) and import it in you preferred IDE, [IntelliJ IDEA](https://www.jetbrains.com/idea/) in my case.

### Project Structure

    ├── README.md
    ├── exercise-api
    │   ├── pom.xml
    │   └── src
    │       └── main
    │           └── java
    │               └── com
    │                   └── contrastofbeauty
    │                       └── tutorial
    │                           └── api
    │                               ├── collectors
    │                               │   └── Collector.java
    │                               ├── domain
    │                               │   ├── AcknoledgeService.java
    │                               │   └── Callback.java
    │                               └── services
    │                                   └── Service.java
    ├── exercise-to-be-corrected
    │   ├── pom.xml
    │   └── src
    │       ...
    │
    ├── exercise-working
    │   ├── pom.xml
    │   └── src
    │       ...
    │
    └── pom.xml


The maven project is composed by four modules:

1. __exercise-api:__ just collection of interfaces used in the other two modules;
2. __exercise-working:__ working application with a test suite;
3. __exercise-to-be-corrected:__ application with bugs where the test suite has to be created in order to find, correct bugs and make it work;
4. __exercise-tdd:__ implementation of some classes by tdd whose description is in the next [tut](/blog/2015/03/21/how-to-test-2-tdd-a-simple-example).

In the module `exercise-to-be-corrected` run the class `RunMeToBeCorrected.java` and see that it fails some some reason, may be a `NullPointerException` is thrown. We do not care so much because we do not want to make it work by a common pattern __running/debugging__ rather we want to _implement missing tests_ demonstrating how they can help us to find and correct bug make our application more robust.

## Application Description
The application is mainly composed by a fake [Cloud Service](https://github.com/blackat/tutorial-howtotest-1-collectors/blob/master/exercise-working/src/main/java/com/contrastofbeauty/tutorial/services/CloudService.java) able to post user's tweets in batch by means of a [Tweet Collector](https://github.com/blackat/tutorial-howtotest-1-collectors/blob/master/exercise-working/src/main/java/com/contrastofbeauty/tutorial/collectors/TweetCollector.java). [Here](http://yuml.me/edit/3555184c) the UML source.

{% img center /images/posts/tutorial_1_how_to_test.png %}

### The workflow

1. The `CloudService` _has a_ certain numbers of different type collectors (hence the association in the UML diagram), in this case just with one able to collect tweets.
2. A user who wants to post tweet in batch has to open a connection invoking the method `service.openConnection()` of the service.
3. The user then starts to post tweets using the method `service.saveObject()`.
4. Once done `service.saveObjectCompleted()` method will be called to tell the service that the user session is finished.

#### The workflow in details

1. The `CloudService` _has a_ `TweetCollector` and a `Callback` implementation instance (Another approach is to leave the service the initialization of the callback function and substitute the _association_ with a _composition_ in the UML diagram).
2. The `Callback` implementation instance will be set into each added collector by the service.
3. Every time a user wants to save a tweet through the service, the collector will stock it in a _list_ as a sort of buffer.
4. Once the collector has stocked a certain amount of tweets, for instance 500, it will generates a _task_.
5. A task has a collection of tweets and the implementation of the method `call()` which will be called by one of the thread of the thread pool. The method specifies how the list of tweets should be treated, for instance: saved, destroyed, sent to Twitter, printed or something else.
6. A collector uses the `Callback` instance to add a task to a _processing list_.
7. The list of active tasks will be then processed by a thread pool.

### How to initiliaze the service and run it
``` java RunMeWorking.java https://github.com/blackat/tutorial-howtotest-1-collectors/blob/master/exercise-working/src/main/java/com/contrastofbeauty/tutorial/RunMeWorking.java
public class RunMeWorking {

    public static void main(String[] args) {

        Callback callback = new CallbackImpl();
        Service service = new CloudService(callback);

        service.addCollector(new TweetCollector());

        service.openConnection(1L);
        service.saveObject(new Tweet("I am Felix the awesome cat."), 1L);
        service.saveObjectCompleted(new AcknoledgeServiceImpl(), 1L);
    }
}
```

__Actually__ the service does not post anything to Twitter, it is just a way to show a bit of java concurrency and how to write _unit tests_ when dealing with concurrent structures like `Future`.

## Let's correct the application
Let's create tests for the application starting from `TweetCollector.java` class, press `cmd + shift + T` on Mac or `crtl + shift + T` on Windows and choose methods you want to test, then Idea will create the test class in the right place

    src/test/java/com/contrastofbeauty/tutorial/collectors/TweetCollectorTest.java.

__Tip__ Make sure the path `src/test/java` exists, otherwise Idea will create the test class together with the source one.

{% img center /images/posts/idea-test-wizard.png %}

### TweetCollector Class
{% img center /images/posts/tweet-collector.png %}

[TweetCollector](https://github.com/blackat/tutorial-howtotest-1-collectors/blob/master/exercise-to-be-corrected/src/main/java/com/contrastofbeauty/tutorial/collectors/TweetCollector.java) is specific implementation of `Collector` interface able to stores an object of a specific type in a `List` by `userId` when its method `accept()` is invoked (e.g. by the service cloud).

Once the tweet collector has collected a certain number of tweet objects, `flush()` method is called, a `TweetTask` object is created and passed an parameter to `callbackFunction.addTask()`. The task in then put in a queue to be processed. A task is a computation unit, it owns:

- `call()` method: called by a thread when the task is available in the task queue;
- processing list: a set of object on which the aforementioned method will work.

Once the task is added to the queue the list is emptied, ready to accept new object to be processed.


### TweeterCollectorTest
Populate the `setUp()` method creating a new `TweetCollector` and annotate it with `@Before` in order to get it run automatically before any method annotated with `@Test` as specfied by [JUnit doc](http://junit.sourceforge.net/javadoc/org/junit/Before.html). Evrytime a test method is run, the `setUp()` method is invoked to have a fresh and clean collector object.
``` java
public class TweetCollectorTest {

    private Collector collector;

    @Before
    public void setUp() throws Exception {
        collector = new TweetCollector();
    }
}
```

#### Method accept() - 3 errors
Let's start working on the method `accept(Object object, long userId)`. Create, or rename the method if already created by the IDE, `testAcceptGoldenPath()` and add the following assertion:
``` java
@Test
public void testAcceptGoldenPath() throws Exception {
    assertTrue(collector.accept(new Tweet("foo tweet"), 1L));
}
```

In the first test we want to test the _standard scenario_ when everything run smooth, no exceptional situations. So for this reason it has been added the suffix _GoldenPath_, an alternative to [Happy Path](http://en.wikipedia.org/wiki/Happy_path). After that some borderline scenario tests will be added. _So run the test!_

##### Error 1
__Issue:__ a `NullPointerException` is thrown.

__Solution:__ the object `processingList` has not been initialized. Add the initialization in the constructor for instance and _rerun the test_.
``` java
public TweetCollector() {
    processingList = new HashMap<>();
}
```

##### Error 2
__Issue:__ another `NullPointerException` is thrown because the data structure is accessed but not initialized for a given user:
``` java
processingList.get(userId).add((Tweet) object);
```

__Solution:__ check if a given `userId` already exists in the map, if not create and add to the map the pair and _rerun the test_.
``` java
if (processingList.get(userId) == null) {
    processingList.put(userId, new ArrayList<Tweet>());
}
```

##### Error 3
__Issue:__ an `AssertionError` is thrown. All the `NPE` have been fixed but it seems the collector has not accepted the `Tweet` object.

__Solution:__ a missing `return true` prevent the method to behave correctly. Add the aforementioned statement and _rerun the test_. Now the test passes and this is the complete code for the method `accept`:
``` java
@Override
public boolean accept(Object object, long userId) {

    if (object instanceof Tweet) {

        if (processingList.get(userId) == null) {
            processingList.put(userId, new ArrayList<Tweet>());
        }

        processingList.get(userId).add((Tweet) object);

        if (customBufferSize != 0) {
            if (processingList.get(userId).size() == customBufferSize) {
                flush(userId);
            }
        } else if (processingList.get(userId).size() == PROCESSING_LIST_BUFFER_SIZE) {
            flush(userId);
        }

        return true;
    }

    return false;
}
```

##### Additional tests
The method now seems correct but has been tested in case of a different obejcts? Add a new test method which will be part of the _automatic test suite_ we are going to create. This automatic test suite will help us during phases such as refactoring, improving code readability and method evolution.
``` java
@Test
public void testAcceptObjectNotAcceptedBecauseDifferentType() throws Exception {
    assertFalse(collector.accept(new Object(), 1L));
}
```

Our automatic net of tests starts to take shape.

#### Method flush() - 1 error
The method `flush()` does not have a return type, so how can we test the _correct behavior_?

__Idea:__ indeed, we want to test the correct behavior! we need to find a way to check if the behavior of the method is the expected one so if it follows the right path. When `flush()` is invoke we expect the creation of a `TweetTask` object and the invocation of `addTask()` method.

Let's try step by step. If not done yet by the IDE, create method `testFlushGoldenPath()` and invoke the method `flush()`, then _run the test!_
``` java
@Test
public void testFlushGoldenPath() throws Exception {
    collector.flush(USER_ID);
}
```

##### Error 1
__Issue:__ a `NullPointerException` is thrown. This is not a good behavior, the map has not been initialized if the method will be directly called.

__Pre-solution:__ check if in the map exist a `List` for the given `userId`, if not simply exit from the method execution (other solutions are acceptable, depends on requirements), _rerun the method_.
``` java
if (processingList.get(userId) == null) {
    return;
}
```

The test passes, but __no verification__ is done, so the test is pretty _useless_. We want to verify that if there is an item, a new `TweetTask` is created and the method `callbackFunction.addTask()` is invoked.

__Idea:__ the `callbackFunction` has not been set yet so a possible `NPE` could arise. May be in the future it will be inject by _DI_. So we can use [Mockito](https://code.google.com/p/mockito/) to create a mock object and verify if the method `addTask()` has been called.

__Solution:__ mock a `Callback.class` class in order to make a verification on an _expected behavior_ and change a bit the `setUp()` method to initialize mocks via annotations. Moreover set the callback and add at least one tweet object just to reproduce a small common scenario.
``` java
@Mock
private Callback callbackFunctionMock;

@Before
public void setUp() throws Exception {
    MockitoAnnotations.initMocks(this);
    collector = new TweetCollector();
}

@Test
public void testFlushGoldenPath() throws Exception {

    collector.setCallbackFunction(callbackFunctionMock);
    collector.accept(new Tweet("foo tweet"), USER_ID);
    collector.flush(USER_ID);

    verify(callbackFunctionMock, times(1)).addTask(any(TweetTask.class), anyInt());
}
```

So we set the callback function, we invoke the `accept()` method to put one tweet into the list for a given user and finally we call the `flush()` method. A mock object is useful to verify behaviors for instance if a given method has been called. In our case we want to verify if the method `addTask()` has been called _exactly one time_. `any(java.lang.Class)` and `anyInt()` belongs to `org.mockito.Matchers` library. A _matcher_ is an entity that helps to match _parameters_ and _arguments_. For instance, in the test, we need to mach method call _parameters_, `addTask(java.util.concurrent.Callable task, long userId)`, but we are not interested to pass specific _arguments_ so we use _matchers_ to say whatever object implementing `Callable` class is fine.

__Why?__ Well, we are not directly call the `addTask`, we just call a method from a public interface that, under certain condition, should call `addTask` with a newly created object. We do not control and we do not want to control the creation of the object, just let it be, but we want to verify if the method has been invoked with some instances of a given class. In Mockito if we use a matcher for an argument, all the other arguments must be substituted with matchers.

__Best practice__ Name a mock variable with `mock` prefix in order to recognize at first sight which variable is a reference to a real object or to a mock. Other patterns are possible, chose the one you like and keep the whole project consistent.

__Attention__ A test class is not just a test on methods but it is a _test to verify the correct behavior of the all unit_.

##### Alternative solution with Mockito.spy()
A _mock object_ has been used, but another kind of test double exists, the _spy_. A `spy` is a _stub_ (state verification) able to record _calling information_, it is a _"partial mock"_. Instead of doing the check on `processingList.get(userId)` at the beginning of the class it is possible to refactor the tweet creation in a separate method as

``` java
@Override
public void flush(long userId) {

    // create a new processing task
    if (callbackFunction != null) {
        TweetTask tweetTask = getTweetTask(userId);
        if(tweetTask != null) {
            callbackFunction.addTask(tweetTask, userId);
        }
    } else {
        throw new IllegalArgumentException("Callback function must be set by the service.");
    }
}

protected TweetTask getTweetTask(long userId) {
    TweetTask tweetTask = new TweetTask(new ArrayList<>(processingList.get(userId)));
    processingList.get(userId).clear();
    return tweetTask;
}
```

and the corresponding modified test using `Mokito.spy()` to avoid the invoke on method `accept()`

``` java
@Test
public void testFlushWithSpyGoldenPath() throws Exception {

    TweetCollector spyOnTweetCollector = Mockito.spy(TweetCollector.class);

    spyOnTweetCollector.setCallbackFunction(callbackFunctionMock);
    doReturn(new TweetTask(new ArrayList<Tweet>())).when(spyOnTweetCollector).getTweetTask(USER_ID);

    spyOnTweetCollector.flush(USER_ID);

    verify(callbackFunctionMock, times(1)).addTask(any(TweetTask.class), anyInt());
}
```
__Best practice__ Name a spy variable with `spyOn` prefix in order to recognize at first sight which variable is a reference to a real object, to a spy or to a mock.

__Attention__ We do not have invoked `accept()` method

##### Another alternative solution without Mockito but with @Override
Just to make a simple comparison, the above test could have been written without _Mockito_ as follows:

``` java
@Test
public void testFlushWithOverrideGoldenPath() throws Exception {

    final AtomicBoolean taskAdded = new AtomicBoolean();

    Callback callbackFunction = new CallbackImpl(){
        @Override
        public void addTask(TweetTask tweetTask, long userId) {
            taskAdded.set(true);
        }
    };

    TweetCollector collector = new TweetCollector(){
        @Override
        protected TweetTask getTweetTask(long userId) {
            return new TweetTask(new ArrayList<Tweet>());
        }
    };

    collector.setCallbackFunction(callbackFunction);
    collector.flush(USER_ID);

    assertTrue(taskAdded.get());
}
```

The use of _Mockito_ makes the code more concise, easy to read, maintain and understand. Moreover the _behavior verification_ is easy to understand than using a variable and check its value. By using 'verify()' we directly verify if the method has been called and how many times, it is __clear__ that we are doing _behavior verification_.

##### Additional tests
Another test can be written to [test the exception](http://junit.org/apidocs/org/junit/rules/ExpectedException.html) along with the message in case the `callbackFunction` is not set.
``` java
@Test
public void testFlushExceptionThrownWithNullCallbackFunction() throws Exception {
    exception.expect(IllegalArgumentException.class);
    exception.expectMessage("Callback function is null, it must be set by the service.");
    collector.accept(new Tweet("foo tweet"), USER_ID);
    collector.flush(USER_ID);
}
```

Exception messages are really important in order to immediately find the root cause of the issue.

***

### CloudService Class

{% img center /images/posts/cloud-service.png %}

Once the service is created, once or more collectors are added in order to provide batch processing to one or more social networks or other services. A user has to open a connection to the service through the method `openConnection()` and then start to post tweets invoking `saveObject()`. Once done the user calls method `saveObjectCompleted()` to post latest tweets and close the connection.


#### CloudServiceTest Class
Create the class as done for the previous example and initialize the _System Under Test (SUT)_. The test class is the API client exposed by the SUT. The `setUp()` method properly initializes the SUT in order to be ready for the test method.

``` java
public class CloudServiceTest {

    private CloudService cloudService;

    private Callback callbackFunction;

    @Before
    public void setUp() throws Exception {

        callbackFunction = new CallbackImpl();
        cloudService = new CloudService(callbackFunction);
    }
}
```

#### Method addCollector() - 2 errors
Create the first test for method `addCollector()`
``` java
@Test
public void testAddCollectorGoldenPath() throws Exception {

    cloudService.addCollector(new TweetCollector());

    assertEquals(1, cloudService.getCollectorSize());
}
```

##### Error 1
__Issue:__ a `NullPointerException` will be thrown when `CloudService.getCollectorSize()` is called in the assertion.

__Solution:__ the object `processingCollectors` has not been initialized. Add the initialization in the constructor, for instance, and _rerun the test_.
``` java
public CloudService(Callback callback) {

    processingCollectors = new HashMap<>();
}
```

##### Error 2 - Test Driven Development (TDD)
Spot this error is not easy, it is a missing code. As it has been seen before, a collector uses a callback function to post a batch processing. When a collector is added the callback function must be set by the service. The collector in this situation acts as a _collaborator_ and we want to verify that a specific method is invoke on the collaborator itself. Create the _test before_ and _implementation later on_ to make the test passes (TDD).

In order to verify a method call, i.e. _indirect output_, we use Mockito, so add `tweetCollectorMock` variable and init mocks inside of the `setUp()` (initialization of the test fixture) method adding `MockitoAnnotations.initMocks(this)` and _run the test_
``` java
@Mock
private TweetCollector tweetCollectorMock;

@Before
public void setUp() throws Exception {

    MockitoAnnotations.initMocks(this);
    callbackFunction = new CallbackImpl();
    cloudService = new CloudService(callbackFunction);
}

@Test
public void testAddCollectorVerifyCallbackFunctionAddedWithMockito() {

    cloudService.addCollector(tweetCollectorMock);
    verify(tweetCollectorMock, times(1)).setCallbackFunction(any(Callback.class));
}
```

__Issue:__ an error is thrown `Wanted but not invoked:...` that means the method `setCallbackFunction()` has not been called.

__Solution:__ in the method `addCollector()` add the call to set the callback function and  _rerun the test_.
``` java
@Override
public void addCollector(Collector collector) throws RuntimeException {

    processingCollectors.add(collector);

    collector.setCallbackFunction(callbackFunction);
}
```

Now the test passes. The `addCollector()` function has been corrected by tests. The tests have been left separated for two reasons:

1. test two different aspects of the same function
2. how _Test Driven Development (TDD)_ helps building a solid net of automated tests and a more reliable code, tests come directly from requirements.

__Attention!__ TDD helps creating a tests net, tests represent the requirements, the documentation as code that can be read to understand the behavior of the class. The code satisfying the tests is consistent with the requirements, its behavior is something to _rely on_ for the next development iteration.

##### Error 2 - Alternative solution with @Override
Instead of using a mocking framework, it is possible to _"intercept"_ the call of the method and register a status that can be used for verification later on as following:
``` java
@Test
public void testAddCollectorVerifyCallbackFunctionAddedWithOverride() {

    final AtomicInteger functionCalled = new AtomicInteger();

    cloudService.addCollector(new TweetCollector(){

    @Override
    public void setCallbackFunction(Callback callback) {

            functionCalled.set(1);
        }
    });

    assertEquals(1, functionCalled.get());
}
```

The use of a mock object and a mocking framework as Mockito allows to have a more concise and readable code. Methods and matchers such as  `verify`, `times()` and `any()` make the comprehension of the test easier. Moreover using a mock for a collector makes clearer that in this context it a _collaborator_ and not the object of the test, it is even closer to a documentation purpose.


#### Method openConnection() - 1 error
Let's create a new test and run it
``` java
@Test
public void testOpenConnectionGoldenPath() {
    cloudService.openConnection(USER_ID);
    assertTrue(cloudService.isUserConnected(USER_ID));
}
```

##### Error 1
__Issue:__ a `NullPointerException` will be thrown.

__Solution:__ the object `processingFutureList` has not been initialized. Add the initialization in the constructor for instance and _rerun the test_.
``` java
public CloudService(Callback callback) {

    processingFutureList = new HashMap<>();

    processingCollectors = new ArrayList<>();
}
```

Now the test passes.

##### Additional test
To build a _net of automatic tests_ is interesting to test different aspects of a method so we can add the following one
``` java
@Test
public void testOpenConnectionUserHasNotOpenConnection() {
    assertFalse(cloudService.isUserConnected(USER_ID));
}
```

#### Method saveObject() - 0 errors
This method does not have errors anymore, but one or more test must be written to build up an efficient _test suite_. It is a small workflow able to test the interface exposed by the class.

__Remember!__ Test the behavior of _all the unit_ not just single and separated methods, the focus is on the all unit.
``` java
@Test
public void testSaveObjectWithOverrideGoldenPath() throws Exception {

    final AtomicBoolean accepted = new AtomicBoolean();

    Collector tweetCollector = new TweetCollector(){
        @Override
        public boolean accept(Object object, long userId) {

            if (object instanceof Tweet) {
                accepted.set(true);
                return true;
            }

            return false;
        }
    };

    cloudService.addCollector(tweetCollector);

    cloudService.openConnection(USER_ID);

    cloudService.saveObject(new Tweet("foo tweet"), USER_ID);

    assertTrue(accepted.get());
}
```

Once again pay attention on the difference between using _override_ and _Mockito_ to test _indirect output_ as follows
``` java
@Test
public void testSaveObjectWithMockitoGoldenPath() throws Exception {

    Collector collectorMock = mock(TweetCollector.class);
    when(collectorMock.accept(any(Tweet.class), anyInt())).thenReturn(true);

    cloudService.addCollector(collectorMock);

    cloudService.openConnection(USER_ID);

    cloudService.saveObject(tweetMock, USER_ID);

    verify(collectorMock, times(1)).accept(any(Tweet.class), anyInt());
}
```

Once again the use of Mockito allows to have a more concise and readable test. The `Mockito.verify()` method highlights the test of the _indirect output_.

#### Additional class behavior tests

##### Additional test 1
It is interesting to enrich the test suite with some other tests, may be a bit redundant but useful to highlight if the code correctly answer to borderline scenarios.
```java
@Rule
public ExpectedException exception = ExpectedException.none();

@Test
public void testSaveObjectObjectNotAcceptedThrowException() throws Exception {

    exception.expect(IllegalArgumentException.class);
    exception.expectMessage("Entity of type " + new Object().getClass() + " cannot be accepted.");

    cloudService.addCollector(new TweetCollector());

    cloudService.openConnection(USER_ID);

    cloudService.saveObject(new Object(), USER_ID);
}
```

##### Additional test 2
The name of the method makes the test self-explanatory.
```java
@Test
public void testSaveObjectUserNotConnected() throws Exception {
    exception.expect(IllegalArgumentException.class);
    exception.expectMessage("User with id " + USER_ID + " has not open any connection, please open a connection " +
        "before trying to save.");

    cloudService.saveObject(mock(Callable.class), USER_ID);
}
```

##### Additional test 3
```java
@Test
public void testSaveObjectCompletedUserNotConnectedThrowException() throws Exception {
    exception.expect(IllegalArgumentException.class);
    exception.expectMessage("User with id " + USER_ID + " has not open any connection, please open a connection " +
        "before trying to save.");

    cloudService.saveObjectCompleted(null, USER_ID);
}
```

##### Additional test 4
```java
@Test
public void testSaveObjectCompletedGoldenPath() throws Exception {

    doNothing().when(acknoledgeServiceMock).sendAckSuccess();

    TweetCollector spyOntweetCollector = spy(new TweetCollector());
    // if you call the real method there will be an exception, use doReturn for stubbing
    doReturn(tweetTaskMock).when(spyOntweetCollector).getTweetTask(USER_ID);
    when(tweetTaskMock.call()).thenReturn(1L);

    cloudService.addCollector(spyOntweetCollector);

    cloudService.openConnection(USER_ID);

    cloudService.saveObject(new Tweet("foo tweet"), USER_ID);

    cloudService.saveObjectCompleted(acknoledgeServiceMock, USER_ID);

    verify(tweetTaskMock, times(1)).call();
    verify(acknoledgeServiceMock, times(1)).sendAckSuccess();
}
```

##### Additional test 5
```java
@Test
public void testSaveObjectCompletedExceptionThrown() throws Exception {
    doNothing().when(acknoledgeServiceMock).sendAckSuccess();

    TweetCollector spyOntweetCollector = spy(new TweetCollector());
    // if you call the real method there will be an exception, use doReturn for stubbing
    doReturn(tweetTaskMock).when(spyOntweetCollector).getTweetTask(USER_ID);
    when(tweetTaskMock.call()).thenThrow(InterruptedException.class);

    cloudService.addCollector(spyOntweetCollector);

    cloudService.openConnection(USER_ID);

    cloudService.saveObject(new Tweet("foo tweet"), USER_ID);

    cloudService.saveObjectCompleted(acknoledgeServiceMock, USER_ID);

    verify(tweetTaskMock, times(1)).call();
    verify(acknoledgeServiceMock, times(1)).sendAckFailed(any(RuntimeException.class));
}
```
