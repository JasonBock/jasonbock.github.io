---
title: "Code Leader"
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# "Code Leader"

I finished this book a couple of weeks ago - I forgot to write my usual book review and since I have some free time now I thought, "hey, time to write that review!"

Overall, I think the book serves its purpose. It does a decent job in laying out the tools and processes developers should follow to make the software they write better. This includes testing, continuous integration, DI/IoC, and other related topics. The book is written well and I think it's a book that any developer can gain something from by reading it (although I think it serves the less experienced crowd better).

That being said ... there were some statements and points made in the book that I didn't get or that I think are errata. That's fine - I've had my own share of mistakes in my books - but I think that they should be pointed out, especially with a book that's trying to get developers to write better code:

* On pg. 67, he makes the statement that the default section in the switch statement "will never be tested." That's incorrect - you can write a test that casts a value that's not in the enumeration (like 37) to the enumeration type, and that'll make its' way down to the default part of the code.
* On pgs. 150 and 151, he keeps switching the precondition around to ensure that the divisor is not zero. The correct one IMO is the one at the bottom of pg. 150, where he ensures it's not equal to zero. The other two only check that it's greater than zero ... which doesn't make sense, as 6/-2 is perfectly valid.
* He never acknowledges that there's a testing framework in Visual Studio. I could be wrong on this, but I don't think I ever saw the phrase "MSTest" in the book. I know that some people have serious issues with MSTest (e.g. only available in certain versions of VS, personal distate for the framework, etc.) but at least mention it. Again, I could be wrong on this point, but I don't recall seeing MSTest in the book.

What I liked in the book overshadows what I didn't like or disagreed with. Even if I disagreed with a statement, at least that shows that he got me to think about a point he was making. Pick it up and read it if you can. It doesn't go as far in-depth as I would've liked, but it's still a good read.

> Published: 08.21.2008 10:33:42 AM CST