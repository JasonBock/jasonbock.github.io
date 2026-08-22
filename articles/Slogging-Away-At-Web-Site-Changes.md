---
title: Slogging Away at Web Site Changes
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Slogging Away at Web Site Changes

This morning I get my monthly "morning alone-time". Basically, every month Liz and I each get one morning alone to do whatever we want. So I'm at the local Dunn Bros Coffee shop in Shakopee [1], trying to get as much coding in as I can. Right now I'm updating my blogging framework to use CSLA.NET. I've been hacking away at that for a week, and I'm now at the point where my blog tests passed. I still have to get through all my blog item and blog item comment tests (among other tests), but so far things are going well. The biggest challenge (I think) is getitng CSLA.NET to have the `DataPortal` working remotely on my web site. I wanted to get away from my home-grown JSON-based messaging stuff, and after taking to Rocky it seems like I'll be able to pull this off (my web hosting company has the necessary permissions to host the `DataPortal` on my web site), but ... first things first - I need the business logic to work correctly!

[1] As of 3/10/2007, their web site says they're open at 6:30 AM on Saturdays. That's wrong; they don't open until 7 AM.

> Published: 03.10.2007 07:22:17 AM CST