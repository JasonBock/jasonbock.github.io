---
title: `InternalsVisibleToAttribute` Changes for .NET 2.0
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# `InternalsVisibleToAttribute` Changes for .NET 2.0

David reports that the RC of VS 2005 requires you to put the trusted assembly's strong name in the metadata. This was optional before RC. Reading David's post, I'm not sure if this is something VS 2005 is requiring, or the attribute itself has been change such that the only way to use it is to specify a strong name, but ... well, while this is a bit better from a security standpoint, I'm still not hot to trot on `InternalsVisibleToAttribute`. The biggest problem I had with it were that either it was way too insecure or it was too secure (if that's possible). If you didn't specify a strong name, anyone could call that internal code if they used an assembly with the right name. But if you required the caller to use a strong name, now you had to mess with strong-naming assemblies, something that isn't impossible, but frankly I don't see a lot of clients doing it because most of the time they don't need to deal with managing a key file and ensuring the strong-naming is OK (again, strong-naming isn't hard, but it can be a bit of a pain in some cases). Now you're forced to using a strong name.

I know there are times where I wanted trusted clients to use specific parts of my code, so in principle I agree with this change, and I'm definitely not against using this attribute. In a few cases it may come in handy, but right now I don't see it being a big part of my .NET 2.0 day-to-day development experience (but I could be proven wrong ... once I **finally** start using 2.0 on a day-to-day basis!).

> Published: 10.07.2005 07:47:36 AM CST