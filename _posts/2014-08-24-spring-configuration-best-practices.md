---
layout: post
title: Spring configuration - best practices
date: '2014-08-24T16:00:00.000+02:00'
author: Tomasz Krug
tags:
- Java
- configuration
- good practices
- Spring
modified_time: '2014-09-06T12:57:46.371+02:00'
blogger_id: tag:blogger.com,1999:blog-5263052209480207518.post-8705285142310717595
blogger_orig_url: http://www.castlecodestein.com/2014/08/spring-configuration-best-practices.html
---

In the previous post I showed you the basics of Spring dependency injection and configuration. In this episode we will take the next step and do it the best way possible.

Spring is so flexible many different approaches will produce the same result. Below I compiled a list of strategies I use to get most out of the framework while keeping things simple.

<!--more-->

#### Use Java config, not XML

Every option available in XML is available in Java config as well. However it is no longer the case the other way around with the coming of Spring Integration Java DSL. Java type safety makes configuring Spring a lot less error-prone plus IDE will help you navigate through bean definitions.

#### Scan normal components, define special beans

By normal components I mean repositories, services and controllers. The classes that are a part of your domain. Just mark them with a proper stereotype and an @Inject annotation. In vast majority of cases you don't need to define them directly as a bean. I haven't had to as of yet.

Configuration should be reserved for special beans customizing Spring and other libraries such as a view renderer (Spring MVC), Jackson object mapper, password encoder (Spring Security), data sources and entity manager factory (Spring ORM, Spring Data).

Place all ApplicationListener implementations in the configuration class as well. If you create a listener and mark it as a component it will still work but the application logic flow won't be so obvious to other developers.

#### Create a separate configuration class for each application aspect

Relatively small, cohesive modules are the way to go. Put all database related information in one place and mail related in the other. If using Spring Integration try not to put too many flows together.

#### Exclude configurations from component scanning, use @Import

```java
@ComponentScan(basePackages = { "your.package" }, excludeFilters = @Filter(type = FilterType.ANNOTATION, value = Configuration.class))
```

It's the same problem as with application listeners. When scanned it is not clear to coders not familiar with a project where do the beans come from exactly. You need to check every package and find them manually. By using @Import you define clear dependencies between configuration modules as it is done in case of normal components.

#### Define an alias for every bean profile and every qualifier

Create a custom annotation meta annotated with @Profile("your profile name") or @Qualifier("") and use them instead. Otherwise you are bound to make an error when typing the profile's name. Java is a strongly typed language, let's not "stringify" it.

How to create a custom annotation recognized by Spring? Check this [blog post](/2014/08/24/spring-meta-annotations/).

#### Do not define beans in web context configuration class unless it is web specific

Keep it clean. Most of the beans should be placed in a normal configuration class. Exceptions include but are not limited to: a web view renderer, view resolver and possibly a JSON converter.

#### Summary

These guidelines do not cover all possible cases but will help you for sure. Do you know of any other? Do you use them yourself in your projects? Don't hesitate to share your knowledge in the comments below.