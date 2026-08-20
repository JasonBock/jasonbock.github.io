---
title: BigInteger is Internal
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# BigInteger is `Internal`

Sigh ... I remember reading about a `BigInteger` class being added to .NET 3.5 ... then somewhere in the beta cycles it got changed to be an `internal` type, and sadly the release version still has it `internal` (you can find it in the `System.Core` assembly as `System.Numeric.BigInteger`). I know I'll get the "please explain when you'd ever need to use this class" inquiry to justify why this type should be opened up (uhhh ... because I did large numerical computations as part of my master's thesis? But I digress ... ), but that's too time-consuming, especially since the DLR has a `BigInteger` type available. Sure, you could wrap this up with `PrivateObject` or do some duck typing magic, but it looks like they're very close ... why don't they make it `public`? Oh, well, it is what it is.

> Published: 12.12.2007 07:16:21 AM CST