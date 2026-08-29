---
title: NUnitAsp Hell May Just be Heck
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# NUnitAsp Hell May Just be Heck

After doing more digging on [this issue](https://jasonbock.net/articles/Im-In-NUnitAsp-Hell.html), I think it has to do something with the pages that the client has made that NUnitAsp isn't handling correctly. I made a small app that simulates the cookie + redirect scenario, and, of course, that works. I did it both using NUnitAsp's `Browser` object as well as using `HttpWebRequest` and `HttpWebResponse` and all is well. I think there something subtle with this app - it's using an `HttpHandler` which my test application is not doing so I'll try to add that in as well. Even though the client and I have talked about this and we both agree it's best to just stop working on this and move on to other things come Monday, I don't like giving up and plus I'm just curious to figure out what the discrepancy really is so I'll try to find the source of the problem on my own time.

> Published: 05.06.2005 05:17:25 PM CST