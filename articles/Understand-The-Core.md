---
title: Understand the Core
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Understand the Core

I'm watching [this vid](https://www.hanselman.com/blog/silverlight-video-of-john-lam-on-ironruby-at-padnug) of John Lam going over IronRuby underneath the covers. It's a very interesting talk and John is definitely excited about what he's working on [1] - I love presentations like this. But what I noticed as I was watching it, it wasn't too hard to follow. And it dawned on my that the commitment I did to write a book on IL years ago is paying off.

Lemme explain. When .NET was first introducted, C# was the language. Sure, we knew VB was in the fray, but, hey, look over here - C#, it's new and shiny! The more I looked at C#, though, I wasn't that interested it. What interested me was the CLR, and the common language that all .NET players needed to talk in: IL. So I started to spend a lot of time learning IL, Reflection (and the Emitter classes), and so on. What I discovered is that this newfound knowledge helped me out immensly in learning C#, VB, and any other language **targeted for .NET**. Sure, seeing new F# or C# features are cool, but I can figure out what's going on underneath the scenes (sort of) because of that hard-won knowledge years ago. Watching John talk about IronRuby ... even though I don't read Ruby, I get what's going on with IronRuby with respect to the DLR.

Now, my point is not to pat my own back ;). My point is that I'm learning that understand the core stuff first makes things that build on top it so much easier. That's why I don't call myself a C# or VB or (insert-language-here) developer. The best qualifier I give myself is a .NET developer (which is why I get really annoyed when I go to interviews at Magenic clients and they ask me, "so how long have you programming in (insert-language-that-works-on-the-CLR"?).

[1] Two nitpicks, John. Slow down - damn, youtalkwaytoofastsometimes! And stop saying, "right". For me, it gets to the point where it's almost too distracting.

> Published: 08.01.2007 11:18:07 AM CST