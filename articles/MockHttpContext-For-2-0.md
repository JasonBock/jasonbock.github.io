---
title: MockHttpContext for 2.0
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# MockHttpContext for 2.0

A couple of days ago, a guy I worked with on a project a couple of years ago e-mailed me and asked me if I knew of any mock HttpContext project out there. Well ... I wrote one a while ago, but it was written for 1.1 and he needed it for 2.0. Since the code needed to do a lot of Reflection on 1.1-specific types and method signatures, it wasn't surprising that I needed a bit of time to port it over to 2.0. But, I had some time to update it, so ... here it is. If you compare the code from 1.1 to 2.0, you'll see that it was much easier to pull off in 2.0 as more of the types and methods were public. Actually, there was only 1 method I had to find and invoke via Reflection. So, if you need a mocked `HttpContext`, give my code a whirl and let me know if you run into any problems - thanks!

> Published: 01.30.2007 06:56:12 PM CST