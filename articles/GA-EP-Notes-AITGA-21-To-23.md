---
title: GA/EP Notes: AITGA 2.1 to 2.3
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# GA/EP Notes: AITGA 2.1 to 2.3

Chapter 2 is all about demonstrating how GAs can solve problems. 2.1 discussed program evolution - that is, using a GA to evolve a Lisp program to model data. It immediately made me think of LINQ expressions ... that may be something I'll take a look at later on. Mitchell started introducing a change in the GA that people do, which is to just grab the top X percentage of fit chromosomes and use those for crossover and mutation, rather than giving all chromosomes in a population the chance to be eligible for crossover + mutation. She also mentioned that some modelers don't even use mutation; sometimes, the argument is that a large enough population with crossover will be enough to evolve suitable solutions.

Mitchell also dived into evolving cellular automata with GAs. This section was somewhat confusing to follow ... one of the things I'm finding a bit frustrating with this book is that the examples are just not clear enough. I feel like Mitchell does a decent job laying the groundwork with the examples but then the figures and resulting conversations around the examples muck things up. Anyway, basically the CAs Mitchell was trying to evolve (this was part of her own research) were tailored to find solutions where the density of the initial configuration was nearly 50% (i.e. equal number of 1s and 0s). This has application into parallel computing, although I found that connection hard to follow. It was interesting to see that that evolved CAs from the GA work was very close to what they were looking for.

2.2. was about data analysis and trend forecasting. Mitchell described the example of predicting blood flow and the stock market along with protien structures as her examples. Again, the diagrams used (esp. Figure 2.13) really made things more confusing that I was hoping for. Overall, this section showed how difficult it is to get the fitness functions and chromosome mappings down correctly. They are so key to achieving an optimal solution.

What's funny is that, after I finished these sections, I thought, "hey, I wonder if GAs have ever been used to evolve neural nets". Then I saw the title to 2.3: "Evolving Neural Networks" - w00t! That's the next section of material I'll tackle.

I'm also thinking of ways to refactor my `GeneticAlgorithm<T>` class to make it easier to test and work with. It needs to be generic enough to allow different approaches of chromosome types, crossover functions, mutators, etc. I have some ideas - one big test of the idea will be to see if I can do the program evolution idea using LINQ expressions.

> Published: 02.09.2009 08:03:04 AM CST