---
title: GA/EP Notes: AITGA Chapters 4, 5 and 6
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# GA/EP Notes: AITGA Chapters 4, 5 and 6

I finished the rest of the "An Introduction to Genetic Algorithms" book during my West Coast trip. Chapter 4 spends its time going deeper into mathematical analysis to understand why GAs work. Chapter 5 talks about different ways to do the GA operations such as selection and mutation. Chapter 6 talks about future areas of research, although the book was written in 1996 so some of these areas may have already been addressed.

Two statements popped out at me as I was reading these chapters. The first one is from Chapter 4 in the "Results of the Formalization":

> " ... they conjectured that the formalism could shed light on the 'punctuated equilibria' behavior commonly seen in genetic algorithms - relatively long periods of no improvement punctuated by quick rises in fitness."

The second is this one (from Chapter 5 in "Tree Encodings"):

> "Tree encoding schemes ... have several advantages, including the fact that they allow the search space to be open-ended ... This open-endedness also leads to some potential pitfalls. The trees can grow large in uncontrolled ways, preventing the formation of more structured, hierarchical candidate solutions ... Also, the resulting trees, being large, can be very difficult to understand and to simplify."

I have seem both behaviors in my ExpressionEvolver. Sometimes during a run I'll have little activity and then all of a sudden a mutation kicks the GA down a new path. I've also seen large expressions kick in and it makes things harder to reason with and work with. Just more things to work on.

Overall, Mitchell's book is OK, but I found myself wanting more. The illustrations seem to actually hinder the conversation flow, rather than help and aid the reader. I was also hoping for better example descriptions. But all in all, I think her book has given me a decent start down the road to understand more about GAs and EC in general. My next book (which I've already started) is called "Evolutionary Computation 1" - we'll see how that goes.

> Published: 03.05.2009 09:40:58 AM CST