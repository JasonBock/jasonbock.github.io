---
title: Never Mind, It Was My Own Dumbass Mistake
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Never Mind, It Was My Own Dumbass Mistake

The caching problem had nothing to do with FTP. It was a completely different problem. It all came down to this line of code:

```c#
XPathDocument quoteDoc = 
  /* Get the document from the cache. */;
XPathNavigator quoteNav = 
  quoteDoc.CreateNavigator();

int quoteCount = 
  int.Parse(quoteNav.Evaluate("count(./Quotes/Quote)").ToString());
Random rnd = new Random();
int randomQuoteIndex = 
  rnd.Next(1, quoteCount + 1);
```

It's that last line of code that was the issue. It used to just have `quoteCount` as the 2nd parameter. The problem is that `Next()` included `minValue` as part of the range, but not maxValue. The SDK is clear on this; I just assumed both parameters were included in the range. Therefore, the new quote that I added this morning was never showing up in the quotes section.

As a side note, just **why** was `Next()` designed this way? While the document is correct, I feel that code should be as self-documenting as possible. It just seems odd that one value is part of the range, but the other one isn't. I'm curious why this method was coded this way.

> Published: 09.20.2004 08:28:58 PM CST