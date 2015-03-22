---
layout: post
title: "How to test 1: find errors to make the application works"
date: 2015-03-07 09:37:33 +0100
comments: true
categories:
---
This is an _experiment tutorial_ to better learn some _101_ practices and how testing can be a better replacement of developing by debugging.

The goal of the _experiment-project_ is find, correct bugs and make the application works by writing tests.

This tutorial is still in __DRAFT 3__.
<!-- more -->

## 1 Clone the project
Clone the project from [Github](https://github.com/blackat/tutorial-howtotest-1-collectors) and import it in you preferred IDE, [IntelliJ IDEA](https://www.jetbrains.com/idea/) in my case.

## 2 Scaffolding

  ├── README.md
  ├── example.log
  ├── exercise-api
  │   ├── exercise-api.iml
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
  │   ├── exercise-to-be-corrected.iml
  │   ├── pom.xml
  │   └── src
  │       ...
  │
  ├── exercise-working
  │   ├── example.log
  │   ├── exercise-working.iml
  │   ├── pom.xml
  │   └── src
  │       ...
  │
  └── pom.xml


The maven project is composed by three modules:

1. __exercise-api:__ just collection of interfaces used in the other two modules
2. __exercise-working:__ working application with a test suite
3. __exercise-to-be-corrected:__ application with bugs where the test suite has to be created in order to find, correct bugs and make it work.

## 3 Application Description
The application is composed by a [Cloud Service](https://github.com/blackat/tutorial-howtotest-1-collectors/blob/master/exercise-working/src/main/java/com/contrastofbeauty/tutorial/services/CloudService.java) able to post user's tweets in batch by means of a [Tweet Collector](https://github.com/blackat/tutorial-howtotest-1-collectors/blob/master/exercise-working/src/main/java/com/contrastofbeauty/tutorial/collectors/TweetCollector.java).

{% img center /images/posts/tutorial_1_tweet_uml.png %}

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

### 3.1 How the service is used
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

Once the service has been started, a user can just open a connection to it and start to submit tweets to be posted. Once a certain amount of tweets has been reached, the service contacts Twitter to post them. The CloudService uses collectors to collect messages to be posted somewhere. When the service starts, one or more collectors can be registered. In the example one one collector has been registered and it is able to collect _tweets_. Other collectors can be implemented and added to the service.

When the user has finished to post tweets, `saveObjectCompleted()` method is called in order to close the connection with the service and sending the latest tweets to Twitter.

__In reality__ the service does not post anything to Twitter, it is just a way to show a bit of java concurrency and how to write _unit tests_ when dealing with concurrent structures like `Future`.

## 4 Let's correct the application
Let's create tests for the application starting from `TweetCollector.java` class, press `cmd + shift + T` on Mac or `crtl + shift + T` on Windows and choose methods you want to test, then Idea will create the test class.

__Tip__ Make sure the path `src/test/java` exists, otherwise Idea will create the test class together with the source one.

{% img center /images/posts/idea-test-wizard.png %}

### 4.1 TweeterCollector.class
[TweeterCollector.class](https://github.com/blackat/tutorial-howtotest-1-collectors/blob/master/exercise-to-be-corrected/src/main/java/com/contrastofbeauty/tutorial/collectors/TweetCollector.java)

// add description of class and how the callback will be called.

### 4.1.1 TweeterCollectorTest.class
Populate the `setUp()` method creating a new `TweetCollector`.
``` java
public class TweetCollectorTest {

    private Collector collector;

    @Before
    public void setUp() throws Exception {
        collector = new TweetCollector();
    }
}
```

#### 4.1.2 Method accept(), 3 errors
Let's start working on the method `accept(Object object, long userId)`, run the first test that has been renamed `testAcceptGoldenPath()` and contains just one assertion:
``` java
@Test
public void testAcceptGoldenPath() throws Exception {
    assertTrue(collector.accept(new Tweet("foo tweet"), 1L));
}
```

##### Error 1
__Issue:__ a `NullPointerException` will be thrown.

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
The method now seems correct but has been tested in case of a different obejct? Add a new test method which will be part of the _automatic test suite_ we are going to be create. This automatic test suite will help us during phases such as refactoring, improving code readability and method evolution.
``` java
@Test
public void testAcceptObjectNotAcceptedBecauseDifferentType() throws Exception {
    assertFalse(collector.accept(new Object(), 1L));
}
```

#### 4.1.3 Method flush(), 1 error
If not done yet by the IDE, create method `testFlushGoldenPath()` and invoke the method `flush()` as following:
``` java
@Test
public void testFlushGoldenPath() throws Exception {
    collector.flush(USER_ID);
}
```

##### Error 1
__Issue:__ a `NullPointerException` is thrown. This is not a good behavior, the map has not been initialized and we do not in advance if the method will be called after an `accept()` or not, for the time being different scenarios are possible.

__Pre-solution:__ check if in the map exist a `List` for the given `userId`, if not simply exit from the method execution (other solutions are acceptable, depends on requirements), _rerun the method_.
``` java
if (processingList.get(userId) == null) {
    return;
}
```

The test passes, but __no verification__ is done, so the test is pretty useless. We want to verify that if there is an item, a new `TweetTask` is created and the method `callbackFunction.addTask()` is invoked.

__Idea:__ the `callbackFunction` has not been set yet so a possible `NPE` could arise or in the future in will be inject by means of _DI_, we still do not know. So we can use [Mockito](https://code.google.com/p/mockito/) to verify if the method `addTask()` has been called.

__Solution:__ mock a `Callback.class` class in order to make a verification on an expected behavior and change a bit the `setUp()` method to initialize mocks via annotations as follows:
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

So we set the callback function and we invoke the `accept()` method to fill the list for a given user. Notice that a test class is not just a tests on methods but it is a _test to verify the correct behavior of the all unit_.


##### Error 1 - Alternative solution with Mockito.spy()
Instead of doing the check on `processingList.get(userId)` at the beginning of the class it is possible to refactor the tweet creation in a separate method as
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

##### Error 1 - Alternative solution with @Override
Just to make a simple comparison, the above test could have been written without _Mockito_ as follows:
``` java
@Test
public void testFlushWithOverrideGoldenPath() throws Exception {

    final AtomicBoolean taskAdded = new AtomicBoolean();

    Callback callbackFunction = new CallbackImpl(null){
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

The use of _Mockito_ makes the code more concise, easy to read and understand. Moreover the _behavior verification_ is easy to understand than using a variable and check its value. By using 'verify()' we directly verify if the method has been called and how many times, it is clear that we are doing _behavior verification_.

##### Additional tests
Another test can be written to test the exception along with the message in case the `callbackFunction` is not set.
``` java
@Test
public void testFlushExceptionThrownWithNullCallbackFunction() throws Exception {
    exception.expect(IllegalArgumentException.class);
    exception.expectMessage("Callback function must be set by the service.");
    collector.accept(new Tweet("foo tweet"), USER_ID);
    collector.flush(USER_ID);
}
```

Exception messages are really important in order to immediately find the root cause of the issue.

***

### 4.2 CloudService.class
Once the service is created, once or more collectors are added in order to provide batch submission to one or more social networks or similar services. A user has to open a connection to the service through the method `openConnection()` and then start to post tweets for instance invoking `saveObject()`. Once done user will call method `saveObjectCompleted()` to post latest tweets and close the connection.


#### 4.2.1 CloudServiceTest.class
Create the class as done for the previous example and initialize the _SUT_.
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

#### 4.2.2 Method addCollector() - 2 errors
Create the first test for method `addCollector()`
``` java
@Test
public void testAddCollectorGoldenPath() throws Exception {

    cloudService.addCollector(new TweetCollector());

    assertEquals(1, cloudService.getCollectorSize());
}
```

##### Error 1
__Issue:__ a `NullPointerException` will be thrown.

__Solution:__ the object `processingCollectors` has not been initialized. Add the initialization in the constructor for instance and _rerun the test_.
``` java
public CloudService(Callback callback) {

    processingFutureList = new HashMap<>();
}
```

##### Error 1 - Alternative solution
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

##### Error 2 - Test Driven Development (TDD)
A collector should be added along with the set of the callback function. A collector should be able to submit the job to be done when necessary using the callback function. The collector in this situation acts as a collaborator and we want to verify that a specific method in invoke on the collaborator.

In order to verify a method call, i.e. _indirect output_, we use Mockito, so add `tweetCollecorMock` variable and init mocks inside of the `setUp()` method adding `MockitoAnnotations.initMocks(this)` and _run the test_
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

Now the test passes. The `addCollector()` function was empty and by tests it has been implemented. The two tests could have been put in a single one and step by step modifying the method call to make the test passed. _Test Driven Development (TDD)_.

#### 4.2.3 Method openConnection() - 1 error
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
``

#### 4.2.4 Method saveObject() - 0 errors
This method does not have any error, but one or more test must be written to build up an efficient _test suite_. Moreover it could be another opportunity to see the difference between using _override_ and _Mockito_ to test _indirect output_.
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

and the same test but done using Mockito

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

##### Additional test 1
``` java
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
``` java
@Test
public void testSaveObjectUserNotConnected() throws Exception {
    exception.expect(IllegalArgumentException.class);
    exception.expectMessage("User with id " + USER_ID + " has not open any connection, please open a connection " +
        "before trying to save.");

    cloudService.saveObject(mock(Callable.class), USER_ID);
}
```
