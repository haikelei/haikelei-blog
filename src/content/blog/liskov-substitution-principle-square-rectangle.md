---
author: Gary Lu
pubDatetime: 2025-01-18T10:00:00+08:00
modDatetime: 2025-01-18T10:00:00+08:00
title: "The Liskov Substitution Principle: Can a Square Inherit from a Rectangle?"
slug: liskov-substitution-principle-square-rectangle
featured: false
draft: false
tags:
  - design-principles
  - liskov-substitution
  - inheritance
  - object-oriented-programming
description: Understanding the Liskov Substitution Principle through the classic Square-Rectangle problem and why subclasses must be substitutable for their base types.
---

# The Liskov Substitution Principle: Can a Square Inherit from a Rectangle?

Object-oriented programming languages have three core characteristics: encapsulation, inheritance, and polymorphism. While these concepts can be learned quickly, mastering them takes considerable time and practice.

Simply put, multiple implementations of an interface represent polymorphism. Polymorphism enables programming against interfaces at compile time while binding to concrete classes at runtime, allowing classes to associate and combine without direct coupling, forming powerful cohesive systems. Most design patterns leverage polymorphic features, including the Open-Closed and Dependency Inversion principles we've discussed.

Polymorphism makes programming feel like magic. Master it, and you've grasped most object-oriented programming techniques.

Encapsulation packages attributes and methods within classes. Using encapsulation effectively means knowing which properties and methods belong in which classes—essentially, how to design objects properly.

Inheritance seems simpler than polymorphism and encapsulation, but inheritance misuse is surprisingly common in practice.

## The Liskov Substitution Principle Defined

There's a design principle governing inheritance relationships and ensuring inheritance doesn't violate the Open-Closed Principle: the **Liskov Substitution Principle**.

The formal definition states: "If for each object o1 of type T1 there is an object o2 of type T2 such that for all programs P defined in terms of T2, the behavior of P is unchanged when o1 is substituted for o2, then T1 is a subtype of T2."

More simply: **Subtypes must be substitutable for their base types.**

In detail: **Everywhere in a program that uses a base class should be able to use a subclass instead.**

Syntactically, any class can be inherited. But whether inheritance makes sense can't be determined from the inheritance relationship alone—you must examine it within application context. Can subclasses actually replace base classes where they're used?

Consider this Horse inheritance design:

```
Horse
├── WhiteHorse
└── Foal
```

Both WhiteHorse and Foal inherit from Horse because they "are" horses. But is this inheritance reasonable?

Let's examine the application context: humans riding horses.

In this scenario, both WhiteHorse and Foal should be able to substitute for Horse. WhiteHorse substitution works fine—humans can ride white horses. But Foal substitution is problematic because foals aren't mature enough to be ridden.

WhiteHorse can replace the base Horse class, but Foal cannot. Therefore, Foal inheriting from Horse violates the Liskov Substitution Principle.

## A Violation Example

Consider this code:

```java
void drawShape(Shape shape) {
    if (shape.type == Shape.Circle) {
        drawCircle((Circle) shape);
    } else if (shape.type == Shape.Square) {
        drawSquare((Square) shape);
    } else {
        // more conditions...
    }
}
```

Here, Circle and Square inherit from Shape, but the application method checks object types and calls different drawing functions accordingly.

This common but terrible approach violates both the Open-Closed Principle and Liskov Substitution Principle.

**Open-Closed violation**: Adding new Shape types requires modifying this method by adding more else-if branches.

**Liskov Substitution violation**: Adding new Shape types without modifying this method means those types can't substitute for the base Shape class.

The solution is simple—define a `draw()` method in the base Shape class that all subclasses implement:

```java
public abstract class Shape {
    public abstract void draw();
}
```

The drawShape() code becomes much simpler:

```java
void drawShape(Shape shape) {
    shape.draw();
}
```

This code satisfies both principles:

- **Open-Closed**: Adding new types requires no code modification
- **Liskov Substitution**: Subclasses can replace the base class and the program works correctly

## Can a Square Inherit from a Rectangle?

Whether inheritance design violates the Liskov Substitution Principle must be examined in specific scenarios.

Imagine we have a Rectangle class:

```java
public class Rectangle {
    private double width;
    private double height;

    public void setWidth(double w) { width = w; }
    public void setHeight(double h) { height = h; }
    public double getWidth() { return width; }
    public double getHeight() { return height; }
    public double calculateArea() { return width * height; }
}
```

This class works well in our applications. Now we need to add a Square class.

Typically, we judge inheritance appropriateness using "IS A" relationships. Since a square IS A special rectangle (where length equals width), Square should be able to inherit Rectangle:

```java
public class Square extends Rectangle {
    public void setWidth(double w) {
        width = height = w;
    }

    public void setHeight(double h) {
        height = width = h;
    }
}
```

This Square design looks normal and seems to work. But does it really?

We must use the Liskov Substitution Principle to judge inheritance reasonableness. The question isn't whether the inheritance design itself makes sense, but whether subclasses can substitute base classes in application scenarios.

Here's how Rectangle gets used:

```java
void testArea(Rectangle rect) {
    rect.setWidth(3);
    rect.setHeight(4);
    assert 12 == rect.calculateArea();
}
```

If we substitute Square for Rectangle in this scenario, `calculateArea()` returns 16, not 12. The program fails to run correctly. This inheritance violates the Liskov Substitution Principle.

## Subclasses Cannot Be More Restrictive Than Parents

A class's public methods represent a contract with users. Users follow this contract and expect the class to behave accordingly, returning reasonable values.

When subclasses inherit from parent classes, according to the Liskov Substitution Principle, users can substitute subclasses wherever parent classes are used. From a contract perspective, **subclass contracts cannot be more restrictive than parent class contracts**—otherwise substitution will fail due to stricter requirements.

In our example, Square inherits Rectangle but has a more restrictive contract—requiring length and width to be equal. Because Square has stricter requirements than Rectangle, Square cannot substitute Rectangle in places where Rectangle is used.

The Foal-Horse example follows the same pattern. Foal has stricter requirements than Horse (cannot be ridden), making the inheritance inappropriate.

In class inheritance, if a parent method has `protected` access, subclasses can override it as `public` but not `private`. `Private` access is more restrictive than `protected`—places that can use the parent's protected method cannot use the subclass's private method, violating the Liskov Substitution Principle. Conversely, making subclass methods `public` is fine—subclasses can have more lenient contracts.

**Generally, subclasses being more restrictive than parents violates the Liskov Substitution Principle.**

This principle seems both reasonable and simple, but if you don't rigorously examine your designs, violations are easy to miss.

In the JDK, Properties inherits from Hashtable, and Stack inherits from Vector—both violating the Liskov Substitution Principle:

- Properties requires String data types while Hashtable accepts Object (subclass more restrictive)
- Stack enforces LIFO behavior while Vector allows random access (subclass more restrictive)

These classes existed since JDK 1.0. If they could redesign, JDK engineers probably wouldn't structure them this way. This demonstrates how inappropriate inheritance happens easily and requires rigorous design review.

## The Bottom Line

In practice, when you inherit from a parent class merely to reuse its methods, you're probably close to incorrect inheritance. If a class wasn't designed for inheritance, don't inherit from it. Bluntly speaking, if it's not an abstract class or interface, avoid inheriting it.

If you need to use another class's methods, **composition is better than inheritance**:

```java
class A {
    public Element query(int id) { ... }
    public void modify(Element e) { ... }
}

class B {
    private A a;

    public Element select(int id) {
        return a.query(id);
    }

    public void modify(Element e) {
        a.modify(e);
    }
}
```

Instead of inheriting A, B composes A and achieves the same method access with greater flexibility (like renaming methods to fit application interfaces). This is the **Object Adapter Pattern**.

Of course, inheriting interfaces or abstract classes doesn't guarantee correct inheritance design. The best approach is checking your design with the Liskov Substitution Principle: Can subclasses substitute parent classes wherever they're used?

Liskov Substitution violations occur not just in inheritance design but also in how parent and child classes are used. Incorrect usage patterns can prevent subclasses from substituting parent classes.

---

**Quick Question**: If a parent class has an abstract method throwing `AException`, and a subclass wants to throw `BException` instead, should `BException` be a parent or child class of `AException`? Why? Use the Liskov Substitution Principle to explain your answer.

_Have you encountered inheritance designs that seemed logical but broke when actually used? Share your experiences in the comments—we've all fallen into these traps!_
