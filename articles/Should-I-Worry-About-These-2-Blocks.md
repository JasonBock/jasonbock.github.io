---
title: Should I Worry About These 2 Blocks?
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Should I Worry About These 2 Blocks?

Consider the following code:

```c#
int result = NativeMethods.CallPInvoke(...);

if (result == 0)
{
  throw new CallPInvokeException("Unexpected error occurred in the call.");
}
else
{
  // Do something useful...
}
```

My tests are currently at 99.78% code coverage. The `throw` line of code is one of the sections that I'm missing.

Now, my question is ... should I really care? The case where result is zero is really bad, and I've never been able to get this call to return zero. Of course, I could set up some IoC and/or mocking thingamadoodle, but frankly, I don't see the worth in doing so. What I really care about is the stuff that happens in the `else` block.

I'm usually pretty gun-ho on hitting 100% code coverage, but in this case ...

So, what would you do? I'm curious to hear your thoughts.

> Published: 05.20.2008 05:56:53 PM CST