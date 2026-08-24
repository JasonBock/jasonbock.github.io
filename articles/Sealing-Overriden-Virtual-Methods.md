---
title: Sealing Overriden Virtual Methods
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Sealing Overriden Virtual Methods

I just found out you can do this:

```c#
public abstract class BaseClass
{
  protected abstract void AMethod();
}

public abstract class PartialBaseClass
  : BaseClass
{
  protected override sealed void AMethod() { }
}

public class ImplementationClass : PartialBaseClass
{ }
```

The interesting thing to note is that `ImplementationClass` cannot override `AMethod()`. I just ran into a design where I could refactor it such that subclasses down the inheritance tree didn't have to worry about "accidentally" overriding a method that they really shouldn't override. By sealing that method, it makes the point moot. It's a small thing, but it's nice to know about - I think it can clean up a lot of designs I make in the future.

> Published: 09.10.2006 07:47:31 PM CST