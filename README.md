# Spring Unit Testing

## Learning Goals

- Create and run unit tests for Spring code.

## Introduction

It's important that we keep unit testing independent of the Spring framework for
2 main reasons:

1. As always, isolating the test to specific functionality makes it easier to
   diagnose what's wrong when a test doesn't pass.
2. Additionally, we need to consider that the Spring Framework can be heavy and
   takes some time to initialize. Having a suite of tests that can run without
   any of the scaffolding associated with the Spring Framework means that they
   can be "cheap" to run, and therefore can run very frequently and integrated
   into a developer's day-to-day work with minimal impact on their velocity.

## Code Along: Create a Spring Project for Testing

Let's create a new Spring project to demonstrate testing within the Spring framework.

1. Go to [https://start.spring.io/](https://start.spring.io/).
2. Select the project properties.
   1. Select "Maven Project", as we will use Maven as the build tool.
   2. Select "Java" as the language.
   3. Select the most recent version of Spring Boot 2. (Make sure it does not
      have "SNAPSHOT" listed after it.)
   4. Change the "Artifact" metadata field to "spring-testing-demo". (This
      will update the "Name" and "Package name" metadata fields too).
   5. Change the "Description" metadata field to "Demo for Testing".
   6. Select the appropriate Java JDK version.
3. Add dependencies.
   1. Click "ADD DEPENDENCIES".
   2. Search for "spring web".
   3. Select "Spring Web" from the list.
4. Click on the "Generate" button on the bottom. This will download a zip file
   containing the Spring Boot Project.
5. Unzip the archive and open it in IntelliJ or a preferred code editor.

![Basic Spring Initializr](https://curriculum-content.s3.amazonaws.com/spring-mod-2/testing/spring-initializr-testing-project.png)

Once you have created the new project, let's create a few packages and classes.
We'll start simple in this lesson and then continue to build and add more
packages and classes as we go.

Create a package called `controller` under the `com.example.springtestingdemo`
directory. In the `controller` directory, create a Java class called
`DemoController`.

Consider the following project structure and ensure yours looks the same:

```text
├── HELP.md
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── example
    │   │           └── springtestingdemo
    │   │               ├── SpringTestingDemoApplication.java
    │   │               └── controller
    │   │                   └── DemoController.java
    │   └── resources
    │       ├── application.properties
    │       ├── static
    │       └── templates
    └── test
        └── java
            └── org
                └── example
                    └── springtestingdemo
                        └── SpringTestingDemoApplicationTests.java
```

Let's add a basic endpoint to this application. Add the following code to the
`DemoController` class:

```java
package com.example.springtestingdemo.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class DemoController {

   @GetMapping("/hello")
   public String hello() {
      return "Hello World";
   }
}
```

If we start up the application and open up Postman, we should now be able to hit
the `/hello` endpoint:

![postman-hello-world](https://curriculum-content.s3.amazonaws.com/spring-mod-2/testing/postman-hello-world.png)

Let's now create a test for our `DemoController` like we would if it wasn't
part a Spring application:

![intellij-generate-test](https://curriculum-content.s3.amazonaws.com/spring-mod-2/testing/intellij-generate-test.png)

Let's add "Unit" to the name of this class to differentiate it from the
integration and acceptance tests we will create later:

![create-demo-controller-unit-test-class](https://curriculum-content.s3.amazonaws.com/spring-mod-2/testing/intellij-create-demo-controller-unit-test-class.png)

In our unit test, we will simply test the return value of the method, just like
we would for a method that is not part of a Spring application. Note that we
will first assert a wrong return value, just to validate that our test fails
like it should, before actually changing it to test for the right return value:

```java
package com.example.springtestingdemo.controller;

import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class DemoControllerUnitTest {

   @Test
   void hello() {
      DemoController demoController = new DemoController();
      assertEquals("Invalid Message", demoController.hello());
   }
}
```

Run this test and see that it fails as expected - the important part here is to
make sure the test error is about the messages not matching, versus being about
any configuration issues:

![hello-wrong-message](https://curriculum-content.s3.amazonaws.com/spring-mod-2/testing/test-fail-1.png)

Now change the assertion:

```java
package com.example.springtestingdemo.controller;

import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class DemoControllerUnitTest {

   @Test
   void hello() {
      DemoController demoController = new DemoController();
      assertEquals("Hello World", demoController.hello());
   }
}
```

When we re-run this test, it should now pass!

Note: This lesson does not follow the test driven development model, which is to
write the unit tests first _before_ writing the code. In a test driven
development model, we would have created the unit tests first.

## Conclusion

In this lesson, we reviewed how to set up a unit test in a Spring project that
does **not** depend on nor care about any of the Spring Framework functionality.
It is very similar to how we created unit tests before. We should continue
creating these types of tests when writing our Spring applications so that we
have a suite of tests that are focused on the smallest pieces of functionality.
These tests will be easier to diagnose if/when something breaks and will be very
cheap to run in our day-to-day development process.

We do, however, want to validate that our functionality is properly integrated
with the Spring Framework, so that we can expose it as API endpoints for public
or managed access. Let's look at that in the next section.
