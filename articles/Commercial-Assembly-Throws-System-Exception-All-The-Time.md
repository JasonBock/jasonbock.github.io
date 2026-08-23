---
title: Commercial Assembly Throws `System.Exception` All The Time
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Commercial Assembly Throws `System.Exception` All The Time

This week I'm helping out the client by doing a bit of R&D; with a number of components they're looking at to do a bunch of Word document manipulation. Today my task was to see how these components would convert nearly 2,000 Word documents into HTML. After getting the test run all set up, I fired off the test, waited a while, and gathered the results. My output grouped exceptions together to see what exceptions were thrown (if any), how many of a given "kind" [1] were thrown, etc. When the test was completed, I was shocked to find out that one of the components throws `System.Exception` for **every** exception it throws when it fails to load a Word document.

YIKES!

This is terrible. This is a component that costs thousands of dollars to purchase, and this is how they design their code? How can I write any kind of intelligent exception handling code around their component? I'd have to do something like this:

```c#
try
{
  // Use lousy component here...
}
// OK, let's "filter" out the exceptions we can't catch...
catch(OutOfMemoryException)
{
  throw;
}
catch(StackOverflowException)
{
  throw;
}
// ...and so on until I die of boredom...
catch(Exception ex)
{
  // Oh, great, NOW I have to pattern searches 
  // on the Message property :(
}
```

This company should be embarrased by their coding practices. As far as I'm concerned, they're out of the picture, and I'll be highly suspect of their products in the future.

If you're selling 3rd-party components these days, don't do stupid things - people will find out. For crying out loud, just run FxCop on your assembly! In fact, that's what I just did, and this component has 11,225 issues, 636 of which are of the DoNotRaiseReservedExceptionTypes issue. Their main competitor in this R&D; effort has 43 DoNotRaiseReservedExceptionTypes issues (still too many in my book), but they only have 1583 issues in total. By way of comparison, mscorlib has 11,435 issues, and they have 109 DoNotRaiseReservedExceptionTypes issue. However, you have to keep in mind that this rule breakage is not just reserved to throwing `System.Exception`; throwing `System.OutOfMemoryException` will cause it as well, and, frankly, mscorlib can throw that whenever it wants as far as I'm concerned. Throwing `System.IndexOutOfRangeException` trips this FxCop rule as well, and I'm not entirely in agreement with this (and the "good" component does that). But, the "bad" component **always** trips this rule by throwing `System.Exception`.

Ugh.

[1] By "kind" I mean the exception type plus its `Message` property value. I wanted to make sure I separated exceptions that may have had different messages in my analysis, just so I wouldn't miss any important exception messages.

> Published: 10.03.2006 11:05:37 AM CST