---
title: Big Breakthrough With My Dynamic Proxies
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Big Breakthrough With My Dynamic Proxies

I finally had some time to sit down and figure out what was going wrong with my dynamic code file for assemblies built with Reflection.Emit. It didn't take long to figure it out - I was using the wrong file name for the `DefineDocument` call. Duh! Oh, well - at least I now know that all my hard work getting this to execute was worth it, as I can now debug my emitted code. It's still not perfect as I have some alignment and code generation issues, but it works. I hope to have something published on my web site this week (I'm taking the entire week off so I should have enough free time to get the bits + code out).

> Published: 01.01.2007 07:20:56 PM CST