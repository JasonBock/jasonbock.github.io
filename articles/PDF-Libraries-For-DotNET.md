---
title: PDF Libraries for .NET
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# PDF Libraries for .NET

My current project deals with creating and manipulating PDFs. I've looked at TallComponents, but I can't figure out for the life of me how I can determine if the text I put into a document is really there. In other words, in my unit test I want to be able to browse the PDF I just created and assert that, yep, the calculated amount of $102,506 is in the document. The PDFKit is good for overlaying text on a given PDF, but it doesn't have a great object browser to persue the current content, and none of the document-related objects and classes contain any information on the current document that help me evaluate if the new content is there. Their TallPDF library is kind of the reverse: a great object model to create PDFs on the fly, but it doesn't appear that you can open a PDF with their objects.

So, what I'm looking for is a PDF component that I can use in .NET that will allow me to add text to a standard form (something like a government tax document or a loan application). It will then allow me to re-open that document such that I can write a unit test to confirm that the new PDF has all of the information I expect given a predefined data set. If you know of a component that can do this for me, please let me know (and thanks in advance).

> Published: 07.20.2004 10:51:05 AM CST