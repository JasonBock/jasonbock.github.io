---
title: Fun PEVerify Errors
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Fun PEVerify Errors

I'm getting closer to releasing version 2.0 of my dynamic proxies assembly. Tonight I ran into a very odd PEVerify error: "[offset 0x0000002F] Unable to resolve token.". Which made me go, "HUH?!?", especially since I looked at the CIL over and over and over again, and everything looked just fine.

Of course, after a 1/2 hour of head-slapping, I saw the obvious problem. The interface the code is trying to use to call a method is internal, so, DUH, it can't resolve the token.

Solutions are always obvious after you figured out the problem :)

Actually, now that I think about it, this brings up a very interesting test case: what **should** I do when I'm faced with a public class that explicitly implements an internal interface? Right now my code will fail miserably with a verification error, and that's not acceptable.

> Published: 10.27.2006 09:40:01 PM CST