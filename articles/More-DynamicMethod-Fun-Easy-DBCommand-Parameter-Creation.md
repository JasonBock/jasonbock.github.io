---
title: More DynamicMethod Fun: Easy DBCommand Parameter Creation
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# More DynamicMethod Fun: Easy DBCommand Parameter Creation

I saw [this post](https://madprops.org/blog/adding-idbcommand-parameters-with-anonymous-types/) a couple of days ago, and I really liked the idea. I suggested to Matt that he changes his implementation to emit a DynamicMethod, thinking that it would give a performance increase.

Tests show that this is faster than the original approach:

```
At 10,000 iterations, Reflection = 0.3002694 seconds, DynamicMethod = 0.0541085 seconds
At 50,000 iterations, Reflection = 1.4428920 seconds, DynamicMethod = 0.2020902 seconds
At 250,000 iterations, Reflection = 7.0968963 seconds, DynamicMethod = 0.9447661 seconds
```

More and more I'm finding using a `DynamicMethod` to hard-code the code paths one finds using Reflection usually leads to a win in terms of performance. Not so much in terms of code maintainability (trying to read `ILGenerator`-based code in a .NET language is nasty, and debugging a `DynamicMethod` is not easy) but once it's rock-solid it's clear sailing from that point on.

> Published: 07.09.2008 03:26:47 PM CST