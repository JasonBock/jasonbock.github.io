---
title: ICustomAttributeProvider
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# `ICustomAttributeProvider`

One small thing that was missed in the "Applied .NET Attributes" book was this: Why do most of the `System.Reflection` classes allow you to search for metadata on their instances? It's a simple answer, but I think it's important to clearly state what's going on.

It all comes down to `ICustomAttributeProvider`. This interface defines two methods: `GetCustomAttributes()` and `IsDefined()`, both of which have two overloads (I talk about the differences between the overloaded versions here if you're interested in the details). There are four classes that implement `ICustomAttributeProvider`: `Assembly`, `MemberInfo`, `Module`, and `ParameterInfo`. These four classes are core base classes in the `System.Reflection` world, especially `MemberInfo` as `EventInfo`, `FieldInfo`, `MethodBase` (which both `ConstructorInfo` and `MethodInfo` derive from), `PropertyInfo`, and `Type`. This is why you can search for attributes on any member of an assembly, because they all implement `ICustomAttributeProvider`, either directly or indirectly.

Again, it's a simple thing to look up, but I think it's cool to reflect upon (no pun intended ... OK, I'll admit it, I intended it) the design of the Reflection API as it pertains to metadata.

> Published: 01.19.2005 01:10:11 AM CST