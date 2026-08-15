---
title: GA/EP Notes: AITGA 1.4 to 1.9
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# GA/EP Notes: AITGA 1.4 to 1.9

I started getting into how a GA works. Mitchell gives a simple definition of the GA in 1.6, going over selection, crossover, and mutation, although I didn't see a discussion on termination (i.e. when do you stop the GA?). The example she gives was OK, but I would've liked a better overall description of the example. When I wrote my master's thesis, my advisor drilled into my head that you can never have enough examples - they are key to helping the reader understand what you're describing with a definition of an algorithm. I think it would've been very beneficial for Mitchell to actually go through a sample run of the example she gave, rather than going through one iteration of it.

I tried writing my own GA in C# (called `GeneticAlgorithm<T>`). It was quite fun to write but what I've noticed is that my chromosomes don't converge. She states in 1.6 that values will "eventually" converge, but that's not what I saw. I realize that there's no guarantee that a GA will yield the optimum solution but it seemed odd that it never even got close. Basically, I took the idea of having a string like this:

```
"001010100101010101010101011010101010101"
```

and having the goal be a string of all 1s. The most optimum solutions I got were around 75% of the alleles being 1s but that's about it. So either the way I have `GeneticAlgorithm<T>` coded up is way off, or GAs converge really slowly, if at all. Still looking into this.

Mitchell spent some time classifying the term "search" and discusses which searches GAs should be used for: "search for solutions", where you're trying to find a solution to a problem in a large search space of candidate solutions. She also discusses different applications of GAs and discusses two cases in detail: the iterative Prisoner's Dilemma and sorting networks. The sorting networks idea was interesting because it dived into diploid chromosomes and how that works in GAs along with coevolution - that is, the fitness function evolves along with the generated solutions.

> Published: 02.06.2009 07:13:29 AM CST