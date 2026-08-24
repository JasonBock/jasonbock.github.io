---
title: Type-Safe Reflection
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Type-Safe Reflection

Ever since I used COM, I fell in love with types, especially when it came to reflecting upon the type and finding out its methods, properties, etc. Unfortunately, this was limited in COM in that the server had to publish a type library for this to be effect (remember tlbinf32.dll?). When I moved to Java for a while, I fell in love with the Reflection API. It was consistent, but it could also be brittle (as well as slow). .NET has the same advantages and disadvantages.

Now, with 3.0 looming on the horizon, there's efforts to make Reflection "type-safe". Check out this article. Amazing! This is wicked-cool in my book. It would be interesting if Daniel added some performance numbers to compare this approach to use the Reflection API straight-up, but conceptually I really like this approach.

> Published: 07.07.2006 07:17:03 AM CST