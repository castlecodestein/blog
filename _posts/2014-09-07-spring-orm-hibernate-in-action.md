---
layout: post
title: Spring ORM + Hibernate in action
date: '2014-09-07T16:00:00.000+02:00'
author: Tomasz Krug
tags:
- Java
- JPA
- ORM
- Hibernate
- embedded database
- Spring
modified_time: '2014-09-07T18:25:32.267+02:00'
blogger_id: tag:blogger.com,1999:blog-5263052209480207518.post-8363628273461559165
blogger_orig_url: http://www.castlecodestein.com/2014/09/spring-orm-hibernate-in-action.html
---

Hello!

In today's post I'm going to show you how to use Java Persistence API in the Spring environment. The source code together with integration tests is available as always on [GitHub](https://github.com/castlecodestein/spring-orm-jpa).

<!--more-->

#### Maven

First of all we need to add dependencies to our project. The important part of the pom.xml file should look like this.

```xml
<!-- Spring -->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-context</artifactId>
  <version>4.1.0.RELEASE</version>
</dependency>

<!-- Spring ORM -->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-orm</artifactId>
  <version>4.1.0.RELEASE</version>
</dependency>

<!-- Hibernate JPA -->
<dependency>   
  <groupId>org.hibernate</groupId>
  <artifactId>hibernate-entitymanager</artifactId>
  <version>4.3.5.Final</version>
</dependency>

<!-- H2 database -->
<dependency>
  <groupId>com.h2database</groupId>
  <artifactId>h2</artifactId>
  <version>1.4.181</version>
</dependency>
```

Nothing fancy in here. Just include the ORM module, your JPA implementation (EclipseLink, Hibernate and OpenJPA are supported as of today), and a database driver. Here I use H2DB driver to run an embedded database. Very useful for testing purposes. Older versions of Spring can be used as well, however I have not tested it on the 3.x.x line.

#### Spring configuration

For JPA support to work we need to register an entity manager factory bean and a data source (embedded or standalone).

```java
@Bean
public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
  LocalContainerEntityManagerFactoryBean em = new LocalContainerEntityManagerFactoryBean();
  em.setDataSource(this.dataSource());
  em.setPackagesToScan(new String[] { "com.castlecodestein.orm" });
  em.setPersistenceUnitName("default");
  JpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
  em.setJpaVendorAdapter(vendorAdapter);
  em.setJpaProperties(additionalProperties());
  return em;
}

/**
 * Embedded data source. Schema.sql script resides in src/main/resources
 * folder and contains table/sequence definitions.
 * @return
 */
@Bean
public DataSource dataSource() {
  return new EmbeddedDatabaseBuilder()
    .setType(EmbeddedDatabaseType.H2)
    .addScript("classpath:schema.sql")
    .build();
}

private Properties additionalProperties() {
  return new Properties() {
    { // Hibernate Specific:
      setProperty("hibernate.dialect", "org.hibernate.dialect.H2Dialect");
    }
  }
}
```

Entity manager factory bean allows you to specify all the information normally contained within persistence.xml. Additionally you have to specify which JPA implementation you want to use (the JpaVendorAdapter part). Spring provides an abstraction layer over embedded database creation. In 4.1.0 version H2, HSQL and Derby databases are supported. Remember to provide a corresponding driver on the classpath.

These two bean are the bare minimum but there's another feature you'd really like to activate and I'm talking about the transaction support.

```java
@Configuration
@EnableTransactionManagement
public class SpringConfiguration {

  ...other beans...

  @Bean
  public PlatformTransactionManager transactionManager(LocalContainerEntityManagerFactoryBean entityManagerFactory) {
    JpaTransactionManager transactionManager = new JpaTransactionManager();
    transactionManager.setEntityManagerFactory(entityManagerFactory
      .getObject());
    return transactionManager;
  }

}
```

@EnableTransactionManagement tells Spring to turn the support on however without the transaction manager bean it won't function properly. Here I register a local JPA transaction manager but there are other implementations i.e. JTA manager.

### Repository class

Now that we have registered all required beans we can go on and implement a repository class. The Student class is a simple entity annotated with standard JPA annotations so I omitted it here for the clarity.

```java
@Repository
public class StudentRepository {

  @PersistenceContext
  private EntityManager em;

  public Student findOne(long id) {
    return em.find(Student.class, id);
  }

  public Collection<Student> findByLastName(String lastName) {
    return em
      .createQuery("from Student where lastName = ?1", Student.class)
      .setParameter(1, lastName).getResultList();
  }

  public Collection<Student> findAll() {
    return em.createQuery("from Student", Student.class).getResultList();
  }

  @org.springframework.transaction.annotation.Transactional
  public Student save(Student student) {
    return em.merge(student);
  }

  @org.springframework.transaction.annotation.Transactional
  public void delete(Student student) {
    em.remove(student);
  }

}
```

@Repository annotation marks this class as a Spring component. @PersistenceContext allows an entity manager to be injected. And @Transactional creates a database transaction if one does not yet exist. Normally I'd place @Transactional on the service level but this example is too simple to justify creating one.

#### Next step

In this post I described how to integrate Spring and JPA but Spring offers a lot better way to create repository classes than by hand as shown above. It's the Spring Data module and I'll cover it in the next post so stay tuned.  And have a nice day!