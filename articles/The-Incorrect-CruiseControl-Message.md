---
title: The Incorrect CruiseControl Message
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# The Incorrect CruiseControl Message

We still use CruiseControl at the client (although that will probably change in the near future ... ). Anyway, we've noticed that sometimes our build will break (due to a test timing out), and if we simply re-run the build, it'll work. Now, let me be clear: this means we have tests that need to change. I'm not faulting CC for this in anyway. But what I find funny is once the build succeeds, CCTray will say: "Recent changes have fixed the build."

Uh ... hey CC, nothing was changed!

I'm thinking that if a build succeeds after a failure, but no changes were detected, the message should be different, like:

* "Nothing fixed the build"
* "Your build is unstabled - fix it!"
* "Oh, sure, it's green ... for now ... "

Basically, a build that goes from red to green with no changes is not a good thing, and it would be nice if CCTray would acknowledge that :).

> Published: 12.19.2007 08:15:05 AM CST