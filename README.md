# Spring Unit Testing

## Learning Goals

- Create and run unit tests for Spring code.

## Introduction

There are 3 levels of functional testing we want to consider when we build APIs
with Spring:

1. Unit Testing: as with all unit testing, this should focus on the smallest
   possible "unit" of our functionality, which means it should not depend on or
   test any of the functionality of the Spring framework, but instead focus on
   the functionality of a specific method.
2. Integration Testing: this phase of testing will start to bring in some of the
   Spring framework functionality and wiring to make sure that the functionality
   we implemented ourselves is properly integrated with the Spring framework.
3. Acceptance Testing: this last phase of functional testing will make sure that
   all the layers of our APIs are tested together and that the functionality
   that is expected by the end users of our APIs functions properly.

We will be looking at unit testing in this lesson.

## Unit Testing

It's important that we keep unit testing independent of the Spring framework for
2 main reasons:

1. As always, isolating the test to specific functionality makes it easier to
   diagnose what's wrong when a test doesn't pass. In this way, the same
   principles that we've discussed before for Unit Testing and TDD apply here.
2. Additionally, we need to consider that the Spring Framework can be heavy and
   takes some time to initialize. Having a a suite of tests that can run without
   any of the scaffolding associated with the Spring Framework means that they
   can be "cheap" to run, and therefore can run very frequently and integrated
   into a developer's day-to-day work with minimal impact on their velocity.

Let's use `Spring Itinitializr` to create a basic version of a Spring Boot
project:

![Basic Spring Initializr](https://curriculum-content.s3.amazonaws.com/java-spring-2/spring-testing-basic-initializr.png)

The only dependency we need to add is `Spring Web`.

This gives us the following basic Spring Boot application:

```java
package com.flatiron.spring.FlatironSpring;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import java.net.URL;
import java.net.URLClassLoader;

@SpringBootApplication
public class FlatironSpringApplication {

	public static void main(String[] args) {
		SpringApplication.run(FlatironSpringApplication.class, args);
	}
}
```

Let's add a basic endpoint to this application with the following class:

```java
package com.flatiron.spring.FlatironSpring;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello World";
    }

}
```

Start your application, and you should now to be able to see the output of your
endpoint in the console:

![Hello World Console](https://curriculum-content.s3.amazonaws.com/java-spring-2/spring-testing-hello-world-console.png)

Or in the browser:

![Hello World Browser](https://curriculum-content.s3.amazonaws.com/java-spring-2/spring-testing-hello-world-browser.png)

If you see an "unauthorized" error message instead of your "Hello World"
message, you may be running into the configuration issue below. If so, go
through the next session. If not, skip the next session and move on to the one
after that.

This is the login form you may see if security is automatically configured:

![Spring Security Simple Authentication](https://curriculum-content.s3.amazonaws.com/java-spring-2/spring-testing-simple-auth.png)

### ! Configuration warning ! Putting Spring Security on hold until the next module

Depending on the version of `Spring Boot` that you use, it may automatically add
Spring Security to your classpath, in which case Spring will automatically
configure a basic authenticator that will protect all URLs you try to expose
through your application.

While that is helpful once you understand how Spring Security works, we need to
disable this functionality while we focus on testing. There are several ways to
do this - we're going to use the easiest one here, and ignore the details of how
this actually works until we discuss security in more details in another module.

For now, simply add the following class to your existing package, and it will
open up all your application URLs. We will cover exactly how and why that works
in our Spring Security module.

```java
package com.flatiron.spring.FlatironSpring;

import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@Configure
@EnableWebSecurity
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().permitAll();
    }
}
```

### Back to our regular programming (see what we did there)

Now that we have validated that our Spring setup is working properly, let's
remove some of our functionality in order to make sure we can set up our testing
properly to work **without** any Spring scaffolding.

Let's start by commenting out the `@GetMapping` annotation in our
`HelloController`:

```java
package com.flatiron.spring.FlatironSpring;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

//    @GetMapping("/hello")
    public String hello() {
        return "Hello World";
    }

}
```

Restart your application, and you should see the following error message when
you try to access your `/hello` URL:

![URL not found](https://curriculum-content.s3.amazonaws.com/java-spring-2/spring-testing-url-not-found.png)

This validates that our `hello()` no longer defines an endpoint for our Spring
application.

Let's now create a test for our `HelloController` like we would if it wasn't
part a Spring application:

![Generate Test](https://curriculum-content.s3.amazonaws.com/java-spring-2/spring-testing-generate-test.png)

Let's add "Unit" to the name of this class to differentiate it from the
integration and acceptance tests we will create later:

![Unit Test Options](https://curriculum-content.s3.amazonaws.com/java-spring-2/spring-test-unit-test-options.png)

In our unit test, we will simply test the return value of the method, just like
we would for a method that is not part of a Spring application. Note that we
will first assert a wrong return value, just to validate that our test fails
like it should, before actually changing it to test for the right return value:

```java
package com.flatiron.spring.FlatironSpring;

import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class HelloControllerUnitTest {

    @Test
    void hello() {
        HelloController helloController = new HelloController();
        assertEquals("Invalid Message", helloController.hello());
    }
}
```

Run this test and see that it fails as expected - the important part here is to
make sure the test error is about the messages not matching, versus being about
any configuration issues:

![Hello Wrong Message](https://curriculum-content.s3.amazonaws.com/java-spring-2/spring-testing-hello-wrong-message.png)

Now change the assertion:

```java
package com.flatiron.spring.FlatironSpring;

import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class HelloControllerUnitTest {

    @Test
    void hello() {
        HelloController helloController = new HelloController();
        assertEquals("Hello World", helloController.hello());
    }
}
```

And see that your test passes:

![Hello test passes ](https://curriculum-content.s3.amazonaws.com/java-spring-2/spring-testing-hello-test-passes.png)

This method that returns a hardcoded value is not very valuable and not very
interesting to test, so let's add a tiny bit of functionality to make it return
a String that reflects a parameter that is passed in:

```java
package com.flatiron.spring.FlatironSpring;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

//    @GetMapping("/hello")
    public String hello(String name) {
        return String.format("Hello %s", name);
    }

}
```

Our unit test will now fail, since we've changed the functionality of our method
without updating it, so let's update it:

```java
package com.flatiron.spring.FlatironSpring;

import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

class HelloControllerUnitTest {

    @Test
    void hello() {
        HelloController helloController = new HelloController();
        String name = "Jamie";
        assertEquals("Hello " + name, helloController.hello(name));
    }
}
```

Now let's restore the endpoint functionality of the controller, and make sure
that our unit test still passes without having to initialize any of the Spring
framework components:

```java
package com.flatiron.spring.FlatironSpring;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello(String name) {
        return String.format("Hello %s", name);
    }

}
```

You should now be able to run your unit test without any changes, and it should
still pass.

## Conclusion

In this section, we looked at how to set up a unit test in a Spring project that
does **not** depend on nor care about any of the Spring Framework functionality.
This allows us to have a suite of tests that are only focused on the actual
functionality we've implemented in our application. These tests will be easier
to diagnose if/when something breaks and will be very cheap to run in our
day-to-day development process.

We do, however, want to validate that our functionality is properly integrated
with the Spring Framework, so that we can expose it as API endpoints for public
or managed access. Let's look at that in the next section.
