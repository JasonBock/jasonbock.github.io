---
title: Cecil Can Be Maddening
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Cecil Can Be Maddening

As much as I like [Cecil](https://github.com/jbevain/cecil), it can also be really frustrating to work with. It has very little documentation, so trying to figure out how things work or why things don't work can be irritating. I'm currently running into an issue where I'm trying to get all of the properties from a `TypeDefinition`, but the `Properties` collection has no values. All of the properties are on the base class, and for some reason Cecil doesn't "flatten" things out, so it doesn't consider those properties to be part of the subclass. I don't know why it's acting this way, but it's definitely not what I expected.

I'm also getting weird duplicate metadata token issues whenever I change method implementations. I have no idea what I'm doing wrong - this almost feels like a bug.

Oh, and if you ever add local variables to a method, make sure you set `InitLocals` to `true`.

Cecil is one of the best options out there for assembly modification, but it also has some warts on it as well.

> Published: 11.05.2008 12:11:29 PM CST