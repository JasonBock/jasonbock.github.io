---
title: Reducing Catch Handlers
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Reducing Catch Handlers

I'm a big fan of writing exception handlers that do not handle the `Exception` type explicity. That said, trying to handle parsing a `Guid` can be a bit of a pain:

```c#
private void SetContext(string contextId)
{
  try
  {
    this.Context = Context.Find(new Guid(contextId));
  }
  catch(ArgumentNullException)
  {
    this.Context = Context.Default;
  }
  catch(FormatException)
  {
    this.Context = Context.Default;
  }
  catch(OverflowException)
  {
    this.Context = Context.Default;
  }
}
```

The point with this code is if the given `contextId` in string format can't be converted into a `Guid`, I have to write three catch handlers that basically do the same thing. It would be nice if C# let you do something like this:

```c#
private void SetContext(string contextId)
{
  try
  {
    this.Context = Context.Find(new Guid(contextId));
  }
  catch(ArgumentNullException, FormatException, OverflowException)
  {
    this.Context = Context.Default;
  }
}
```

It's less code and it's more readable (in my mind at least). I realize that this is tricky, because what if you wanted to use the exception object that came in? What would you do ... this?

```c#
private void SetContext(string contextId)
{
  try
  {
    this.Context = Context.Find(new Guid(contextId));
  }
  catch(ArgumentNullException ane, FormatException fe, OverflowException oe)
  {
    this.Context = Context.Default;
  }
}
```

It seems like the two code examples I gave are the only options: you either don't care about using the `Exception` object or you declare an object reference for each type. The 1st approach seems like the only one that would make sense. Otherwise, if you really care about handling that specific `Exception` (and the corresponding object that's thrown) just break them out into separate handlers.

Of course, if `Guid` had a `TryParse()` method on it, this would be a wash. But that doesn't eliminate my point - there are other cases where I've had the same code in multiple catch blocks, and even if that code is just a call to a common method, having the ability to declare one catch block in code that's executed for multiple `Exception` types would be very nice to have.

> Published: 06.22.2007 07:32:27 AM CST