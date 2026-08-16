---
title: Exception Inconsistencies
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Exception Inconsistencies

While I'm preparing for my exceptions talk next Saturday, I stumbled across this odd fact, which I noticed when I was reading the docs for [this Code Analysis error](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca2208): The arguments between `ArgumentException` and `ArgumentNullException` are reversed. That is, in `ArgumentException`, the offending argument name goes as the second parameter, but in `ArgumentNullException`, it's the first one.

It's a small inconsistency, but man, that's quite bizarre!

> Published: 10.03.2008 01:09:50 PM CST