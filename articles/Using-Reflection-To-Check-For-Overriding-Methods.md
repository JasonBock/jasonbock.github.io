---
title: Using Reflection To Check for Overriding Methods
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Using Reflection To Check for Overriding Methods

I just ran into this issue and I'm going to share the love. Here's a small code snippet to see if a given MethodInfo overrides a base class definition:

```c#
MethodInfo methodInfo = /* code that gets the reference goes here... */;
bool isOverriding = 
  methodInfo.GetBaseDefinition().DeclaringType.Equals(
  methodInfo.DeclaringType);
```

You'd think there would be an `IsOverriding` property on `MethodInfo`, but there isn't. This seems to work for the cases I've seen, but if you find a problem with it please let me know (I'm sure there's some edge case or scenario in a language other than C# that I completely missed).

> Published: 03.03.2005 02:13:09 PM CST