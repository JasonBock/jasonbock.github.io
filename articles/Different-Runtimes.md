---
title: Different Runtimes?
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Different Runtimes?

Note: The following was written as kind of a stream-of-though conflagulation. Reader discretion is advised :)

I read [this](https://weblogs.asp.net/podwysocki/how-would-the-clr-be-different/), and it spoke to some thoughts I've been having as of late. Basically, when will we be able to generate runtimes?

Right now, M is all the rage (sort of) in that it's making it easier to create languages (DSLs, whatever term you want to use). But why can't we just keep going with this? Create runtimes that are tailored to our specific needs.

Now, I get it, in that the CLR is the package for .NET. Trying to write DSL-like runtimes seems a bit odd and difficult, and frankly, would anyone want to do that anyway?

Of course, I'm not entirely convinced that everyone is going to want to create their own languages either.

There's always versioning issues and compatiabilities and whatnot to consider, but that's the tradeoff: you do something custom, there's always the danger that someone may need to use it in the future, and unless you are very clear in your intents, it may end up being a maintenance nightmare for the rest of the folks who have to use you transmorgrification inspired by Lovecraftian thoughts.

I remember back when the CLR was a baby, someone was showing how you could add a new opcode like "exp" to do exponentiation. This was done via a custom build of the SSCLI. Coolio idea, but limited to anyone who could use the entire toolset to emit, understand, and create assembly code around seeing this custom opcode.

My point is we're getting OK with creating new languages, but maybe we should see what else can be customizable. And sure, trying to write custom, one-off runtimes and execution engines and so on may seem stupid or extremely difficult, but in certain scenarios, why not?

> Published: 01.16.2009 02:04:14 PM CST