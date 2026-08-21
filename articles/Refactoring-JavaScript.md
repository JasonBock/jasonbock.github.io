---
title: Refactoring JavaScript
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Refactoring JavaScript

My current task is to do a refactoring sweep of our JavaScript. I've tried to introduce OO concepts into the JavaScript I've written, but there's still a lot of functions all over the place. Developers have done the "copy-n-paste" approach to code reuse, and ... well, it's kind of ugly. I spent a half-day going through all the JS code I could find and came up with a plan of attack. I have a love-hate relationship with dynamic languages. There's a lot of power there, but one little thing missing and you could end up going down a black hole of despair.

Speaking of JS refactoring ... we tried to use one of Microsoft's ASP.NET Ajax controls in our app, and it doesn't work. The reason? Deep within their JS, they look at everything within `this.prototype`. Well ... we have code that adds methods to `Object` and their code rudely bombs if anyone else dared to add things to that member. Oh, well ...

> Published: 05.03.2007 08:03:24 AM CST