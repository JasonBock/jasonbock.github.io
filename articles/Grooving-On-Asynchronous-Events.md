---
title: Grooving on Asynchronous Events
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Grooving on Asynchronous Events

A while back I lamented that the "Add Web Reference ... " function in VS 2005 didn't generate the traditional `BeginXXX()` and `EndXXX()` methods I was so fond of. Now ... I'm seeing the light with the asynchronous eventing model, and how `AsyncOperation`, `AsyncOperationManager`, and `SynchronizationContext` all flow and work together. It's really cool, and it makes coordination and composition of asynchronous work pretty straightforward. This will definitely change my talk around a bit for the Dec. 2006 Twin Cities .NET User Group but it's for the better.

> Published: 09.18.2006 01:18:21 PM CST