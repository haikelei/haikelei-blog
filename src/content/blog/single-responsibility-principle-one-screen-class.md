---
author: Gary Lu
pubDatetime: 2025-01-25T10:00:00+08:00
modDatetime: 2025-01-25T10:00:00+08:00
title: "The Single Responsibility Principle: Why Class Files Should Fit on One Screen"
slug: single-responsibility-principle-one-screen-class
featured: false
draft: false
tags:
  - design-principles
  - single-responsibility
  - code-quality
  - software-design
description: Understanding the Single Responsibility Principle through real-world examples, from a massive 10,000-line SQL processing class to web application architecture evolution.
---

# The Single Responsibility Principle: Why Class Files Should Fit on One Screen

During my time at Intel, I inherited a big data SQL engine development project. When I took over, the project had completed early technical validation and architectural design, capable of processing relatively simple standard SQL statements. The company planned to form a dedicated team to develop support for complete standard SQL syntax and eventually commercialize the product.

When I opened the project, I broke out in a cold sweat. The project consisted of just a few classes, with the largest one—responsible for SQL syntax processing—containing nearly 10,000 lines of code. The code was filled with massive switch/case and if/else statements, methods calling each other randomly, and global variables being passed everywhere.

You could only understand what each line meant by running test SQL statements in debug mode. And this was 10,000 lines implementing less than 10% of SQL syntax features. If we implemented all SQL features, how massive would this class become? How complex would the logic be? How difficult would maintenance be? And we were supposed to have a team collaborating on development! Imagine multiple people committing code to such a giant file—just thinking about it was painful.

Here's one method from that SQL syntax processing class—there were hundreds more like it:

```java
/**
 * Digest all Not Op and merge into subq or normal filter semantics
 * After this process there should not be any NOT FB in the FB tree.
 */
private void digestNotOp(FilterBlockBase fb, FBPrepContext ctx) {
    // recursively digest the not op in a top down manner
    if (fb.getType() == FilterBlockBase.Type.LOGIC_NOT) {
        FilterBlockBase child = fb.getOnlyChild();
        FilterBlockBase newOp = null;
        switch (child.getType()) {
        case LOGIC_AND:
        case LOGIC_OR: {
            // not (a and b) -> (not a) or (not b)
            newOp = (child.getType() == Type.LOGIC_AND) ? new OpORFilterBlock()
                : new OpANDFilterBlock();
            FilterBlockBase lhsNot = new OpNOTFilterBlock();
            FilterBlockBase rhsNot = new OpNOTFilterBlock();
            lhsNot.setOnlyChild(child.getLeftChild());
            rhsNot.setOnlyChild(child.getRightChild());
            newOp.setLeftChild(lhsNot);
            newOp.setRightChild(rhsNot);
            break;
        }
        case LOGIC_NOT:
            newOp = child.getOnlyChild();
            break;
        case SUBQ: {
            switch (((SubQFilterBlock) child).getOpType()) {
            case ALL: {
                ((SubQFilterBlock) child).setOpType(OPType.SOMEANY);
                SqlASTNode op = ((SubQFilterBlock) child).getOp();
                revertRelationalOp(op);
                break;
            }
            // ... many more cases
            }
            newOp = child;
            break;
        }
        // ... more cases
        }
        fb.getParent().replaceChildTree(fb, newOp);
    }
    if (fb.hasLeftChild()) {
        digestNotOp(fb.getLeftChild(), ctx);
    }
    if (fb.hasRightChild()) {
        digestNotOp(fb.getRightChild(), ctx);
    }
}
```

At that moment, I thought: _This is going to be tough._

## The Single Responsibility Principle Defined

Software design has two fundamental principles: low coupling and high cohesion. Most design principles I've discussed (and design patterns we'll cover) focus on achieving low coupling. Cohesion examines the functional relatedness of elements within a module or class.

When designing classes, we should place strongly related elements inside a class and weakly related elements outside. This maintains high class cohesion. The specific design principle is:

**A class should have only one reason to change.**

This is the **Single Responsibility Principle**. If a class takes on too many responsibilities, it couples those responsibilities together. This coupling makes the class fragile—when changes occur, they cause unnecessary modifications and introduce bugs.

Too many responsibilities also lead to excessive code. When a class becomes too large, it's difficult to maintain the Open-Closed Principle. If you must open and modify the class file, massive amounts of code appear on screen, making errors easy to introduce.

This leads to a programming best practice: **A class file should ideally fit within one screen when opened.** This approach offers several benefits: less code with single responsibility makes reuse and extension easier (better Open-Closed Principle compliance), and simpler reading with convenient maintenance.

## A Single Responsibility Violation Example

To determine if a class has single responsibility, check if it has only one reason to change.

Consider this design:

```
Rectangle Class
├── draw() method
└── area() method
```

The Rectangle class has two methods: `draw()` for rendering and `area()` for calculating area. Two applications depend on this Rectangle class: a geometric calculation application and a graphical interface application.

Drawing requires area calculation, but calculating area doesn't need drawing capabilities. Screen rendering is complex and requires dependency on a specialized GUI component package.

This creates an awkward situation: when developing a geometric calculation application, I need to depend on the Rectangle class, which depends on a GUI package that might be tens or hundreds of megabytes. A geometric calculation program should primarily contain mathematical computation code. Now the packaged program must include an unrelated GUI package, growing from potentially hundreds of kilobytes to hundreds of megabytes.

The Rectangle class design violates the Single Responsibility Principle. Rectangle handles two responsibilities: geometric shape calculation and screen graphics rendering. The class has two reasons to change, creating unnecessary coupling that not only bloats scientific computing applications but also forces recompilation of geometric calculation applications when graphical interface applications modify Rectangle.

Better design separates these responsibilities by splitting Rectangle into two classes:

```
GeometricRectangle Class
└── area() method

Rectangle Class
└── draw() method (can use GeometricRectangle for area)
```

Geometric area calculation gets extracted to an independent `GeometricRectangle` class responsible for shape area calculation. Rectangle retains only single drawing responsibility. Now rectangle rendering can use area calculation methods, while geometric calculation applications don't need to depend on unrelated drawing methods and GUI components.

## Web Application Architecture Evolution and Single Responsibility

Web application technology development and evolution represents a continuous process of responsibility separation, implementing the Single Responsibility Principle.

Over a decade ago, in the early internet application days, business was simple and technology primitive. Typically, one class handled one request.

In Java, this meant one Servlet handled one request:

```
[User Request] → [Servlet] → [Database] → [HTML Response]
```

This approach had significant problems: all request processing and response operations happened within the Servlet. The Servlet retrieved request data, performed logical processing, accessed databases, obtained results, and constructed returning HTML based on results. All these responsibilities were completed in one class, particularly HTML output, requiring line-by-line HTML string construction in Servlet:

```java
response.getWriter().println("<html><head><title>servlet program</title></head>");
```

This was painful. HTML files could be large, and piecing together strings bit by bit in code made programming difficult, maintenance difficult—basically everything difficult.

Later came JSP. If Servlet meant outputting HTML within programs, JSP meant calling programs within HTML:

```
[User Request] → [JSP] → [Business Model] → [Dynamic HTML Response]
```

JSP development was easier than Servlet—at least no more painful HTML string concatenation. JSP-based web programs typically achieved basic responsibility separation: page-constructing JSP separated from logic-processing business models. But this separation was incomplete—JSP still contained substantial business logic code coupled with HTML tags.

True view-model separation came with MVC frameworks, which completely separated views from models through controllers. Views contained only HTML tags and template engine placeholders, while business models specialized in business processing. This separation enabled frontend and backend development to become distinct roles—frontend engineers handled view template development, backend engineers handled business development, with no direct dependencies or coupling.

```
[User Request] → [Controller] → [Model] → [View] → [Response]
```

MVC enabled logical business model layering, separating business models into business layer, service layer, and data persistence layer, achieving further responsibility separation and better Single Responsibility Principle compliance.

## The Bottom Line

Returning to our article title: class responsibilities should be single—meaning only one reason should cause class changes. This typically results in less code. In development practice, class files opened in IDEs should ideally not exceed one screen.

In the big data SQL engine example opening this article, the main problem with the SQL syntax processing class was too many functional responsibilities placed in one class. After studying the prototype code and discussing with the original developers, I split this class's responsibilities across two dimensions.

One dimension was the processing workflow: the entire process could be divided into syntax definition, syntax transformation, and syntax generation phases. Each SQL statement required all three phases. Additionally, each SQL statement generated a syntax tree composed of many nodes during processing. From this perspective, each syntax tree node should be handled by a single-responsibility class.

I split the original nearly 10,000-line class along these two dimensions into hundreds of classes, each with relatively single responsibility, handling only one processing phase of one syntax tree node. Many small classes had just a few lines of code, occupying only small portions of the IDE screen—clear at a glance, easy to read and maintain. Classes weren't coupled but were constructed into trees at runtime based on SQL syntax trees, then traversed using the **Composite Pattern** from design patterns.

Subsequent developers joining the project only needed to develop corresponding syntax transformers and syntax tree generators for unsupported SQL syntax features, without modifying or even calling existing classes. At runtime, when syntax processing encountered corresponding syntax nodes, it simply handed them to related classes.

After refactoring, although class count expanded hundreds of times, total code lines decreased significantly. Despite having hundreds of classes instead of one monolithic class, the codebase became more maintainable, extensible, and aligned with the Open-Closed Principle.

---

_What designs in your software development could benefit from applying the Single Responsibility Principle? Have you encountered similar "god classes" that tried to do everything? Share your refactoring experiences in the comments._
