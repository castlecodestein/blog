---
layout: post
title: Dependency injection in Spring
date: '2014-08-17T16:00:00.000+02:00'
author: Tomasz Krug
tags:
- Java
- dependency injection
- configuration
- Spring
modified_time: '2014-09-07T11:16:34.749+02:00'
blogger_id: tag:blogger.com,1999:blog-5263052209480207518.post-7284399924522907371
blogger_orig_url: http://www.castlecodestein.com/2014/08/dependency-injection-in-spring.html
---

At its core Spring is a dependency injection container. All other modules are dependent on it. In this post I will show you how to register beans in the application context and how to inject / autowire dependencies. The next one will focus on Spring configuration best practices.

<!--more-->

Concepts and strategies will be explained in the following order:

* autowiring
* direct bean reference
* component scan
* configuration import
* bean profiles
* bean qualifiers

There are two ways to configure Spring: by an XML file or through a Java configuration class. Here I will cover only the latter but you can easily find many XML examples on the internet. All of my code samples are available at [Castle Codestein repository](https://github.com/castlecodestein/spring-dependency-injection) bundled together with apropriate integration tests.

#### Autowiring

First you need to create a class annotated by @Configuration. Every bean definition inside has to be marked as a @Bean. Otherwise Spring won't register it in the context.

```java
@Configuration
public class SpringDirectBeanConfiguration {

    @Bean
    public PostRepository postRepository() {
      return new PostRepository();
    }

    @Bean
    public PostService postService(PostRepository repository) {
      return new PostService(repository);
    }

    @Bean
    public PostApi postApi(PostService service, PostRepository repository) {
      return new PostApi(repository, service);
    }

}
```

All dependencies specified as method parameters are resolved by the framework. When creating a bean it tries to find suitable candidates for each of the arguments. In case none or more than one can be found an exception is thrown.

Candidate beans are found based on 3 aspects: 
* bean / dependency class type (subclassing included)
* generic argument (if present)
* bean qualifier (if present)

PostRepository does not depend on any other class and it can be created right away. PostService needs a PostRepository instance to function properly so it retrieves one from the context. The same goes for the PostApi bean. Every bean declared in this way becomes a singleton inside the application context.

#### Direct bean reference

Instead of letting Spring resolve the dependencies one can provide them directly.

```java
@Configuration
public class SpringDirectBeanAltConfiguration {

  @Bean
  public PostRepository postRepository() {
    return new PostRepository();
  }

  @Bean
  public PostService postService() {
    return new PostService(this.postRepository());
  }

  @Bean
  public PostApi postApi() {
    return new PostApi(this.postRepository(), this.postService());
  }

}
```

The second example gives exactly the same result as the first one. This approach is very convenient when dealing with many beans of the same class type. No need to create a separate qualifier for each of them.

#### Component scan

In a bigger project defining beans directly would prove to be a tedious and error-prone task. We can simplify the configuration by using Spring's component scan. Component scan will search through the specified packages to find all classes marked by Spring annotations (@Component, @Configuration). Each configuration class will be resolved the same way as our parent one. Every component will be registered as a bean in the context.

```java
@Configuration
@ComponentScan(basePackages = { "your.package.name" })
public class SpringBeanScanConfiguration {

}
```

Configuration requires only a single annotation but how does a component class look like?

```java
@Component
public class PostService {

  private final PostRepository repository;

  @Inject
  public PostService(PostRepository repository) {
    this.repository = repository;
  }

  public void createPost(Post post) {
    this.repository.save(post);
  }

  public void deletePost(long id) {
    this.repository.delete(id);
  }

}
```

First the type itself must be annotated by @Component class. Secondly a constructor must be marked by @javax.inject.Inject or Spring's @Autowire. Spring conforms to JSR 330 so they both produce the same result. Dependencies can also be wired through setter methods but I'm not a fan of this approach. 

#### Import

Bean definitions can be separated into multiple configuration classes. What's more they can depend on each other just as other components do. To achieve this Spring introduced the @Import annotation. 

```java
@Configuration
@Import(value = { SpringSubmoduleConfiguration.class })
public class SpringTopAltConfiguration {

  @Inject
  private SpringSubmoduleConfiguration moduleConfiguration;

  @Bean
  public PostApi postApi() {
    return new PostApi(this.moduleConfiguration.postRepository(), this.moduleConfiguration.postService());
  }

}
```

Imported class will be resolved first and all beans scanned / defined by it will be available in the importing one for autowiring.

#### Bean profiles

Beans can be assigned to a profile. It's a useful mechanism when dealing with tests. Define a normal profile with a ConnectionFactory bean pointing at a standalone database and another testing profile with an embedded database. Your application will work when deployed on a server and when executing integration tests. To activate a profile use either a -Dspring.profiles.active=profileName1,profileName2 JVM parameter or by using an @ActiveProfiles annotation on a test class.

The "default" profile is built-in and it is active only when no other profile is explicitly activated.

```java
@Configuration
public class SpringProfileConfiguration {

  @Bean
  @Profile("default")
  public Integer defaultNumber() {
    return 0;
  }

  @Bean
  @Profile("production")
  public Integer luckyNumber() {
    return 24;
  }

  @Bean
  @Profile("dev")
  public Integer badLuckNumber() {
    return 11;
  }

}
```

#### Bean qualifiers

The last concept I'd like to describe in this post are bean qualifiers. When using profiles many beans of the same type can be present in one configuration file. However most of time only one version is active depending on the profile(s) selected. If you need many implementations of the same interface in the same profile then qualifiers are the answer.

```java
@Configuration
public class SpringQualifierConfiguration {

  @Bean
  @Qualifier("lucky")
  public Integer luckyNumber() {
    return 24;
  }

  @Bean
  @Qualifier("badLuck")
  public Integer badLuckNumber() {
    return 11;
  }

}
```

To use them each of required dependencies (autowired constructor/method parameters) must be annotated by a qualifier as well. If no qualifier is specified then in the example above Spring will find two candidate beans for an Integer dependency and will not know which one to choose. And that leads to an exception.

#### Summary

Want to learn more? Check out Spring's [documentation](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#beans).

These are just the basics. In the next post I will show how to create clean configuration and how to avoid headache caused by an error in it. 