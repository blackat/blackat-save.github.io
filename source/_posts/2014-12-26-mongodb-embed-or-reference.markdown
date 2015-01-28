---
layout: post
title: "MongoDB: Embed or Reference"
date: 2014-12-26 08:54:11 +0100
comments: true
categories: [NoSQL, MongoDB]
published: false
---
embed means pre-joined and is characterized by
pros: fast performance (data are store sequentially, only one seek and then just a sequential read)
cons: potentially too many data are retrieved, no flexibility in queries, order and restrict do not work

when embed: fixed query pattern, small documents (less than 16MB)


reference:
pros:
cons:


##No JOINS so how to?
Using an ODM such as Mongoose
http://mongoosejs.com/docs/populate.html

In MongoDB how?

Mongoose
