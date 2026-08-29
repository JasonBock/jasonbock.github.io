---
title: What Is the Next Card? Here's the Solution
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# What Is the Next Card? Here's the Solution

Yesterday I saw this puzzle. I spent some time thinking of a solution, but over lunch a fellow Magenic consultant and I figured it out. This entry documents how I came to the solution I did. By the way, if you don't want to know what the answer is, just read the problem statement and stop there. The solution is actually fairly simple, but like anything in life, it's simple once you figure it out.

In case you can't get to the page that mentions the problem, here's the original text (edited and modified a bit, but the essence of the problem remains):

> Jason and Bob are captured by some foreign government. The guardian says that he can free them, if they win the following game. First, Bob must choose five random cards from a standard 52 card pack. Then, he must send one card at a time (through the guardian) to Jason. When Jason receives the fourth card, it must correctly guess the fifth card that Bob still has. If he guesses correctly, they are both free. Otherwise, they will both stay in prison forever.
>
> Can Jason and Bob exit the prison in these conditions?
>
> Assumptions:
>
> * Before the actual game, Jason and Bob can talk together for one day and decide their playing strategy. But, after that, they cannot communicate in any way - they are now completely isolated forever (or, at least, until they get out from the prison). Also, they are completely isolated from the outer world (no cell phones) etc. The only method to communicate between them will be through the cards that Bob sends to Jason.
> * The time interval between delivering the cards cannot be used as a method to communicate information. In fact, the guardian might randomly delay delivering the cards to Jason just to make sure that they are not using this trick to send additional information.
> * Bob cannot alter the cards the he sends in any way to send additional information. In fact, after receiving a card from Bob, the guardian might send to Jason another (almost identical) card of the same type, to make sure he didn't use the card to send additional clues to Jason.

When I started reading the problem, I didn't want to read the comments as I **really** wanted to find a solution for myself. Unfortunately, I saw a comment that said something about the fact that you'll always have two cards that are of the same suit, but I immediately stopped reading the comments so I wouldn't get any more hints. As it turns out, that's a part of the solution, so I'm a bit bummed that I wasn't able to get to the solution all on my own. But as you'll see, I ended up "pairing" with another guy and we both figured it out.

Anyway, while I knew I had to come to a general solution, I started with some easy cases. For example, let's say Bob [1] draws a straight flush, like 2H, 3H, 4H, 5H, and 6H [2]. In this case, he gives me the first four cards in sequence and I'll know that the 5th card is 6H. This will also work for four in a row, or even three in a row. For example, if I see 5S, 6S, 2D, and JH, I know that the 5th card is 7S; it doesn't matter what the 3rd and 4th cards were.

Another set of easy cases are when Bob would draw four of a kind or three of a kind, like 4S, 4H, 9S, 4D, and 4C. With four of a kind, Bob gives me three of the four up front. The 4th card doesn't matter; I'll know that the 5th card is the other card in the set of four. With three of a kind, it's a little harder, but manageable. For example, if Bob draws 6H, 3S, 6D, 6C, and 8S, Bob could send two of the sixes first, but the order of the first two cards matter as the suits of those cards convey the suit of the last card. So, let's say Bob gave me 6H, 6D, 3S, and 8S. The 3rd and 4th cards don't matter. I know from the first two that the last card is a 6, but is it a club or a spade? Well, we'd have some agreement that if I see H-D, then the 5th card is a club. If I saw D-H, then it would've been a spade. Similar rules could be created for the other cases.

While these easy cases are fine, there's still too many cases that can occur that they don't address. I needed to come up with a general solution, but other things took precedence last night so I put the problem solving on hold until lunch. I mentioned the problem to Bob, but I didn't want him to figure it out and then tell me the answer. I enjoy problem solving, but for some reason it always takes time for me to come up with an answer, so I didn't want any help. I should've known better. Recently in my career as a developer I've tried to do "pair programming" as it's usually better (and more efficient) to work with someone to solve problem rather than sweat it out on your own. As you'll see, Bob and I both had pieces to the answer, but only through a collaborative effort did we come to a solution.

So now that Bob knows the problem, he can't help himself. He starts doing all sorts of calculations and guesses in his head, completely ignoring the juicy burger on his place. Sure enough, after a couple of minutes he announces that he thinks he has the solution. I'm crushed. What's the magic that my brain doesn't have to solve problems like this so quickly? So I gave in and asked him how in the hell did he solve it so fast? He proceeds to tell me his solution, but after we debate it for a while, we both realize it's not good enough. Then it hits me. He's on the right track, but...aha! Then it all falls into place. We reviewed the plan, and yep, it works [3].

Here's the solution - I'll use the hand 3C, 4D, 9S, 10C, and JH as an example hand to illustrate how the solution works. First, Bob and Jason know that they'll always have at least two cards that are of the same suit. Bob takes the two cards (if Bob has more than two of the same suit, he'll just pick two - as you'll see in a moment it won't matter which two he picks) - one will be the first card Bob sends over and the second will be the one Jason will guess. So, you have the first problem out of the way - after Jason sees the first card, he'll know what suit the 5th card is. However, they have to be a bit careful which card they send first, because the next three cards will tell Jason what the value of the 5th card is, and if Bob sends the wrong one over the guess will be incorrect.

Here's the problem. Bob has to communicate with the other three cards what the value of the 5th card is. What he does is use the 1st card's value as a starting point. Then, with the next three cards, he tells Jason what to "add" to the card to get the value of the 5th card. He does this by putting the cards in a specific order, and Jason will interpret the 2nd, 3rd, and 4th cards in a "greater than" fashion. See, Bob and Jason decided that a C is bigger than a S, which is bigger than a H, which is bigger than a D. If two cards are of the same suit, then the value of the card itself determines which one is "greater" (with a K being the highest value and an A being the lowest). Then, Bob and Jason agree that the order of the three cards corresponds to a specific value. Let's say the 2nd card is "greater" than the 3rd card, which is "greater" than the 4th card. This results in a 1-2-3 pattern, which translates to the value 1. If card #3 was greater than #4, which is greater than #2, this results in 2-3-1 pattern, which translates to the value 4. See the pattern? Jason determines how the three middle cards are ordered, and uses that information to get an offset value. It doesn't really matter how the ordering-to-offset translation is determined; so long as Bob and Jason agree on the mapping, all will be well with the world. Let's say they agree on the following mapping [4]:

* 1-2-3 == 1
* 1-3-2 == 2
* 2-1-3 == 3
* 2-3-1 == 4
* 3-1-2 == 5
* 3-2-1 == 6

This is where the selection of the 1st card is important. We only have 6 values to use as offset value. In our example hand, our two cards are 3C and 10C. The difference between them is 7, if one limits themselves to only looking at the set of cards in a given suit going from A, 2, 3, ..., J, Q, K. If you think of that set as being circular, you can get to a 3 from a 10 by going 10, J, Q, K, A, 2, 3. In other words, Bob only need to transmit a value of 6 to Jason if he uses 10C as the starting card. Then, Bob send over 4D, JH, and 9S. Jason will see a 6 encoded in the other cards (9S is bigger than JH, which is bigger than 4D, so that's the 3-2-1 pattern), so he uses the following formula to get the value of the 5th card:

> endingCardValue = ((startingCardValue + offset) MOD 13)

In our example, that comes to ((10 + 6) MOD 13), or (16 MOD 13), or 3. Hey, the 3C! [4] They're out of jail and partying hard!!

[1] By the way, "Bob" is Bob Knutson, the guy I worked with to come to a solution. He's a great guy to talk and he's one of the smartest guys I've ever met.

[2] "H" stands for hearts, "D" for diamonds, "S" for spades, and "C" for clubs. "J" is for jack, "Q" for queen, "K" for king, and "A" for ace.

[3] This is just another illustration of why solving problems seems to work better when there's more than 1 person around.

[4] Bob's original idea was to use the 4 cards to encode the value of the 5th card. As he realized, there weren't enough combinations to encode all 52 values, but his idea led us to the right solution.

[5] Note that a value of 0 from the formula means the 5th card is an ace.

> Published: 06.17.2005 02:25:18 PM CST