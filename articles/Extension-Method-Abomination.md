---
title: Extension Method Abomination
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Extension Method Abomination

For some reason this code came to mind today, and I shuddered a bit:

```c#
public static class ObjectExtensions
{
  public static bool IsNull(this object obj)
  {
    return obj == null;
  }
}
```

Because now you can write code like this:

```c#
object o = null;
Assert.IsTrue(o.IsNull());
```

All I can do is look at the code and stare in wonder ...

(BTW I didn't compile this, but I think this should work!)

> Published: 10.11.2007 02:47:51 PM CST