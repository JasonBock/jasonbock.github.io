---
title: HttpContext: Don't Mock
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# `HttpContext`: Don't Mock

In the past I've done work mocking `HttpContext`. But after reading more articles, I'm convinced that mocking `HttpContext` is usually not worth the pain and effort. Rather, change your code that uses `HttpContext` and its' properties to using an object that contains `HttpContext`. That makes your code easier to test because now you can pull lots of UI code into workflows and models and controllers and have tests around them.

> Published: 10.16.2007 07:52:28 AM CST