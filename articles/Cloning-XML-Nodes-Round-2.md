---
title: Cloning XML Nodes - Round 2
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Cloning XML Nodes - Round 2

Someone at Magenic pointed me to the `ImportNode()` method on an `XmlDocument`. Here's the code demonstrating how this works - it's an update of the code from my previous post and I've bolded the text to highlight the change:

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

  nodeToAppendTo.AppendChild(
    destinationDocument.ImportNode(
    nodeToCopy, true));
  XmlNode nodeToTest = 
    destinationDocument.SelectSingleNode(
    "./outer/middle/inner/middle/inner");
  Assert.IsNotNull(nodeToTest, "The copy did not work.");
  Assert.AreEqual(nodeToTest.InnerText, "My inner x text.", 
    "The inner text is incorrect.");

  XmlNode nodeYToCopy = sourceDocument.SelectSingleNode(
    "./outer/middle[@value='y']");

  nodeToAppendTo.AppendChild(
    destinationDocument.ImportNode(
    nodeYToCopy, true));

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

It's a much cleaner approach in my book - I knew there had to be a better way.

> Published: 12.15.2004 10:01:03 AM CST