---
layout: post
title: What Spring Framework is for?
date: '2014-08-10T16:00:00.000+02:00'
author: Tomasz Krug
tags:
- Java
- Spring
modified_time: '2014-09-05T14:58:49.268+02:00'
blogger_id: tag:blogger.com,1999:blog-5263052209480207518.post-1048360971326607599
blogger_orig_url: http://www.castlecodestein.com/2014/08/what-spring-framework-is-for.html
---

The first time I have been developing an application with Spring Framework it was just an experiment. An experiment so successful Spring is now on top of my favourite 3rd party Java libraries. In this post series I will show you why it should be on your top list as well or at least why you should consider it a viable option.

Spring consists of multiple modules. I will iterate over the more important ones to give an overview.

<!--more-->

* Core - dependency injection container. The heart of Spring Framework. Almost all other modules depend on this one. Responsible for finding and registering services, controllers, repositories and other components as beans in the main application context. Context can be configured through XML or directly in Java configuration class. I highly recommend the latter despite the fact most of Spring documentation focuses on the XML way.
* ORM - you no longer need persistence.xml. Just configure the entity manager factory bean, specify packages to be scanned for entity mappings and you're good to go.
* Data - repository pattern support. Dependent on the ORM module. Create repository classes in a matter of minutes without all the boilerplate code. Extend the base repository interface, declare your own methods and Spring will take care of creating an implementation at runtime and registering it as a bean. You only need a method with name following the Spring Data convention or a @Query annotation attached. Big time saver.
* Web MVC - request-based web framework. Provides a very clean separation between the service layer, view model and the view itself. Can be used to implement RESTful web services and a traditional web client as well. You are not tied to any particular view renderer. There are many that can be used such as Thymeleaf, Rythm, Mustache and many more.
* Security - restrict access to your web resources in just a few lines. CSRF is supported out of the box as well as password hashing algorithms (BCrypt), login, logout and redirect on access attempt to a restricted page. LDAP, OAuth, Kerberos, almost everything you need.
* Integration - inter- and intra-JVM communication. If you want to use a messaging system in your application this is the module you need. All of the Enterprise Integration Patterns implemented in one library. It does not support as many external systems as Apache Camel does, however with the incoming Integration Java DSL it gives you type safety.
* Test - allows you to initialize the application context to perform integration tests without actually deploying the application on a server.
* Boot - to run your application from the command line or with an embedded Tomcat server.

These are the ones I'm using the most and each of them will be described in detail in its own post. However it doesn't mean there are no other modules that can be a good fit for your use case. You can see for yourself on the official [Spring page](http://spring.io/projects).

Each module's page links to an extensive reference documentation. Bookmarking it is highly recommended as you will be visiting it frequently. Unfortunately there are not many examples included and even if you can find some they are configured by XML instead of Java. Sometimes it's just easier to learn by fiddling with a working example and seeing it in action than to work your way through a whole documentation chapter.

In the future posts I will focus on a single module at a time to give you a detailed description and to convince you give it a try.