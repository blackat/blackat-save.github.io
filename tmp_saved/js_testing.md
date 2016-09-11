- set up a basic specification
- load an AngularJS module
- instantiate a controller
- run a spec testing a value on scope

General structure

- injecting dependencies
- defining a context
- writing specs

- unit testing: test the functionality of a piece of code in isolation
- angular-mocks.js loads the ngMock module to inject and mock AngularJS services
- mock module function configure information ready for the Angular injector
- mock inject function creates and injectable wrapper around its argument, it creates a new instance of $injector for each test, $injector instance is used to resolve all the references for this test. 
