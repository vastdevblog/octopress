---
layout: post
title: "Handling Dependency Injection in Scala test cases with TestNG"
date: 2012-03-23 15:46
comments: false
categories: [scala, testng]
author: Alex Moffat <alex.moffat@gmail.com>
---

This is a short post about using [Guice](http://code.google.com/p/google-guice/) dependency injection 
in Scala test cases that are configured with [TestNG](http://testng.org/doc/index.html) annotations.
The two interesting points are

* How in Guice do you override production dependencies with test ones when you want to execute a test case?
* How do you use TestNG with Scala?

<!-- more -->

Here's a simple example showing how this works.

``` scala 
import org.testng.Assert._
import org.testng.annotations.{Guice, Test}
import org.testng.{ITestContext, IModuleFactory}

@Guice(moduleFactory = classOf[ExampleTestModuleFactory])
class ExampleTest {
 
  @Inject
  var idGenerator: UniqueIdGenerator = _

  /**
   * Make sure we're using the UniqueIdGenerator that
   * returns IDs from a known list.
   */
  @Test
  def notUnique() {
    assertEquals(idGenerator.nextId(), "7411c0c8-358c-4e5c-879d-96112806c9aa")
  }
}

class ExampleTestModuleFactory extends IModuleFactory {

  def createModule(context: ITextContext, testClass: Class[_]) = {
    Modules.`override`(new ProductionModuleOne, new ProductionModuleTwo)
      .`with`(new ExampleTestModule)
  }
}
```

Starting at the top with the `@Guice` annotation. This identifies the class that TestNG will
use to create the Guice module to provide the dependencies for the test. This class, in 
the example ExampleTestModuleFactory, has
to extend the IModuleFactory interface and return the module to use from the 
`createModule(context: ITextContext, testClass: Class[_])` method. Instead of a moduleFactory The `@Guice` annotation can also
be used by simply listing the guice modules `@Guice(module = Array(classOf[ProductionModuleOne], classOf[ProductionModuleTwo]))` but this doesn't allow for replacing production dependencies with test ones.

In ExampleTextModuleFactory the `override` and `with` methods on the 
[Modules](http://google-guice.googlecode.com/git/javadoc/com/google/inject/util/Modules.html) and
[OverriddenModuleBuilder](http://google-guice.googlecode.com/git/javadoc/com/google/inject/util/Modules.OverriddenModuleBuilder.html) 
classes are used to combine the production dependencies and then override some of them with test dependencies. 
The bindings in the ExampleTestModule override those for the same key in the production modules.
This lets you replace just the dependencies you need with test ones without having to replicate the complete production configuration. 
Backticks must be used around override and with because they are reserved words in Scala.

To use the dependencies you just need to add the `@Inject` annotation to the test class to inject values for variables, as in the example. 
Unfortunately you can't inject parameters into test methods using Guice, you have to use TestNG's facilities for that.