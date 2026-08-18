---
title: Fault Blocks Can't Be Emitted Via DynamicMethod Using ILGenerator
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Fault Blocks Can't Be Emitted Via DynamicMethod Using ILGenerator

I'm preparing for my exceptions talk at TCCC5 - it's going to be a springboard to figure out what I would say in an eBook ... if I write it. Anyway, I'm amazed at how much good information there is on teh internets...and it's also sad because exception handling is twisted and misued all the time in .NET code. Anyway, I ran across a blog entry and I read that you can't emit fault blocks in a `DynamicMethod`. A workaround (proposed by [the docs](https://learn.microsoft.com/en-us/dotnet/api/system.reflection.emit.ilgenerator.beginfaultblock)) says you could use `DynamicILInfo` to do this. As this post shows, using `DynamicILInfo` takes a lot more work to create a method than `ILGenerator` does (and that blog post doesn't show how to set up an exception handler ... but [this one](https://learn.microsoft.com/en-us/archive/blogs/haibo_luo/turn-methodinfo-to-dynamicmethod) does), but at least there's a way to do it.

So for the millions of .NET developers out there who are emitting methods on-the-fly with fault blocks, there's a way to do it! :)

And yes, I love obscure stuff like this.

> Published: 08.14.2008 10:41:13 AM CST