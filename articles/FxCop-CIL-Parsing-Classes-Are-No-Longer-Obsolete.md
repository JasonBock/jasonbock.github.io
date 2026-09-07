---
title: FxCop CIL Parsing Classes Are No Longer Obsolete
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# FxCop CIL Parsing Classes Are No Longer Obsolete

A while back I wrote an article for Pinnacle Publishing on CIL parsing where I used the FxCop classes. Unfortunately, I had some issues with the API (having a structure called `EHCLAUSE` and a class called `EHClause` didn't help matters). I wasn't ticked off though as all of the classes within the `Microsoft.Tools.FxCop.Sdk.IL` namespace were marked with the `ObsoleteAttribute`. The latest version, .ver 1:0:9:1, has cleaned things up. Not that this is amazing news, but it is cool that these classes can now be used by developers without any issues. Of course, now I'm wondering what is within `Microsoft.Tools.FxCop.CodeAccessSecurityPolicy`!

> Published: 04.22.2003 01:38:36 PM CST