---
title: Creating a Read-Only Collection From a HashSet
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Creating a Read-Only Collection From a HashSet

This may be a very inefficient way to get a read-only collection from a `HashSet`, but there is no `AsReadOnly()` method on `HashSet` (at least I didn't see one), so I added an extension method to create one:

```c#
public static class HashSetExtensions
{
  public static ReadOnlyCollection<T> AsReadOnly<T>(this HashSet<T> @this)
  {
    return new List<T>(@this).AsReadOnly();
  }
}
```

It ends up working very nicely:

```c#
ReadOnlyCollection<string> data = 
  new HashSet<string> {"A", "B", "A"}.AsReadOnly();
```

As a side note, I'm using "@this" in extension methods for the first argument - for some reason I'm finding that a simple way to preserve the "this-ness" of that first argument.

> Published: 02.14.2008 08:23:32 AM CST