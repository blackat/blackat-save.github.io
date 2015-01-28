---
layout: post
title: "Scaffolding a Web Application"
date: 2014-12-28 09:25:15 +0100
comments: true
categories:
published: false
---
## Node.js Basics

#### Module
A module is a file, one-to-one correspondence

#### require
Simple Node module loading system.


#### exports
exports: a special object used to add functions and objects to the root of the module. Variable local to the module are private as the module has been wrapped in a function. (describe the public private modifiers with the collection example)


#### module.exports
Useful to export a complete object in one assignment instead of building it one property at time.

The root of the module'exports is a function such as a constructor

### Best Practices
what about
- naming conventions of the variables
- use strict at the beginning of the file
- scaffolding
- json file where tu put semicolon, attached to the key name or not
