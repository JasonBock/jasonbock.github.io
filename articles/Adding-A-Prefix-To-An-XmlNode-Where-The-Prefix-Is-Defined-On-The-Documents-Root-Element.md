---
title: Adding a Prefix to an XmlNode Where the Prefix is Defined on the Document's Root Element
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Adding a Prefix to an XmlNode Where the Prefix is Defined on the Document's Root Element

OK, the title of this post is a little long-winded. Here's what I ran into today [1] - it has a simple solution, but the solution wasn't inspired from me. I had to get some help from someone to figure this out. Here's the problem. Let's say you had a document that looked like this:

```xml
<root xmlns:drums="http://www.drums.com">
  <count>2</count>
</root>
```

Now I want to add a `<count>` tag that is prefixed with drums - the document should look like this after all is said and done:

```xml
<root xmlns:drums="http://www.drums.com">
  <count>2</count>
  <drums:count>2</drums:count>
</root>
```

My first attempt was to use `CreateElement()` like so:

```c#
drumsCountNode = drumDoc.CreateElement("drums", "count");
```

But that didn't work...

```xml
<root xmlns:drums="http://www.drums.com">
  <count>2</count>
  <count xmlns="drums">2</count>
</root>
```

My next attempt was to name the element all in one shot:

```c#
drumsCountNode = drumDoc.CreateElement("drums:count");
```

But that was no good either ...

```xml
<root xmlns:drums="http://www.drums.com">
  <count>2</count>
  <count>2</count>
</root>
```

Finally, someone suggested I use the 3rd overload of `CreateElement()`:

```c#
drumsCountNode = drumDoc.CreateElement("drums", "count", "http://www.drums.com");
```

That did the trick! Again, for those XML gods out there, this is probably child's play. It confused the hell out of me!

[1] For those who are wondering how I ran into this, think of the `<comments>` and `<slash:comments>` elements in an RSS feed ...

> Published: 01.24.2005 07:37:08 PM CST