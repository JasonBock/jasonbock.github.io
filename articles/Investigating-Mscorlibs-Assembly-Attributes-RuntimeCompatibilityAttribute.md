---
title: Investigating mscorlib's Assembly Attributes: `RuntimeCompatibilityAttribute`
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Investigating mscorlib's Assembly Attributes: `RuntimeCompatibilityAttribute`

Today I was briefly looking at mscorlib in [Reflector](https://www.red-gate.com/products/reflector/), and I noticed a bunch of attributes at the assembly level that I've never seen before. Since attributes have always interested me (enough, even, to write a book on them), I've decided to spend some time looking at a handful of them. Today's topic is focused on `RuntimeCompatibilityAttribute`:

```c#
[assembly: RuntimeCompatibility(WrapNonExceptionThrows=true)]
```

This is the way mscorlib is tagged with this attribute. As you can see, `WrapNonExceptionThrows` is set to `true`. But what is it that this attribute does?

A while ago [I blogged](https://jasonbock.net/articles/Throwing-Exceptions-That-Are-Not-Exceptions.html) about how you can throw exceptions that do no derive from Exception. According to the SDK, this attribute has ... well, **something** to do with the CLR and its default behavior of wrapping non-`Exception` types into a `RuntimeWrappedException`. Here's what the SDK says:

> You can use the RuntimeCompatibilityAttribute class to specify whether exceptions should appear wrapped inside catch blocks and exception filters for an assembly. Many language compilers, including the Microsoft C# and Visual Basic compilers, apply this attribute by default to specify the wrapping behavior.

This is true - I verified assemblies that I've compiled in 2.0 and they automatically get this attriubte applied with `WrapNonExceptionThrows` set to `true` (although you can explicitly use this attribute with that property set to `false` and the compiler will honor that). However, the SDK continues on with a comment that doesn't make a lot of sense to me:

> Note that the runtime still wraps exceptions even if you use the RuntimeCompatibilityAttribute class to specify that you do not want them wrapped. In this case, exceptions are unwrapped only inside catch blocks or exception filters.

So, in other words, even if I turn off the wrapping, I still get it? I checked this attribute using the code example I had in my blog entry, and the results were quite interesting. When I compile CallingBadClass.cs with `WrapNonExceptionThrows` set to `false`, the compiler no longer gives me the CS1058 warning. When I run the EXE, here's what I see on the console:

```
CatchWithNoException called.
In try block...
Catch entered.
Finally entered.
CatchExceptionAndWithNoException called.
In try block...
Catch entered.
Finally entered.
CatchException called.
In try block...
```

At this point, the application acts like it was compiled in 1.1 in that it stops. But it doesn't show the debugging services window; I get the following window:

![WrapNonExceptionThrows to False](https://jasonbock.net/images/RuntimeCompatiabilityAttribute-1.png "WrapNonExceptionThrows to False")

If I select the *Close* button, the application pauses for a while, and then I get the following window:

![WrapNonExceptionThrows to False](https://jasonbock.net/images/RuntimeCompatiabilityAttribute-2.png "WrapNonExceptionThrows to False")

When I select the *OK* button, the application finally stops with this final output to the console:

```
Unhandled Exception: System.Runtime.CompilerServices.RuntimeWrappedException: An
 object that does not derive from System.Exception has been wrapped in a Runtime
WrappedException.
   at BadClass.BadMethod()
   at CallingBadClass.CatchException()
   at CallingBadClass.Main()
```

This is actually a bit worse than what happens in 1.1, because in 1.1 I'm told what the type of the exception is. In this case, all I'm told is that I got some kind of non-`Exception` thrown.

So it seems like setting that property on the attribute isn't such a good idea. I can't think of a case in 2.0 where I wouldn't want to a `RuntimeWrappedException`. In fact, as that last stream of console output shows, I **still** get a `RuntimeWrappedException` (just like the SDK said I would!) - it's just not handled very gracefully.

> Published: 07.05.2006 06:21:59 PM CST