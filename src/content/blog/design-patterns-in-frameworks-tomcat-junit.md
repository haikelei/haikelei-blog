---
author: Gary Lu
pubDatetime: 2025-01-15T10:00:00+08:00
modDatetime: 2025-01-15T10:00:00+08:00
title: "Design Pattern Applications: Design Patterns in Programming Frameworks"
slug: design-patterns-in-frameworks-tomcat-junit
featured: false
draft: false
tags:
  - design-patterns
  - frameworks
  - tomcat
  - junit
  - strategy-pattern
  - template-method
description: Examining how popular frameworks like Tomcat and JUnit leverage design patterns such as Strategy, Template Method, and Composite to create flexible and extensible architectures.
---

# Design Pattern Applications: Design Patterns in Programming Frameworks

In most cases, when developing applications, we don't build from scratch. For example, when developing a Java web application, we don't need to write code to listen on HTTP port 80, handle binary HTTP data packets over networks, or manually allocate processing threads for each user request. Instead, we simply write a Servlet and work with an HttpRequest object. We don't even need to extract request parameters from HttpRequest—through Controllers, we can directly receive objects constructed from request parameters.

When writing code, we only need to focus on our business logic. Generic functionality like listening on HTTP ports and constructing parameter objects from HTTP requests is handled by common frameworks like Tomcat or Spring.

## What is a Framework?

**A framework is a reusable design and implementation for a certain class of architectural solutions.** All web applications need to listen on HTTP ports and process request parameters. This functionality shouldn't be repeatedly developed in every web application—it should be reused as common components.

However, not all reusable components are called frameworks. Frameworks typically define a software's main structure and can support the overall or partial architectural form of software. For example, Tomcat completes the main flow of web application request-response processing. We only need to develop Servlets to handle request processing logic and construct response objects. Therefore, Tomcat is a framework.

Another category of reusable components doesn't control software's main flow or support overall architecture. For example, Log4J provides reusable logging functionality, but logging isn't the software's main structure. We typically call Log4J a tool rather than a framework.

Generally, when using frameworks for programming, we need to follow framework specifications. With frameworks like Tomcat, Spring, MyBatis, and JUnit, **frameworks call our written code, while our code calls tools** to complete specific functionality like logging or regex matching.

I emphasize the difference between frameworks and tools not for semantic precision, but because I've seen ambitious engineers claim to have designed new frameworks when they've merely provided functional interfaces for developers to call—far from what we expect from frameworks.

Based on our framework description, when designing a framework, you're actually designing a common architecture for a class of software and implementing it through code. Simply providing functional interfaces for program calls cannot support software architecture or standardize software structure.

So how do we design and develop a programming framework?

I've previously discussed the Open-Closed Principle. Frameworks should satisfy this principle—facing different application scenarios, frameworks shouldn't need modification (closed for modification). But various application functionalities should be extensible (open for extension), allowing applications to extend various business functions based on frameworks.

Frameworks should also satisfy the **Dependency Inversion Principle**—frameworks shouldn't depend on applications (since applications don't exist when developing frameworks), and applications shouldn't depend on frameworks (enabling flexible framework switching). Both frameworks and applications should depend on abstractions. For example, Tomcat's programming interface is the Servlet specification. Applications only need to implement the Servlet interface to run on Tomcat without depending on Tomcat, enabling switching to other web containers like Jetty.

While design principles can guide framework development, they don't provide specific design methods. In fact, frameworks are developed using various design patterns. The relationship between programming frameworks, applications, design patterns, and design principles is shown below:

```
Design Goals (Low Coupling, High Cohesion)
    ↓
Design Principles (Open-Closed, Dependency Inversion, etc.)
    ↓
Design Patterns (GoF 23, MVC, etc.)
    ↓
Programming Frameworks (Tomcat, Spring, etc.)
    ↓
Applications
```

Object-oriented design aims for low coupling and high cohesion. To achieve this, people proposed design principles including Open-Closed, Dependency Inversion, Liskov Substitution, Single Responsibility, and Interface Segregation principles. Based on these principles, people summarized design patterns, most famously the GoF 23 design patterns and patterns familiar to web developers like MVC. Following these design patterns, people developed various programming frameworks enabling developers to develop applications simply and quickly.

## Design Patterns in Web Containers

I've repeatedly mentioned Tomcat as a framework. What design patterns do Tomcat and similar web containers use? How is code executed by web containers? Where does the HttpServletRequest object in programs come from?

Web containers primarily use the **Strategy Pattern**—multiple strategies implementing the same strategy interface. When programming, Tomcat depends on strategy interfaces and loads different strategy implementations based on different runtime contexts.

The strategy interface here is the Servlet interface, and our developed code implements this interface to handle HTTP requests. The J2EE specification defines the Servlet interface with three main methods:

```java
public interface Servlet {
    public void init(ServletConfig config) throws ServletException;
    public void service(ServletRequest req, ServletResponse res)
            throws ServletException, IOException;
    public void destroy();
}
```

When web containers load our developed Servlet concrete classes, they call the `init` method for initialization. When HTTP requests reach the container, it deserializes binary encoding from HTTP requests, encapsulates them into ServletRequest objects, then calls the Servlet's `service` method for processing. When containers shut down, they call the `destroy` method for cleanup.

When developing web applications, we only need to implement this Servlet interface and develop our own Servlets. Containers monitor HTTP ports and encapsulate received HTTP data packets into ServletRequest objects, calling our Servlet code. Code only needs to retrieve request data from ServletRequest for processing, returning results through ServletResponse to clients.

Here, Tomcat is the Client program in the Strategy pattern, Servlet interface is the strategy interface, and our developed concrete Servlet classes are strategy implementations. Using the Strategy pattern, web containers like Tomcat can call various Servlet application code, and various Servlet applications can run in other web containers like Jetty, as long as these containers support the Servlet interface.

Web containers complete the main HTTP request processing flow, specify Servlet interface specifications, and implement the main web development architecture. Developers only need to develop specific Servlets under this architecture. Therefore, we can call web containers like Tomcat and Jetty frameworks.

Actually, when developing specific Servlet applications, we don't directly implement the Servlet interface—we inherit the HttpServlet class, which implements the Servlet interface. HttpServlet also uses the **Template Method Pattern**, where parent classes define computational skeletons and processes using abstract methods, leaving abstract method implementations in subclasses.

Here, the parent class is HttpServlet. HttpServlet implements the Servlet interface by inheriting GenericServlet and calls corresponding methods for different HTTP request types in its service method. HttpServlet's service method is a template method:

```java
protected void service(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException {
    String method = req.getMethod();
    if (method.equals(METHOD_GET)) {
        doGet(req, resp);
    } else if (method.equals(METHOD_HEAD)) {
        long lastModified = getLastModified(req);
        maybeSetLastModified(resp, lastModified);
        doHead(req, resp);
    } else if ...
}
```

Since HTTP requests have seven types (GET, POST, etc.), HttpServlet provides corresponding execution methods (doGet, doPost, etc.) for programming convenience. The service template method determines HTTP request types and executes different methods accordingly. Developers only need to inherit HttpServlet and override corresponding HTTP request type methods like doGet and doPost without determining HTTP request types themselves.

## Design Patterns in JUnit

JUnit is a Java unit testing framework. Developers only need to inherit JUnit's TestCase, develop their test case classes, and execute tests through the JUnit framework to get results.

Developing test cases looks like this:

```java
public class MyTest extends TestCase {
    protected void setUp() {
        // Test initialization
    }

    public void testSome() {
        // Test method
    }

    protected void tearDown() {
        // Cleanup
    }
}
```

Each test case inherits TestCase, performs test initialization in the `setUp` method (like loading test data), writes multiple methods prefixed with "test" as test case methods, and has a `tearDown` method for cleanup after testing (like deleting test data from databases).

How are our written test cases executed by JUnit? How is the execution order of methods in test cases guaranteed? JUnit also uses the **Template Method Pattern**—test case method execution order is fixed in JUnit framework's template method:

```java
public void runBare() throws Throwable {
    setUp();
    try {
        runTest();
    } finally {
        tearDown();
    }
}
```

`runBare` is a method in the TestCase base class. When test cases execute, they actually only execute the runBare template method, which first executes setUp, then various test-prefixed methods, and finally tearDown, ensuring each test case gets initialization and necessary cleanup. Our test classes only need to inherit the TestCase base class and implement setUp, tearDown, and other test methods.

Additionally, software test cases are numerous. You might want to execute all cases or just some. JUnit provides TestSuite for managing and organizing test cases:

```java
public static Test suite() {
    TestSuite suite = new TestSuite("all");
    suite.addTest(MyTest.class);        // Add a TestCase
    suite.addTest(otherTestSuite);      // Add a TestSuite
    return suite;
}
```

TestSuite can add multiple TestCase classes to a test suite through the `addTest` method and can add other TestSuites. When executing this TestSuite, added TestCase classes execute, other added TestSuites' test classes execute, and if other test suites contain additional test suites, they all execute.

This means TestSuite is recursive—actually a tree structure. When traversing from the tree's root node, we can execute all test cases. Traditional tree traversal requires recursive programming, but using the **Composite Pattern**, we can traverse trees without recursion.

First, both TestSuite and TestCase implement the Test interface:

```java
public interface Test {
    public abstract void run(TestResult result);
}
```

When calling TestSuite's addTest method, TestSuite places input objects into an array:

```java
private Vector<Test> fTests = new Vector<Test>(10);

public void addTest(Test test) {
    fTests.add(test);
}
```

Since both TestCase and TestSuite implement the Test interface, addTest can accept both TestCase and TestSuite. When executing TestSuite's run method, it retrieves each object from this array and executes their run methods:

```java
public void run(TestResult result) {
    for (Test each : fTests) {
        each.run(result);
    }
}
```

If the test object is TestCase, it executes testing. If it's TestSuite, it continues calling that TestSuite's run method, traversing and executing each Test's run method in the array, implementing recursive tree traversal.

## The Bottom Line

There's a common misconception about architects' work—that architects only need to do architectural design without writing code. In fact, if architects only draw architecture diagrams and write design documents, how can they ensure their architectural designs are followed by development teams and implemented properly?

Architects should implement their architectural designs through code—by developing programming frameworks that establish software development standards. Development teams develop programs according to framework interfaces, ultimately being called and executed by frameworks. Architects don't need to repeatedly explain software architecture with diagrams—they just need to write framework-based demos, and everyone understands the architecture and their roles.

Therefore, every programmer aspiring to become an architect should learn how to develop frameworks.

---

_Have you worked with frameworks that clearly demonstrated specific design patterns? Which patterns do you find most useful when building reusable components? Share your framework development experiences in the comments._
