---
title: Mirroring a GroupBox Control - Solved (I Think)
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Mirroring a GroupBox Control - Solved (I Think)

Wow, did I get lucky. I now have a solution for this problem. I found it buried in a message posted on .NET 247. Basically, you set the FlatStyle property to System, and everything works as expected:

![Mirroring a GroupBox - Solved](https://jasonbock.net/images/Mirroring-GroupBox-Solved.png "Mirroring a GroupBox - Solved")

I'm going to do more testing, but this seems to be the solution I'm looking for.

> Published: 12.21.2004 11:14:29 AM CST