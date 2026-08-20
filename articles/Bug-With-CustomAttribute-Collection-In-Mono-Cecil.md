---
title: Bug With CustomAttribute Collection in Mono.Cecil?
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Bug With `CustomAttribute` Collection in Mono.Cecil?

UPDATE: Never mind ... chalk this up to PEBKAC :(

I spent some time playing with Cecil this afternoon, and I ran into something that feels a lot like a bug. If I add a custom attribute on a method, method parameter, etc., it never shows up in the collection from the `CustomAttribute` property. If I look at the target assembly in Reflector or through the Reflection API, it finds it just fine. I didin't see anything that says that Cecil doesn't support custom attributes, so ... has anyone else seen this?

> Published: 11.09.2007 03:17:41 PM CST