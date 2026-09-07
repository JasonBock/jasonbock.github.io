---
title: The Runtime is Smuggling My Methods!
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# The Runtime is Smuggling My Methods!

Today, we ran into a slight bug in our unit tests. We haven't resolved it yet (a very weird issue with how NUnit loads the tests in a different `AppDomain` and updating one of our strong-named assemblies from a different team), but when I examined the exception stack trace, I found this method: `System.Runtime.Remoting.Messaging.SmuggledMethodCallMessage.FixupForNewAppDomain()`.

Sometimes, I love the names that are buried deep in the .NET Framework.

> Published: 03.22.2004 09:01:45 PM CST