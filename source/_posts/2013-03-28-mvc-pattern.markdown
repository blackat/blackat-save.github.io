---
layout: post
title: "MVC Pattern"
date: 2013-03-28 22:42
comments: false
categories: [mvc, design pattern]
---
{% blockquote Bates and Sierra, http://shop.oreilly.com/product/9780596007126.do Head First Design Patterns %}
The MVC Pattern combines three patterns into a solution that separate the responsibilities in managing an interaction.
{% endblockquote %}
<!-- more -->

##Class diagram
{% img /images/posts/design-patterns/mvc_pattern.png %}

* __View__ present the model usually in graphic interface.
* __Controller__ takes user input, interpreting them to change the model, translates input into actions on the model.
* __Model__ holds all the data, state and application logic, sends notifications of state changes to the observers.

According to the interaction shown

1. The _user_ does something.
2. The _controller_ gets the input (action and/or parameters), interprets them and figures out how to change the state of the _model_ based on that action. The controller can also ask the view to change such as enable or disable certain buttons.
3. The _model_ handles all the application data and logic.
4. The _model_ notifies the _view_ that its state has changed.
5. The _view_ gets the state of the _model_ to update itself.

##Patterns

###Strategy Pattern
Implemented by the _view_ and the _controller_. It decouples the view from the model because the controller is responsible for interacting with the _model_ to carry out user requests. The view doesn't know how the request will be interpreted to change the model.

The view delegates to the controller to handle the user actions. The controller is the object that knows how to handle the user actions.

###Composite
The view is a composite of GUI components, the top level contains other components which contain components and so on until the leaf.

###Observer
The model is the _observable_ and the view is the _observer_ which registers with the model.