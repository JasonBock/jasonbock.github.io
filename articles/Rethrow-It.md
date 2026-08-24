---
title: Rethrow It
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Rethrow It

I just saw a post on the FxCop forum where someone is catching the general `Exception` type, and he doesn't understand why his exception management strategy is causing FxCop fits:

> I have about 50 places in my code that are at the top level of the food chain that do the handling of exceptions through the Exception Management Application Block and several hundred lower level places in the code that simply rethrow the exception up to the higher level. At every one of these 50 places in my code, fxCop declares an issue "Do Not Catch General Exception Types" and that doesn't seem to make any sense to me. Am I supposed to let these exceptions go unhandled??

No, but he's falling for the "if-I-catch-and-log-an-Exception-then-that's-OK" fallacy. Here's what his code looks like:

```c#
try
{
  // Some code to do something...
}
catch (Exception ex)
{
  HandleError(ex);
}
```

His `HandleError()` method logs the exception details. That's good, right? So why should he get the "Do Not Catch General Exception Types" error from FxCop?

The problem is that he's not reading the explanation in FxCop:

> You should not catch Exception or SystemException. Catching generic exception types can hide run-time problems from the library user, and can complicate debugging. You should catch only those exceptions that you can handle gracefully.

In this case, the coder is saying, well, something went wrong that I didn't anticipate, so I should log it. That's a good thing, but the problem is that he allows his code to continue to execute. That's bad! If he changes his code to this (note the code in bold):

```c#
try
{
  // Some code to do something...
}
catch (Exception ex)
{
  HandleError(ex);
  throw;
}
```

Then FxCop won't bark with this rule violation, because you're acknowledging that you really can't handle a generic `Exception` type. In a nutshell, yes, you should let exceptions like this go unhandled, because if an `Exception` occurs, at that point you're screwed in your code. Did you get an `OutOfMemoryException`? Or a `StackOverflowException`? Do you really think your code should continue? Maybe you should show a window that says, "We're sorry, but an unanticipated error occurred. The error has been logged and the application will be shut down", giving the user the option to restart the application. But the code should stop as soon as it can, because you're really in an indeterminate state at this point.

That doesn't sound resilient, but it actually is. If you know a method can throw an exception that you can recover from, then make a specific catch block for that exception type. But having the "catch-all" catch block is asking for trouble. And if you're code is riddled with generic exception handling blocks, that's a clear sign that you haven't tested your code enough, and you should go back and find out what kind of exceptions you're catching and having specific handling blocks for those exceptions.

I used to think this was all ivory-tower architect mumbo-jumbo until I read "Framework Design Guidelines", specifically the section on exceptions. Now I'm a complete convert from having generic exception handling blocks, and I'm trying to root them out of my current and old code bases.

> Published: 07.14.2006 09:33:33 AM CST