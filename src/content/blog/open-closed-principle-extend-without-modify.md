---
author: Gary Lu
pubDatetime: 2025-01-22T10:00:00+08:00
modDatetime: 2025-01-22T10:00:00+08:00
title: "The Open-Closed Principle: How to Implement Changes Without Modifying Code"
slug: open-closed-principle-extend-without-modify
featured: false
draft: false
tags:
  - design-principles
  - open-closed-principle
  - extensibility
  - design-patterns
description: Understanding how to implement requirement changes without modifying existing code through the Open-Closed Principle and various design patterns like Strategy, Adapter, Observer, and Template Method.
---

# The Open-Closed Principle: How to Implement Changes Without Modifying Code

In my previous article, I explained that software design should embrace requirement changes and respond to them flexibly and quickly. Great programmers should welcome changing requirements because continuous evolution means their software remains vibrant, and their change-ready designs finally get to shine. This creates a virtuous cycle for both technology and business.

But requirement changes mean existing functionality must evolve, which typically means modifying code. If we handle changes by constantly editing code, that code will inevitably become unrecognizable and decay over time.

Is there a way to implement requirement changes _without_ modifying existing code?

This might sound impossible, but it's actually the most fundamental principle in software design: the **Open-Closed Principle**.

## The Open-Closed Principle Defined

The Open-Closed Principle states: **Software entities (modules, classes, functions, etc.) should be open for extension but closed for modification.**

- **Open for extension** means the software entity's behavior can be extended. When requirements change, you can extend the module to meet new needs.
- **Closed for modification** means when extending software entities, you don't need to change the current implementation—no code edits, no recompiling existing class files, no rebuilding deployed modules.

Simply put: **Software functionality can be extended, but software entities cannot be modified.**

This seems contradictory—how can you extend functionality without modifying code?

## A Violation of the Open-Closed Principle

Let's start with a negative example to illustrate the problem.

Imagine designing a phone with buttons for dialing. The core objects are Button and Dialer. A simple design might look like this:

```java
public class Button {
    public final static int SEND_BUTTON = -99;

    private Dialer dialer;
    private int token;

    public Button(int token, Dialer dialer) {
        this.token = token;
        this.dialer = dialer;
    }

    public void press() {
        switch (token) {
            case 0: case 1: case 2: case 3: case 4:
            case 5: case 6: case 7: case 8: case 9:
                dialer.enterDigit(token);
                break;
            case SEND_BUTTON:
                dialer.dial();
                break;
            default:
                throw new UnsupportedOperationException("unknown button: " + token);
        }
    }
}

public class Dialer {
    public void enterDigit(int digit) {
        System.out.println("enter digit: " + digit);
    }

    public void dial() {
        System.out.println("dialing...");
    }
}
```

This code works and meets requirements. The design seems reasonable at first glance.

But it violates the Open-Closed Principle. When you want to:

- Add new button types (like \* and # keys), you must modify Button
- Use buttons to control a password lock instead of a dialer, you must modify Button
- Have buttons control multiple devices, you must modify Button

Any functional extension requires modifying the Button class, clearly violating the principle.

The consequences are severe:

- **Rigidity**: The Button class becomes inflexible—any requirement change demands code modification
- **Fragility**: Large switch/case blocks are brittle and error-prone
- **Poor reusability**: Button is tightly coupled to Dialer and mixed with various button types

**When you see `else` or `switch/case` keywords in code, you can usually assume the Open-Closed Principle is being violated.**

## Using the Strategy Pattern

Many design patterns solve extensibility problems while adhering to the Open-Closed Principle. Let's redesign our example using the **Strategy Pattern**.

We introduce an abstract `ButtonServer` interface between Button and Dialer. Button depends on ButtonServer, while Dialer implements ButtonServer.

```java
public interface ButtonServer {
    void buttonPressed(int token);
}

public class Button {
    private ButtonServer server;
    private int token;

    public Button(int token, ButtonServer server) {
        this.token = token;
        this.server = server;
    }

    public void press() {
        server.buttonPressed(token);
    }
}
```

When a button is pressed, it calls the ButtonServer's `buttonPressed` method, which is actually implemented by Dialer. This completes the required functionality while decoupling Button from Dialer.

Now Button can be reused in other scenarios—any class implementing ButtonServer (like a password lock) can use Button without any code modifications.

The Strategy Pattern is a behavioral pattern where multiple strategies implement the same interface. The client program depends on the strategy interface and receives different strategy implementations at runtime based on context.

## Using the Adapter Pattern

Button now follows the Open-Closed Principle, but Dialer violates it because it must implement ButtonServer and use if/else or switch/case logic based on the token parameter.

The solution? The **Adapter Pattern**.

Instead of having Dialer directly implement ButtonServer, we create two adapters: `DigitButtonDialerAdapter` and `SendButtonDialerAdapter`. The adapters implement ButtonServer and call Dialer's methods appropriately, while Dialer remains unchanged.

```java
public class DigitButtonDialerAdapter implements ButtonServer {
    private Dialer dialer;

    public DigitButtonDialerAdapter(Dialer dialer) {
        this.dialer = dialer;
    }

    public void buttonPressed(int token) {
        dialer.enterDigit(token);
    }
}

public class SendButtonDialerAdapter implements ButtonServer {
    private Dialer dialer;

    public SendButtonDialerAdapter(Dialer dialer) {
        this.dialer = dialer;
    }

    public void buttonPressed(int token) {
        dialer.dial();
    }
}
```

The Adapter Pattern bridges incompatible interfaces, allowing them to work together without modification.

## Using the Observer Pattern

What if one button needs to control multiple devices? Maybe pressing a button should trigger dialing _and_ play sounds _and_ light up different colored LEDs?

With our current design, we'd need to modify the adapter to call multiple devices, violating the Open-Closed Principle again.

The solution? The **Observer Pattern**.

```java
public class Button {
    private List<ButtonListener> listeners;

    public Button() {
        this.listeners = new LinkedList<ButtonListener>();
    }

    public void addListener(ButtonListener listener) {
        listeners.add(listener);
    }

    public void press() {
        for (ButtonListener listener : listeners) {
            listener.buttonPressed();
        }
    }
}
```

Now we can add multiple observers to a button. When new devices need to respond to button presses, we simply add them to the listener list without modifying Button.

The Observer Pattern solves one-to-many object relationships, notifying multiple observers when a subject's behavior occurs.

## Using the Template Method Pattern

What if buttons need to perform internal operations when pressed—updating member variables or changing state differently for each button type?

The **Template Method Pattern** provides the solution:

```java
public abstract class Button {
    private List<ButtonListener> listeners;

    abstract void onPress();

    public void press() {
        onPress();  // Template method calls abstract method
        for (ButtonListener listener : listeners) {
            listener.buttonPressed();
        }
    }
}

public class SendButton extends Button {
    void onPress() {
        // Perform send-button-specific operations
    }
}
```

The Template Method Pattern defines the skeleton of an algorithm in a parent class while letting subclasses implement specific steps.

## The Key Insight

**The key to implementing the Open-Closed Principle is abstraction.**

When a module depends on an abstract interface, you can extend that interface freely. You don't need to modify existing code—you just add new implementations of the interface, leveraging polymorphism to handle requirement changes.

Different extension scenarios lead to different design patterns, and most design patterns exist to solve extensibility problems.

The Open-Closed Principle is the _principle of principles_ in software design—the core principle that guides all others. While other design principles are more technical and specific, the Open-Closed Principle provides direction. During software design, constantly ask yourself: "When requirements change, can my current design extend functionality without modifying code?"

If not, it's time to apply other design principles and patterns to redesign.

## The Bottom Line

Great software design anticipates change from the beginning. When you find your design violating the Open-Closed Principle as requirements evolve, refactor immediately to maintain strong, clean code.

The best developers don't fear changing requirements—they design systems that make change effortless. That's the difference between code that lasts and code that rots.

---

_Have you encountered codebases that constantly required modification for new features? How did you handle the technical debt? Share your experiences in the comments._
