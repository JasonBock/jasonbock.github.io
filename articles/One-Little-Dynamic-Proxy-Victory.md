---
title: One Little (Dynamic Proxy) Victory
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# One Little (Dynamic Proxy) Victory

Tonight I finished a major change to my proxy method implementation. Coding in IL/Reflection.Emit is always tricky, but sufficient planning made this change virtually painless (one minor IL verification error and another very sneaky error with generic methods I'm still chasing down). I was able to run my performance tests (as they aren't affected by my 2 minor issues) to see if proxy call invocation times went down at all. I thought my new design would work better, but, hey, you never know until you take numbers. The results? The "old" way: total proxy call time was 3.88 seconds (that's the **total** time of thousands and thousands of method invocations that are actually proxies to handle pre- and post-invocation semantics, not just one call!). The "new" time? 0.37 seconds! That's an order of magnitude difference and my tests didn't have to change one bit (in other words I didn't break the client).

It's the small things in life that make me smile sometimes :)

> Published: 10.25.2006 10:22:18 PM CST