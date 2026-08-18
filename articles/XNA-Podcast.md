---
title: XNA Podcast
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# XNA Podcast

Jeff recently posted a podcast with fellow Magenicon Chris on XNA development. It's a pretty decent podcast on all things XNA-related, like the upcoming 3.0 release, the gaming community, etc. There was one point about random numbers, though, that I'd like to address. It starts around 28:09, where Chris is talking about the XBox architecture and performance (the actual quote I'm pulling is about 30 seconds after the time I mentioned - just wanted you to get some context for the quote):

> " ... certain parts of the framework where it just doesn't really make sense to put into the game. Some stuff that's just not performant. I mean, if you're using a lot of random numbers for example, you wouldn't want to use System.Crypto [1], it's just not fast. Right, you want to roll your own random number generator."

I get what Chris is saying. For most games, you don't need secure random number generation. A pseudo-random one is probably good enough (and Chris has written one that you can use outside of the Random class, which is discussed here). But be aware that hackers are tenacious. They will try to find every single frackin' hole in a game to exploit it. Here are a couple of links describing the issues: 1, 2. Again, for most hobby-like games, it's probably not an issue to be concerned about, but if you think you're going to create a huge selling game, then choose your generator wisely.

[1] There is no System.Crypto namespace; Chris is alluding to System.Security.Cryptography.

> Published: 07.01.2008 08:40:58 AM CST