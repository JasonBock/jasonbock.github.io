---
title: Fixed the CodeDom Problem
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Fixed the CodeDom Problem

I figured out what the problem was. As soon as you access the `CompiledAssembly` property, like this:

```c#
string result = results.CompiledAssembly.Location;
```

The assembly is locked. Fortunately, there's the `PathToAssembly` property which gives you the location of the assembly without locking it.

> Published: 03.17.2005 09:38:32 AM CST