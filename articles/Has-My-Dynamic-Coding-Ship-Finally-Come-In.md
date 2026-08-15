---
title: Has My Dynamic Coding Ship Finally Come In?
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Has My Dynamic Coding Ship Finally Come In?

I couldn't save this link for one of my "Flagged Links" posts - it needs a post all by itself.

Ever since I started in .NET, I fell in love with assembly reading/writing, metadata, dynamic code execution, etc. This naturally pulled me to the `System.Reflection` and `System.Reflection.Emit` namespaces and I was able to do some interesting things with them, but I always felt the picture was incomplete. What about reading PDBs? What about reading method implementations with a nice API?

When [FxCop](https://learn.microsoft.com/en-us/visualstudio/code-quality/net-analyzers-faq) burst on to the scene, I quickly delved into the supporting assemblies to discover that yes, it had that nice API to read assembly contents all the way down to the IL level ... but it didn't have that assembly modification hook.

When [Cecil](https://github.com/jbevain/cecil) came on my radar, I dove into its infrastructure. I really liked it, and I think it can do even more in the future, but ... I really can't put my finger on why it feels somewhat incomplete, but it does.

Now, the [CCI](https://github.com/microsoft/cci) framework has been released on CodePlex.

It looks like it has everything I could ever want.

My first idea? See if I can remove `System.Reflection.Emit` from [DynamicProxies](https://github.com/JasonBock/DynamicProxies) and [EmitDebugger](https://github.com/JasonBock/EmitDebugging/tree/main/EmitDebugging) and use CCI. I'm not saying it's a great idea, but that's the first one that popped into my head.

Thank you Microsoft. I think this should've showed up a long time ago, but it's finally here.

> Published: 04.15.2009 08:20:06 AM CST