---
layout: post
title: "Tasty Cookie Recipes"
date: 2013-02-15 19:33
comments: false
categories: [cookie, request, response, resource]
published: false
---
![Alt text](/images/posts/cookies.jpg)
##Intro
Today I have got in my inbox a request about a possible performance degradation due to cookies overhead. Well it is a long time that I do not deal with these nice and sweet pieces of code and probably I have never done something serious with them leaving some entity to take care about them. But I was quite interested in understanding why cookies can decrease the performance of the request/response round trip time (RTT) and how to overcome this issue. So here we are.
<!-- more -->

##The Issue
Every time a request is sent by the browser cookies are attached allowing the server to know that this request is connected to the previous one. But looking at the browser inspection console, there are a lot of requests for **static resources** such as images, CSS and Javascript. On static resources there is not any user interaction, but just the retrieval by a GET method. In this case **cookie overhead** is not necessary, they just increase the **size** of the request and then  request/response transaction **RTT**.

##A Bit of History
HTTP is a request/response protocol in the client-server computing model [[1]]. The client submit a request for a **resource** to the server which responds providing one of

* resource such as HTML page, an image, a CSS file and so on;
* message as response of an action it has performed.

The response contains completion status information and the requested content in its body if any. A **transaction** is a two-way communication between the client and the server, so it is a request/response roundtrip. An **HTTP session** is a sequence of transactions. A session has a **lifetime** and only **one connection** with the server shared by all the transactions.

HTTP is a **stateless protocol** [[2]] which means it treats each request a separated transaction, each pair request/response is independent from the previous and the next one. Stateless protocol means the server is not required to keep any **status** about among transactions in a session lifetime.

At the beginning of the web navigation a stateless protocol was enough, simple and fast, no need to clean and manage information about the status. Web applications have become more and more complex and keeping the status of the transactions in a session is mandatory. Some solutions have been adopted to keep the status:

* HTTP cookies.
* Query string parameters such as `/logout.action?session_id=uniqe_id`
* Hidden variables within the web forms

##HTTP Cookies
A cookies is simple a piece of data stored in the web browser designed to remember the state (memory of action done so far) of a web site as consequence of different activities the user has done in the past. For instance an e-commerce web site could store in the cookie the items the client has put in the basket or more sensible information making the validity of the cookies restricted to a specific session.

So there are different cookie flavors:

* **Session cookie.** Known also as transient or in-memory cookie, it exists in the temporary memory of the web browser, it is valid for a given user session and deleted when the browser is closed.
* **Persistent cookie.** Known as tracking cookie, it outlasts the user session and it is sent back to the server during each request till the validity period of the cookie which could also be 1 year.
* **Secure cookie.** Has secure attribute enable, used via HTTPS and it ensures the cookie is always encrypted when is sent to the server.
* **HTTPOnly cookie.** 

One of the main uses of the cookies is session management to maintain user data across transactions in the same session. The server generates and sends to the client the cookie containing a unique session identifier which will be sent back by the browser during all the subsequent transactions. The server then updates the state of the user data associated with this unique identifier keeping the browsing status across transaction in the session lifetime.

Therefore cookies can be generated and maintained by the server but also a script language such as javascript can do that. For instance, at the first request the browser sends

	GET /index.html HTTP/1.1
	Host: http://blackat.github.com/
	
and the server responds with

	HTTP/1.1 200 OK
	Content-type: text/html
	Set-Cookie: name1=value1
	Set-Cookie: name2=value2; Expires=Fri, 09-Jun-2017 07:22:19 GMT
	
where `Set-Cookie` is a browser **directive** to store the bowser. The next request will be

	GET /spec.html HTTP/1.1
	Host: http://blackat.github.com/
	Cookie: name=value; name2=value2
	Accept: */*

receiving this request, the server knows that this request is related to the previous one.

Keep in mind that a request brings a

* required resource (static, dynamic),
* method (GET, POST, etc) to be applied to the resource,
* cookies to identify the state of the navigation.

**Static resources** do not require any user interaction so, removing cookies, the request overhead is reduced and the request latency as well.

##Minimize the Request Overhead


##References
[1]: http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol
[2]: http://en.wikipedia.org/wiki/Stateless_protocol
