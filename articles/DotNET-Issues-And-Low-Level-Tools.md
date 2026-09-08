---
title: .NET Issues and Low-Level Tools
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# .NET Issues and Low-Level Tools

How low do you go?

Recently, a couple of incidents have made me think about .NET and programming in general. Sam wondered if there was a way to find out if an assemlby was already registered in the GAC. I knew that the Fusion API had something to do with it, but it was undocumented (and I was also looking at the wrong DLL; if I would've looked at mscorcfg.dll I would've ILDasm/Anakrino'ed the sucker). So I grabbed the output of "gacutil /l" and created a couple of classes to search through the list. While this did the job, Mattias showed the true way.

Another incident revolves around the VB .NET compatiability layer. Someone suggested using `StrDup()`, but Drew showed via ILDasm that using one of `String`'s custom constructors was the way to go. Essentially, `StrDup()` is just a wrapper method with some extra code bloat to boot, so why not use the constuctor?

On the one hand, I love it that it's easy to figure out what is really going on in a method in .NET. Tools like ILDasm, Reflector, and Anakrino make it painless to discover the hidden behaviors of a method. At the same time, though, I find myself wondering, "aren't methods supposed to hide those details from us anyway?" The intent of Design-By-Contract (DBC) is desirable. To know what the method expects from me and what it will uphold when it's done **without** having to see the source code would make development time easier. In essence, I'd like to know the method's behaviour without having to see it. I just find it funny how OO development is supposed to encapsulate implementations and you end up needing ILDasm to figure out what the encapsulated behavior is!

> Published: 05.30.2002 12:00:00 AM CST