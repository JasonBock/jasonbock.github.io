---
title: Quixo Improvements
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Quixo Improvements

I'm currently trying to tweak my [Quixo](https://github.com/JasonBock/Quixo) game. I moved it to 2.0 a month ago (refactoring to using generic collections in place of type-safe 1.x collections, etc.), and for the past couple of days I've been playing with changing the representation of the board to a long and twiddling the bits to get and set board piece values. I just got all of my tests to pass again, and here's the results:

* 1.1 Timing: 36.44
* 2.0 Timing: 21.32
* 2.0 Timing (Using Bit-Twiddling): 16.83

Basically I've cut the time that the "smart" engine takes to process a move in half. I'm pretty happy with that. The next step is to get engines do their next move calculation on a background thread so the UI can be more responsive, but that's for another day ...

> Published: 04.27.2006 09:10:55 PM CST