---
layout: post
title: "Get and Post"
date: 2013-03-08 19:33
comments: false
categories: spring. spring webflow, form, html
published: false
---
![Alt text](/images/posts/get_request.jpg)
##Intro
I am working on a sample Spring Web Flow application without using custom tags but just standard HTML ones
	<form method="post">
	    <input type="hidden" name="_flowExecutionKey" value="${flowExecutionKey}"/>

		<c:forEach items="${productList}" var="product">
		<p>
			<input type="radio" name="product" value="${product.productName}">
				<c:out value="${product.productName}"/>
			</input>
		</p>

	    <p><input type="submit" name="_eventId_addProduct" value="Add" /></p>
	    <p><input type="submit" name="_eventId_removeProduct" value="Remove" /></p>
	    <p><input type="submit" name="_eventId_checkout" value="Checkout" /></p>
	</form>
At the beginning I have forgotten to specify the `method="post"` and the `_eventId` was not sent into the body of the request but as a key-value pairs in the query string [[1]].
<!-- more -->

##HTTP Methods
Etymologically the word **verb** is used to describe an action, state or occurrence and forms the main part of the predicate of a sentence. HTTP defines **methods** or **verbs** to express the **action** to be executed on the resource identified. The **resource** represents either existing data or data generated dynamically according to the server implementation.

###Forms
Basically a `form` is a component of a Web page inside which are placed **controls**. Most of the controls are represented by the  `input` element (radio button, text field, checkbox etc.). Each part of a form is considered a paragraph [[4]] and it is a good practice to separate it by other parts using the `<p>` elements.
	
	<form>
		<p><label>First name: <input></label></p>
	</form>
	
Form submissions are exposed to servers in a variety of ways, most commonly as HTTP `GET` or `POST` requests [[3]].

###GET
_The GET method means retrieve whatever information (in the form of an entity) is identified by the Request-URI._ [[2]] 

It means that _with the HTTP GET method, the form data set is appended to the URI specified by the action attribute (with a question-mark ("?") as separator) and this new URI is sent to the processing agent._ [[6]]

HTML4 user agent conventions: _if the method is "get" and the action is an HTTP URI, the user agent takes the value of action, appends a `?' to it, then appends the form data set, encoded using the "application/x-www-form-urlencoded" content type. The user agent then traverses the link to this URI. In this scenario, form data are restricted to ASCII codes._ [[7]]

From the `HttpServlet.java` class
_The GET method should also be idempotent, meaning that it can be safely repeated. Sometimes making a method safe also makes it idempotent. For example, repeating queries is both safe and idempotent, but buying a product online or modifying data is neither safe nor idempotent._

###POST
_The POST method is used to request that the origin server accept the entity enclosed in the request as a new subordinate of the resource identified by the Request-URI in the Request-Line._ [[2]]  

So _with the HTTP POST method, the form data set is included in the body of the form and sent to the processing agent._ [[6]]

HTML4 user agent conventions: _if the method is "post" and the action is an HTTP URI, the user agent conducts an HTTP "post" transaction using the value of the action attribute and a message created according to the content type specified by the enctype attribute._ [[7]]

From the `HttpServlet.java` class
_This method does not need to be either safe or idempotent. Operations requested through POST can have side effects for which the user can be held accountable, for example, updating stored data or buying items online._

safe and idempotent methods

###Distinction matters

###Differences in form submission

###Server side processing 

<!-- References -->
[1]: http://en.wikipedia.org/wiki/Query_string
[2]: http://tools.ietf.org/html/rfc2616#section-9
[3]: http://www.w3.org/html/wg/drafts/html/master/forms.html#forms
[4]: http://www.w3.org/html/wg/drafts/html/master/dom.html#paragraph
[5]: http://www.w3.org/html/wg/drafts/html/master/grouping-content.html#the-p-element
[6]: http://www.w3.org/TR/REC-html40/interact/forms.html#h-17.13
[7]: http://www.w3.org/TR/REC-html40/interact/forms.html#form-data-set