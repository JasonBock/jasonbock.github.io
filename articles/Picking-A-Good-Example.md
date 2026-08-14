---
title: Picking a Good Example
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Picking a Good Example

This weekend I saw someone give a presentation on unit testing and the benefits in having unit tests and following TDD. Overall, the talk was good, but one thing I noticed was the problem he picked seemed awkward for an example. Basically, he wanted to test an algorithm that would return all the prime factors for a given integer. With the TDD approach, he'd write a test for 1 that would fail, update the implementation to return the factors for 1, rinse, lather and repeat.

I don't think that's a good example.

Why?

You end up writing tests that drive the implementation to be basically a table-driven switch statement. This feels backwards. I've done this myself using Roman numerals as the example and it felt awkward seeing the developers writing tests that drove an implementation that ended up looking like "if 1 then "I", else if 2 then "II", else ...".

Don't get me wrong - you absolutely want tests for your code, even if the algorithm is generalized. And I also get that the intent of the exercise is just that: an exercise to understand the practice of TDD. But for the purposes of illustration, you want to pick a problem where the boundary conditions are minimal, like FizzBuzz. You can actually show all the tests that you would need to ensure the problem is solved.

> Published: 05.05.2010 08:34:27 AM CST