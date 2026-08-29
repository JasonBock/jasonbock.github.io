---
title: When The Obvious Move Isn't So Obvious
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# When The Obvious Move Isn't So Obvious

I'm still working on the [Quixo .NET](https://github.com/JasonBock/Quixo) game - it's currently up to 0.3.0.1. I tried adding dockable windows, but the two freeware .NET components I found ... well, they didn't play very nice, at least they weren't as nice as I wanted them to be, so I just put stuff in panels with splitters for now. One thing that I thought was interesting as I'm playing the alpha-beta pruning engine is how the min-max algorithm works, because sometimes it surpises the hell out of me. For example, take the following game where X is me and O is the `AlphaBetaPruningEngine`:

![Obvious Moves](https://jasonbock.net/images/Obvious-Move-1.png "Obvious Moves")

Now, O can make 4 in a row by moving {3, 4} to {3, 0}, which would give this:

![Obvious Moves](https://jasonbock.net/images/Obvious-Move-2.png "Obvious Moves")

But here's what it did:

![Obvious Moves](https://jasonbock.net/images/Obvious-Move-3.png "Obvious Moves")

This surpised me. It happily made 3 in a row with its' previous move, so why doesn't it go for 4 in a row? The issue lies with the minimax algorithm. A potential next move may seem great, but your opponent's responses may render that useless, and it fact the response may be more damaging than another potential move, so you miminize your losses and go with the move conservative move. Or, a previous move scores the same as a move you think is good, but since it's no better the algorithm discards it as it thinks it already has a "good enough" move.

Let's get into a bit more detail with this. I logged the engine's search tree when it had to make a move after X's 4th move (the 7th move). It searches 9 potential moves before it gets to {3, 0} to {3, 4} (the move it ends up making for move #8). It scored this move as -16. Why? Well, it thinks that the next best move for X that it finds first is {2, 4} to {2, 0}. Again, since it's trying to minimize damage, it won't anticipate that the opponent will do something stupid (like {3, 0} to {4, 0}), so it assigns the {3, 0} to {3, 4} response as the best response with minimum damage.

Now, by the time it gets to the move I thought it would pick ({3, 4} to {3, 0}), it gave it the score of -16. Even though this gives O a big advantage in the short run, X can easily break the line of 4 with {2, 4} to {2, 0}, and then the engine thinks it's at a -16. Again, X **could** do something really stupid at move #9 like {4, 0} to {0, 0} (which would give O the win with {0, 4} to {0, 0} at move #10), but the minimax approach assumes the opponent will play well, so it doesn't pursue this line. Since {3, 4} to {3, 0} scores the same as {3, 0} to {3, 4}, it assumes it already has the best move with {3, 0} to {3, 4} and keeps that one. **That's** why it doesn't make a line of 4. It's not that it's any worse (in its' "mind") than the move it takes; it's just that it "thinks" it's no better than a move it already found.

I find all of this a bit confusing yet fascinating at the same time. I'm wondering if I throw in a bit of randomness to the search that it would "spice" things up or make it worse (i.e. find the top 2 or 3 moves and randomly pick one). Right now, my engine is "predictable" because it will always play the same move to a given board configuration, and that may not be the "best" way of doing things. There's so much I don't know about game theory and related topics, and that's why this stuff interests me so much.

> Published: 04.19.2005 11:21:00 AM CST