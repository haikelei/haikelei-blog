---
author: Gary Lu
pubDatetime: 2025-01-05T10:00:00+08:00
modDatetime: 2025-01-05T10:00:00+08:00
title: "Component Design Principles: Where Are Component Boundaries?"
slug: component-design-principles-boundaries
featured: false
draft: false
tags:
  - software-architecture
  - component-design
  - design-principles
  - software-engineering
description: Exploring component design principles, examining both cohesion and coupling aspects to determine optimal component boundaries in complex software systems.
---

# Component Design Principles: Where Are Component Boundaries?

Software complexity has an **exponential** relationship with its scale. A software system with complexity 100, if split into two unrelated subsystems of equal size, should have complexity 25 for each subsystem, not 50. The software development industry established a consensus long ago: complex software systems should be decomposed into multiple lower-complexity subsystems, which can be further broken down into smaller components. In other words, software needs modular and component-based design.

In fact, programmers began attempting component-based software design as early as the punch card era. Relatively independent, reusable programs were punched onto paper tape cards and stored in boxes. When a program needed to reuse a component, developers would take that stack of tape cards from the box and place them before or after other tapes to be run, scanning and executing them together with photoelectric readers.

Our current component development and reuse isn't much different. In Java development, we compile independent components into JAR packages—essentially encapsulating components in individual boxes. When reuse is needed, programs simply depend on these JARs and place them in the `classpath` at runtime for unified JVM loading and execution.

Today, any reasonably-sized software system is decomposed into many components. Component-based design enables us to develop complex systems.

So how do we design components? How large should component granularity be? How do we divide component functionality? Where are component boundaries?

As we've said before, software design's core goal is **high cohesion and low coupling**. Today, let's examine component design principles from these two dimensions.

## Component Cohesion Principles

**Component cohesion principles** mainly discuss which classes should be aggregated in the same component so that components can provide relatively complete functionality without becoming too large. In specific design, we can follow these three principles:

### Reuse-Release Equivalence Principle

The Reuse-Release Equivalence Principle states: **The granularity of software reuse should be equivalent to its release granularity.** In other words, if you want others to reuse your software at a certain granularity, you should release your software at that same granularity. This is essentially the definition of components—components are the smallest granular software units for both reuse and release.

Additionally, if your released components will continuously change, you should manage component versions with version numbers so component users can know whether they need to upgrade versions and whether component incompatibilities will occur. Therefore, component version numbers should follow commonly accepted conventions.

Here's a suggested version numbering convention: Major.Minor.Patch (e.g., 1.3.12, where major version is 1, minor version is 3, patch version is 12). Major version upgrades indicate non-backward-compatible significant revisions; minor version upgrades indicate important functional revisions or bug fixes that remain backward-compatible; patch upgrades indicate unimportant functional revisions or bug fixes.

### Common Closure Principle

The Common Closure Principle states: **We should place classes that change at the same time and for the same reasons into the same component.** Classes that don't change simultaneously or for the same reasons should be placed in different components.

While components exist for reuse, what often causes problems during development is component maintainability. If components must undergo various changes throughout their lifecycle, it's best not to involve other components—all related changes should occur within the same component. This way, when changes happen, only that component needs republishing rather than affecting many components.

Perhaps placing certain classes in a component is convenient and reasonable for reuse, but if component reuse conflicts with maintenance—for example, if these classes' future changes aren't synchronized with the overall component's future changes—maintainability should be carefully considered when deciding whether to include these classes in the component.

### Common Reuse Principle

The Common Reuse Principle states: **Don't force component users to depend on things they don't need.**

On one hand, this principle means we should place mutually dependent, commonly reused classes in the same component. For example, a data structure container component providing arrays, hash tables, and various data structure containers should also include classes for data structure traversal and sorting, enabling classes within the component to jointly provide external services.

On the other hand, this principle indicates that classes not commonly depended upon shouldn't be placed in the same component. If non-dependent classes change, it triggers component changes, which in turn causes programs using the component to change. This creates unnecessary trouble for component users, potentially causing them to dislike using such components and making component reuse difficult.

These three component cohesion principles conflict with each other. For example, the Common Reuse Principle and Common Closure Principle—one emphasizes easy reuse while the other emphasizes easy maintenance, creating conflict. Therefore, these principles can guide component design considerations, but making correct design decisions requires architects' experience and scenario understanding to balance these principles.

## Component Coupling Principles

Component cohesion principles discuss what functionality and classes components should contain, while component coupling principles discuss how coupling relationships between components should be designed. Component coupling relationship design should follow three principles:

### Acyclic Dependencies Principle

The Acyclic Dependencies Principle states: **Component dependency relationships should not contain cycles.** If Component A depends on Component B, Component B depends on Component C, and Component C depends on Component A, a circular dependency forms.

Often, circular dependencies gradually form during component changes. Component A version 1.0 depends on Component B version 1.0. Later, Component B upgrades to 1.1, and some upgraded functionality depends on Component A version 1.0, creating circular dependency. If component design boundaries are unclear, component development lacks reviews, developers only focus on their own components, and projects lack unified component dependency management rules, circular dependencies are likely.

Once circular dependencies appear in systems, they become extremely unstable. A minor bug can cause chain reactions and mysterious problems elsewhere. Sometimes systems that worked fine yesterday won't start today without any changes.

Developing code in systems with serious circular dependencies feels like programming in tar pits—teams are afraid to change anything and can't move anything, experiencing only anxiety and frustration.

### Stable Dependencies Principle

The Stable Dependencies Principle states: **Component dependency relationships must point toward more stable directions.** Components with few changes are stable; components with frequent changes are unstable. According to this principle, unstable components should depend on stable components, not vice versa.

Conversely, if a component is depended upon by many components, it needs to be relatively stable because changing a component depended upon by many others is inherently difficult. Similarly, if a component depends on many other components, it's relatively unstable because changes in any dependency might cause its own changes.

Simply put, the Stable Dependencies Principle means: **Components shouldn't depend on components that are less stable than themselves.**

### Stable Abstractions Principle

The Stable Abstractions Principle states: **A component's abstraction level should be consistent with its stability level.** Stable components should be abstract, while unstable components should be concrete.

This principle's guidance for concrete development is: if your designed component is concrete and unstable, you can design interface sets for classes providing external services and encapsulate these interfaces in specialized components, making them relatively abstract and stable.

In practice, this abstract interface component design should follow the Dependency Inversion Principle I discussed earlier. Abstract interface components shouldn't be defined by low-level concrete implementation components but by high-level using components. High-level using components depend on interface components for programming, while low-level implementation components implement interface components.

JDBC in Java exemplifies this. JDK defines JDBC interface components in the `java.sql` package. When developing applications, we only need to program using JDBC interfaces. When releasing applications, we specify concrete implementation components—either MySQL's JDBC components or Oracle's JDBC components.

## The Bottom Line

Component boundaries and dependency relationship division must consider not only technical issues but also business scenarios. Variability versus stability, dependence versus being depended upon—all need examination within business contexts. Sometimes it's not just technical and business issues; human factors must be considered too. In complex organizations, component dependencies and design need to account for human elements. If component functionality division involves departmental responsibility boundaries, it might even connect to company politics.

Therefore, company technical depth and strength, business situations, departmental and team relationships, and even company history can all influence component design. Those who deeply understand these situations are usually company "veterans." So older programmers don't necessarily need to compete with younger programmers on technology or stamina—they should leverage their strengths to do more valuable things for themselves and companies.

---

_Have you encountered systems with circular dependencies? How did you break the cycles? What other examples of stable abstraction principles (like JDBC) have you worked with? Share your experiences in the comments._
