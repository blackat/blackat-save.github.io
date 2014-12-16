---
layout: post
title: "Singleton Pattern"
date: 2013-03-14 20:58
comments: false
categories: [design patterns, singleton, java]
---
{% blockquote Bates and Sierra, http://shop.oreilly.com/product/9780596007126.do Head First Design Patterns %}
The Singleton Pattern ensures a class has only one instance and provides a global point of access to it.
{% endblockquote %}
<!-- more -->

##Class Diagram
{% img /images/posts/design-patterns/singleton.jpg %} 

`Singleton` is name of the class but also the type of the unique `instance` variable. The variable is `static` so it belongs to the class. The method `getInstance()` is `static` so can be access using `Singleton.getInstance()`.

##Key Points
* __Private Constructor.__ The class cannot be instantiated.

* __Core.__ The `Singleton` class
	* manages the single instance,
	* prevent other class from creating their instances,
	* provide a _global access_ point to the instance,
	* can have its own data and methods.
	
* __Lazy Initialization.__ The instance is created only the first time the method `getInstance()` is invoked. When the class is loaded by the class loader the instance is not yet created. Beneficial in case of _resource intensive objects_ such as database connection.

* __Multiple Class Loaders.__ Each class loader defines a _namespace_ so the same class is loaded multiple times. A custom class loader could be define to address the issue.

* __No Subclassing.__ Due to the `private` constructor the class cannot be _instantiated_ and _subclassed_. If the constructor will be changed to public, the parent and the subclasses will share the _same instance variable_. A `static` variable means it belongs to the class.

* __Garbage Collector 1.2__ A curious issue happens in its implementation because the GC collected the singleton without a global reference to them. The _only reference_ to a singleton is the singleton itself.

##Implementation
{% codeblock lang:java %}
	public class Singleton {
		private static Singleton instance;

		private Singleton() { }

		public static Singleton getInstance() {
			if (instance == null) {
				instance = new Singleton();
			}
			return instance;
		}
		// other data and methods
	}
{% endcodeblock %}

##Multi-Threaded Application
The `Singleton` could be used in a multithreaded environment. Two threads could enter the `getInstance()` method at the same time. Ensure that each thread waits its turn before entering the method.

Different implementations are feasible in order to address the issue.

###Lazy Implementation
{% codeblock lang:java %}
	public class Singleton {
		private static Singleton instance;

		private Singleton() { }

		public static synchronized Singleton getInstance() {
			if (instance == null) {
				istance = new Singleton();
			} 
			return instance;
		}
		// other data and methods
	}
{% endcodeblock %}
__Note.__ Synchronizing the `getInstance()` method creates an important overhead for the application, performance can decrease by a factor of 100.

###Eager Implementation
{% codeblock lang:java %}
	public class Singleton {
		private static Singleton instance = new Singleton();

		private Singleton() {}

		public static Singleton getInstance() {
			return instance;
		}
		// other data and methods
	}
{% endcodeblock %}
__Note.__ The JVM will create an instance of Singleton when the class is loaded. Fine if the application always uses a Singleton instance. 

###Double-Checked Locking Implementation
{% codeblock lang:java %}
	public class Singleton {
		private volatile static Singleton instance;

		private Singleton() {}

		public static Singleton getInstance() {
			if (instance == null) {
				synchronized (Singleton.class) {
					if (instance == null) {
						instance = new Singleton();
					}
				}
			}
			return instance;
		}
		// other data and methods
	}
{% endcodeblock %}
__Note.__ Just the creation of the instance is `synchronized` because the issue comes from a possible concurrent creation and avoid the reduction of the performance. Synchronization happens _only_ the first time the issue has to be created. The `volatile` keyword

Prior Java5 an out-of-order write may allow the reference to be returned before the object was created. The `new` operator is not atomic.

##Drawbacks
* __Global state.__ Singletons introduce a _global state_ into a program 
	* anyone can access to the reference variable provided by Singleton ignoring the scope;
	* global state state are very difficult to test;
	* singleton user and singleton class become coupled together so really difficult to test
		* _solution:_ pass the singleton as parameter in the client's constructor so the singleton can be mocked
	* factory class or the client could enforce the singularity eliminating the global state
		* _issue:_ violation of the Single Responsibility Principle, the class should take care about its own task and the singularity

* __Dependencies.__ Singleton can be accessed anywhere thanks to the static method `getInstance()`, no need to pass the reference variable as parameter.
	* _issues:_ 
		* the signatures of the methods do not show their dependency anymore, the method could pull a singleton _"out of thin air"_ [[4]];
		* becomes more difficult to use and test the code, the developer should have a look at the inner code.
		
* __Test-driven and Agile Development.__ Have small tests covering most of the code
	* test must be run in any order
	* two test could modify a shared resource that is the singleton


##References
IBM - User Your Singletons Wisely [[1]]
Portland Pattern Repository - Pages about singleton [[2]]
Portland Pattern Repository - Singletons Are Evil [[3]]
Google - Why Singletons Are Controversial [[4]]
JavaWorld - Simply Singleton [[5]]
Interview questions about singleton [[6]]

[1]: http://www.ibm.com/developerworks/webservices/library/co-single/index.html
[2]: http://c2.com/cgi/wiki?search=Singleton
[3]: http://c2.com/cgi-bin/wiki?SingletonsAreEvil
[4]: https://code.google.com/p/google-singleton-detector/wiki/WhySingletonsAreControversial
[5]: https://code.google.com/p/google-singleton-detector/wiki/WhySingletonsAreControversial
[6]: http://javarevisited.blogspot.fr/2011/03/10-interview-questions-on-singleton.html