---
title: SOLID Principles
date: 2019-09-15 18:32:11
tags:
- Programming
- Reading
---

The SOLID principles tell us how to arrange our functions and data structures into classes, and how those classes should be interconnected. The use of the word “class” does not imply that these principles are applicable only to object-oriented software. A class is simply a coupled grouping of functions and data. Every software system has such groupings, whether they are called classes or not. The SOLID principles apply to those groupings.

<!-- more -->

The goal of the principles is the creation of mid-level software structures that:
- Tolerate change,
- Are easy to understand, and
- Are the basis of components that can be used in many software systems.


> **Single responsibility principle**
>
> A module should be responsible to one, and only one, actor.
>
> A class should only have a single responsibility, that is, only changes to one part of the software's specification should be able to affect the specification of the class.

> **Open–closed principle**
>
> Software entities should be open for extension, but closed for modification.
> 
> The goal is to make the system easy to extend without incurring a high impact of change. This goal is accomplished by partitioning the system into components, and arranging those components into a dependency hierarchy that protects higher-level components from changes in lower-level components.

> **Liskov substitution principle**
>
> Objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program.

> **Interface segregation principle**
>
> Many client-specific interfaces are better than one general-purpose interface. Depending on something that carries baggage that you don’t need can cause you troubles that you didn’t expect.

>**Dependency inversion principle**
>
> One should depend upon abstractions, **not** concretions. Most flexible systems are those in which source code dependencies refer only to abstractions, not to concretions.
>
> Don’t refer to volatile concrete classes; Don’t derive from volatile concrete classes; Don’t override concrete functions; Never mention the name of anything concrete and volatile.
