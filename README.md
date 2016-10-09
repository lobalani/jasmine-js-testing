Automated JavaScript Testing using Jasmine
==========================================

Overview
--------
- Principles of Unit Testing
    - What are Unit Tests?
    - Unit Testing Concepts
    - Why Resist?
- Examples of Hard to Test Code
- Unit Testing Front-End App
- Mocking Front-End App
- Prototyping Front-End App
- Integrating with Front-End Frameworks

What are Unit Tests?
--------------------
A Unit Test is composed of a small section of a code, refered to as a *Unit*. We test this Unit of code, verifying its functionality in order to increase confidence and reduce the overall risk of our application.

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

Lets see below on how to use them more efficiently.
