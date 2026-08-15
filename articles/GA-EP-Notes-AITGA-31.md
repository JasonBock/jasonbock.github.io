---
title: GA/EP Notes: AITGA 3.1
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# GA/EP Notes: AITGA 3.1

In Chapter 3, Mitchell starts to dive into using GAs to model/simulate the real world. Evolution is a messy theory to test for many reasons, but insights into the process can be gained through computer simulations. GAs can help out with these investigations. 3.1 talks about a number of models that have been done to look at the Baldwin effect (no, this has nothing to do with the actors). Basically, it states this:

> ... the capacity to acquire a certain desired trait allows the learning organism to survive preferentially, thus giving genetic variation the possibility of independently discovering the desired trait...In this indirect way, learning can guide evolution, even if what is learned cannot be directly transmitted genetically.
Mitchell presents two models that demonstrate learning definitely has an effect on the evolution of a population.

I'm also still working on what I call ExpressionEvolver. Basically, this takes an initial population of LINQ expressions and evolves them using a GA according to the rules Mitchell gave in the "Evolving Lisp Programs" section in Chapter 2. This required that the expression is mutable, which it isn't, but using this library got around that problem. My main issue now is dealing with infinities. Let's say I'm trying to find an expression that matches this one:

```
a => a * a * a
```

What I do in my program is generate a list of objects that contain an input value and the expected result based on the given expression. I then evaluate the chromosomes using this list. The problem is if I get an evolved expression like this:

```
a => (((a * (((a * a) * (a * a)) * ((a * a) * (a * a)))) * (a * a)) * 
  (((((a * a) * ((((a * a) * (a * a)) * ((a * a) * (a * a))) * a)) * 
  (((((a * a) * ((((a * a) * (a * a)) * ((a * a) * (a * a))) * a)) * 
  ((a * a) * (a * a))) * a) * (a * a))) * a) * (a * a)))
```

Which is basically this:

```
a => (a ^ 43)
```

So, when I give an input of 47.23, I should get 105354.681067, but with that monster expression I get 9.803e+71. I'm using a mean squared error approach to determine the fitness, but as you can imagine this can quickly end up to an positive infinity double value. This causes all sorts of havoc that I'm not dealing with effectively (I basically end up getting an unhandled exception). I'm not really sure what to do with generated expressions like this. Obviously, it's so far off from the target that it should probably be elimited, but then I have to deal with the possibility that my entire population is completely out of whack.

I think I'm going to go down the path of simply eliminating the "negative infinite fitness" chromosomes (I end up multiplying the fitness result by -1 so the less negative the fitness, the better) and somehow keep the population size the same. Of course, having a arbitray precision data type would be nice, and I know .NET 4.0 will have `BigInteger`, but what I really need is `BigDecimal`. If you have any other ideas on how to handle this, please let me know.

> Published: 02.16.2009 07:55:37 AM CST