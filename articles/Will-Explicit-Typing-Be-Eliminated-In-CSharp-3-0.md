---
title: Will Explicit Typing Be Eliminated in C# 3.0?
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Will Explicit Typing Be Eliminated in C# 3.0?

In [this post](https://web.archive.org/web/20140426090333/http://blogs.msdn.com/b/ericlippert/archive/2005/09/27/474462.aspx), Eric shows two cases where type inference (via the `var` keyword) will be beneficial for the developer. The generic `Dictionary` example really caught my eye - not forcing a developer to type that in will be a great time-saver. But then I thought ... will there ever be a great need to explicitly type anything in C# anymore, and just let the compiler figure it out?

I'm sure I'll find cases where that's the case, but right now I can't think of one. If the compiler can do it right, then frankly why would I care to type the full class name in?

> Published: 09.27.2005 02:58:21 PM CST