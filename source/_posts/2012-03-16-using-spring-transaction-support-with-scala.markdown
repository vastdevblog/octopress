---
layout: post
title: "Using Spring transaction support with Scala"
date: 2012-03-16 14:30
comments: false
categories: [scala, spring]
author: Alex Moffat <alex.moffat@gmail.com>
---

What's the best way to use [Spring](http://www.springsource.org/)'s [transaction management](http://static.springsource.org/spring/docs/3.1.x/spring-framework-reference/html/transaction.html) features from a Scala program? This is my second attempt to come up with an answer to that question. 

My preference for Spring transactions with Java is to use the [@Transactional](http://static.springsource.org/spring/docs/3.1.x/spring-framework-reference/html/transaction.html#transaction-declarative-annotations) annotation. I'd like to be able to achieve the same effect in Scala.

The intention for the Java annotations, and what we want in Scala, is to 

* Have some piece of code execute inside a transaction.
* Be able to specify the behavior of the transaction.
* Don't overwhelm the rest of the funactionality with the transactional code.

<!-- more -->

In Java with Spring the recommended approach is to define an interface, implement it and annotate the implementation. Using an interface lets Spring use the [JDK's proxying mechanism](http://docs.oracle.com/javase/1.5.0/docs/api/java/lang/reflect/Proxy.html). Below is an example of this usage.

``` java Example Java
public interface SomeInterface {
    
    int someMethod(String name);

    int someReadOnlyMethod(String name);
}

public class SomeClass implements SomeInterface {
    
    @Transactional    
    public int someMethod(String name) {
        // Some code that gets executed in a read/write transaction.
    }

    @Transactional(readOnly = true)
    public int someReadOnlyMethod(String name) {
        // Some code that gets executed in a read only transaction.
    }
}
```

In Scala we can't use the interface approach but we can get the same, effect using language features. What we can write is code like this.

``` scala Example Scala Usage

  def someMethod(name: String): Int = 
      inTransaction() {
          // Some code that runs in a read/write transaction.
      }

  def someReadOnlyMethod(name: String): Int = {
      // We can do stuff here
      inTransaction(readOnly = true) {
          // Some code that runs in a read only transaction.
      }
      // and here outside the transaction
  }
```

A side effect is that you don't have to use [Spring's proxying mechanisms](http://static.springsource.org/spring/docs/3.1.x/spring-framework-reference/html/aop.html#aop-proxying), which requires either a Java interface or using CGLIB.

The signature for the inTransaction method should be something like

``` scala inTransaction  
  def inTransaction[T](readOnly: Boolean = false)(value: =>T): T
```

It takes two lists of parameters. The first configures the transaction, in the same waht the arguments to the Spring Transactional annotation do. The second is a single call-by-name parameter, so that it is not evaluated before it's used, that provides the value. 

The complete implementation is shown below. A trait is used so that transactional support can be added to any class by including the trait. The including class needs to provide an instance of [PlatformTransactionManager](http://srcrr.org/java/spring/3.1.0/reference/org/springframework/transaction/PlatformTransactionManager.html) for the trait to use. This can be done with a val constructor parameter as shown. 

{% gist 2053120 TransactionalSupport.scala %}