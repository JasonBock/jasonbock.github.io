---
title: GA/EP Notes: AITGA 1.10
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# GA/EP Notes: AITGA 1.10

The last section of Chapter 1 dealt with discussing how GAs actually work. The algorithm is fairly easy to implement, but it feels like "magic" when a solution just "pops out". This section goes through some math to illustrate how GAs do what they do. I spent some time trying to understand schemas, the Schema Theorem, etc. and I kind of get it but it's still a little nebulous. Future chapters should cover these topics in detail.

I also figured out that GeneticAlgorithm<T> was working just fine. I had written a LINQ query wrong so I wasn't getting the best chromosome; I was getting the worst. Adding "descending" fixed that! Here's a plot of a simple run that shows the fitness and best solutions going up, up, and up:

![Average Fitness](https://jasonbock.net/images/Average-Fitness.png "Average Fitness")

![Best Fitness Score](https://jasonbock.net/images/Best-Fitness-Score.png "Best Fitness Score")

> Published: 02.06.2009 02:30:10 PM CST