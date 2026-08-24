---
title: This Feels Like Cheating
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# This Feels Like Cheating

```c#
using System;

namespace GenericInference
{
  class Program
  {
    static void Main(string[] args)
    {
      Something noInference = Create<Something>();

      Something inference;
      Create(out inference);
    }

    static T Create<T>() where T: new()
    {
      return new T();
    }

    static void Create<T>(out T value) where T: new()
    {
      value = new T();
    }
  }

  class Something { }
}
```

In the first case, I have to specify the generic type. In the second, I don't (although I have to use two lines of code, not one).

Nothing much else to see here, move along ...

> Published: 05.19.2006 07:00:54 AM CST