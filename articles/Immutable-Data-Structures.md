---
title: Immutable Data Structures
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Immutable Data Structures

Read [this post](https://learn.microsoft.com/en-us/archive/blogs/ericlippert/path-finding-using-a-in-c-3-0-part-two):

> Immutable data structures are the way of the future in C#. It is much easier to reason about a data structure if you know that it will never change. Since they cannot be modified, they are automatically threadsafe. Since they cannot be modified, you can maintain a stack of past “snapshots” of the structure, and suddenly undo-redo implementations become trivial. On the down side, they do tend to chew up memory, but hey, that’s what garbage collection was invented for, so don’t sweat it.

Sometimes I drive fellow developers nuts because I make my classes read-only/immutable by default. But as Eric said, it makes things easier to reason about if you can do that.

> Published: 10.04.2007 12:08:28 PM CST