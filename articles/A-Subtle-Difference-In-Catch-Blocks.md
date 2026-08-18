---
title: A Subtle Difference in Catch Blocks
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# A Subtle Difference in Catch Blocks

What's the difference in the following two catch blocks?

```c#
public static int DifferencesInCatch(int x)
{
  int value = x + 20;
    
  try
  {
    value = value / x;
  }
  catch(Exception) {}
    
  try
  {
    value = value / x;
  }
  catch {}
    
  return value;
}
```

At first glance ... not much. They both seem like they're going to catch the general exception type. And you'd be right ... but there is a difference.

Take a look at the IL:

```
.method public hidebysig static int32 DifferencesInCatch(int32 x) cil managed
{
  .maxstack 2
  .locals init (
    [0] int32 'value')
  L_0000: ldarg.0 
  L_0001: ldc.i4.s 20
  L_0003: add 
  L_0004: stloc.0 
  L_0005: ldloc.0 
  L_0006: ldarg.0 
  L_0007: div 
  L_0008: stloc.0 
  L_0009: leave.s L_000e
  L_000b: pop 
  L_000c: leave.s L_000e
  L_000e: ldloc.0 
  L_000f: ldarg.0 
  L_0010: div 
  L_0011: stloc.0 
  L_0012: leave.s L_0017
  L_0014: pop 
  L_0015: leave.s L_0017
  L_0017: ldloc.0 
  L_0018: ret 
  .try L_0005 to L_000b catch [mscorlib]System.Exception handler L_000b to L_000e
  .try L_000e to L_0014 catch object handler L_0014 to L_0017
}
```

Look at the 2nd catch block's type. It's `object`, not `Exception`.

Now, in the CLR it's valid to throw anything; it doesn't have to be a subclass of `Exception`. This is used by some languages (e.g. C++), but most .NET developers don't even know about this. In fact, in 2.0 if code throws something that's not `Exception`-based, it'll wrap that object with a `RuntimeWrappedException` (I've discussed this feature in detail [here](https://jasonbock.net/articles/Throwing-Exceptions-That-Are-Not-Exceptions.html)).

Anyway, here's the design issue I'm struggling with. In my [ExceptionFinder](https://github.com/JasonBock/ExceptionFinder) code, I'm trying to iron bugs out. A couple of weeks ago, I **thought** I saw some bizarre types listed in the results - they were more than just the "object" type. Unfortunately, I lost all my notes on what I had seen (damnit!), so I thought I could add code in that would not allow non-`Exception` types to be reported. In fact, I considered this to be a bug in the program and I changed the implementation to throw a custom exception when a type is reported that's not an `Exception` type. But ... it's also a completely valid case! For example, if code calls "throw" in a `catch` block like the 2nd one in the original code sample, what's on the stack is an object of ... `System.Object`. Most likely it's some kind of `Exception`-based object, but it's possible that it could be something else.

I'm thinking of breaking my `LeakedExceptionsCollection` into two separate nodes: one that contains all the `Exception`-based exceptions, and those that don't. Frankly, most of the analysis will find `Exception`-based exceptions, but at least this way I'll know if I find an exception of type `System.String` I can trace down what method it occurred in and on what instruction.

> Published: 08.22.2008 03:42:43 PM CST