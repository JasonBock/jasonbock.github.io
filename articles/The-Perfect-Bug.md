---
title: The Perfect Bug
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# The Perfect Bug

Today I was trying to get my game program buttoned up to the point where I could release a 0.1 version. There's a lot of frills and features that need to be added, but, first things first - I need to make sure the board is working correctly. The game framework seems solid, so I moved onto the UI portion. After a while, I got it functional and stable, and I was able to play a game the way the rules are defined. The odd thing happened when I tried to print out the move history. I have unit tests around this, but when I displayed the results in a dialog box, nothing was right. This really confused the hell out of me - how can the game work when the move history is way off.

Then I noticed it. Each position was in the wrong place, but it was pefectly mirrored. What I mean is, if I picked a piece on a 4x4 grid (with the origin in the lower-left hand corner) at what I thought was position {3, 4} and I moved it to {3, 0}, what what being printed out was {1, 0} to {1, 4}. Since the game is symmetrical, it didn't matter from a playing perspective that I had the layout all wrong. Only when I printed out the history did the bug manifest itself.

I have to admit, that was a first for me. I had a bug that was so bad it was **perfect**!!

> Published: 04.06.2005 09:25:44 PM CST