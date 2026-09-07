---
title: A .NET Assembly Parser ... Written in Java?
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# A .NET Assembly Parser ... Written in Java?

One of the areas I'd love to see .NET expand upon is the ability to read from and write to .NET assemblies on the fly. The second part is pretty much covered by the Emitter namespace, but the first part isn't. There have been some 3rd-party efforts to bridge the gap (some of which I've mentioned here before), and today I just found out that somebody is taking BCEL and making/transforming it into something that's ".NET-aware": MBEL. At first glance, it seems like it's a cool idea. You can do read/write operations on an assembly, all the way down to the CIL level. However, there's a couple of problems. The first one is that the links to get the source code and the jar file don't work. In fact, as I'm writing this entry, I just noticed that the links to either the binaries or the source have been completely removed. Hmmmm ... anyway, the second "issue" is that this is all written in Java!

Now, I could care less that it's written in Java. What suprises me is that this is a tool that .NET developers would love to use, but since the binaries are Java bytecodes, they'd be left hanging. I wanted to get the source code and either port it to C# or try it in J# and see how much success I'd have there, but there's no source code to get. Oh, well. Hopefully they create a native, .NET managed version of MBEL as it seems to have some promising aspects.

> Published: 09.11.2003 03:05:56 PM CST