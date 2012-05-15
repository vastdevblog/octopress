---
layout: post
title: "Generating Unique JUnit Test Names with TestNG Data Providers"
date: 2012-05-14 19:35
comments: true
categories: [testng, junit, tricks]
author: Olivier Modica <omodica@vast.com>
---

We use [TestNG](http://www.testng.org/) with the [Maven Failsafe Plugin](http://maven.apache.org/plugins/maven-failsafe-plugin/) to implement integration tests for our Java platform services.

For this specific platform service we've defined a set of functional test suites each containing between a few and a few hundred test payloads and expected results, with each test payload and expected result pair constituting one logical unit test. Each TestNG test would run a single functional test suite, iterating over all test pairs &mdash; so while we may have hundreds of unit tests the TestNG reports would only show one test per functional test suite and would fail the entire functional test suite if a single unit test within failed.

This really wasn't ideal for a couple of reasons:

* Since each unit test is independent from each other it makes little sense for a single unit test to fail all the unit tests within the functional test suite
* Test reports wouldn't adequately show the number of unit tests which succeeded, failed or were skipped 

<!-- more -->

Because of the large number of unit tests (hundreds of them) it just wasn't an option to list each unit test individually in either code or within a TestNG `testng.xml` file (although one could in theory generate this file at build time). 

So over the week-end I decided that it was time to take another look at this.

I immediately thought the TestNG `@DataProvider` annotation was a great fit for our use case, where we "feed" a test payload and expected result pair for each test so I implemented the following:

``` java
public class CarsAndTravelIT {

    @DataProvider(name = "testSpec")
    public Iterator<Object[]> createData(Method method) {
        // Retrieve the suite name from the annotation and pass it to the iterator which loads the input files within the test suite folder.
        return new TestSpecIterator(method.getAnnotation(TestSuite.class).suiteName());
    }
    
    @TestSuite(suiteName = "cars")
    @Test(dataProvider = "testSpec")
    public void carsTestSuite(TestSpec testSpec) throws Exception {
        performTestAndAssert(testSpec);
    }
        
    @TestSuite(suiteName = "travel")
    @Test(dataProvider = "testSpec")
    public void travelTestSuite(TestSpec testSpec) throws Exception {
        performTestAndAssert(testSpec);
    }
    
    @Retention(RetentionPolicy.RUNTIME)
    protected @interface TestSuite {
        String suiteName();
    }
    
    protected class TestSpecIterator implements Iterator<Object[]> {
    
        protected TestSpecIterator(String suiteName) {
            ...
        }
        
        ...
        
        @Override
        public Object[] next() {
            return new Object[] {fileIterator.next()};
        }
    }
}    
```

This code connects each test with a data provider named "testSpec", which is itself using a custom annotation to specify the functional test suite used to look up the test files.

So assuming the following test resources:

``` html
resources
    cars
        input1.xml
        input2.xml
        input3.xml
    travel
        input1.xml
        input2.xml
```

TestNG will execute something equivalent to:

``` html
carsTestSuite("input1.xml")
carsTestSuite("input2.xml")
carsTestSuite("input3.xml")
travelTestSuite("input1.xml")
travelTestSuite("input2.xml")
```

At this point I was pretty happy, and I could see that in the TestNG XML report and in the command line each individual test was being counted, so for the above example running `mvn integration-test` would show 5 tests executed &mdash; exactly what I wanted!

I checked the build results in our continuous build system only to discover that only 2 tests were being reported! What? How could I get 5 tests reported locally yet have 2 tests reported in our build system?

Eventually I tracked it down to the test names that TestNG generates in its JUnit XML report:

``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<testsuite failures="0" time="..." errors="0" skipped="0" tests="5" name="TestSuite">
  ...
  <testcase time="..." classname="..." name="carsTestSuite"/>
  <testcase time="..." classname="..." name="carsTestSuite"/>
  <testcase time="..." classname="..." name="carsTestSuite"/>
  <testcase time="..." classname="..." name="travelTestSuite"/>
  <testcase time="..." classname="..." name="travelTestSuite"/>
</testsuite>  
```

So while 5 tests were executed only 2 distinct test names were generated in the JUnit XML report ("carsTestSuite" and "travelTestSuite") which causes the build system to report only 2 tests run. Whether that's legitimate or not I had to find a solution.

While there may other ways to achieve the same result (by extending the TestNG JUnit reporter for example) my goal was to keep the changes to a minimum and eventually settled on using the `getTestName` method from the TestNG `ITest` interface:

``` java
public class CarsAndTravelIT implements ITest {

    private static final ThreadLocal<String> testName = 
        new ThreadLocal<String> () {
            @Override 
            protected String initialValue() {
                return "undefinedTestName";
            }
        };

    @DataProvider(name = "testSpec")
    public Iterator<Object[]> createData(Method method) {
        // Retrieve the suite name from the annotation and pass it to the iterator which loads the input files within the test suite folder.
        return new TestSpecIterator(method.getName(), method.getAnnotation(TestSuite.class).suiteName());
    }
    
    @TestSuite(suiteName = "cars")
    @Test(dataProvider = "testSpec")
    public void carsTestSuite(TestSpec testSpec) throws Exception {
        performTestAndAssert(testSpec);
    }
        
    @TestSuite(suiteName = "travel")
    @Test(dataProvider = "testSpec")
    public void travelTestSuite(TestSpec testSpec) throws Exception {
        performTestAndAssert(testSpec);
    }
    
    @Override
    public String getTestName() {
        return testName.get();
    } 
    
    @Retention(RetentionPolicy.RUNTIME)
    protected @interface TestSuite {
        String suiteName();
    }
    
    protected class TestSpecIterator implements Iterator<Object[]> {
    
        protected TestSpecIterator(String suiteMethodName, String suiteName) {
            ...
        }
        
        ...
        
        @Override
        public Object[] next() {
            // We're assuming the caller has called hasNext() before calling this method.
            TestSpec next = fileIterator.next();
            // Set the test name from the suite method name and the input file name, since this is called
            // right before getTestName().
            testName.set(suiteMethodName + "-" + next.getInputFile().getName());
            return new Object[] {next};
        }
    }
}    
```

Because the `getTestName` method get invoked right after the `next` method I chose to set the test name right then (since I used an iterator-backed data provider I could do that), which didn't require any additional changes to TestNG or its configuration. For safety I used a `ThreadLocal` but this will not be necessary if you can guarantee single threaded execution.

With those changes the JUnit XML report is now:

``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<testsuite failures="0" time="..." errors="0" skipped="0" tests="5" name="TestSuite">
  ...
  <testcase time="..." classname="..." name="carsTestSuite-input1.xml"/>
  <testcase time="..." classname="..." name="carsTestSuite-input2.xml"/>
  <testcase time="..." classname="..." name="carsTestSuite-input3.xml"/>
  <testcase time="..." classname="..." name="travelTestSuite-input1.xml"/>
  <testcase time="..." classname="..." name="travelTestSuite-input2.xml"/>
</testsuite>  
```

And now not only the is JUnit XML report had more specific test case names but our continuous build system was also reporting 5 tests. Voila!

I will file an enhancement request with the TestNG project to support this exact use case. 

Hopefully this post will be helpful to others.

