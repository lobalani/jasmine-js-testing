Principles of Unit Testing
==========================

Overview
--------
- What are Unit Tests?
- Properties of a good Unit Test
- Unit Testing Concepts
- Why Resist?

What are Unit Tests?
--------------------
A Unit Test is composed of a small section of a code, refered to as a *Unit*. We test this Unit of code, verifying its functionality in order to increase confidence and reduce the overall risk of our application.

**DEFINITION**
A *Unit Test* is a piece of code that invokes a unit of work and checks one specific end result of that unit of work. If the assumptions on the end result turn out to be wrong, the unit test has failed. A unit test’s scope can span as little as a method or as much as multiple classes. A unit test is almost always written using a unit testing framework. It can be written easily and runs quickly. It’s trust- worthy, readable, and maintainable. It’s consistent in its results as long as pro- duction code hasn’t changed.

Properties of a good Unit Test
------------------------------
A unit test should have the following properties:
- It should be automated and repeatable.
- It should be easy to implement.
- It should be relevant tomorrow.
- Anyone should be able to run it at the push of a button.
- It should run quickly.
- It should be consistent in its results (it always returns the same result if you don’t change anything between runs).
- It should have full control of the unit under test.
- It should be fully isolated (runs independently of other tests).
- When it fails, it should be easy to detect what was expected and determine how
to pinpoint the problem.
- It should be incremental, meaning that the unit test should be updated if a new relevant defect is detected in the code, which means that this defect will not happen again as long as this unit test is running periodically.


Unit Testing Concepts
---------------------
The Unit Tests should:
- Be Predictable
    - If we have certain set of inputs, then we should get the same Output each time we run the tests.
    - Tests which use **dynamic data**, like current Date, are more prone to errors, so use **static data** which are more predictable.
- Be easily Reproducible
- Pass or Fail
    - Tests should either Pass or Fail
    - Tests that always Pass or always Fail are BAD tests
- Be Self Documenting
    - The naming of the tests should describe what each test is meant to do
    - Helps someone looking at the test to quickly assert what the test is testing
- Have Single Responsibility
    - Unit Tests should be testing only one thing at a time
    - To test more things, split them into multiple unit tests
- Provide Useful Error Messages
    - Each test should have descriptive error messages, so as to tell what went wrong when a Unit Test fails

*NOTE:* Unit Tests are not Integration Tests. The focus should not be to test multiple components working together, but rather, to focus on testing one feature of one component at a time.

Why Resist?
----------
Despite common excuses writing Unit Tests are valuable and can save time, increase confidence, and reduce risk. It might be time consuming to write tests at first, but it becomes easier with time and practice.

Unit Tests should be predictable, atomic, self documenting, focused, and easy to diagnose.