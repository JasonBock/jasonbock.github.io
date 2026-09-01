---
title: Potential Bug in FxCop
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Potential Bug in FxCop

I think I've found a bug in the `ExceptionHandlers` property of the `Microsoft.Cci.Method` class (version 1.312.0.0). Take a look at the implementation:

```c#
public virtual ExceptionHandlerList ExceptionHandlers
{
  get
  {
    if (this.exceptionHandlers == null)
    {
      this.exceptionHandlers = new ExceptionHandlerList();
    }
    return this.exceptionHandlers;
  }
  set
  {
    this.exceptionHandlers = value;
  }
}
```

The problem is the getter. For some reason in the debugger, I'm seeing that my method in question has four handlers, which is correct - it has three catch blocks and a finally block. However, when I run the code, it's always zero. I thought maybe it's an initialization error as it's always initialized to a new list, but that doesn't seem to be the case - another method with just a catch and finally block yields two elements in the debugger, but zero when the test is run. This is really frustrating. Something odd is going on. I'll keep on looking into it, but if you've already done some work with FxCop 1.312 and you've seen this problem please let me know.

**Update #1**: Looks like the evaluation engine is a bit lazy - I needed to call `this.Visit(method);` in my `Check()` method before I do any processing. This loads all the exception handlers correctly. Thanks to John for figuring it out (it's buried in his article under the "Exception Documentation Rules" section).

> Published: 02.21.2005 01:24:14 PM CST