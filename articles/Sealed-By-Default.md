---
title: Sealed By Default
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Sealed By Default

Ted suggest having sealed classes by default. Other people have voiced their opinions about this issue, and I tend to fall towards the side of non-extensibility. That is, designing classes for inheritance takes more effort than it does to make a class sealed, and generally I've found that most classes I write are not used in inheritance hierarchies. Therefore, I'm moving towards making my classes sealed by default these days. If I know I'm designing a class that will be inherited from, then the sealed keyword stays off. If developers want to extend my class, then we'll talk about it, and if it's needed, then I can open up the class. It's a lot easier to unseal a class than it is to seal a class that was used as a base class (that's why I also tend to make methods non-virtual and/or private if there's no good reason for a subclass to have access to a method. Same idea goes for fields, properties, etc.).

And man, is it **windy** in the Twin Cities today!

> Published: 04.18.2004 05:24:12 PM CST