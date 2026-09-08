---
title: Description in Attribute is Solved!
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Description in Attribute is Solved!

The problem is finally solved! Thanks to Hun Boon who finally help me see the light. Actually, the problem is pretty obvious once you see the solution, but isn't that always the case?

Anyway, the problem was I was combining two problems into one. I know the implementation of the getter of `Description` will set `description` to `null` unless the given description is one that the internal `Res` class can map to. So when I started debugging it in VS .NET, I started seeing `description` as `null` **after** the constructor was run. But that's the problem - I was looking at the value in the debugger. VS .NET is also trying to return the value of the `Description` property, and it must be doing this before it get the value of the private `description` field.

In fact, this code illustrates that the constructor does the expected thing:

```c#
ServiceProcessDescriptionAttribute lastOne =
  new ServiceProcessDescriptionAttribute("Give this a shot");

FieldInfo lastFI = lastOne.GetType().BaseType.GetField("description", BindingFlags.Instance |
  BindingFlags.Public | BindingFlags.NonPublic);

string fieldValue = lastFI.GetValue(lastOne).ToString();

Console.WriteLine("fieldValue is " + fieldValue);
```

Yep, the console prints out, "fieldValue is Give this a shot". Next time I'll remember not to blindly trust the results of the debugger's *Locals* window.

> Published: 07.22.2002 01:36:00 PM CST