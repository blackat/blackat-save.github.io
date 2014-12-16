---
layout: post
title: "Java and Zeros"
date: 2013-02-17 11:54
comments: false
categories: java, puzzle, zero
published: false
---
![Alt text](/images/posts/zeros_and_ones.jpg)
##Intro
I am working on some random collection implementation just for the sake to do it and I have faced with an issue about zeros. Calling java.util.Random.nextInt(0) I get java.lang.IllegalArgumentException: n must be positive. Well the first think I though is that there is no sense to generate a random number when the upper bound is 0, returning 0 could be an alternative, but the message is **n must be positive** so 0 is a negative number in the Java ecosystem. After a quick research I have found that each language consider 0 in different way and there are negative and positive 0s. So here we are.
<!-- more -->

##Zeros in Programming languages