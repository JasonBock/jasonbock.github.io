---
title: "Working Effectively with Legacy Code"
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# "Working Effectively with Legacy Code"

I finished this book a week or so ago. Basically it's a book that catalogs ways to approach legacy code such that you can get it into a manageable state (hopefully). Feather's definition of legacy code is a good one - it's any code that doesn't have unit tests around it. I like that as it makes it very clear that if you're not writing tests, you're already in a bad position. The key point I got out of the book wasn't all of the strategies (although they're pretty good) - it was that one guiding principle that, if not followed, puts you into a myriad of situations that the book is trying to get you out of. If you write good unit tests, then, well, really, you don't need this book, unless you're put on a project that has legacy code.

Some of the examples I found somewhat useless (the C and C++ ones) but that's only because I haven't been in those languages for years.

I think the book is written well, and the guidance Feathers gives is good. I only wish all developers [1] would have strong testing in place from the get-go. Then we wouldn't need a book like this. That's not a cut on Feather's effort - just a comment on the sorry state of code bases in our industry. I don't know how many times I've run into developers that resist unit testing or, frankly, don't even know that they should be doing it. At least with the second group you can educate them; the first group may require mental hand-to-hand combat to change their ways. Maybe this book, if thrown on their desk, would wake them up to the perils ahead if they don't write good tests right away.

[1] This includes me - I don't always write tests up-front, but it's always cost me in the long run.

> Published: 06.21.2010 07:58:33 PM CST