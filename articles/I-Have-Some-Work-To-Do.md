---
title: I Have Some Work To Do
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# I Have Some Work To Do

Recently I received a bug report for ExceptionFinder. Basically, it has to do with my bad assumptions on the previous instruction before the `throw` opcode. So I decided to run every method in mscorlib through my analyzer.

The results weren't encouraging:

```
Total number of methods: 20314
Total number of errors: 3636
```

Yikes, I'm only handling 82% of the methods correctly! Granted, once I fix one I'll probably fix a bunch, but I will not make this add-in 1.0 until I can analyze every method in mscorlib with no exceptions. That doesn't mean I'm going to give the right results per-se, but I should not bomb on the instruction analysis. That's probably going to take some time to plod through, though ... hence the title of this post.

UPDATE (4/20/2008 - 1:02 PM): Error count is now 1928 ...

UPDATE (4/20/2008 - 3:27 PM): Error count is now 321 ...

UPDATE (4/20/2008 - 9:14 PM): Error count is now 211 ...

UPDATE (4/20/2008 - 9:45 PM): Error count is now 15 ...

UPDATE (4/20/2008 - 9:56 PM): Error count is now 0 - w00t! I'm not entirely thrilled with how I squashed the last bug, but I can live with it for now.

> Published: 04.19.2008 09:15:17 PM CST