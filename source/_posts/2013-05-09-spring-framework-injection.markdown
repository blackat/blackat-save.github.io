---
layout: post
title: "Spring Framework Injection"
date: 2013-05-09 00:30
comments: false
categories: [spring, dependency injection]
published: false
---
Dependencies are injected from beans defined in the application context which can be configured in a deployment descriptor such as the `web.xml`.

* __Type of injection.__
    * _setter injection_
    * _field injection_
    * `@Autowired` annotation for method and fields means that dependencies are injected _by type_.
    * `@Resource` annotation for method and fields means that dependencies are injected _by name_.
* __Application context configuration.__
    * `web.xml`
        * `<context-param>` defines resource locations of XML configuration metadata.
    * `@ContextConfiguration`
