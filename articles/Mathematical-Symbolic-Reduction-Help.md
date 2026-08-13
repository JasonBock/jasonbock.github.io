---
title: Mathematical Symbolic Reduction - Help!
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Mathematical Symbolic Reduction - Help!

One of my many side projects evolves mathematical expressions using LINQ expression trees. One issue that I'm trying to address is reducing or compressing [1] the size of the tree by finding simple conditions in the tree. For example, if I find "a * 1", I know that can be turned into "a", which gets rid of two nodes. The problem is that I know there's a lot more that I can do. For example, one time my program came up with this formula:

```
((a * ((a * a) - a)) + (a * a))
```

Doing some simple algebraic manipulations:

```
a * ((a ^ 2) - a) + (a ^ 2)
(a ^ 3) - (a ^ 2) + (a ^ 2)
(a ^ 3)
```

So what I really have is something that's far simpler than the original formula [2]. Unfortunately, my rules are extremely simplistic at this point and don't catch the reduction that I can do by hand.

Now, I could figure out a lot of these rules (eventually), but what I'm wondering is, are there any engines/papers/information/etc. out there that can guide me in creating my own framework to reduce my expression trees. I know there has to be stuff out there - I mean, Mathematica handles my example with ease. But finding information hasn't been easy for some reason (I'm guessing my search-fu isn't using the right terms).

Any ideas are appreciated - just leave a comment. Thanks!

[1] I call it "compressing" because there's a Reduce() method on the Expression class, but it doesn't do anything that I'm looking for.

[2] I ran some tests that showed that the original expression is quicker than the one where you have to use Math.Pow(). But my tests were thrown together really quick - I'd have to do more investigation to solidify my initial observations.

> Published: 06.29.2010 01:46:29 PM CST