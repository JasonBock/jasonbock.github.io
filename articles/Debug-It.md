---
title: "Debug It!"
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# "Debug It!"

I finished [this book](https://pragprog.com/titles/pbdp/debug-it/) on a plane from Boston to Minneapolis. It's a high-level book that's designed to give developers a good perspective on how to find and fix bugs in their code properly.

Frankly, I wish this book would be read by a lot of developers. It's not long - you can devour it in a couple of readings. It's not too detailed on the specifics of a programming language or environment - while Butcher uses some Java code examples it's not a requirement to understand Java to read the book. Butcher focus on how a developer should systematically root out where a bug is and how to remove it effectively. Debugging is not about firing up a debugger - in fact, that has very little to do with debugging. Developers need to ensure they've found the bug and that they have steps in place to ensure it doesn't come back again. I also learned a couple of new tricks as well (the in-memory logger is a nice one).

There's one section of the book, though, that really irked me. Butcher creates a Dispatcher class in Chapter 2 where he demonstrates the basics of logging. However, his catch block catches the general Exception type. Argh!! This is a bad programming practice that no development book should ever repeat, even in code in a book. I've repeatedly run into teams who do this and defend their choice because "they read it in a book". Ultimately we're all responsible for the choices we make in life, and if we write code we're responsible for what we send to the compiler. But having books show this as an example doesn't help. Catching the general exception type causes so many more problems than people think it solves - I hope Butcher fixes this in a future edition.

I should emphasize, though, that even with this irritation, I would not hesitate to recommend this book to other developers. I've been on teams where the developers are severely lacking in debugging analysis skills. This book may not fix that problem in its entirely, but it's a very good starting point.

> Published: 02.22.2010 08:16:32 PM CST