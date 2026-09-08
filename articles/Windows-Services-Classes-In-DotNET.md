---
title: Windows Services Classes in .NET
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Windows Services Classes in .NET

Yesterday I was playing with the Windows Services classes in .NET. Oh, man, it is so much easier to write services than it was pre-.NET days! The only annoying thing is that you can't add a description to your service, even though there is an attribute called `ServiceProcessDescriptionAttribute`. The installer classes don't do anything with it. However, I'm hoping that by using some Win32 APIs, hooking the `AfterInstall` event and using some .NET reflection code, I can get around this issue.

> Published: 07.19.2002 08:55:00 AM CST