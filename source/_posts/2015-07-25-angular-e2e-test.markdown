---
layout: post
title: "angular e2e test"
date: 2015-07-25 16:42:43 +0200
comments: true
categories:
published: false
---
An end-to-end (e2e) test aims to test an application from the user perspective, so how to automate the test of the application logic on the full system using the user interface via a web browser, for instance, to check if the application functionalities satisfy the specifications. __[draft 2]__

<!-- more -->

## Ingredients
Following applications will be used to setup the environment and run e2e tests.

- [Node.js][nodejs], an event-driven framework for server-side and networking applications.
- [Protractor][protractor] developed by AngularJS team, running on Node.js, run e2e tests in a real browser using a WebDriver.
- WebDriver such as [Selenium][selenium] to control the browser and simulate user actions.
- [Google Chrome][chrome] browser.
- [Karma][karma] test runner.

### Install vs. Grunt
In this tutorial the Grunt approach has been chosen instead of installing Protractor and Selenium WebDriver globally and run them. The advantage is to have everything easier to setup and run, but paying the performance lost of starting a server each time.

Grunt can define and automate task running such as run the application we want to test against a server, run e2e test and then shutdown the server. Everything is described in the `Gruntfile.js` and can be run by a continuous integration server.

## Project Scaffolding

    ├── Gruntfile.js
    ├── LICENSE
    ├── README.md
    ├── css
    │   └── ui-navbar.css
    ├── demo
    │   ├── app.js
    │   ├── index.html
    │   ├── package.json
    │   ├── partials
    │   │   ├── home.html
    │   │   ├── state1.html
    │   │   └── state2.html
    │   └── server.js
    ├── dist
    │   ├── css
    │   │   ├── ui-navbar.css
    │   │   └── ui-navbar.min.css
    │   └── js
    │       ├── ui-navbar.js
    │       └── ui-navbar.min.js
    ├── e2e
    │   ├── navbar.po.js
    │   └── navbar.spec.js
    ├── package.json
    ├── src
    │   └── navbar.js
    └── template
        ├── navbar-li.html
        ├── navbar-li.html.js
        ├── navbar-ul.html
        └── navbar-ul.html.js

The name of the folders is self-explanatory, the focus is on the `e2e` folder where acceptance tests are created. Each test file ends with `.spec.js` to identify a __behavior specification__, simply __spec__,  that should be tested.

## Protractor, Jasmine, Karma and... what?
There some ingredients in the recipe, but how are they mixed up? and how do they interact?

1. Each `.spec.js` file written in _Jasmine_ defines a __test suite__ which is a set of __behavior specifications__, Behaviour Driven Development or [BDD][bdd]. BDD is a way to use natural language to define behaviors that should be tested. In BDD a test [_"it’s a sentence describing the next behaviour in which you are interested."_][bdd]
2. [Jasmine][jasmine] is a BDD testing framework, it comes with a web-based test runner to continuously execute test during the development phase. _Behaviors_ are described in _specs_ which are grouped by _spec suites_, for instance:
`describe` function introduces a _spec suite_ defined by the code block of the function. The Jasmine function `it` defines a _spec_. Jasmine can be used to write both unit tests and e2e tests.

        describe("A suite is just a function", function() {
          var a;

          it("and so is a spec", function() {
            a = true;

            expect(a).toBe(true);
          });
        });

3. [Protractor][protractor] is a `Node.js` application able to run e2e tests written in JavaScript. It uses a WebDriver such as Selenium to control the browser and simulate user interaction with the web application.
4. [Karma][karma] is a test runner to automate test calling based on `karma.conf.js` configuration file. Jasmine comes with `SpecRunner.html` which must load test files we want to run and additional libraries and frameworks such as AngularJS, jQuery and so on. Karma configuration is simpler, some plugins can be used to better monitor the test running and can run tests against multiple browsers. Moreover Karma can be invoked by a continuous integration server and via the `watch` feature runs tests whenever the application code changes.

## Karma configuration

## Write an e2e test
Before start to write an e2e, the concept of [page object][pageobject-wiki] has to be introduced because it will simplify the way to write, maintain and read e2e tests in the future.

### Simon says: "You are do it wrong"
{% blockquote Simon Stewart, http://blog.rocketpoweredjetpants.com/ PageObject %}
If you have WebDriver APIs in your test methods, You're Doing It Wrong.
{% endblockquote %}

``` javascript
void testLoginPage() {
  driver.findId("username").setText("foo-user");
  ...
}
```

In a e2e test method is better to hide the UI details and structure to have a simpler and more readable test, what happens to an e2e test if the __UI implementation changes__? What about [a programmatic API to drive and interrogate a UI][pageobject-selenium] in order to __expose the service__ you interact with and __not the implementation__.


### PageObject
An e2e test is a defined interaction of an agent simulating a human user with a web page. The interaction needs to refer to element in the html page to click and links and evaluate what is displayed. Writing a test that _directly_ interact and manipulate the web page means the test will be subject to change when the web page changes. A __page object__ wraps an html page providing an _"application-specific API"_ to interact and manipulate page elements without accessing them directly. The e2e test will use the API which decouples the the test code from the web page.

A web page could be really complex but _[the complexity shouldn't be revealed by the page objects][pageobject-fowler]_.

{% blockquote Martin Fowler, http://martinfowler.com/bliki/PageObject.html PageObject %}
The rule of thumb is to model the structure in the page that makes sense to the user of the application.
{% endblockquote %}

What should be a page objects finally? Is just a model or something more?

{% blockquote Martin Fowler, http://martinfowler.com/bliki/PageObject.html PageObject %}
Page objects are commonly used for testing, but should not make assertions themselves. Their responsibility is to provide access to the state of the underlying page. It's up to test clients to carry out the assertion logic.
{% endblockquote %}

The great advantage of a page object is to encapsulate a part of the UI along with its structure complexity, so a page object hides details of the UI from the tests, so:

1. the logic that manipulates the UI is just in one place and can be modified or evolved without touching the test code;
2. the tests are more readable, no additional and complex code to make access to UI elements, UI details are simply hidden at test level;
3. if the UI changes the [fix has to be applied in only one place][pageobject-wiki];
4. e2e test writer interacts with a __service__ offered by a __particular part__ of a web page, _an test should test the functionality not its implementation_;

An e2e test turns out to be simpler, easier to be maintained and a real source of documentation providing only the code necessary to test the expected behavior as when a human user interacts with the web page.

[nodejs]: http://nodejs.org ""
[protractor]: https://github.com/angular/protractor ""
[protractor-browser]: https://github.com/angular/protractor/blob/master/docs/browser-setup.md ""
[selenium]: https://code.google.com/p/selenium/wiki/GettingStarted ""
[chrome]: www.google.co.uk/chrome ""
[grunt]: http://gruntjs.com ""
[karma]: http://karma-runner.github.io/0.13/index.html
[bdd]: http://dannorth.net/introducing-bdd/
[jasmine]: http://jasmine.github.io
[pageobject-fowler]: http://martinfowler.com/bliki/PageObject.html
[pageobject-wiki]: https://code.google.com/p/selenium/wiki/PageObjects
[pageobject-selenium]: https://docs.google.com/presentation/d/1682BnhVsmBudDkHWN_Bj_WAZ2iEJTwLwkHOnpYyP1Ac/edit?pli=1#slide=id.g339080ef4_028
