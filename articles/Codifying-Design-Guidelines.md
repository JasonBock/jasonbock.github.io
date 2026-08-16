---
title: Codifying Design Guidelines
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Codifying Design Guidelines

I was reading a post a couple of days ago, and the thing that resonated with me was not so much the specific principles they have on ASP.NET MVC but that they had designs they wanted to enforce in the code base. This is a perfect example of why custom rules in CodeAnalysis (or Gendarme or NDepend) are so helpful. All of the rules they give could easily be written up in these static analysis engines and code could be automatically checked/reviewed at every build. No need to actually just write them down; implement them!

> Published: 11.04.2008 03:44:25 PM CST