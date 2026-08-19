---
title: "Beautiful Code"
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# "Beautiful Code"

I finished this book a week ago - I'm finally getting around to posting my thoughts. It was very hit and miss. Some of the chapters were very good (I'll list a couple later on), and some were just awful. Jeff's comments pretty much sum up my thoughts on the content, although there's one thing that he didn't mention that caught my eye. It's this:

```c#
catch(Exception ex)
{
  // ...
}
```

This is a book about beautiful code, and developers are showing code that catches the base `Exception` type? That's a **big** no-no, and I'm amazed that this was in the book. It may seem like a minor nit to some, but to me this knocked the book down a notch. I realize that each author was only responsible for their own content, so I don't blame the authors as a whole. I just wish the chapters would have been edited by developers who knew the target language and could provide feedback before the book went to print (and if they were, then the Java reviewer missed the boat on these code examples).

As I said before, there were some real gems in the book. Here's a list of my favorite chapters:

* "The Most Beautiful Code I Never Wrote", by Jon Bentley
* "Finding Things", by Tim Bray
* "Correct, Beautiful, Fast (In That Order): Lessons From Designing XML Verifiers", by Elliotte Rusy Harold (and his exception handlers were much better!)
* "On-The-Fly Code Generation For Image Processing", by Charles Petzold
* "Writing Programs for 'The Book'", by Brian Hayes

I really can't recommend buying the book, because it's just not solid enough through and through to warrant the price tag (although all royalties go to Amnesty International, which isn't such a bad thing either).

> Published: 02.28.2008 08:56:02 PM CST