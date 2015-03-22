---
layout: post
title: "How to test 2: TDD a simple example"
date: 2015-03-21 12:37:14 +0100
comments: true
categories:
---
Test Driven Development (TDD) is a software development process based on the _test-first_ approach of [extreme programming](http://en.wikipedia.org/wiki/Extreme_programming). Roughly the idea is creating tests before any implementation and then start development guided by test, finally refactor. We will quickly explore the technique and its _rhythm_ to highlight advantages and disadvantages. __DRAFT 1__
<!-- more  -->

### Components and material
This tutorial is part of an experiment done to simply explain fundamental concepts of testing. In the [previous one](http://blackat.github.io/blog/2015/03/07/how-to-test-1-find-errors-to-make-the-application-works/) you can find concepts and jargon of the testing world along with some exercises which can help you to get in confidence with unit testing and mock objects. The code experiment used in this tutorial is described in the previous article and can be found on [GitHub](https://github.com/blackat/tutorial-howtotest-1-collectors), so please clone it to experiment or just follow some code snippet on this page.

## A bit of theory

## Learning by example
Imagine to write a new collector unit which can be plugged into the Cloud Service. At the beginning not all the details are clear and well defined, but with some imagination we can define some functionalities. Just a sort of rough idea and then start to apply the TDD steps one by one.

### First test

##### RED
So let's create the `FacebookCollectorTddTest.class` and implement the first
``` java
public class FacebookCollectorTddTest {

    @Test
    public void testAcceptGoldenPath() throws Exception {

        FacebookCollector collector = new FacebookCollector();
        assertTrue(collector.accept(new FacebookPost("Just a foo post"), 1L));
    }
}
```

the code won't compile because the classes still do not exist. You can ask your IDE to implement the classes for you. For instance in [IntelliJ IDEA](https://www.jetbrains.com/idea/) you can exploit the class creation functionality as the screenshot below
{% img center /images/posts/idea_class_creation.png %}

The class we are going to implement should implement the `Collector.java` interface
``` java
public class FacebookCollector implements Collector {
}
```

The `Collector` interface is already the result of some rough idea about the service, but can be improved or extended in orde to support new features or requirements.

Once the code compile, run the test and make it fails, __Red__. But wait a minute! The assertion does not have any message and when the test fails does not tell us anything interesting.

``` java
java.lang.AssertionError
	at org.junit.Assert.fail(Assert.java:86)
	at org.junit.Assert.assertTrue(Assert.java:41)
	at org.junit.Assert.assertTrue(Assert.java:52)
	at com.contrastofbeauty.tutorial.collectors.FacebookCollectorTddTest.testAcceptGoldenPath(FacebookCollectorTddTest.java:15)
```

Let's improve the assertion with a test message
``` java
@Test
public void testAcceptGoldenPath() throws Exception {

    FacebookCollector collector = new FacebookCollector();
    FacebookPost facebookPost = new FacebookPost("Just a foo post");
    assertTrue("Object " + facebookPost + " for user " + USER_ID + " has not been accepted.", collector.accept(facebookPost, USER_ID));
}
```

##### GREEN
Make the test passes with the minimum and simplest amount of code. As result the _first version_ of the constructor and the `accept` method could be
``` java
public class FacebookCollector implements Collector {

private Map<Long, List<FacebookPost>> facebookPostMap;

public FacebookCollector() {
    facebookPostMap = new HashMap<>();
}

@Override
public boolean accept(Object object, long userId) {

    if(facebookPostMap.get(userId) == null) {
        facebookPostMap.put(userId, new ArrayList<FacebookPost>());
    }

    facebookPostMap.get(userId).add((FacebookPost) object);

    return true;
}
```

Now re-running the test it passes. Comparing this way of implementing with the one seen to correct the class by testing you can notice we have some quite a good amount of time because of `NullPointerException`, they have been spotted immediately and corrected without allowing them to propagate.

##### REFACTOR
Refactor both test and the class to make them more readable such as the `setUp()` method, the assertion message and the constant
``` java
public class FacebookCollectorTddTest {

    private static final long USER_ID = 1L;

    private FacebookCollector collector;

    private FacebookPost facebookPost;

    @Before
    public void setUp() {
        collector = new FacebookCollector();
        facebookPost = new FacebookPost("Just a foo post");
    }

    @Test
    public void testAcceptGoldenPath() throws Exception {
        String message = "Object " + facebookPost + " for user " + USER_ID + " has not been accepted.";
        assertTrue(message, collector.accept(facebookPost, USER_ID));
    }
}
```

***

### Second test

##### RED
The test is marked as _GoldenPath_ in order to highlight it is the most common case, where everything should run smooth. But what happens at the borders? Our implementation always return true except when an exception happens. So let's implement another test.
``` java
@Test
public void testAcceptWrongObjectType() throws Exception {
    Integer fooObject = new Integer(123);
    String message = "Object " + fooObject + " for user " + USER_ID + " must not be accepted.";
    assertFalse(message, collector.accept(fooObject, USER_ID));
}
```

The test will fail because of a `java.lang.ClassCastException`, the declaration of the list `List<FacebookPost>` prevents some wrong addition to the list.


##### GREEN
It is enough doing a check on the type of object to make the test passes
``` java
@Override
public boolean accept(Object object, long userId) {

    if (object instanceof FacebookPost) {
        if (facebookPostMap.get(userId) == null) {
            facebookPostMap.put(userId, new ArrayList<FacebookPost>());
        }

        facebookPostMap.get(userId).add((FacebookPost) object);

            return true;
    }

    return false;
}
```

##### REFACTOR
Test and class should be fine.

***

### Third test
Roughly a collector starts to collect posts from a user, when a certain number of posts is reached, the collector calls the method `flush()` to submit the job.

##### RED
Let's write a failing test
``` java
@Test
public void testAcceptCallFlushMethod() throws Exception {

    collector.setNewBufferSize(2);
    collector.setCallbackFunction(callbackMock);

    collector.accept(facebookPost, USER_ID);
    collector.accept(facebookPost, USER_ID);

    verify(callbackMock, times(1)).addTask(any(Callable.class), anyInt());
}
```

The test sets the _buffer size_ equal to two so it is enough adding two posts to have the `flush` method called. By using Mockito it is possible to verify if a certain method has been called and how many times. This is _behavior verification_ and happens on _indirect outputs_.

##### GREEN
A solution for the method could be
``` java
@Override
public boolean accept(Object object, long userId) {

    if (object instanceof FacebookPost) {
            if (facebookPostMap.get(userId) == null) {
                facebookPostMap.put(userId, new ArrayList<FacebookPost>());
            }

        facebookPostMap.get(userId).add((FacebookPost) object);

        if (newBufferSize != 0) {

            if (facebookPostMap.get(userId).size() == newBufferSize) {
                flush(userId);
            }

        } else {
            if (facebookPostMap.get(userId).size() == BUFFER_SIZE) {
                flush(userId);
            }
        }

        return true;
    }

    return false;
}
```

 To make the test work we have to implement other three methods such as `flush()`, `setNewBufferSize()` and setCallbackFunction(). SO it is possible to temporary move to other tests and then come back or just provide a fake implementation to produce the call of the method we want to verify. It depends a lot on experience, confidence with the task and how strict we want to adhere to TDD approach. Two methods are setters and they should not be tested, the third one could be temporary implemented as

``` java
@Override
public void flush(long userId) {

    callbackFunction.addTask(new Callable() {
        @Override
        public Object call() throws Exception {
            return null;
        }
    }, userId);
}
```

The test passes.

##### REFACTOR
Improve a bit the `accept()` method to make it more readable and compact. One solution could be
``` java
@Override
public boolean accept(Object object, long userId) {

    if (object instanceof FacebookPost) {
         if (facebookPostMap.get(userId) == null) {
             facebookPostMap.put(userId, new ArrayList<FacebookPost>());
         }

         facebookPostMap.get(userId).add((FacebookPost) object);

         int bufferSize = newBufferSize != 0 ? newBufferSize : BUFFER_SIZE;

         if (facebookPostMap.get(userId).size() == bufferSize) {
             flush(userId);
         }

         return true;
    }

    return false;
}
```

***

### Fourth test
It's time to test the `flush()` method. Write a unit test, make it fails and re-implement it in a more appropriate way than what done before.
When the `flush()` method is called, it creates a task and invokes a method against a callback function, the list for a given user is cleared.

##### RED
The failing test
``` java
@Test
public void testFlushGoldenPath() throws Exception {

    collector.setCallbackFunction(callbackMock);

    collector.accept(facebookPost, USER_ID);
    collector.accept(facebookPost, USER_ID);

    String message1 = "The list size for the user " + USER_ID + " must be equal to 2";
    assertEquals(message1, 2, collector.getListSizeByUserId(USER_ID));

    collector.flush(USER_ID);
    String message2 = "The list size after a call to flush() must be 0";
    assertEquals(message2, 0, collector.getListSizeByUserId(USER_ID));

    verify(callbackMock, times(1)).addTask(any(Callable.class), anyInt());
}
```

We test

    - posts have been correctly added
    - after the call to the `flush()` method the list is empty
    - the method `addTask()` from callback is invoked

##### GREEN
Solution
``` java
@Override
public void flush(long userId) {

    List<FacebookPost> list = new ArrayList<>(facebookPostMap.get(userId));
    facebookPostMap.get(userId).clear();

    FacebookTask facebookTask = new FacebookTask(list);

    callbackFunction.addTask(facebookTask, userId);
}
```

The test run fine.

##### REFACTOR
The class seems fine.

***

### Fifth test
Add exception in case the callback function is not set. This is a borderline test
