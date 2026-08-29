---
title: A Quixo Battle
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# A Quixo Battle

I'm still working on my [Quixo .NET](https://github.com/JasonBock/Quixo) game. Just as an experiment, I pitted my engine against another Quixo game. I turned it up to its "genius" level, and my depth search was at 3 ply. It basically ended up a draw - here's the state it ended in:

![Quixo Draw](https://jasonbock.net/images/Quixo-Draw.png "Quixo Draw")

The rival program kept moving its' O piece at {3, 4} to {4, 4}, and my program countered with {4, 1} to {4, 0}. So, both programs wanted to keep the positions they had, and we ended up repeating the positions, so I finally stopped the experiment.

It's interesting to note that the rival program took up approximately 3 times less memory than mine (5.8 MB to 16.3 MB). However, the rival program took a long time to generate a move, and it pegged the processor very hard. In fact, it got so bad that it would adversely affect the performance of other applications.

I may try this again - I'm interested to see if other games end up in the exact same state.

> Published: 04.16.2005 10:05:18 PM CST