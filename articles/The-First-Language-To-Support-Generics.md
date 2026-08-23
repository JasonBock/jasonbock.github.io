---
title: The First Language to Support Generics
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# The First Language to Support Generics

I saw [this article](https://www.infoq.com/news/2007/01/FSharp-Two-Years/) this morning, and there was a comment [1] challenging F#'s claim "to be the first .NET language to support generics". Eric stated:

> This is pure bollocks. Eiffel has generics and was ported to .Net long before F# even saw daylight.

I think there's a technical point here that needs clarification. Eric is correct that Eiffel was ported to .NET before F# was even on the scenes, and since Eiffel supports the notion of generics, they had to support it in their compiler. What I think is missing here is that Eiffel needed to do this **over** the runtime as the CLR didn't support generics until 2.0. Since Don Syme worked on adding generics to .NET and he's the primary force behind F#, it makes sense that **CLR-based** generics were added to F# before C# and VB had them baked in.

[1] I would've added a comment to InfoQ's site, but they require registration and I refuse to do that. Of course, comments on my site are currently closed but I'll get around to fixing that shortly!

> Published: 01.10.2007 08:25:12 AM CST