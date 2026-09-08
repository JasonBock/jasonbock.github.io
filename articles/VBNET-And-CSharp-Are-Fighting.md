---
title: VB.NET and C# Are Fighting
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# VB.NET and C# Are Fighting

I think I just hit what looks like a **very** weird bug in .NET, but I'm too tired to sort it all out tonight. Here's the jist of it. I was playing out with Reflection, and I had a VB .NET class implement a C# interface and another VB .NET interface. One method in the VB .NET class was implementing a method from the C# and VB .NET interface. When I tried to load that type via `GetType()`, it threw an exception.

I immediately ran PEverify on my VB .NET assembly, and I was getting an IL error on my type. The funny thing was, I had an `AssemblyCulture` attribute in my C# assembly (en-US), which, if I took it out and recompiled my VB .NET assembly, PEverify doesn't choke on it anymore. But I'm **still** not able to load it via Reflection!

I haven't spent enough time to determine if it's just me or there's bug in the compiler I found. After I get some sleep I'll try to isolate it and see just what the problem is. I'll give more details later - stay tuned ...

> Published: 10.02.2002 11:53:00 PM CST