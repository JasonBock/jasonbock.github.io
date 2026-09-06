---
title: Cloning XML Nodes
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Cloning XML Nodes

I've run into a case where I need to copy XML nodes from one document to another. Unfortunately, using `Clone()` doesn't work as you'll get a `System.ArgumentException`, but using a combination of `InnerXml` and `OuterXml` does the trick. I'm sure this has been figured out by many people before me, but I don't want to forget how I did it, and in case you ever run into it, keep this code in mind:

```c#
public void CloneElement()
{
  XmlDocument sourceDocument = new XmlDocument();
  XmlDocument destinationDocument = new XmlDocument();

  sourceDocument.LoadXml(
    "My inner x text." + 
    "My inner y text." + 
    "");
  destinationDocument.LoadXml(
    "");

  XmlNode nodeToCopy = sourceDocument.SelectSingleNode(
    "./outer/middle[@value='x']");
  XmlNode nodeToAppendTo = destinationDocument.SelectSingleNode(
    "./outer/middle/inner");

  nodeToAppendTo.InnerXml = nodeToCopy.OuterXml;

  XmlNode nodeToTest = 
    destinationDocument.SelectSingleNode(
    "./outer/middle/inner/middle/inner");
  Assert.IsNotNull(nodeToTest, "The copy did not work.");
  Assert.AreEqual(nodeToTest.InnerText, "My inner x text.", 
    "The inner text is incorrect.");

  XmlNode nodeYToCopy = sourceDocument.SelectSingleNode(
    "./outer/middle[@value='y']");

  nodeToAppendTo.InnerXml += nodeYToCopy.OuterXml;

  nodeToTest = 
    destinationDocument.SelectSingleNode(
    "./outer/middle/inner/middle[@value='y']/inner");
  Assert.IsNotNull(nodeToTest, "The y copy did not work.");
  Assert.AreEqual(nodeToTest.InnerText, "My inner y text.", 
    "The y inner text is incorrect.");

  nodeToTest = 
    destinationDocument.SelectSingleNode(
    "./outer/middle/inner/middle[@value='x']/inner");
  Assert.IsNotNull(nodeToTest, "The x copy was lost.");
  Assert.AreEqual(nodeToTest.InnerText, "My inner x text.", 
    "The x inner text is incorrect.");
}
```

This is a NUnit test, which is why the `Assert` class is present, but you get the idea. There may be better, far superior ways of doing this (and I'm open to suggestions), but at least I know that there's one way to solve the problem.

> Published: 12.14.2004 03:23:30 PM CST