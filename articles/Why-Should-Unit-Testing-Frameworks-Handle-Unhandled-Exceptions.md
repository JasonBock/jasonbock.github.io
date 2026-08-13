---
title: Why Should Unit Testing Frameworks Handle Unhandled Exceptions?
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Why Should Unit Testing Frameworks Handle Unhandled Exceptions?

A couple of days ago I was glancing though a book on unit testing sitting on a fellow dev's desk, and one of the early examples showed how you can write a unit test all by yourself. The author was trying to show the concept without introducing a framework at the same time and I liked that. What I didn't like was the way exception handling was done:

```c#
try
{
  // Unit test code goes here...
}
catch(Exception e)
{
  // Reporting the exception goes here...
}
```

Ugh. I so cannot stand when I see books and articles do this. This is such a bad practice in coding - the number of times I write code to handle the general exception is ... well, I honestly can't remember when I've done it. It's not completely verboten - there are valid cases when you have to - but there's so rare that in 99.999999856% of your exception handlers you should not do this.

But wait - this is a knee-jerk reaction on my part. Think of all the unit testing frameworks you've used. They usually do this **exact** thing. In other words, you have a bunch of tests in your system, and suddenly a change you made causes a test to fail somewhere else due to an unhandled exception. However, you'll see all tests reported at the end with one failing.

But hold on ... think about this for a second.

Why should that have happened?

We've always talked about failing fast ... OK, I usually say that in my talks, so I won't say "we" as in "all developers". But I believe it's true - the quicker you fail, the better. I personally feel apps should just die quickly (with as much logging information as possible in the face of an unhandled exception) rather than swallow exceptions (and no, showing a message box to a user showing them the exception's message is **not** handling the exception).

Why shouldn't unit test frameworks act the same way?

If you're test has an assert and that fails, fine, report that the user. But if the code has an unhandled exception, then ... well, stop! So what if it is a unit test. Unit tests should be written with the same standards and principles as you do with your "regular" code, so why do unit test frameworks try to "keep on truckin'" in the face of an unhandled exception?

I realize that I've completely glossed over isolation - it's possible that frameworks can put the test in its own `AppDomain` or process to allow other tests to run at the cost of performance (or maybe some of them do this already). But I'm starting to wonder if unit test frameworks are doing it "right".

> Published: 08.15.2010 02:46:42 PM CST