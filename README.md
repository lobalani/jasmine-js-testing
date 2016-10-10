Automated JavaScript Testing using Jasmine
==========================================

Overview
--------
- [Principles of Unit Testing](intro_unit_testing.md)
- Examples of Hard to Test Code
    - Tightly Coupled Components
    - Private Parts
    - Singletons
    - Anonymous Functions
    - Mixed Concerns
    - New Operators
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

Tightly Coupled Components
--------------------------
Having Tightly Coupled Components means that two or more components have a direct reference to each other. Tight coupling makes it difficult to test one component from another.

This makes the code more brittle, as change in one component could break another.

**Example**

 Here we have a polls object with two methods, add and getList. The following submit and view objects rely on polls and interact with its methods. The concern here is that there's a direct reference from submit and view to the polls object. If we change the name of the polls object or its methods or parameters, then we'd have a problem on our hands.

```JavaScript
(function(){
    "use strict";

    var polls = {
        add: function(poll) {
            jQuery.post("/polls", poll);
        },
        getList: function(callback) {
            jQuery.get("/polls", function(data) {
                callback(data.list);
            });
        }
    };

    var submit = {
        add: function(poll) {
            polls.add(poll);
        }
    };

    var view = {
        init: function() {
            polls.getList(this.render);
        },
        render: function(list) {
            list.forEach(function(poll) {
                jQuery("<li>" + poll + "</li>").appendTo("#output");
            });
        }
    };
})();
```

**Refactored/Improved Code**

Here we have our polls objects, which has an add and getList method. These are communicating in the server either to append new polls or to retrieve a list of polls. The submit and view objects have a direct reference to polls, that's where the tight coupling is happening.  Here we're adding a whole bunch of new poles to the server, we're calling view.init, which gets the list from the server, calls render when it's done, loops over them, and inserts a new list item into the DOM for each item in the loop. And that's considered poor performance.

But what if someone comes in here and changes polls add to addPoll. This will break the code. So what can we do to help protect ourselves from that?

This is a small code set so it's easy to see that something is broken, but that would become harder to spot the larger the code base becomes. And we don't want to go and have to search replace a bunch of things. So let's create a pollBridge, which will act as a contract between submit, view, and polls. And so, submit is looking for something called add. So we're going to map that to polls.add and the view is looking for something called getList and so we're going to map that to polls.getList. However, submit and view need to know about this pollBridge and so let's make an init method where we'll get to pass in our pollBridge and we'll save this off locally. And anytime where we use polls, we'll use our local version. In the same way, we'll do the same thing for view, so we'll type polls in here, save this off. And anytime we use polls we'll use the local version. And then the only thing we have to do is call the init methods and pass in pollBridge. So what did that save us? 

If we decide to go to polls and change add to addPoll, then we don't have to go down and change submit and view code, it can stay the same, we just have to change the pollBridge.

It's not good to touch the DOM too many times, so we're going to try to bundle up a whole bunch of changes and then do it one time. So we are introducing a new variable called markup, it's an array. And in our loop here, instead of actually touching the DOM every time, we are going to add a new item to markup array every time. And then when done, we'll say markup.join, and say join on an empty string. That makes one huge string with all the markup from every item in the array. Then put it inside the jQuery function and it will convert all those markup tags in the string to actual real DOM elements. So now it's in memory, so we need to append it to the output, which is where it was before. 

It works, but now we have loosely coupled our components and made the DOM manipulation code a little bit more performant.

```JavaScript
(function(){
    "use strict";

    var polls = {
        add: function(poll) {
            jQuery.post("/polls", poll);
        },
        getList: function(callback) {
            jQuery.get("/polls", function(data) {
                callback(data.list);
            });
        }
    };

    var pollBridge = {
        add: polls.add,
        getList: polls.getList
    };

    var submit = {
        init: function(polls) {
            this.polls = polls;
        },
        add: function(poll) {
            this.polls.add(poll);
        }
    };

    var view = {
        init: function(polls) {
            this.polls = polls;
            this.polls.getList(this.render);
        },
        render: function(list) {
            var markup = [];

            list.forEach(function(poll) {
                markup.push("<li>" + poll + "</li>");
            });
            jQuery(markup.join("")).appendTo("#output");
        }
    };

    submit.init(pollBridge);
})();
```

Private Parts
-------------
Encapsulation and data hiding are great. However, these practices can make testing harder. This might be acceptable, it just depends on what we need.

**Example**

Here we have a person object. We're using the revealing module pattern to give us public and private parts. The way this works is, whatever is returned in the last statement will be public. So here we have a public eat method and everything else is private. So the chew and the swallow functions will be private.

If we really wanted to unit test chew and swallow functions, we would need to expose those as public. So a way we could do that is we could just say chew, map it to the chew function in the return object.

It's not always the best option to make everything public, but if we want to unit test our things, we need to have access to it somehow. We don't have to have 100% unit test coverage, so we might not need to expose everything.

```JavaScript
var person = (function(){
    "use strict";

    var chew = function() {
        console.log('chew');
    };

    var swallow = function() {
        console.log('swallow');
    };

    var eat = function() {
        for (var index = 0, length = 10; index < length; index++) {
            chew();
        }
        swallow();
    };

    return {
        eat : eat
    }
})();

person.eat();
```

Singletons
----------
