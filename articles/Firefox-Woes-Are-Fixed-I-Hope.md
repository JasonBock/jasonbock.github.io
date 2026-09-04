---
title: Firefox Woes Are Fixed, I Hope
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Firefox Woes Are Fixed, I Hope

OK, I thought I fixed the `textarea` thing, but then someone noticed that hovering over the comments area changed the color of the labels to yellow. It didn't take too long to figure out what it was, but the "fix" puzzles me. Here's the deal. My anchor tag to mark the comments section looked like this:

```xml
<a name="comments" />
```

IE was OK with this, but Firefox seemed to consider that as an open anchor tag. So I changed it to this:

```xml
<a name="comments"></a>
```

Now both browsers are happy. Go figure.

> Published: 01.12.2005 03:34:24 PM CST