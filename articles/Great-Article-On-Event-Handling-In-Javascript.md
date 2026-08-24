---
title: Great Article on Event Handling in Javascript
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Great Article on Event Handling in Javascript

Today I ran into an issue where I needed to handle a button's `onclick` event that I was dynamically creating in my page. That sounds easy, but I wanted to handle the event via an instance method, not a static method, which is trickier because you need to preseve the `this` reference. We're trying to use OO principles where it makes sense in our Javascript code and I didn't want to suddenly break out of that in my design. Fortunately, I ran across an article, and ... wow, did things just click as I read this. I love how Brockman slowly goes through the process of how he got to his `bind()` solution. He really solidified some concepts I was still a bit murky on (i.e. closures).

> Published: 04.13.2006 01:10:07 PM CST