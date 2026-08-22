---
title: One Minor CSLA Glitch
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# One Minor CSLA Glitch

Last night I ran into an odd issue with CSLA. If you get a `SerializationException` with a message that starts with "Type is not resolved for member ... " and it has to do with your custom principal and identity objects, then you're running into the same issue I am. I tried to apply the workaround mentioned by John Lewicki in the comments section. Basically it has to do with implementing `ISerializable` and detecting what the serialization context is. However, you still need to implement the "special" constructor in your identity class to handle the non-`CrossAppDomain` case and you need to manually serialize the `IsAuthenticated` value when `GetObjectData` is called and the context is not `CrossAppDomain` - (i.e. don't throw an exception per John's recommendation). Plus, you also need to implement this on the principal object, but you're stuck with the "special" constructor because `BusinessPrincipalBase` exposes only one constructor that takes an identity object, and that stymies a good implementation of `ISerializable`.

I'm still working out the final details on this. I have to try and change `BusinessPrincipalBase` to do this, and I always hate changing someone else's code :). Hopefully I'll have a solution by tonight, and then the last thing to do is to get the dates working correctly on the site (I save all datetime values as UTC so I have to make sure I display that correctly on the page so it makes some sense to the viewer).

**UPDATE**: This was fixed by implementing some Reflection to find the `_identity` field in `BusinessPrincipalBase`. I'm not entirely happy with this as it is prone to breaking due to internal changes, but it's better (in my opinion) than changing the source code. I still need to write a cross-`AppDomain` unit test but the proof-of-concepts works.

> Published: 04.10.2007 11:08:54 AM CST