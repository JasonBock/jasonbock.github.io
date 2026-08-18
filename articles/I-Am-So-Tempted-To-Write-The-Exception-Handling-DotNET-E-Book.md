---
title: I Am So Tempted to Write the "Exception Handling in .NET" E-Book
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# I Am So Tempted to Write the "Exception Handling in .NET" E-Book

I've had it.

I'm done with seeing exception handlers in .NET written that catch every kind of exception.

This one finally broke me:

```c#
public bool IsAuthorized()
{
  try
  {
    return AuthorizationManager.DoSomeKindOfFancyAuthorizationChecking();
  }
  catch (Exception)
  {
  }

  return true;
}
```

Guess what this returns if there's an exception ...

ARGH!!!!

If I can find the time, I'm going to write the 50-page e-book on exception handlers in .NET and charge $1 for it (or hell, just give it away for free!). Get it into every .NET developer's hands and make them follow it. Update it as soon as others find issues with it.

It's time.

> Published: 07.16.2008 01:09:25 PM CST