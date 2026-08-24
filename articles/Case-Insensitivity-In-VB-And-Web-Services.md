---
title: Case-Insensitivity in VB and Web Services
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Case-Insensitivity in VB and Web Services

I've added a Ladybug entry on this issue. Basically, if your service exposes fields in the message that have the same names, except one ends with "Field", you're going to run into trouble if you add it as a web reference in a VB project. Personally, I've never liked VB's case-insensitivity, but (as Rocky mentioned in an internal Magenic list), in this case I think the code generation tool is to blame. It should not generate code that can't be compiled, and I also found out you can't use the "Rename" refactoring tool either to fix this; it has to be a manual fix or a carefully invoked Find-And-Replace action. Yuck!

> Published: 07.05.2006 12:48:25 PM CST