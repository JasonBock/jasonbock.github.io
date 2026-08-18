---
title: The Monster API
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# The Monster API

I'm currently on a project that requires interfacing to a system (can't say what it is, sorry) that has a C API, so this will require p/invokes. That's not the problem (although that's a little bit more work than necessary), but here's the fun thing. The function takes two strings (more or less), an in string and and out string. The two parameters are basically containers for literally hundreds of input and output values. In other words, the string will (probably) look like this:

> "000100220300P000010100..."

where each position (or a group of positions) represents a input value.

And, of course, there are no examples or unit tests that demonstrate how this is called.

Wow. I think I'm back in the 90s again ...

And hell yes. There will be a nice .NET API I'm going to make around this with unit tests so the rest of my code doesn't have to deal with monstrosity.

> Published: 04.24.2008 01:30:52 PM CST