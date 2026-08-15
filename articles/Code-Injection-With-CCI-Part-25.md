---
title: Code Injection With CCI - Part 2.5
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Code Injection With CCI - Part 2.5

Previous Parts

* [Part 1](https://jasonbock/articles/Code-Injection-With-CCI-Part-1.html)
* [Part 2](https://jasonbock/articles/Code-Injection-With-CCI-Part-2.html)

You might be able to tell that I'm writing this series "on-the-fly" a little bit. That is, I'm finding out things as I go along from the Cecil-to-CCI migration. One thing I found out is that you can override the `OpenBinaryDocument()` methods and use `UnmanagedBinaryMemoryBlock` to prevent `MetadataReaderHost` from locking the assembly you load:

```c#
public override IBinaryDocumentMemoryBlock OpenBinaryDocument(
  IBinaryDocument parentSourceDocument, string childDocumentName)
{
  return UnmanagedBinaryMemoryBlock.CreateUnmanagedBinaryMemoryBlock(
    childDocumentName, parentSourceDocument);
}

public override IBinaryDocumentMemoryBlock OpenBinaryDocument(
  IBinaryDocument sourceDocument)
{
  return UnmanagedBinaryMemoryBlock.CreateUnmanagedBinaryMemoryBlock(
    sourceDocument.Location, sourceDocument);
}
```

Now I can save my changed assembly right back to the file:

```c#
PeWriter.WritePeToStream(module, host,
  File.Create(module.Location));
```

Sweet!

I'll continue on in Part 3 with method modification - specifically, adding label support to correct branch offset values.

> Published: 04.29.2009 07:59:16 AM CST