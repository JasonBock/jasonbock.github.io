---
title: QuickCounters
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# QuickCounters

Tonight I went to the local .NET User's Group, and Scott Colestock gave a presentation on his open-source toolkit called QuickCounters. It's basically a wrapper around performance counters, so by adding a couple of lines of code, you get performance counting metrics. The library also has support for serialization so it can work in BizTalk orchestrations. There's also code being written for WCF to intercept service calls so you don't have to support the counters explicitly in code. I mentioned to Scott an idea to integrate Mono.Cecil to weave in the counter support. Then, all you'd have to do is this:

```c#
[QuickCounter]
public int DivideXByY(int x, int y)
{
 return x / y;
}
```

And the weaver would inject the QuickCounter code for you. This weaver could do dual work by installing the counters based on the `QuickCounter` attributes that are present, so you wouldn't need an XML configuration file either. If I have time ... I'll talk to Scott about this in more detail and I'll try to contribute this to the project if he thinks it's a good idea. At the very least, I'm going to suggest that we use this on my current project as it makes performance counter support pretty painless.

> Published: 11.02.2006 07:28:21 PM CST