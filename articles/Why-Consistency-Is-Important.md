---
title: Why Consistency Is Important
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Why Consistency Is Important

I'm currently working on a code base that was developed by one person at the client. That person is no longer at the client, and a bunch of us are adding new features to the application.

It's **painful**.

Why? Because it's not consistent with what we've deemed as important to do in the code we write:

* Our coding standards weren't used, so navigating through the code is a much different experience that most of our code.
* Code Analysis wasn't used. We're trying to add that in right now (as that's a standard part of our build process) and we're finding all sorts of minor architecture and coding issues that, if Code Analysis was used from day 1, they would've been addressed much sooner.
* Unit testing is extremely sparce. One assembly has 30% coverage, and another assembly has 3% coverage. That's not good - to be blunt, it sucks.

Recently I've talked about why I think these tools are important in creating great code. Now, I'm back in a code base that didn't use them, and it's solidifying in my head why these tools and concepts are important. Does this code base run? Yes. Is it maintainable? No. And that puts us in a space that we've been trying to avoid for the past year: having to deal with code that's not easy to maintain.

> Published: 02.21.2008 08:30:34 AM CST