---
title: Secure Random Numbers
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Secure Random Numbers

Friday I had a discussion with a co-worker about choosing numbers for lotteries (and no, I didn't win the Powerball, like pretty much everyone else who played), and to make a long story short, I wrote a program to see if his argument made sense. When I did this, I ran into a familiar issue with random number generation in .NET. There's the `Random` class, but that's not as "good" as implementations of `RandomNumberGenerator`. Therefore, I decided to write up a little class that extends `Random` and implements its virtual methods with a secure implementation. You can get the code drop [here](https://github.com/JasonBock/SpackleNet). There are a couple of things I'd like to add/improve, but I have the feeling I'll find uses for this class in the future.

> Published: 02.19.2006 06:07:34 PM CST