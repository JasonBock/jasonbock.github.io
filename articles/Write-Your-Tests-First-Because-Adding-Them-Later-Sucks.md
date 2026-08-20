---
title: Write Your Tests First ... Because Adding Them Later Sucks
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Write Your Tests First ... Because Adding Them Later Sucks

Seriously. Right now I'm trying to add tests around a section of code that doesn't have any, and it's not fun. Why? Here's a couple of thoughts that spring to mind:

* *It's boring*. Write the obvious tests. Check code coverage - what did I miss? Write more tests capturing more edge cases. Wash, rinse, repeat. Ugh.
* *It's not trivial*. I'm trying to put tests around code that was created months ago, and trying to capture the mental state of the coder and what he was doing is damn near impossible.

Write your tests first, or at least write them in conjunction with your new code. I find that it's so much easier to have your tests in place with new code rather than trying to bang out tests with untested code that's been around for a while. In the end, having tests is better than having no tests (assuming the tests are good ones) but don't wait to write them until it's too late.

> Published: 12.10.2007 07:56:13 AM CST