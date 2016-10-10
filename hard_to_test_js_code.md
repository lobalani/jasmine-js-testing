Hard to Test JavaScript Code
============================

Overview
--------
- Facts of Life
- Introduction
- Unit vs Integration: What's the difference?
- Example : Traditional Implementation
- What makes it hard?
- Hard to Test Parts
- Tightly Coupled <Components></Components>
- Private Parts
- Singletons
- Anonymous Functions
- Mixed Concerns
- New Operators
- Guiding Priciples

Facts of Life
-------------

>*You will design your code*
>
>*You will test your code*
>
>*You will refactor your code*
>
>*Other people will use your code*
>
>*There will be BUGS!*

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

Example : Traditional Implementation
------------------------------------

```JavaScript
var tmplCache = {};

function loadTemplate (name) {
  if (!tmplCache[name]) {
    tmplCache[name] = $.get('/templates/' + name);
  }
  return tmplCache[name];
}

$(function () {

  var resultsList = $('#results');
  var liked = $('#liked');
  var pending = false;

  $('#searchForm').on('submit', function (e) {
    e.preventDefault();

    if (pending) { return; }

    var form = $(this);
    var query = $.trim( form.find('input[name="q"]').val() );

    if (!query) { return; }

    pending = true;

    $.ajax('/data/search.json', {
      data : { q: query },
      dataType : 'json',
      success : function (data) {
        loadTemplate('people-detailed.tmpl').then(function (t) {
          var tmpl = _.template(t);
          resultsList.html( tmpl({ people : data.results }) );
          pending = false;
        });
      }
    });

    $('<li>', {
      'class' : 'pending',
      html : 'Searching &hellip;'
    }).appendTo( resultsList.empty() );
  });

  resultsList.on('click', '.like', function (e) {
    e.preventDefault();
    var name = $(this).closest('li').find('h2').text();
    liked.find('.no-results').remove();
    $('<li>', { text: name }).appendTo(liked);
  });

});
```

On any given line, we might be dealing with presentation, or data, or user interaction, or application state. Who knows! It’s easy enough to write integration tests for this kind of code, but it’s hard to test individual units of functionality.

What makes it hard?
-------------------

- A general lack of structure; almost everything happens in a `$(document).ready()` callback, and then in anonymous functions that can’t be tested because they aren’t exposed.
- Complex functions; if a function is more than 10 lines, like the submit handler, it’s highly likely that it’s doing too much.
- Hidden or shared state; for example, since `pending` is in a closure, there’s no way to test whether the pending state is set correctly.
- Tight coupling; for example, a `$.ajax` success handler shouldn’t need direct access to the DOM.

Hard to Test Parts
------------------

- Tightly Coupled Components
- Private Parts
- Singletons
- Anonymous Functions
- Mixed Concerns
- New Operators

Tightly Coupled Components
--------------------------

Our first hard to test scenario is when we have tightly coupled components. This means that two or more components have a direct reference to each other. This code is characterized by the following:

- It is difficult if you need to test one component apart from the other
- Makes code more brittle because a change to one piece could break another

**Example**

Let's take a look at the following code snippet:

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

The polls object has two methods: `add` and `getList`. The `submit` and `view` objects rely on `polls` and interact with its methods.

The concern here is that there is a direct reference from `submit` and `view` to the `polls object`. If we changed the name of the `polls` object or its methods or parameters, then we would have a problem on our hands. In this case the code is small, but imagine if these were large pieces of code.

In order to resolve this issue we could relax the coupling by passing `polls` in the `submit` and view objects. We would basically be manually injecting the dependency at this point. Another way to solve this issue is to have the components communicate to each other using a message bus.

**Refactored Code**

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

In the above code refactor we introduced a `pollBridge` object that will act as a contract between the various components. We pass the bridge into the `submit` and `view`.

By adding a bridge we are reducing the tight coupling between the various components. For example, if the `polls` `add` method was changed to `addPoll` we would only need to change the `pollBridge` mapping to `add: polls.addPoll`, and wouldn't need to change any code in the submit object since it uses the bridge and not the `polls` object directly.

Private Parts
-------------

Encapsulation and data hiding are great. However, doing so can make testing harder. This might be acceptable, it just depends on what we need.

We don't need to have 100% unit test coverage so this may be okay. However, if we do want to test these areas then we may need to expose them somehow.

**Example**

Let's take a look at the following code snippet:

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

In the above code snippet we are using the revealing module pattern and returning a person object with a public method called eat. If this pattern sounds unfamiliar, the idea is that whatever is returned at the bottom of the IIFE (Immediately Invoked Function Expression) will be public while everything else will be private to the closure.

Internally there are 2 private functions named `chew` and `swallow`. The public `eat` method calls the `chew` function 10 times followed by the `swallow` method.

If we wanted to unit test either the `chew` or `swallow` functions then we'd be out of luck. There isn't a way to get at that functionality directly. However, if we did want to unit test that code then we'd need to expose those methods as well.

Singletons
----------

A singleton is one of the Gang of Four design patterns and it's a known and popular pattern. The essence of the pattern is that you can have only one instance of an object. Some concerns that can come into play are:

- It isn't so good when unit testing multiple use cases
- You may need to reset the singleton's state for each test

**Example**

Let's take a look at the following code snippet:

```JavaScript
var data = {
    token: null,
    users: []
};

function init (username, password) {
    data.token = username + password;
}

function addUser (user) {
    if (data.token) { data.users.push(user); }
}
```

The previous code snippet has a `data` object, which serves as our singleton. The singleton is internally being used by the `ini`t and `addUser` methods.

If we were writing a unit test on the `ini`t function and then moved to test the `addUser` function we would run into a problem if we wanted to verify what happens when there isn't a token set. The token would have been already set in the data singleton since it is shared across tests. In cases like this we'll probably need to reset the data singleton between tests. This is just something we'll need to consider and keep in mind if you use singletons.

**Refactored Code**

```JavaScript
var newData = function() {
    return {
        token: null,
        users: []
    };
};

var users = {
    init: function(data) {
        this.data = data;
    },
    setToken: function(username, password) {
        this.data.token = username + password;
    },
    addUser: function(user) {
        if ( this.data.token ) { this.data.users.push(user); }
    
};

users.init(newData());
users.setToken( "eliahmanor", "password" );
users.addUser({ username: "elijahmanor" });
```

In the above refactor code we created a factory function called `newData` that will make a new object with properties that are initialized to the correct values. By doing so each unit test can create its own fresh version of data to use as it is being tested. Using this technique can get around the need to reset the values to some known state between each test.

Anonymous Functions
-------------------

Another technique that can be problematic when testing is using anonymous functions. These are very convenient to use.

The issue with having so many anonymous functions is that it isn't easy to test the callback in isolation since there is no name or handle to target the function.

**Example**

Let's take the following code snippet for example and examine why this might be a problem.

```JavaScript
$.ajax({
    url: "/people",
    success: function (data) {
        var $list = $("#list");
        $.each(data.people, function (index, person) {
            $("<li />", {
                text: person.fullName
            }).appendTo($list);
        });
    }
});
```

In the above snippet we are calling the jQuery `ajax` method to request a list of people from the server. We have set an anonymous callback function to the `success` option parameter. Although this is a very common way of coding this, it does make testing the code in the callback difficult without actually making an Ajax request.

If we really wanted to unit test the callback, we'd either need to make the actual Ajax request or simulate the Ajax request using a stub or a library like Mockjax. However, with some minimal refactoring we can provide a clean separation that will enable us to test the callback in isolation.

**Refactored Code**

A possible refactored solution could be the following:

```JavaScript
function render(data) {
    var $list = $("#list");

    $.each(data.people, function (index, person) {
        $("<li />", {
            text: person.fullName
        }).appendTo($list);
    });
}

$.ajax({
    url: "/people",
    success: render
});
```

All we did was moved out the `success` method into its own function so that we can unit test the `render` function apart from actually making an Ajax request.

Mixed Concerns
--------------

We should not write code that tries to do too many things in one method, especially if they don't go together. For example, it's a good idea to separate DOM code from data manipulation code.

Some issues we can run into when having mixed concerns are:
- Testing code with mixed concerns can be very awkward and require more setup and verification code
- It often requires that we know more details about the Implementation

**Example**

Let's take the following code snippet for example and examine why this might be a problem.

```JavaScript
var people = {
    list: [],
    add: function (person) {
        this.list.push(person);
        $("#numberOfPeople").html(this.list.length);
    }
};
```

The above snippet has mixed concerns. The `add` method takes its parameter and adds it to an internal `list` array property, which seems appropriate. However, it also updates the DOM in the same method.

The method mixes data and presentation concerns, which is typically not a good idea. It would be better if the `add` method didn't update the DOM directly, but rather publish a message or possibly a higher level method could update the DOM after the `add` method was called.

**Refactored Code: Solution#1**

A possible refactored solution could be the following:

```JavaScript
var people = {
    list: [],
    add: function (person) {
        this.list.push(person);
        $(this).trigger("person.added", [person]);
    },

    addAndRender: function(person) {
        this.add(person);
        this.render(person);
    },

    render: function(person) {
        $("#numberOfPeople").html(this.list.length);
        $("#lastPersonAdded").html(person.name);
    }
};

people.addAndRender({ name: "Brendan Eich" });
people.addAndRender({ name: "John Resig" });
```

The above code refactor uses a new `addAndRender` method to help combine the two different actions that are being performed.

**Refactored Code: Solution#2**

Let's take a look at another implementation that uses an event to communicate that something has happened.

```JavaScript
var people = {
    list: [],
    init: function() {
        $(this).on("person.added", this.render.bind(this));
    },

    add: function (person) {
        this.list.push(person);
        $(this).trigger("person.added", [person]);
    },

    render: function(e, person) {
        $("#numberOfPeople").html(this.list.length);
        $("#lastPersonAdded").html(person.name);
    }
};

people.init();
people.add({ name: "Brendan Eich" });
people.add({ name: "John Resig" });
```

The above code uses a custom event called `person.added` to handle the communication that the DOM needs to be updated. Our `init` method helps wire-up the render method to be triggered when our `person.added` message is triggered.

Regardless either refactor solution is better than the initial code in that the concerns are now split up into separate methods.

New Operators
-------------

Another issue that can make unit testing difficult is using the `new` operator frequently in the application code. 

Instead, we might consider injecting dependencies into component or provide enough information needed for it to create itself.

**Example**

Let's take the following code snippet for example and examine why this might be a problem.

```JavaScript
// ... more code ...

nextBirthday: function () {
    var now = new Date(),
        next = Number.MAX_VALUE;
        person;
    $.each(this.list, function (index, item) {
        /* ... */
    });

    return person;
}

// ... more code ...
```

The previous code is a small snippet to determine whose birthday is next from an array of individuals. The implementation is fairly straightforward, but the reason it is problematic is because it creates a `new Date` which ends up being the current date and time. That sounds reasonable at first thought, but it becomes a nuisance when we want to unit test the behavior.

**Refactored Code**

The following code shows a simple way to get around this issue:

```JavaScript
var people = {
    list: [],
    add: function(person) { this.list.push(person); },

    nextBirthday: function (date) {
        var now = date ? new Date(date) : new Date(),
            next = Number.MAX_VALUE, person;

        $.each(this.list, function (index, item) {
            var dob = new Date(item.dob),
                year = dob.setFullYear(now.getFullYear()) > now ?
                    now.getFullYear() : now.getFullYear() + 1,
                diff = dob.setFullYear(year) - now;

            if (diff < next) { next = diff; person = item; }
        });

        return person;
    }
};

people.add({ name: "Mark", dob: "12/19/1976" });
people.add({ name: "Jane", dob: "10/18/1979" });
people.add({ name: "Jim", dob: "8/17/1983" });
people.add({ name: "Mary", dob: "7/9/1981" });
people.add({ name: "Alex", dob: "8/17/2010" });
people.add({ name: "Bob", dob: "3/1/1985" });
people.add({ name: "John", dob: "1/1/1982" });
people.add({ name: "Sue", dob: "2/1/1987" });

console.log(people.nextBirthday("12/15/2013").name);
```

At the bottom we are building up a list of people with their birthdays and then console.logging whose birthday is next.

This seems great at first, but then we realize that we aren't necessarily testing all the code paths in the `nextBirthday` method. It is dependant on the current Date! Are you testing for birthdays that have already happened this year? It would be much better if we could control the so-called-current-date so we could be certain what edge cases are in fact being tested.

The easiest way to fix this code is to pass in an optional date parameter that can override what the date is set to. It's a pretty easy fix, but it provides much more flexibility to the system.

So, the main point here is to just be aware of what objects are created and make sure there is a way to control them for testability.

Guiding Priciples
-----------------

The above problematic code snippets aren't the only types of code you should watch out for, but it is a good start and should hopefully open your eyes to some of the common pitfalls you could run into when developing your application. To review here are some concepts you should keep in mind:

- Try not to tightly couple your components.
- Be aware that anything you make private will be unavailable to test.
- Limit the use of singletons, otherwise you'll need to reset them. Use constructors to create instances.
- Be careful of using too many anonymous functions.
- Try not to mix various non-related concerns in your code. Write Pure Functions.
- Keep methods simple.
- Do not intermingle responsibilities.
- Be aware of the new operator when the constructor does work for you.