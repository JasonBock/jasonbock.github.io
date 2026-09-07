---
title: No Parser for C#
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# No Parser for C#

Well, I should've done my research on this beforehand, but I didn't expect `CreateParse()` to return `null` when I wanted to parse a .cs file. From what I've been able to find on the Internet, it looks like the 1.0 version doesn't supply a parser for C#. This really sucks. I wanted to take a C# code file, parse it, change it in a couple of places, and then compile it. But since there's no parser, I either have to write one myself or look for a different approach. Maybe there's a different approach I can use, or someone has written a parser for C# that uses the CodeDOM classes, but it doesn't look like it.

> Published: 04.21.2003 12:23:00 PM CST