---
layout: post
title: "Kii Web Application Sign In With Twitter"
date: 2015-02-13 18:54:58 +0100
comments: true
categories:
published: false
---
introduction
The only way to authenticate a user is via a Twitter application because Twitter does not anymore support basic authentication protocol, @Anywhere has been deprecated.

<!-- read more -->

## Browser sign in flow

## OAuth 1.0a at glance

## Implementing sign in with Twitter

#### Step 1: obtaining a request token
To start a _sign in_ flow the application has to obtain a __request token__ as described in [1].

https://api.twitter.com/oauth/request_token

authentication header








[1]: https://dev.twitter.com/web/sign-in/implementing "Implementing Sign in with Twitter"
[2]: https://developers.facebook.com/docs/facebook-login/login-flow-for-web/v2.2 "Facebook Login for the Web with the JavaScript SDK"
[3]: https://dev.twitter.com/web/sign-in/desktop-browser "Browser sign in flow"
[4]: https://dev.twitter.com/overview/api/twitter-libraries "Twitter Libraries"
[5]: http://devcenter.kinvey.com/html5/tutorials/how-to-implement-safe-signin-via-oauth "How to Implement Safe Sign-In via OAuth"
[6]: https://twittercommunity.com/t/how-do-i-implemente-sign-in-with-twitter-from-javascript-now-that-anywhere-will-be-deprecated-soon/1180/5 "Sign in implementation in Javascript from twitter blog"
[7]: https://dev.twitter.com/oauth/reference/post/oauth/request_token POST oauth/request_token
