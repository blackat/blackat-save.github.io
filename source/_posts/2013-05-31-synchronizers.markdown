---
layout: post
title: "Synchronizers"
date: 2013-05-31 05:54
comments: false
categories: [java, multi threading, synchronizer]
---
Synchonizers are high-level abstraction in order to synchonize activities of two or more threads.

The group of synchronizers includes:

* __[Semaphore.](#semaphore)__ Controls access to one ore more shared resources.
* __[Phaser.](#phaser)__ Used to support a synchronization barrier.
* __[CountDownLatch.](#countdownlatch)__ Allows threads to wait for a countdown to complete.
* __[Exchanger.](#exchanger)__ Supports exchanging data between two threads.
* __[CyclicBarrier.](#cyclicbarrier)__ Enables threads to wait at a predefined execution point.

<!-- more -->

##Modeling Approach
In the following examples a scenario is taken from reality modeled using threads and synchonizers. In general, each participant to the real world scenario is modeled as a thread and all of them share a resource.

The common and recurring pattern is: a group of threads and their coordination to access a shared resource or a limited and shared number of resources. A resource could be seen as the _synchronizer_.

It is possible to describe entities in the following way:

* _synchonizer or resource_ is something someone want to have access to, or is waiting for,
* _thread_ is a participant trying to use, conquer, access a resource.

In all the scenario a _challenge_ can be identified and can help to understand how to model the piece of the reality by using threads and synchronizers.

##<a id="semaphore">Semaphore</a>

###Scenario
It is the classical one where many people want to get cash from a small number of cash machines.

Basically each person is modeled as a thread because is the participant to a challenge to use the cash machine which is the shared resource. Being the shared resource implies that the cash machine is the synchronizer, the _semaphore_.

###Code
To model a person as a thread just create a class extending `java.lang.Thread` class or implementing `java.lang.Runnable` interface.

The use of the semaphore is extremely simple, a thread tries to _acquire_ the control or _lock_ on the object calling `cashMachine.acquire()`. The thread starts in `runnable` state and then is put in `wait` state waiting to acquire the semaphore.

Once the semaphore will be acquire, the thread will simulate the cash withdrawal step and then it will release the resource so other people can use the cash machine.
{% codeblock lang:java %}
package com.contrastofbeauty.tuts.concurrency.synchronizers.semaphore;

import java.util.concurrent.Semaphore;

public class Person implements Runnable {

    public static final int WITHDRAWAL_TIME = 1000;

    private Semaphore cashMachine;
    private String name;

    public Person(final Semaphore machine, final String personName) {
        cashMachine = machine;
        name = personName;
    }

    @Override
    public final void run() {
        System.out.println(Thread.currentThread().getName() + " is waiting for the cash machine.");

        try {
            cashMachine.acquire();
            System.out.println(Thread.currentThread().getName() + " is using the cash machine.");

            Thread.sleep(WITHDRAWAL_TIME); // simulate the user withdrawal

            System.out.println(Thread.currentThread().getName() + " has done with the cash machine.");
            cashMachine.release();
        } catch (InterruptedException e) {
            System.err.println(e);
        }
    }
}
{% endcodeblock %}

A `java.util.concurrent.Semaphore` instance is used to simulate three cash machines (the counter starts from 0). Six people/threads are created to model six possible concurrent access to the three resources.

The semaphore models the access to a pool of cash machines or resources and it is the mean to get access to.

__Remember__ to start each thread invoking `start()` method over each new created thread.
{% codeblock lang:java %}
package com.contrastofbeauty.tuts.concurrency.synchronizers.semaphore;

import java.util.concurrent.Semaphore;

public final class ClientExecutor {

    private ClientExecutor() {
    }

    public static void main(final String[] args) {

        // semaphore control the access to the 3 available cash machines
        Semaphore cashmachines = new Semaphore(2);

        new Thread(new Person(cashmachines, "Einstein"), "Einstein").start();
        new Thread(new Person(cashmachines, "Fermi"), "Fermi").start();
        new Thread(new Person(cashmachines, "oppenheimer"), "oppenheimer").start();
        new Thread(new Person(cashmachines, "Majorana"), "Majorana").start();
        new Thread(new Person(cashmachines, "Turing"), "Turing").start();
        new Thread(new Person(cashmachines, "von Neumann"), "von Neumann").start();
    }
}
{% endcodeblock %}

##<a id="phaser">Phaser</a>

###Scenario

###Code
{% codeblock lang:java %}
package com.contrastofbeauty.tuts.concurrency.synchronizers.phaser;

import java.util.concurrent.Phaser;

public class AssemblerRobot implements Runnable {

    private Phaser workPhaser;

    private String name;

    public AssemblerRobot(final Phaser phaser, final String robotName) {
        workPhaser = phaser;
        name = robotName;
        // the party register itself to synchronize
        workPhaser.register();
    }

    @Override
    public void run() {
        //To change body of implemented methods use File | Settings | File Templates.
    }
}
{% endcodeblock %}

{% codeblock lang:java %}
package com.contrastofbeauty.tuts.concurrency.synchronizers.phaser;

import java.util.concurrent.Phaser;

public final class ClientExecutor {

    private ClientExecutor() {

    }

    public static void main(final String args) {

        // only one party is register, other parties can further register themselves
        Phaser assembler = new Phaser(1);

        new Thread(new AssemblerRobot(assembler, "External assembler")).start();

        new Thread(new AssemblerRobot(assembler, "Interior assembler")).start();

        new Thread(new AssemblerRobot(assembler, "Wheel assembler")).start();
    }
}

{% endcodeblock %}

##<a id="countdownlatch">CountDownLatch</a>

###Scenario

###Code
{% codeblock lang:java %}
package com.contrastofbeauty.tuts.concurrency.synchronizers.countdownlatch;

import java.util.concurrent.CountDownLatch;

public class Runner implements Runnable {

    private CountDownLatch timer;

    public Runner(final CountDownLatch counter) {
        timer = counter;
    }

    @Override
    public final void run() {
        System.out.println(Thread.currentThread().getName() + " waiting to run.");
        try {
            // the thread waits for the timer to reach 0 and be released
            timer.await();

        } catch (InterruptedException e) {
            System.err.println(e);
        }
        System.out.println(Thread.currentThread().getName() + " started to run.");
    }
}
{% endcodeblock %}

{% codeblock lang:java %}
package com.contrastofbeauty.tuts.concurrency.synchronizers.countdownlatch;

import java.util.concurrent.CountDownLatch;

public final class ClientExecutor {

    public static final int COUNTDOWN_SECONDS = 10;

    public static final int SECONDS_TO_WAIT = 1000;

    private ClientExecutor() {
    }

    public static void main(final String[] args) {

        CountDownLatch timer = new CountDownLatch(COUNTDOWN_SECONDS);

        // all the runners each one represented by one thread
        new Thread(new Runner(timer), "Pietro Paolo Mennea").start();
        new Thread(new Runner(timer), "Sara Simeoni").start();
        new Thread(new Runner(timer), "Luigi Beccali").start();
        new Thread(new Runner(timer), "Adolfo Consolini").start();
        new Thread(new Runner(timer), "Maurizio Damilano").start();

        System.out.println("Timer started");
        Long count = timer.getCount();

        while (count > 0) {
            try {
                Thread.sleep(SECONDS_TO_WAIT); // simulate the passing of 1 sec.
                System.out.print(count + " ");

                if (count == 1) {
                    System.out.println("GO!");
                }
                timer.countDown(); // decrement the timer of 1 unit
                count = timer.getCount();
            } catch (InterruptedException e) {
                System.err.println(e);
            }
        }
    }
}
{% endcodeblock %}

##<a id="exchanger">Exchanger</a>

###Scenario

###Code
{% codeblock lang:java %}
package com.contrastofbeauty.tuts.concurrency.synchronizers.exchanger;

import java.util.concurrent.Exchanger;

public class Friend implements Runnable {

    private Exchanger<String> presents;

    private String name;

    private String present;

    public Friend(final Exchanger<String> exchanger, final String friendName, final String presentToGive) {
        presents = exchanger;
        name = friendName;
        present = presentToGive;
    }

    @Override
    public final void run() {

        String received;

        System.out.println(name + " give as a present " + present);
        try {
            received = presents.exchange(present);
            System.out.println(name + " get as a present " + received);
        } catch (InterruptedException e) {
            System.err.println(e);
        }
    }
}
{% endcodeblock %}

{% codeblock lang:java %}
package com.contrastofbeauty.tuts.concurrency.synchronizers.exchanger;

import java.util.concurrent.Exchanger;

public final class ClientExecutor {

    private ClientExecutor() {
    }

    public static void main(final String[] args) {

        // object used to exchange presents between two friends
        Exchanger<String> presents = new Exchanger<String>();

        new Thread(new Friend(presents, "Alessandro Delpiero", "De Bello Gallico, di Giulio Cesare")).start();
        new Thread(new Friend(presents, "Michel Platini", "Fabulae, di Fedro")).start();
    }
}
{% endcodeblock %}

##<a id="cyclicbarrier">CyclicBarrier</a>

###Scenario

###Code
{% codeblock lang:java %}
package com.contrastofbeauty.tuts.concurrency.synchronizers.cyclicbarrier;

public class MeetingRoom extends Thread {

    @Override
    public final void run() {
        System.out.println("All the participants have arrived at the meeting room.");
    }
}
{% endcodeblock %}

{% codeblock lang:java %}
package com.contrastofbeauty.tuts.concurrency.synchronizers.cyclicbarrier;

import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class Participant implements Runnable {

    private CyclicBarrier meetingPoint;

    private String name;

    public Participant(final CyclicBarrier barrier, final String partecipantName) {
        meetingPoint = barrier;
        name = partecipantName;
    }

    @Override
    public final void run() {
        System.out.println(name + " arrived at the meeting point.");
        try {
            meetingPoint.await();
        } catch (InterruptedException e) {
            System.err.println(e);
        } catch (BrokenBarrierException e) {
            System.err.println(e);
        }
    }
}
{% endcodeblock %}

{% codeblock lang:java %}
package com.contrastofbeauty.tuts.concurrency.synchronizers.cyclicbarrier;

import java.util.concurrent.CyclicBarrier;

public final class ClientExecutor {

    private ClientExecutor() {
    }

    public static void main(final String[] args) {

        /**
         * Manage the meeting point, represented by a thread, for all the
         * registered threads. When all the threads have reached the meeting
         * point, the run() method of the meeting point thread is executed.
         */
        CyclicBarrier barrier = new CyclicBarrier(5, new MeetingRoom());

        // participant to the meeting
        new Thread(new Participant(barrier, "Charlie Chaplin")).start();
        new Thread(new Participant(barrier, "Rodolfo Valentino")).start();
        new Thread(new Participant(barrier, "Buster Keaton")).start();
        new Thread(new Participant(barrier, "Roscoe Arbuckle")).start();
        new Thread(new Participant(barrier, "Max Linder")).start();
    }
}
{% endcodeblock %}
