---
title: Sorting Sections of a List
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Sorting Sections of a List

Today I had to tackle the problem of sorting only a section of a list, not the entire thing. Fortunately, `Sort()` has an overload that already does this for you:

```c#
using System;
using System.Collections.Generic;

namespace PartialSorts
{
  internal static class Program
  {
    private static void Main(string[] args)
    {
      List<int> values = new List<int>() { 5, 2, 8, 6, 1 };
      values.Sort(1, 3, null);
            
      foreach(int value in values)
      {
        Console.Out.WriteLine(value);
      }
    }
  }
}
```

Running the code produces this:

```
5
2
6
8
1
```

Note that the 2nd, 3rd, and 4th values were sorted, but the 1st and 5th stayed where they are. Nice!

> Published: 01.17.2008 02:06:52 PM CST