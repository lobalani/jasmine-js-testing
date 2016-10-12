JavaScript Design Patterns
==========================

Overview
--------
- What is a Pattern?
- Why are Patterns important?
- Design Patterns
- Coding Patterns
- AntiPatterns


What is a Pattern?
------------------

In software development, a pattern is a solution to a common problem. A pattern is not necessarily a code solution ready for copy-and-paste but more of a best practice, a useful abstraction, and a template for solving categories of problems.

Why are Patterns important?
---------------------------

Patterns are important because:

- They help us write better code using proven practices and not reinvent the wheel.
- They provide a level of abstraction. The brain can hold only so much at a given time, so when we think about a more complex problem, it helps if we don’t bother with the low-level details but account for them with self-contained building blocks (patterns).
- They improve communication between developers and teams, which are often in remote locations and don’t communicate face to face. Simply putting a label on some coding technique or approach makes it easier to make sure we’re talking about the same thing. For example, it’s easier to say (and think) “immediate func- tion,” than “this thing where you wrap the function in parentheses and at the end of it put another set of parentheses to invoke the function right where you’ve de- fined it.”

Design Patterns
---------------

Design patterns are those initially defined by the “Gang of Four” book. Examples of design patterns are singleton, factory, decorator, observer, and so on. 

The thing about design patterns in relation to JavaScript is that, although language-independent, the design patterns were mostly studied from the perspective of strongly typed languages, such as C++ and Java. Sometimes it doesn’t necessarily make sense to apply them verbatim in a loosely typed dynamic language such as JavaScript. Sometimes these patterns are workarounds that deal with the strongly typed nature of the languages and the class-based inheritance. In JavaScript there might be simpler alternatives.

We will discuss JavaScript implementations of several Design Patterns.

Coding Patterns
---------------

They are JavaScript-specific patterns and good practices related to the unique features of the language, such as the various uses of functions. JavaScript coding patterns are our main topic of discussion.

AntiPatterns
------------

An antipattern is not the same as a bug or a coding error; it’s just a common approach that causes more problems than it solves.