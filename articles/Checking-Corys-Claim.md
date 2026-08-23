---
title: Checking Cory's Claim
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Checking Cory's Claim

I saw [Cory's claim](http://addressof.com/posts/comparing-apples-to-apples-the-myth-of-benchmarks/), I decided to run the tests on my own machine. This is what I got when I ran run.cmd:

```
Start C# benchmark
Int      : 8151
Double   : 2263
Long     : 14210
Trig     : 1692
IO       : 2693
Total C# : 29009 ms
End C# benchmark
Start VB benchmark
Int      : 8262
Double   : 2283
Long     : 14080
Trig     : 1702
IO       : 3044
Total VB : 29371 ms
Stop VB benchmark
```

Here are the results from run1.cmd:

```
Start VB benchmark
Int      : 8112
Double   : 2273
Long     : 14210
Trig     : 1763
IO       : 3245
Total VB : 29603 ms
Stop VB benchmark
Start C# benchmark
Int      : 8191
Double   : 2263
Long     : 14190
Trig     : 1702
IO       : 2593
Total C# : 28939 ms
End C# benchmark
```

In both cases, C# was "faster", although the difference in my mind is so small that it validates my premise: it's not the language, it's the IL that's generated underneath the scenes. There may be slight differences between the two, but at the end of they day they're performing the same.

> Published: 11.15.2006 09:10:45 AM CST