JavaScript Styleguide
=====================

Overview
--------
- Basic Formatting
- Indentation Levels
- Statement Termination
- Line Length
- Line Breaking
- Blank Lines
- Naming

Basic Formatting
----------------

At the core of a style guide are basic formatting rules. These rules govern how the code is written at a high level. Basic formatting rules guide developers toward writing code in a particular style. These rules often contain information about syntax that developers may not have considered, but every piece is important in creating a coherent piece of code.

Indentation Levels
------------------

Properly indented code is much easier to understand. Ensuring proper indentation is the first step.

There is no universal agreement on how to accomplish indentation in code. There are two schools of thought:

**Use Tabs for indentation**
Each indentation level is represented by a single tab character. So indents of one level are one tab character, second-level indentation is two tab characters, and so on. 

*Pros*
- There is a one-to-one mapping between tab characters and indentation levels, making it logical.
- Text editors can be configured to display tabs as different sizes, so developers who like smaller indents can configure their editors that way, and those who like larger indents can work their way, too.

*Cons*
- Systems interpret tabs differently.
- We may find that opening the file in one editor or system looks quite different than in another, which can be frustrating.

**Use Spaces for indentation**

Each indentation level is made up of multiple space characters. Within this realm of thinking, there are three popular approaches: two spaces per indent, four spaces per indent, and eight spaces per indent. These approaches all can be traced back to style guidelines for various programming languages. 

For reference, here are some indentation guidelines from various style guides:
- The jQuery Core Style Guide specifies indents as tabs.
- Douglas Crockford’s Code Conventions for the JavaScript Programming Language specifies indents as four spaces.
- The SproutCore Style Guide specifies indents as two spaces.
- The Google JavaScript Style Guide specifies indents as two spaces.
- The Dojo Style Guide specifies indents as tabs.

*Pros*
- Files are treated exactly the same in all editors and all systems. Text editors can be configured to insert spaces when the Tab key is pressed. That means all developers have the same view of the code. 

*Cons*
- It is easy for a single developer to create formatting issues by having a misconfigured text editor.

**RECOMMENDATION:**
Use four spaces per indentation level.

>**WARNING!**
>
>Even though the choice of tabs or spaces is a preference, it is very important not to mix them. Doing so leads to horrible file layout and requires cleanup work.

Statement Termination
---------------------



Line Length
-----------


Line Breaking
-------------


Blank Lines
-----------


Naming
------

