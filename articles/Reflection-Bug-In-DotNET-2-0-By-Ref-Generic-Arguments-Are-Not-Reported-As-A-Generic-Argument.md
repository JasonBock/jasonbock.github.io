---
title: Reflection Bug in .NET 2.0? By-Ref Generic Arguments Are Not Reported As a Generic Argument
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Reflection Bug in .NET 2.0? By-Ref Generic Arguments Are Not Reported As a Generic Argument

Consider the following method:

```c#
public virtual void ByRefArgument<T>(int arg1, T arg2, ref T arg3)
```

Now, I wrote some code to get this method as a `MethodInfo`. I rip through the parameters via `GetParameters()`, and I found what appears like a bug. I get the `Type` via the `ParameterType` property, and here are the relevent property values on each type:

arg1, FullName = "System.Int32", IsByRef = false, IsGenericParameter = false
arg2, FullName = null, IsByRef = false, IsGenericParameter = true
arg3, FullName = null, IsByRef = true, **IsGenericParameter = false**

Notice the bolded text? The parameter type isn't viewed as being a generic parameter, when it clearly is. I can use the fact that `FullName` is `null` when the parameter type is a generic, but this feels really wrong.

I'll check the Microsoft bug tracking tool to see if this has been reported before. If not, I'll submit it.

> Published: 10.26.2006 08:46:17 PM CST