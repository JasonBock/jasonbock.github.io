---
title: TestContext Property
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# TestContext Property

I found out today that the `TestContext` object that you get in a `ClassInitialize` method doesn't work across multiple test calls (at least `WriteLine` cacks on me after the first test). This link shows that can add a property to your class such that the framework will set the context for you ... but why wasn't a `TestContextAttribute` created in the framework so you didn't have to use a specific name for the attribute? Seems like an oversite to me ...

> Published: 11.29.2006 11:18:22 AM CST