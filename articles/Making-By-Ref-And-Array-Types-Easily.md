---
title: Making By-ref and Array Types ... Easily!
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Making By-ref and Array Types ... Easily!

I just found out something tonight that was introduced in .NET 2.0. Before 2.0, if you wanted to create a type that was by-ref or an array, you had to do this:

```c#
Type byRefType = Type.GetType("System.String&");
Type arrayType = Type.GetType("System.String[]");
Type arrayByRefType = Type.GetType("System.String[]&");
```

That always felt ugly to me. However, now you can use the `MakeByRefType()` and `MakeArrayType()` methods so you don't have to worry about messing up the string definition:

```c#
Type byRefType = typeof(string).MakeByRefType();
Type arrayType = typeof(string).MakeArrayType();
Type arrayByRefType = typeof(string).MakeArrayType().MakeByRefType();
```

There's also a `MakePointerType()` for completeness sake. Cool stuff!

> Published: 12.05.2006 08:32:13 PM CST