---
title: `MethodBuilder` and `GetParameters` ... ARGH!
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# `MethodBuilder` and `GetParameters` ... ARGH!

Yesterday I talked about a cool new Reflection-based feature to get by-ref and array type. However, there's something that's in the Reflection API that's really bugging me:

```c#
MethodBuilder method = /* create method here...*/;
ParameterInfo[] arguments = method.GetParameters();
```

The `GetParameters()` call fails with a `NotSupportedException` on a `MethodBuilder` object. If the dynamic type is created, then the method call is safe, but until that time, you get an exception.

I can't begin to tell you how annoying this is ... because it would take too long to explain **why** I need this functionality. If I could all of the parameter information from a dynamic method before it's "baked", I'd be in a much happier world, but because the method fails pre-"baking", I have to come up with some goofy way to figure out what the parameters are.

It's not going to be pretty ...

> Published: 12.06.2006 02:13:03 PM CST