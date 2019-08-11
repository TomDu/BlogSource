---
title: Clean Code - Smells and Heuristics
date: 2019-08-11 16:49:27
tags:
- Programming
- Reading
- Java
---

From ***Clean Code, [Robert C. Martin](https://en.wikipedia.org/wiki/Robert_C._Martin)***

---

# COMMENTS

**C1: Inappropriate Information**

Comments should be reserved for technical notes about the code and design.

**C2: Obsolete Comment**

 Comments get old quickly. It is best not to write a comment that will become obsolete. If you find an obsolete comment, it is best to update it or get rid of it as quickly as possible.

<!-- more -->

 **C3: Redundant Comment**

 Comments should say things that the code cannot say for itself.

 **C4: Poorly Written Comment**

 A comment worth writing is worth writing well.

 **C5: Commented-Out Code**

 When you see commented-out code, delete it!

---

# ENVIRONMENT

**E1: Build Requires More Than One Step**

You should be able to check out the system with one simple command and then issue one other simple command to build it.

**E2: Tests Require More Than One Step**

You should be able to run all the unit tests with just one command.

---

# FUNCTIONS

**F1: Too Many Arguments**

No argument is best, followed by one, two, and three. More than three is very questionable and should be avoided with prejudice.

**F2: Output Arguments**

Output arguments are counterintuitive.

**F3: Flag Arguments**

Boolean arguments loudly declare that the function does more than one thing. They are confusing and should be eliminated.

**F4: Dead Function**

Methods that are never called should be discarded.

---

# GENERAL

**G1: Multiple Languages in One Source File**

The ideal is for a source file to contain one, and only one, language.

**G2: Obvious Behavior Is Unimplemented**

Any function or class should implement the behaviors that another programmer could reasonably expect.

**G3: Incorrect Behavior at the Boundaries**

Don’t rely on your intuition. Look for every boundary condition and write a test for it.

**G4: Overridden Safeties**

It is risky to override safeties.

**G5: Duplication**

Find and eliminate duplication wherever you can.

**G6: Code at Wrong Level of Abstraction**

It is important to create abstractions that separate higher level general concepts from lower level detailed concepts. Isolating abstractions is one of the hardest things that software developers do, and there is no quick fix when you get it wrong.

**G7: Base Classes Depending on Their Derivatives**

In general, base classes should know nothing about their derivatives.

**G8: Too Much Information**

Well-defined modules have very small interfaces that allow you to do a lot with a little. Good software developers learn to limit what they expose at the interfaces of their classes and modules.

Hide your data. Hide your utility functions. Hide your constants and your temporaries. Don’t create classes with lots of methods or lots of instance variables. Don’t create lots of protected variables and functions for your subclasses. Concentrate on keeping interfaces very tight and very small. Help keep coupling low by limiting information.

**G9: Dead Code**

When you find dead code, do the right thing. Give it a decent burial. Delete it from the system.


**G10: Vertical Separation**

Variables and function should be defined close to where they are used.

Private functions should be defined just below their first usage.

**G11: Inconsistency**

Be careful with the conventions you choose, and once chosen, be careful to continue to follow them.

**G12: Clutter**

Keep your source files clean, well organized, and free of clutter.

**G13: Artificial Coupling**

Things that don’t depend upon each other should not be artificially coupled.

**G14: Feature Envy**

The methods of a class should be interested in the variables and functions of the class they belong to, and not the variables and functions of other classes. When a method uses accessors and mutators of some other object to manipulate the data within that object, then it envies the scope of the class of that other object. It wishes that it were inside that other class so that it could have direct access to the variables it is manipulating.

**G15: Selector Arguments**

In general it is better to have many functions than to pass some code into a function to select the behavior.

**G16: Obscured Intent**

Make code as expressive as possible.

**G17: Misplaced Responsibility**

Code should be placed where a reader would naturally expect it to be.

**G18: Inappropriate Static**

In general you should prefer nonstatic methods to static methods. If you really want a function to be static, make sure that there is no chance that you’ll want it to behave polymorphically.

**G19: Use Explanatory Variables**

One of the more powerful ways to make a program readable is to break the calculations up into intermediate values that are held in variables with meaningful names.

**G20: Function Names Should Say What They Do**

Yep.

**G21: Understand the Algorithm**

Know how your function works.

**G22: Make Logical Dependencies Physical**

If one module depends upon another, that dependency should be physical, not just logical.

**G23: Prefer Polymorphism to If/Else or Switch/Case**

Consider polymorphism before using a switch.

**G24: Follow Standard Conventions**

Every team should follow a coding standard based on common industry norms.

**G25: Replace Magic Numbers with Named Constants**

It is a bad idea to have raw numbers in your code. You should hide them behind well-named constants.

**G26: Be Precise**

When you make a decision in your code, make sure you make it precisely. Know why you have made it and how you will deal with any exceptions.

**G27: Structure over Convention**

Enforce design decisions with structure over convention.

**G28: Encapsulate Conditionals**

Extract functions that explain the intent of the conditional.

**G29: Avoid Negative Conditionals**

Negatives are just a bit harder to understand than positives.

**G30: Functions Should Do One Thing**

Functions that do more than one thing should be converted into many smaller functions, each of which does one thing.

**G31: Hidden Temporal Couplings**

Temporal couplings are often necessary, but you should not hide the coupling.

**G32: Don’t Be Arbitrary**

Have a reason for the way you structure your code, and make sure that reason is communicated by the structure of the code.

**G33: Encapsulate Boundary Conditions**

Boundary conditions are hard to keep track of. Put the processing for them in one place. Don’t let them leak all over the code.

**G34: Functions Should Descend Only One Level of Abstraction**

The statements within a function should all be written at the same level of abstraction, which should be one level below the operation described by the name of the function.

**G35: Keep Configurable Data at High Levels**

Make it easy to config.

**G36: Avoid Transitive Navigation**

In general we don’t want a single module to know much about its collaborators.

---

# JAVA

**J1: Avoid Long Import Lists by Using Wildcards**

Long lists of imports are daunting to the reader.

**J2: Don’t Inherit Constants**

A hideous practice.

**J3: Constants versus Enums**

Use enums.

---

# NAMES

**N1: Choose Descriptive Names**

Names in software are 90 percent of what make software readable. You need to take the time to choose them wisely and keep them relevant. Names are too important to treat carelessly.

**N2: Choose Names at the Appropriate Level of Abstraction**

Choose names the reflect the level of abstraction of the class or function you are working in.

**N3: Use Standard Nomenclature Where Possible**

Names are easier to understand if they are based on existing convention or usage.

**N4: Unambiguous Names**

Choose names that make the workings of a function or variable unambiguous.

**N5: Use Long Names for Long Scopes**

The length of a name should be related to the length of the scope.

**N6: Avoid Encodings**

Names should not be encoded with type or scope information.

**N7: Names Should Describe Side-Effects**

Names should describe everything that a function, variable, or class is or does.

---

# TESTS

**T1: Insufficient Tests**

A test suite should test everything that could possibly break. The tests are insufficient so long as there are conditions that have not been explored by the tests or calculations that have not been validated.

**T2: Use a Coverage Tool!**

Coverage tools reports gaps in your testing strategy. They make it easy to find modules, classes, and functions that are insufficiently tested.

**T3: Don’t Skip Trivial Tests**

They are easy to write and their documentary value is higher than the cost to produce them.

**T4: An Ignored Test Is a Question about an Ambiguity**

Sometimes we are uncertain about a behavioral detail because the requirements are unclear. We can express our question about the requirements as a test that is commented out, or as a test that annotated with @Ignore.

**T5: Test Boundary Conditions**

Take special care to test boundary conditions.

**T6: Exhaustively Test Near Bugs**

Bugs tend to congregate. When you find a bug in a function, it is wise to do an exhaustive test of that function.

**T7: Patterns of Failure Are Revealing**

Sometimes you can diagnose a problem by finding patterns in the way the test cases fail. Complete test cases, ordered in a reasonable way, expose patterns.

**T8: Test Coverage Patterns Can Be Revealing**

Looking at the code that is or is not executed by the passing tests gives clues to why the failing tests fail.

**T9: Tests Should Be Fast**

A slow test is a test that won’t get run.
