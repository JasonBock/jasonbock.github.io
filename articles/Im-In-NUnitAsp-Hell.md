---
title: I'm In NUnitAsp Hell
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# I'm In NUnitAsp Hell

Today I tried to add tests for a set of pages in the application, and none of them were succeeding. Here's the flow:

* I make a page request with some query string values, which will add an item to a cart, does some cookie work, and then redirects to a sign-in page.
* I then go to the cart page.

If I do this in IE, it works - I see the item in the cart. But in my NUnitAsp test, no dice - the cart is empty.

I think there's a bug in NUnitAsp with cookies and redirects, but I'm not sure. There's not a lot of material on the Net about NUnitAsp but here's what I found:

* First, this link. Basically, it seems like they knew there was an issue and they said they put in a fix on 6/6/2002.
* Next, this link. Some developers posted fixes for some NUnitAsp problems, one of which is `HttpClient` not supporting cookies correctly. Note the date of this post - 11/25/2003, or almost 1 1/2 years after they put in a fix. The response (click here) doesn't say if these fixes are necessary, or, if they are, if they were ever implemented in the main code line.
* Finally, this link. Matt hints at a problem with cookies and redirects. He has a solution but it doesn't seem to address the problem I'm seeing. Again, note the date of the post - 4/14/2004, nearly 2 years after the bug was supposedly fixed.

So, I think I'm screwed unless I dive into the NUnitAsp code base and try to figure out if there is a bug and then try to fix it. Unfortunately, there's other things that are important to get done, and debugging NunitAsp is not very high on the client's list. So...if you've run into this before and have a solution (or at least some insight) please post a comment - thanks!

> Published: 05.06.2005 03:20:46 PM CST