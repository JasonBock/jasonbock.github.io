---
title: The 15-Second Delay Cause
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# The 15-Second Delay Cause

[5 days ago](https://jasonbock.net/articles/A-Great-E-Mail-Title.html) I mentioned that we were seeing a 15-second delay with our ASP.NET application running IIS. For some reason, using anything in `System.Web.Extensions.dll` (the JSON serialization stuff in MS's AJAX framework) caused a paused the first time it was loaded. Well, we **finally** found out what the problem was. It's all due to `WinVerifyTrust`. Check out these articles:

* [Measurement Studio .NET Assemblies Take More Than 10 Seconds to Load at Run Time](https://web.archive.org/web/20161113000730/http://digital.ni.com/public.nsf/allkb/18E25101F0839C6286256F960061B282)
* [Educating or Capitulating](https://web.archive.org/web/20100628184859/http://blogs.xceedsoft.com/plantem/PermaLink,guid,3dde0262-1b7f-45d3-9a6e-164c842e422d.aspx)

Seems like the solutions/workarounds are not that good. We don't want to turn off CRL, nor do we want to install a hot-fix to minimize the timeout value. We also have no control over this assembly since it comes from Microsoft [1]. (It's funny to see that the vendors gave up on Authenticode signing because of this issue). Therefore, we're probably going to abandon Microsoft's code and see if we can use other alternatives.

[1] I **could** run my add-in and do some fancy-dancy recompilation, or I could do the ildasm/ilasm dance, but both are very sucky (and probably "illegal") options.

> Published: 03.21.2007 10:51:07 AM CST