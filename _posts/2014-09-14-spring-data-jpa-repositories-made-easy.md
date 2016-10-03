---
layout: post
title: Spring Data JPA - repositories made easy
date: '2014-09-14T16:00:00.000+02:00'
author: Tomasz Krug
tags:
- Java
- Spring Data
- JPA
- Spring
modified_time: '2014-09-15T14:37:29.645+02:00'
blogger_id: tag:blogger.com,1999:blog-5263052209480207518.post-4171478870497694648
blogger_orig_url: http://www.castlecodestein.com/2014/09/spring-data-jpa-repositories-made-easy.html
---

Spring Data is a set of project aimed at simplifing the process of repository class creation. Each of them focuses on a different persistence technology but they do have a lot in common.

In this post I will show you how to use Spring Data to enhance JPA experience. The sample project is available at this [location](https://github.com/castlecodestein/spring-data-jpa). 

<!--more-->

#### Basic setup

First we need to include Spring Data JPA as a dependency in pom.xml.

```xml
<dependency>
  <groupId>org.springframework.data</groupId>
  <artifactId>spring-data-jpa</artifactId>
  <version>1.7.0.RELEASE</version>
</dependency>

<!-- Omitted database driver / JPA implementation -->
```

Then we have to tell Spring to scan the classpath and find repositories (interfaces extending the Repository interface) to implement.

```java
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(basePackages = { "your.base.package.with.repositories" })
public class SpringConfiguration {

  ...database and entity manager factory beans omitted...

}
```

And that's all. Basically it boils down to a single annotation @EnableJpaRepositories. Now Spring will create concrete implementations and register them in the application context.

I omitted beans not specific to Spring Data configuration. If you don't know how to connect to a database or how to configure Hibernate in the Spring environment then check my [Spring ORM post](/2014/09/07/spring-orm-hibernate-in-action/).

#### Repository declaration

Each repository is an interface extending org.springframework.data.repository.Repository or any of its subinterfaces such as CrudRepository and PagingAndSortingRepository.

The simplest repository declaration capable of inserting, updating, deleting and retrieving entities looks like this.

```java
public interface CourseRepository extends org.springframework.data.repository.CrudRepository<Course, Long> {

}
```

CrudRepository contains methods like save(entity), findOne(id), delete(id) and similar. PagingAndSortingRepository adds some more on top of them.

These methods take care of the basics however oftentimes it is necessary to execute custom queries. There are two ways to specify them in the repository interface. 

* By declaring a method with a name conforming to Spring Data convention

  ```java
  import org.springframework.data.repository.CrudRepository;

  public interface CourseRepository extends CrudRepository<Course, Long> {

  /**
   * Returns a single Course instance described by a provided title. 
   * If more than one found then error is thrown.
   */
  public Course findByTitle(String title);

  /**
   * Overrides the CrudRepository definition returning a Collection instead of an Iterable.
   */
  public Collection<Course> findAll();

  }
  ```

  Full repository method name syntax can be found [here](http://docs.spring.io/spring-data/jpa/docs/1.7.0.RELEASE/reference/html/#jpa.query-methods).

* By annotating a method with @Query annotation

  ```java
  import org.springframework.data.repository.CrudRepository;

  public interface CourseRepository extends CrudRepository<Course, Long> {

    @Query("from Course c where c.title = ?1")
    public Course findBy(String title);

  }
  ```

  This way you can execute any valid JPQL query. The method name is irrevelant so you can avoid too complicated names like findByFirstNameAndLastNameAndActiveTrue(firstName, lastName). SQL queries can be executed as well with nativeQuery parameter set to true.

#### Base repository interface

Spring Data provides three base interfaces for you to work with: Repository, CrudRepository and PagingAndSortingRepository. Methods defined there return Iterable<Entity> while most of the time you will need either a Collection object or one of its specialized subclasses. To avoid overriding the return type in every repository you can declare your own base interface and extend it instead of a Spring one.

```java
import org.springframework.data.repository.PagingAndSortingRepository;

@NoRepositoryBean
public interface CustomBaseRepository<E, ID> extends PagingAndSortingRepository<E, ID> {

  public Collection<E> findAll();

}
```

The @NoRepositoryBean is important as otherwise Spring will try to create a concrete implementation of this interface and will throw an error as it does not know the entity and the id type.

#### How to use them?

You can inject your repositories just like any other bean. For each repository there is a registered singleton bean created by Spring. Below you will find an example of a service class.

```java
@Service
public class CourseService {

  private final CourseRepository repository;

  @Inject
  public CourseService(CourseRepository repository) {
    super();
    this.repository = repository;
  }

  @Transactional
  public Course createCourse(String title) {
    Course course = Course.empty(title);
    return this.repository.save(course);
  }

}
```

#### Interacting with EntityManager

Spring Data query system is quite powerful. If there is any feature you would like to use that Spring Data does not support yet you can always provide your own implementation to the declared methods in repository interface.

To do it you need three elements. Your normal repository interface, additional interface with methods you want to implement yourself and the class implementing your custom methods. The naming convention is very important! If you do not conform to it Spring won't be able to find your implementation.

If the additional interface is named AbcRepositoryCustom then the class must be named AbcRepositoryCustomImpl. The required postfix can be changed with a @EnableJpaRepositories parameter.

```java
/**
 * Main repository interface. The one to be injected into services.
 */ 
public interface CourseRepository extends CriteriaRepository<Course>, CourseRepositoryCustom {

}

/**
 * Additional interface with custom methods.
 */
public interface CourseRepositoryCustom {

  public Collection<Course> findByTitle(String title);

}

/**<
 * Custom implementation class.
 */
public class CourseRepositoryCustomImpl implements CourseRepositoryCustom {

  @PersistenceContext
  EntityManager em;

  @Override
  public Collection<Course> findByTitle(String title) {
    return em.createQuery("from Course c where c.title = ?1", Course.class)
      .setParameter(1, title).getResultList();
  }

}
```

#### Summary

These are the basics of using Spring Data JPA in your project. As always it is a good idea to read through [Spring's reference documentation](http://projects.spring.io/spring-data-jpa/).

If any part of this note or the example project is unclear or you have some questions regarding Spring Data post it in the comment section. I will try to help.