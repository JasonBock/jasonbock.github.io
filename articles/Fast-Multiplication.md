---
title: Fast Multiplication
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Fast Multiplication

I'm a sucker for large arithmetic. [My master's thesis](https://epublications.marquette.edu/theses/3962/) was based on using the [number theoretic transform (NTT)](https://en.wikipedia.org/wiki/Discrete_Fourier_transform_over_a_ring#Number-theoretic_transform) to calculate pi. [Today](https://terrytao.wordpress.com/2008/10/02/the-lucas-lehmer-test-for-mersenne-primes/) on [Terence's blog](https://terrytao.wordpress.com/) [1] I saw him mention the [Schönhage-Strassen algorithm](https://en.wikipedia.org/wiki/Sch%C3%B6nhage%E2%80%93Strassen_algorithm), which is a specific kind of NTT. Reading about that algorithm took me back for a bit, talking about rings and primitive roots of unity and whatnot. What's interesting is that for astronomically large numbers there's actually a faster algorithm called [Fürer's algorithm](https://en.wikipedia.org/wiki/Multiplication_algorithm#Further_improvements).

Just remember, though, 640 K of memory was considered a lot not so far in our history :).

[1] I make no claims in being able to follow Tao's blog. His posts are way, **way** over my head. Today I just got lucky :).

> Published: 10.02.2008 12:27:47 PM CST