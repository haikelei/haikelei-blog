---
author: Gary Lu
pubDatetime: 2025-01-28T10:00:00+08:00
modDatetime: 2025-01-28T10:00:00+08:00
title: "The Purpose of Software Design: What Separates Great Developers from Mediocre Ones?"
slug: software-design-purpose-great-vs-mediocre-developers
featured: true
draft: false
tags:
  - software-design
  - software-engineering
  - code-quality
  - technical-leadership
description: Exploring the fundamental difference between great and mediocre developers through their approach to changing requirements and software design principles.
---

# The Purpose of Software Design: What Separates Great Developers from Mediocre Ones?

There's a saying in software development: great programmers are 100 times more productive than mediocre ones. That sounds dramatic, but honestly, I think it's conservative. In my experience, poor programmers often have _negative_ productivity—their work makes projects harder, code more obscure, timelines longer, and bugs multiply mysteriously. This decay spreads, corrupting even previously healthy codebases until projects become unmaintainable and fail entirely.

When you only look at the process, the difference between great and mediocre developers isn't obvious. But looking at outcomes? If the result is project failure, the productivity isn't just zero—it's negative infinity compared to successful projects.

Developer quality shows up in two ways: raw programming ability (not everyone can write a compiler) and software design skills. Even in straightforward domains like building an order management system, where the requirements are clear and most experienced developers can deliver working code, the gap between great and mediocre programmers remains vast.

In software design, the difference between good and bad design comes down to one thing: **how well it handles changing requirements**. And here's the key distinction—mediocre programmers fear requirement changes because every change spawns endless bugs. Great programmers _welcome_ requirement changes because they designed for flexibility from day one. Without changes, their elegant designs never get to shine.

This difference in attitude reflects a fundamental gap in design capability.

Once a developer gets used to writing flexible, change-friendly code, they can never go back to rigid, fragile, confusing implementations. Even _reading_ such code becomes physically uncomfortable. I remember a colleague asking to leave early one afternoon, looking pale and exhausted. When I asked what was wrong, he said, "I was code-reviewing for another team. The code was so awful that I threw up in the bathroom halfway through. I need to go home and recover."

Shocking? Maybe. But terrible code really does have that power—it crashes systems during execution, creates unending bugs during testing, traps developers in maze-like debugging sessions, and makes readers dizzy just trying to understand it.

## Characteristics of Bad Design

Bad design and code share several destructive traits:

### Rigidity

Components are so tightly coupled that tiny changes require modifications everywhere. What should be a simple feature request turns into a system-wide refactor.

### Fragility

Even worse than rigidity—small changes cause mysterious failures in seemingly unrelated parts of the system. You make one simple change, restart the application, and it crashes for no apparent reason. If rigidity makes 3-hour tasks take 3 days, fragility makes developers question their sanity.

### Immobility

You can't easily extract and reuse parts of the system. Want to use one component elsewhere? Good luck separating it from the tangled dependencies.

This is why many microservices migrations fail. Teams try to break apart monoliths without first solving the coupling problem. Microservices require loosely coupled modules—if your monolith can't achieve that, your "microservices" will just amplify the problems.

### Viscosity

When hacky solutions are easier to implement than proper ones, the software inevitably degrades. Projects start with good architecture but slowly rot through countless quick fixes and shortcuts.

### Opacity

Code is written for humans first, computers second. When code is cryptic and hard to understand, maintainers will modify it in ways the original designer never intended, leading to system decay. Clear, expressive code helps future developers follow the intended patterns.

## A Real Example of Design Decay

Most software isn't write-once-and-forget. It evolves throughout its lifetime—look at how Taobao grew from a small website to a system maintained by thousands of engineers, or how Facebook evolved from Zuckerberg's dorm room project to serve billions globally.

Let's trace how software rots through seemingly innocent requirement changes.

Imagine you need a program that copies keyboard input to a printer. Simple enough:

```cpp
void copy()
{
    int c;
    while ((c=readKeyBoard()) != EOF)
        writePrinter(c);
}
```

You ship it, other developers depend on it, everyone's happy. Months later, your boss says it needs to support paper tape input. You modify the code:

```cpp
bool ptFlag = false;
// Please reset this flag before use
void copy()
{
    int c;
    while ((c=(ptFlag? readPt() : readKeyBoard())) != EOF)
        writePrinter(c);
}
```

Now you have a global flag and a comment hoping people remember to reset it. Some forget, others misunderstand what it does, bugs emerge. You're frustrated but soldier on.

Then your boss returns: "We need paper tape _output_ too." You reluctantly add:

```cpp
bool ptFlag = false;
bool ptFlag2 = false;
// Please reset these flags before use
void copy()
{
    int c;
    while ((c=(ptFlag? readPt() : readKeyBoard())) != EOF)
        ptFlag2? writePt(c) : writePrinter(c);
}
```

You thoughtfully updated the comment from "this flag" to "these flags," but now even more developers are confused by the cryptic flags. Bugs multiply. You start polishing your resume.

## The Solution

In just two requirement changes, clean code became rigid, fragile, viscous, and opaque. This pattern appears everywhere in software development.

The industry has developed design principles and patterns to combat these problems. Following these practices prevents code rot and creates robust, flexible software.

For our example, a more flexible approach might look like:

```java
public interface Reader {
    int read();
}

public interface Writer {
    void write(int c);
}

public class KeyBoardReader implements Reader {
    public int read() {
        return readKeyBoard();
    }
}

public class Printer implements Writer {
    public void write(int c) {
        writePrinter(c);
    }
}

Reader reader = new KeyBoardReader();
Writer writer = new Printer();

void copy() {
    int c;
    while((c = reader.read()) != EOF)
        writer.write(c);
}
```

By abstracting input and output behind interfaces, the `copy` function becomes stable. Adding new input or output devices doesn't require changing the core logic—we just implement new classes.

**The best way to handle changing requirements is to design for change from the beginning, then continuously refactor based on real requirement changes to maintain flexibility.**

## The Bottom Line

Great software design anticipates change from day one. During development, we must stay alert for signs of decay and use design principles to identify problems and design patterns to solve them.

In interviews, I primarily assess programming capability through questions about design principles and patterns. Understanding these concepts is what separates developers who create lasting value from those who create technical debt.

In the upcoming "Software Design Principles" series, I'll dive deep into how design principles and patterns help you build robust, flexible, reusable, and maintainable programs.

## Your Experience

Have you encountered terrible code in your career? Did it exhibit the characteristics we discussed—rigidity, fragility, immobility, viscosity, or opacity? What problems did it create for your team?

---

_What's the worst codebase you've had to work with? How did you handle it? Share your war stories in the comments—we've all been there!_
