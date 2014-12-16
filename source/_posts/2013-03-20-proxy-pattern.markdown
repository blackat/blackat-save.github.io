---
layout: post
title: "Proxy Pattern"
date: 2013-03-20 23:00
comments: false
categories: [design patterns, proxy, java]
---
{% blockquote Bates and Sierra, http://shop.oreilly.com/product/9780596007126.do Head First Design Patterns %}
The Proxy Pattern provides a surrogate or placeholder for another object to control access to it.
{% endblockquote %}
<!-- more -->

##Class Diagram
{% img /images/posts/design-patterns/proxy.jpg %}

`Subject` interface is implemented by both `RealSubject` and `Proxy`. So they are _interchangeable_ and it allows the _client_ to treat the `Proxy` as `RealSubject`.

`RealSubject` does the _real work_ and the `Proxy` <u>_controls the access_</u> to it.

`Proxy` handles the _creation_ of the `RealSubject` keeping a _reference_ to the `Subject` in order to be able to forward the request coming from the client to the real implementation.

Proxy can controls the access to the _real object_ in different ways

* [__Remote proxy.__](/blog/2013/03/22/remote-proxy-pattern)
	* It <u>controls the access</u> to a _remote object_.
	* The proxy acts as a _local representative_ for an object that _lives on different JVM_.
	* _The object actually lives somewhere else on the network._
* [__Virtual proxy.__](/blog/2013/03/22/virtual-proxy-pattern)
	* It <u>controls the access</u> to a _resource_ that is expensive to create.
	* The proxy acts as a _representative_ for an object that may be expensive to create. 
	* It _defers the creation_ of the object until it is needed. 
	* Virtual proxy acts as a _surrogate_ for the object _before and while_ it is being created.
	* Object is expensive to create because it has to be retrieved from a database over the network.
* [__Protection proxy.__](/blog/2013/03/22/protection-proxy-pattern)
	* It <u>controls the access</u> to a _resource_ based on access rights.
* __Caching proxy.__ Similar to a virtual proxy but it _caches_ the expensive objects it creates to reduce the latency of the forwarded request.

##Key Points
* __Proxy pattern__
	* provides a surrogate or place holder for another object,
	* creates a representative object that <u>_controls access_</u> to another object which can be _remote_, _expensive to create_ or _need secure access_ to have its methods used.
* __Wrapper__
	* In some way the proxy _wraps_ (`HAS-A`) the real object.
	* Proxy _"intercepts"_ the requests from the client and forwards them to the real object acting a some form of control on them.
	* If the real object is remote, there is a sort of _"remote proxy"_ receiving the request or call over the network invoking the call of the method against the real object it owns.
* __Factory method__ 	Using a _factory method_ it is possible to _wrap_ the _real object_ into the proxy and return it to the client as it was the real one.
	

##Comparisons
All proxies have in common the ability to intercept a method call that a client has done. The client always invoke method on a _proxy_ thinking that it is the _real object_. _Proxies always acts as surrogates._

###Proxy vs. Decorator and Adapter
* __Proxy__
	* It always _control the access_ to a _class_. 
	* Proxy decouples the client from the real object. Controlling the access, the proxy allow the client the use of the real object only when it is available, so the client doesn't have to wait for it.
	* The protection proxy may provides a partial interface to a real object, in this it is similar to an adapter.
	* It __wraps the subject__. The client doesn't know what object has been wrapped. Virtual proxy _wraps_ an object that even doesn't exist at the beginning.
* __Decorator__ 
	* It always _adds behavior_ to a _class_.
	* It wraps an __object__.
	* It _never_ instantiates anything.
* __Adapter__ 
	* It forwards the request from the client to another object, but it _changes the interface_ of the objects it adapts. 
	* It is _different_ from the _proxy_ which always implement the _same interface_ of the real object.
