---
layout: post
title: Spring meta annotations
date: '2014-08-31T16:00:00.001+02:00'
author: Tomasz Krug
tags:
- Java
- meta annotations
- Spring
modified_time: '2014-09-05T16:26:53.848+02:00'
blogger_id: tag:blogger.com,1999:blog-5263052209480207518.post-1189633723523934597
blogger_orig_url: http://www.castlecodestein.com/2014/08/spring-meta-annotations.html
---

Spring Framework introduces meta annotations to simplify configuration. The concept can be easily explained as a substitute for the non-existent type hierarchy mechanism in Java annotations. While it is not backed by the language itself you still can emulate subtyping by writing an appropriate annotation parser and that's the way Spring has chosen.

<!--more-->

The first example comes right from spring-context library. In org.springframework.stereotype package you can find 4 annotation types: @Component, @Repository, @Controller and @Service. @Component is the base type while the latter ones are its subtypes. @Service definition looks like this:

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Service {
  String value() default "";
}
```

The first three meta annotations are the standard Java ones. The forth one is the type @Service "extends". Thanks to that every class annotated by @Service will be treated exactly the same as the one with @Component annotation declared on it. In this case @Service works simply as an alias for @Component. As of Spring 4.0 there is no additional processing associated with @Service. However it is still a good idea to use @Service/@Repository/@Controller over @Component where appropriate as it conveys more information about the class.

You can create your own, even more specialized subtypes.

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Service
public @interface ApplicationService {
  String value() default "";
}
```

As you can see now the Service type acts as the base but nonetheless classes annotated with @ApplicationService will still be registered as beans as any other Spring component would be.

Up to this point we have been creating aliases for a single base annotation type but it doesn't stop there. The single inheritance restriction known from Java class hierarchy does not apply here. That allows us to combine several related annotations into one. Here's a snippet of what I'm using to simplify integration testing.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@ContextConfiguration(classes = { WebAppConfig.class, SecurityConfig.class, ContextConfig.class, OperatorConfiguration.class },
  loader = AnnotationConfigWebContextLoader.class)
@WebAppConfiguration
@ActiveProfiles(profiles = { "dev" })
@DirtiesContext(classMode=ClassMode.AFTER_EACH_TEST_METHOD)
public @interface IntegrationTesting {

}
```

@ContextConfiguration specifies classes used to configure Spring context. Because of @DirtiesContext after each method the context will be created anew. @ActiveProfiles is present as the "dev" profile contains an embedded database to be used instead of a standalone one.

Now tests look a lot cleaner.


```java
@RunWith(SpringJUnit4ClassRunner.class)
@IntegrationTesting
public class HomeControllerIT {

  ...test methods...

}
```

Unfortunately the meta annotation mechanism is only available inside the Spring framework. I really hope this concept will be included in a future Java version or just implemented in other frameworks. In Hibernate I'd love to replace 4 id related annotations with just one. Until that time however we will have to stick to the old ways of Ctrl-C and Ctrl-V.