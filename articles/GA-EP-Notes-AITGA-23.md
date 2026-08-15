---
title: GA/EP Notes: AITGA 2.3
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# GA/EP Notes: AITGA 2.3

2.3 is all about evolving neural networks. Mitchell gave a very bried description of neural nets, what weights are, how feedback is used, etc. She then dove into a couple of examples where GAs were used to evolve the weight and configuration schemas of the networks themselves. She described the different techniques used - again, representation is key (i.e. the differences between direct and grammatical encodings).

I also spent some time refactoring my `GeneticAlgorithm<T>` class. What I've found is that the algorithm itself is pretty simple - there's really not much to the general GA concept. What seems hard is getting the representations correct and what tweakings you need to apply (e.g. what's the mutation probability? how should I do crossover? etc.).

While I can't spend all my time doing all the examples in the book, things become more concrete once I try to actually reproduce what was described in the book (e.g. creating the `GeneticAlgorithm<T>` class). My next step is to spend some time working with LINQ expressions to see what I can do there. After that, it's on to Chapter 3, which is called "Genetic Algorithms in Scientific Models".

> Published: 02.10.2009 08:51:46 AM CST