---
title: Dispose Method to Null
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Dispose Method to Null

I wish the following code never existed ... but it did:

```c#
public class Source
{
  private XmlDocument document;

  // ...

  public void Dispose()
  {
    this.document = null;
  }
}
```

The class doesn't implement `IDisposable`, but it has a `Dispose()` method. Go duck-typing! And ... it sets an `XmlDocument` (which doesn't implement `IDisposable`) to `null`. Which, of course, will immediately free memory and ... oh wait, never mind.

What's worse is that it was used by other sections in the code without using using:

```c#
var source = new Source();

// ...

source.Dispose();
```

I should stress the words "was used" - that code has been destroyed :).

> Published: 08.19.2008 10:26:11 AM CST