---
author: Gary Lu
pubDatetime: 2025-01-12T10:00:00+08:00
modDatetime: 2025-01-12T10:00:00+08:00
title: "Design Pattern Fundamentals: If You Can't Apply Design Patterns Flexibly, You Haven't Mastered OOP"
slug: design-pattern-fundamentals-oop-mastery
featured: false
draft: false
tags:
  - design-patterns
  - object-oriented-programming
  - polymorphism
  - software-design
description: Exploring why polymorphism is the essence of OOP and how design patterns represent the flexible application of polymorphic features in software development.
---

# Design Pattern Fundamentals: If You Can't Apply Design Patterns Flexibly, You Haven't Mastered OOP

In interviews, I like to ask: "Which design patterns are you most familiar with?" The typical response is "Singleton" and "Factory." Honestly, this answer isn't satisfying. While Singleton and Factory are classic design patterns, these creational patterns don't capture the essence of design patterns.

**The essence of design patterns lies in the flexible application of polymorphism—and polymorphism is the very essence of object-oriented programming.**

## The Essence of OOP is Polymorphism

Sometimes I ask interviewees, "What is an object?" I get various answers: "Objects combine data and methods." "Objects abstract problem domains." "Everything is an object." "Object characteristics are encapsulation, inheritance, and polymorphism."

This is an open question, and these answers are all correct in describing different aspects of objects. But what's the essence of object-oriented programming? What's the core difference between OOP and procedural programming?

We often say OOP's main characteristics are encapsulation, inheritance, and polymorphism. But are these three characteristics what truly distinguish object-oriented programming from other programming techniques?

Let's examine **encapsulation** first. OOP languages provide class definitions. Through classes, we can encapsulate member variables and methods, using access modifiers (private, protected, public) to control visibility.

The most basic design granularity in OOP is the class. Classes form relatively independent entities by encapsulating data and methods. Classes interact through access control constraints, completing object-oriented programming. However, encapsulation isn't unique to OOP languages. Procedural languages like C can achieve encapsulation by defining methods in header files (.h) and implementing structures and methods in implementation files (.c), allowing external programs to access only methods defined in headers.

**Inheritance** seems unique to OOP languages, but C can actually implement inheritance too. If struct A contains struct B's definition, we can understand A as inheriting from B, and methods defined for B can directly execute (through type casting) on A's data.

As a programming technique, this method of implementing inheritance through struct definitions was already frequently used before OOP languages appeared.

Looking at **polymorphism**, it can actually be implemented in C through function pointers. But using function pointers for polymorphism is dangerous because this polymorphism lacks syntax and compilation constraints—it relies purely on programmer agreements. When bugs occur, debugging is extremely painful. Therefore, this type of polymorphism isn't frequently used in procedural language development.

In object-oriented languages, polymorphism is simple: subclasses implement abstract methods from parent classes or interfaces, programs use abstract parent classes or interfaces for programming, different subclasses are injected at runtime, and programs exhibit different behaviors—this is polymorphism.

The greatest benefit is **implementation independence** during software programming. Programs code against interfaces and abstract classes without concerning themselves with specific implementations.

Remember the case I discussed in [Article 10]: for a program copying characters from input devices to output devices, if specific device implementations are coupled with the copy program, adding any input or output device requires modifying program code, eventually making the copy program increasingly complex and difficult to use and understand.

By using interfaces, we define Reader and Writer interfaces describing input and output devices respectively. The copy program only needs to code against these interfaces without caring about specific devices, keeping the program stable and easily reusable. Specific devices are created at runtime and passed to the copy program. Whatever specific device is passed determines where copy logic operates—devices can be plugged and unplugged like plugins, making programs exhibit polymorphic characteristics.

Polymorphism also **inverts dependency relationships** between program modules. In traditional programming thinking, if Module A calls Module B, Module A must depend on Module B—meaning A's code must import or use B's code. But through polymorphism, we can invert this dependency: Module A calls Module B, yet A doesn't depend on B—instead, B depends on A.

This is the Dependency Inversion Principle I mentioned in [Article 12]. More precisely, Module B doesn't depend on Module A either—it depends on abstract interfaces defined by Module A. Module A codes against abstract interfaces and calls them, Module B implements these interfaces. At runtime, Module B is injected into Module A, enabling A to call B without depending on B.

Polymorphism often makes object-oriented programming exhibit magical characteristics, and polymorphism is the essence of OOP. It's polymorphism that makes OOP vastly different from previous programming approaches.

## The Essence of Design Patterns is Using Polymorphism

Even knowing about OOP's polymorphic characteristics, it's difficult to leverage polymorphism effectively to develop powerful object-oriented programs. How exactly do we use polymorphic features well? Through continuous programming practice, people have summarized a series of design principles and patterns.

Our previous articles discussed design principles:

1. **Open-Closed Principle**: Software classes and modules should be closed for modification but open for extension
2. **Dependency Inversion Principle**: High-level modules shouldn't depend on low-level modules; both should depend on abstractions defined by high-level modules
3. **Liskov Substitution Principle**: Wherever parent classes are used, subclasses should be substitutable (considering runtime scenarios, not just static relationships)
4. **Single Responsibility Principle**: Classes should have only one reason to change (practically, class files shouldn't exceed one screen)
5. **Interface Segregation Principle**: Don't force callers to depend on methods they don't need (achieved through multiple interface inheritance)

Most design principles relate to polymorphism, but they're more guidance-oriented. Programming requires more specific design methods—these are design patterns.

**Patterns are reusable solutions.** In programming practice, people discovered certain problems recur repeatedly. While scenarios differ, problem essences remain the same, and solution methods are reusable. People call these reusable programming methods design patterns. **The essence of design patterns is flexible application of polymorphism.**

Let's examine the Decorator Pattern to see how to flexibly apply polymorphic features. First, define an interface `AnyThing` with an `exe` method:

```java
public interface AnyThing {
    void exe();
}
```

Multiple classes implement this interface. The Decorator Pattern's key characteristic is passing same-type objects through constructors—each class implements the same interface as objects passed to constructors:

```java
public class Moon implements AnyThing {
    private AnyThing a;
    public Moon(AnyThing a) {
        this.a = a;
    }
    public void exe() {
        System.out.print("明月装饰了");  // "Moon decorates"
        a.exe();
    }
}

public class Dream implements AnyThing {
    private AnyThing a;
    public Dream(AnyThing a) {
        this.a = a;
    }
    public void exe() {
        System.out.print("梦装饰了");  // "Dream decorates"
        a.exe();
    }
}

public class You implements AnyThing {
    private AnyThing a;
    public You(AnyThing a) {
        this.a = a;
    }
    public void exe() {
        System.out.print("你");  // "You"
    }
}
```

When designing these classes, they have no coupling. But during object creation, through different constructor orders, we can make these classes call each other, presenting different decoration results:

```java
AnyThing t = new Moon(new Dream(new You(null)));
t.exe();
// Output: 明月装饰了梦装饰了你 ("Moon decorates Dream decorates You")

AnyThing t = new Dream(new Moon(new You(null)));
t.exe();
// Output: 梦装饰了明月装饰了你 ("Dream decorates Moon decorates You")
```

Polymorphism's charm lies in how individual class code seems unremarkable, but once running, exhibits complex and varied characteristics. Sometimes polymorphism creates code reading difficulties that intimidate OOP newcomers. This is exactly why design patterns matter—through class names like Observer or Adapter, you immediately understand what pattern the designer is using, enabling faster code comprehension.

## The Bottom Line

Simply using object-oriented programming languages doesn't mean you've mastered OOP. Only by flexibly applying design patterns to make programs exhibit polymorphic characteristics—creating robust, flexible, clear, maintainable, and reusable programs—have you truly mastered object-oriented programming.

So next time an interviewer asks about design patterns, maybe you could respond: "Besides Singleton and Factory, I prefer Adapter and Observer patterns. Also, the Composite pattern is extremely useful for handling tree structures."

Design patterns are highly practice-oriented programming skills. Learning design patterns helps us appreciate OOP's various subtleties. Truly mastering design patterns requires continuous practical application, making programs more robust, flexible, clear, and reusable.

At that point, better interview responses about design patterns might be: "In my work, I particularly like using Template and Strategy patterns. In my last project, to solve different users needing different recommendation algorithms, I..."

In fact, design patterns aren't limited to the 23 patterns in the "Design Patterns" book. Any reusable design solution for specific problem scenarios can be called a design pattern.

There's a famous saying about design patterns: "Mastering design patterns means forgetting design patterns"—somewhat like Zhang Wuji learning Tai Chi. When you truly understand design patterns thoroughly, your programs contain design patterns everywhere. You might use two or three design patterns in just three to five lines of code. You become a design pattern master, potentially creating your own patterns. At that point, interviewers won't ask about design patterns anymore—and if they do, whatever you say will be correct.

---

_Have you experienced the "aha moment" when a design pattern suddenly made a complex problem simple? Which patterns have become second nature in your coding? Share your experiences in the comments._
