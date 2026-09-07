---
title: Proxies, .NET, and Java
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Proxies, .NET, and Java

When this came out, I just ignored it. Debating whether product/language/framework/etc. A is better or worse than product/language/framework/etc. B can easily end up going into the proverbial wormhole where the energy is way too unstable to keep it open. However, I eventually skimmed Carlos's points, and point 87, "Dynamic Proxies," kind of bothered me. Here's the statement in all of its' detail:

> "Java Dynamic Proxies type safe dynamic interception of method invocations. This is extremely useful in dynamically creating remote stubs rather than using a compilation approach. JBoss uses this mechanism to allow for a more productive development environment for EJBs. Dynamic Proxies have also been used in other ways like in AOP and Design By Contract implementations. Dynamic Proxies are absent in .NET"

Well, they're "absent" in the sense that there is no `Proxy` class that intercepts virtual method invocations on object instances (see my article on Java proxies for details). However, .NET has a wonderful interception story via `ContextBoundObject`-derived classes. I won't go into the details of how it works here - it's been described in many books and articles before (the most recent one I saw is by Juval Lowy in the March 2003 issue of MSDN, called "Contexts in .NET").

Of course, you can always roll your own as well. In fact, I made the mistake of not finding the proxy stuff in .NET right away, so I ended up writing my own dynamic proxies architecture using the `Reflection.Emit` classes (you can get the code [here](https://github.com/JasonBock/DynamicProxies)). It was fun to do and **very** educational, but it was also a major PITA (for those who have dynamically created code in .NET via the Emitter classes, you know the pain ;) ). There is a difference in what I did compared to .NET proxies - the biggest one is that you don't have to derive your class from `ContextBoundObject` to get the interception. However, my proxies don't intercept non-virtual methods like .NET proxies do.

By the way, I've never understood **why** .NET proxies have the requirement of deriving from a specific base class to facilitate interception. It's not that it's bad or has no technical merit, but it seems to me an alternative design would be to use an attribute, like a `ContextBoundObjectAttribute`. Ingo mentioned this before, and I agree with him.

The reason why I bring this all up is that I feel Carlos's point about proxies is misleading, if not wrong. There are many ways to cruft up proxies in .NET. It may not work exactly like it does in Java, but that doesn't mean the concept isn't there.

> Published: 02.06.2003 10:24:44 AM CST