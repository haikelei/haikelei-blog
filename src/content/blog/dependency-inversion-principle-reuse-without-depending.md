---
author: Gary Lu
pubDatetime: 2025-01-08T10:00:00+08:00
modDatetime: 2025-01-08T10:00:00+08:00
title: "The Dependency Inversion Principle: How to Reuse Code Without Depending On It"
slug: dependency-inversion-principle-reuse-without-depending
featured: false
draft: false
tags:
  - design-principles
  - dependency-inversion
  - software-architecture
  - framework-design
description: Understanding how frameworks like Spring and Tomcat enable code reuse without direct dependencies through the Dependency Inversion Principle and interface ownership inversion.
---

# The Dependency Inversion Principle: How to Reuse Code Without Depending On It

In software development, we constantly use frameworks. If you're a Java developer, you're familiar with Spring and MyBatis. Even web containers like Tomcat and Jetty are frameworks. Here's something remarkable about frameworks: developers can use their functionality without ever calling the framework's code directly.

Your application doesn't call Spring's code to get dependency injection and MVC features—it just works. Your program never calls Tomcat's code, yet it can listen on HTTP ports and handle requests.

We use these frameworks daily and take this for granted, but stop and think—isn't this magical? When we write code, can we enable other engineers to use our functionality without calling our code? Most developers I've observed cannot. So how do Spring and Tomcat achieve this?

The answer lies in a fundamental object-oriented design principle: **Dependency Inversion**.

## The Dependency Inversion Principle Defined

The Dependency Inversion Principle states:

1. **High-level modules should not depend on low-level modules. Both should depend on abstractions.**
2. **Abstractions should not depend on details. Details should depend on abstractions.**

Layered software design is widely accepted among developers. Originally, layers were introduced to establish clear relationships where high-level modules depend on low-level ones. Typically, policy layers depend on mechanism layers, and business logic layers depend on data storage layers.

But what are the drawbacks of high-level modules depending on low-level ones?

**Maintenance becomes difficult.** High-level modules contain business logic and policies—the core that makes your software unique. Low-level modules contain technical details. When high-level modules depend on low-level ones, business logic depends on technical details. Changes to technical implementation force business code changes, which is unreasonable.

**Reusability suffers.** Generally, higher-level modules have greater reuse value. But if high-level modules depend on low-level ones, reusing high-level modules requires dragging along all their low-level dependencies.

Actually, we already use dependency inversion in many places. In Java database development, application code doesn't directly depend on database drivers—it depends on JDBC. Database drivers implement JDBC, so applications can switch databases without code changes. This works because the application (high-level) doesn't depend on the driver (low-level)—both depend on JDBC (abstraction).

Similarly, Java web applications don't depend on Tomcat specifically—they depend on the J2EE specification. Applications implement the Servlet interface, package themselves, and run on any compliant container. The container could be Tomcat, Jetty, or any J2EE-compliant server.

MVC frameworks and ORM frameworks all follow the dependency inversion principle.

## The Key: Interface Ownership Inversion

Let's dig deeper into how dependency inversion works so we can apply it in our own designs.

Here's traditional layered dependency—policy layer depends on mechanism layer, mechanism layer depends on utilities:

```
[Policy Layer] → [Mechanism Layer] → [Utilities Layer]
```

The problem with this approach is transitive dependency. The policy layer transitively depends on both lower layers, so any change below cascades upward, making maintenance nightmarish.

The solution is dependency inversion. Each high-level module declares abstract interfaces for the services it needs, and low-level modules implement these interfaces:

```
[Policy Layer] ← [Policy Service Interface] ← [Mechanism Layer]
[Mechanism Layer] ← [Mechanism Service Interface] ← [Utilities Layer]
```

Now high-level modules don't directly depend on low-level modules. Instead, low-level modules depend on interfaces defined by high-level modules, inverting the dependency relationship.

In typical programming, low-level modules own their interfaces and high-level modules depend on those interfaces. For example, the DAO layer defines its interface, and the Service layer depends on the DAO interface.

**But with dependency inversion, interface ownership is inverted.** Interfaces are defined by high-level modules and implemented by low-level modules. High-level modules don't depend on low-level interfaces—low-level modules depend on high-level interfaces.

For the Service/DAO example, **Service would define the interface, and DAO would implement it**—that's true dependency inversion.

## Using Dependency Inversion for High-Level Module Reuse

Let's examine a concrete example. Imagine a Button controlling a Lamp—when pressed, the lamp turns on or off.

The naive design has Button directly depend on Lamp:

```
[Button] → [Lamp]
```

Problems with this design:

- Any change to Lamp might require Button changes
- Button can't be reused to control other devices (like motors)
- Button is tightly coupled to Lamp's implementation

The solution is dependency inversion. Instead of depending on concrete implementation, depend on abstraction. The abstraction here is: "turn target object on/off."

Here's the refactored design:

```
[Button] → [ButtonServer Interface] ← [Lamp]
```

Button defines the `ButtonServer` interface describing the abstraction: turn target objects on/off. Lamp implements this interface to fulfill the requirement.

Through this inversion:

- Button no longer depends on Lamp—it depends on the ButtonServer abstraction
- Lamp also depends on ButtonServer
- Button can control any device implementing ButtonServer (motors, lights, anything)
- Lamp changes don't affect Button

Notice the interface ownership inversion—`ButtonServer` belongs to the high-level Button module, not the low-level Lamp. The name itself reflects this ownership.

This answers our opening question: How can other engineers use our code's functionality without calling our code? If we're the Button developers, other engineers just implement our `ButtonServer` interface. Button calls their device code, giving devices button functionality without the device developers calling Button code directly.

This is also called the **Hollywood Principle**: "Don't call me, I'll call you." Tomcat and Spring are designed this way—applications don't call the framework; the framework calls applications. But applications must implement framework interfaces (like Servlet) first.

## The Bottom Line

Dependency inversion means high-level modules don't depend on low-level modules—both depend on abstractions, typically defined by high-level modules and implemented by low-level ones.

**Coding guidelines for dependency inversion:**

1. **Use abstract interfaces in application code, avoid volatile concrete classes**
2. **Don't inherit from concrete classes**—if a class wasn't designed as abstract, avoid inheriting from it
3. **Don't override functions with concrete implementations**

The most typical application of dependency inversion is **framework design**. Frameworks provide core functionality (HTTP handling, MVC, etc.) and interface specifications. Applications follow these specifications to be called by the framework. Applications use framework functionality without calling framework code—they implement framework interfaces and get called by the framework.

We can apply dependency inversion principles in our code development, following framework design philosophies to create flexible, loosely-coupled, reusable software.

Software development sometimes feels like magic, displaying counter-intuitive characteristics that dazzle the mind. This is precisely the artistry of software programming. When you feel this magic and embody it in your designs, you've stepped through the doorway to becoming a software craftsperson.

---

_What examples of dependency inversion have you encountered in your work? How has it improved (or complicated) your codebases? Share your experiences in the comments._
