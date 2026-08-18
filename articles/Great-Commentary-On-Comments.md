---
title: Great Commentary on Comments
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Great Commentary on Comments

I like [this](https://blog.codinghorror.com/coding-without-comments/). What's funny is Jeff makes a key observation - the more experience you get, the less comments you add to your code:

> Junior developers rely on comments to tell the story when they should be relying on the code to tell the story. Comments are narrative asides; important in their own way, but in no way meant to replace plot, characterization, and setting.

Well, at least that's what I've been doing as a software developer. I'm not 100% against comments, but I rarely use them. I've tried to use XML comments, but most of the time they feel redundant - the only use I get out of them is what the developer reports as exceptions that can be thrown from a method. I'm really trying to write code that can be easily understood just by reading it, and **only** using comments as the exception, not the rule.

Oh, and I'm also against putting the TODO or FIX comments in code. Because ... they never get addressed. Either fix it immediately when you see it, or if it's too hard to address at the moment, get a story in the development pipeline to refactor the code.

Oh, one more thing ... I hate it when people comment out code, especially when source control is in place. It does nothing but add confusion into the mix. And I love it when people respond to me after I've torn through a code base and removed all the commented code with, "hey!, I was going to add that code in later ... probably ... sometime soon ... ". If it's not compiled it, YAGNI.

> Published: 07.25.2008 08:37:27 AM CST