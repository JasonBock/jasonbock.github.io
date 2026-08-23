---
title: `XPathNavigator` and the `xmlns` Namspace
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# `XPathNavigator` and the `xmlns` Namspace

I'm not an XML guy. I don't live in an angle-bracket world, and that's fine by me. Because of that, I get bit by things like this. Let's say you have an XML document defined like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Values xmlns="http://www.iamgoingtodriveyounuts.com">
  <Value></Value>
  <Value></Value>
  <Value></Value>
</Values>
```

If I want to find all the `Value` nodes, I have to do this:

```c#
XPathNavigator navigator = (new XPathDocument("Data.xml")).CreateNavigator();

XmlNamespaceManager manager = new XmlNamespaceManager(navigator.NameTable);
manager.AddNamespace("v", "http://www.iamgoingtodriveyounuts.com");

XPathNodeIterator iterator = navigator.Select(
  "./v:Values/v:Value",
  manager);
Console.Out.WriteLine(iterator.Count);
```

Basically, I'm defining a `v` namespace with the same value as the `xmlns` namespace in the document. Since you can't redefine `xmlns`, this was the only option I could find (and I didn't figure this out on my own; somebody on an internal Magenic list did it for me).

I didn't find a solution via my standard Internet searches, so I figured I post this in the hopes that I save someone else from the pain and suffering I inflicted upon myself today.

> Published: 11.08.2006 01:40:04 PM CST