---
title: Bizarre, Yet Interesting, Decompiled C# Code in Reflector
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Bizarre, Yet Interesting, Decompiled C# Code in Reflector

Today I stumbled across some weird decompiled code in [Reflector](https://www.red-gate.com/products/reflector/) (version 4.2.48.0) - note the code in bold:

```c#
MethodInfo info1 = (MethodInfo) methodof(object.Equals);
```

Um ... what is methodof?

Actually, some searches turned up discussions about adding more "type-safe" Reflection operations to C# in the future, similar to what typeof gives you over `Type.GetType(string)`. But it's definitely not in C# 2.0, not at least that I can tell. I'm trying to find out from Lutz why this shows up in the code generation because it seems kind of odd to do that ...

> Published: 10.11.2006 10:45:56 AM CST