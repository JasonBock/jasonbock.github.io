---
title: New Code Pages
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# New Code Pages

I just started working on creating separate .aspx file for my code drops that I have on my site. Currently I have three, ExceptionViews, UriViewer, and FileGenerator for Reflector. I'll keep adding ones for other code drops that I have on my site and for future ones as well. This should help put some organization around the code samples I publish. The three pages need some polish, but at least I got step one done.

By the way, FileGenerator now supports creating VS 2005 projects. Take a look at this screen shot:

![Generated mscorlb](https://jasonbock.net/images/Generated-Mscorlb.png "Generated mscorlb")

That's the `AppDomain` class in VS 2005. Now, don't get your hopes up and put your evil reverse-engineering hat on - the generated code doesn't compile, so you can't augment mscorlib.dll with your own stuff. You also don't have the .snk file that Microsoft uses to sign their assembly, which is a big problem as well. But I find it nice to be able to generate the source code and use the power of VS's Intellisense to traverse the inner workings of the Framework to see what's really going on underneath the scenes.

> Published: 01.29.2006 01:31:25 PM CST