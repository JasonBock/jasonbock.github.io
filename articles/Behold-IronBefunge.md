---
title: Behold, IronBefunge!
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Behold, IronBefunge!

A week or so ago, I did a talk on esoteric programming languages for the Twin Cities Languages User Group. One of the languages that I dived into was [Befunge](https://en.wikipedia.org/wiki/Befunge), a language that is quite devilish. Take a look at this program:

```
v  > .v
   3
>#v?4.@
  12
  >> .^
```

It randomly prints a number from 1 to 4 and then quits. Obvious, right? Just keep this in mind: follow the arrows, "?" randomly changes your direction, "#" is a trampoline instruction, a period prints what's on the stack, and "@" means "quit".

Anyway, I got to talking with some attendees and it seemed like a fair amount of attendees found Befunge to be quite interesting (probably from an insanity standpoint, but never mind that detail :) ). The more I thought about it, the more I wondered...just how hard would it be to write a compiler and/or interpreter written in .NET for Befunge ... like [IronBefunge](https://github.com/JasonBock/IronBefunge)? I've had some free time lately so I dove in to see what I could come up with.

Well, it's not easy. Part of the reason is that the "specs" for the language (both [Befunge-93](https://catseye.tc/view/Befunge-93/doc/Befunge-93.markdown) and [Befunge-98](https://esolangs.org/wiki/Funge-98)) are not as complete as I'd like. There's good information for sure, but there are some ambiguities (e.g. if an instruction expects a certain amount of values to be on the stack but the stack doesn't have that many ... you should just magically pop enough values on the stack for the operation to work). All that said, I do have a minimally working interpreter (i.e. it'll run the random number generator program above along with standard "Hello World" programs). I'd like to have the code available, but as of the time I originally posted this article, CodePlex's TFS was offline. Keep checking the IronBefunge site - once I can get my code up there you can check out the 0.1 version. The code is pretty messy right now - I need to do a lot of refactorings and cleanups, add support for more instructions, up the code coverage numbers (which are currently at 66%) and add some execution dump information for debugging purposes - but it's been fun to get it up to this point. Enjoy!

> Published: 07.16.2009 02:58:48 PM CST