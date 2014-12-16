---
layout: post
title: "Request and Response Dissected"
date: 2013-03-10 11:35
comments: true
categories: 
published: false
---
##Intro
Debugging a Web application just put a breakpoint on the handler method and see the content of the request and response objects, but who has created them?

##Request and Response
Usually a Web application works as a sequence of request and response between a client and a server. The client sends a request with some data to server. The client can request a data such as an image or it can request an action over some data present in the query string or in the body of the request.

In the first case the server sends back the resource, in the second case the server execute an action answering with a response.

<!-- some example can help to understand different scenario -->

##Web Server
A web server is an application listening on a specific port implementing some standard contracts enabling it to get request from a user agent and dialog with a backend to serve the request.

Consider Jetty[1] as a web server, it is J2EE compliant implementing specific interfaces. According to the specifications (**citation**) a web application must have the following elements:
* web.xml: web deployment descriptor, in xml format where an handler method (servlet) is described along with the path it is responsible for.
* HttpServlet implementation: implementing this abstract class and overriding methods such as doGet and doPost the developer gives the methods to be invoked when the user agent will send a request on a matching path.
As shown in the example [[2]], the methods are responsible of executing some business code and producing a response to the user agent.

##How it works
Basically hitting a link produces the user agent to create a request compliant to the HTTP protocol. The request is sent over the TCP/IP protocol to the server which is listening on a specific port.

In Jetty there is class names HttpConnection which does a lot of work to make things ready for the business logic:
* reads from a raw TCP/IP socket,
* parses Http headers of the request
* create request and response objects

Another interesting point to have a look at is the `Server.handle` method where some actions happen just after the incoming stream has been parsed.

Basically the actions are:
* Jetty starts reading its configuration file and deploying all the Web applications present inside of the `webapp` folder.
* Deployment process involves reading the web deployment descriptor, that is the `web.xml` file of each one of the Web applications and registering each handler or servlet to serve a request coming from a specific path.
* The user agent sends a request over the TCP/IP in the format specified by the HTTP protocol.
* Jetty, listening on a specific port, gets the request, reads the data on the TCP/IP socket and creates the `request` and `response` objects compliant to J2EE specifications.
* Jetty will call the `init()` method of the specific servlet class and then the `doGet()` or `doPost()` method passing the request and response objects it has just created.

[1]: http://wiki.eclipse.org/Jetty
[2]: http://www.seas.upenn.edu/~cis550/jetty.html