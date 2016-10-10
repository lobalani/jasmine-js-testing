Hard to Test JavaScript Code
============================

Overview
--------
- Introduction
- Unit vs Integration: What's the difference?
- Example
- Tightly Coupled Components
- Private Parts
- Singletons
- Anonymous Functions
- Mixed Concerns
- New Operators

<http://alistapart.com/article/writing-testable-javascript>
<https://www.pluralsight.com/blog/software-development/6-examples-of-hard-to-test-javascript>
<https://www.toptal.com/javascript/writing-testable-code-in-javascript>

Introduction
------------
We’ve all been there: that bit of JavaScript functionality that started out as just a handful of lines grows to a dozen, then two dozen, then more. Along the way, a function picks up a few more arguments; a conditional picks up a few more conditions. And then one day, the bug report comes in: something’s broken, and it’s up to us to untangle the mess.

As we ask our client-side code to take on more and more responsibilities two things are becoming clear. One, we can’t just point and click our way through testing that things are working as we expect; automated tests are key to having confidence in our code. Two, we’re probably going to have to change how we write our code in order to make it possible to write tests.

Really, we need to change how we code? Yes—because even if we know that automated tests are a good thing, most of us are probably only able to write integration tests right now. Integration tests are valuable because they focus on how the pieces of an application work together, but what they don’t do is tell us whether individual units of functionality are behaving as expected.

That’s where unit testing comes in. And we’ll have a very hard time writing unit tests until we start writing testable JavaScript.

Unit vs Integration: What's the difference?
-------------------------------------------
Writing integration tests is usually fairly straightforward: we simply write code that describes how a user interacts with our app, and what the user should expect to see as they do.

Whereas an integration test is interested in a user’s interaction with an app, a unit test is narrowly focused on a small piece of code:

>When I call a function with a certain input, do I receive the expected output?

If we write our code with our future unit testing needs in mind, we will not only find that writing the tests becomes more straightforward than we might have expected, but also that we’ll simply write better code, too.

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
