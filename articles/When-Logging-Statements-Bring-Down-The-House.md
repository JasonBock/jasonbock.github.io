---
title: When Logging Statements Bring Down the House
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# When Logging Statements Bring Down the House

Can you think of anything that could possibly go wrong with this code?

```c#
// query is a method argument.
int i = 0;
logger.Debug(string.Format("My query " + query + 
  " was processed at index {0}.", i));
```

We were running into some problems with one of our WCF services occasionally crapping out and giving us the all-encompassing `FaultException`. Since that exception is kind of opaque, we had to set `includeExceptionDetailInFaults` to true in our service behavior configuration so we could get more information. That's when we noticed that what was really going on was a `FormatException`. We only had one call to `string.Format()` in our service method, and after looking at it some more, we realized that query could contain the "{" and/or "}" characters, which would really mess up the formatting. Why this code was ever written this way in the first place is beyond me [1]; it should've been written like this:

```c#
// query is a method argument.
int i = 0;
logger.Debug(string.Format("My query, {0}, was processed at index {1}.", query, i));
```

That solved all of our woes.

This debugging story is kind of interesting because the initial analysis assumed that the service was dying due to either heavy loads or some other unknown factor. The reality was that it was a stupid implementation of a logging call; WCF was working just fine. I'm continually reminded that when I'm debugging a problem that I have to force my brain to look at all avenues and not assume I'm already down the right path with my first or second guess.

[1] After looking through source control history, guess who was the numbnut who wrote that code... ;)

> Published: 04.12.2007 06:43:34 AM CST