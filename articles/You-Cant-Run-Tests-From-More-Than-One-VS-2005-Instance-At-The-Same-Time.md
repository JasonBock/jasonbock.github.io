---
title: You Can't Run Tests From More Than One VS 2005 Instance at the Same Time?
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# You Can't Run Tests From More Than One VS 2005 Instance at the Same Time?

Wow. I just ran into this scenario this morning (can't believe I haven't seen it before). When the 2nd VS instance tried to run the test, I got this:

> Test did not execute.  The test/run was aborted on agent '{my-pc-name-deleted}' due to error: 'Code coverage collection error: Another test run has locked collection engine. The test run execution on that machine cannot continue.'

That's awful. I hope this has been addressed/fixed/refactored in VS 2008.

UPDATE (12/18/2008): I just ran into this tonight ... on VS 2008. Ugh

> Published: 10.11.2007 09:47:33 AM CST