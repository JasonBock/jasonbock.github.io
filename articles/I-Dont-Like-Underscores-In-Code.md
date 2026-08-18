---
title: i_dont_like_underscores_in_code
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# i_dont_like_underscores_in_code

I tried to put my comment into a post, but I couldn't tell if it was accepted or not, and I've seen this kind of API style popping up more and more lately, so ... my post is born!

The thing about having a public .NET API using underscores is it doesn't fit the recommended coding style. If you're in Eiffel, that's actually what you do - in fact, you use upper-case and underscores (e.g. `GET_CUSTOMER`). But in .NET, public APIs should be Pascal-cased with no underscores - e.g. `GetCustomer`. CodeAnalysis rule [CA1707](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1707) helps enforce this style as well.

I actually like the interface that Jean-Paul came up with. I've seen other "fluent" designs pop up lately, and they all seem to use the underscore approach. The idea is nice ... just follow the .NET style.

> Published: 08.27.2008 08:12:49 AM CST