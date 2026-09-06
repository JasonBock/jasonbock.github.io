---
title: If You're Creating a Custom Collection, Then, Damnit, Make it a Collection!!!
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# If You're Creating a Custom Collection, Then, Damnit, Make it a Collection!!!

I'm currently looking at a 3rd-party library to read, edit, and create PDF documents [1]. Since I'm not very familiar with PDFs and the vendor's object model, I decided to whip up a view ala my UriViewer to check out the main document object's properties. Well, it was able to load the PDF I'll eventually need to alter at run-time, but what I saw in the object browser irked me a bit. You see, the main object has properties called `Pages` and `Fields`, which, by looking at the names, you'd **think** were collections or arrays. Unfortunately, the property grid control wasn't expanding the items out like I'm used to, because the underlying objects were **not** collections. They simply descended from `System.Object`, and they only had the `Count` and `Item` properties that made them **feel** like collections. But in reality, they weren't.

If you're going to create a custom collection, then for god's sake, **make it** a custom collection! I can't use the **foreach** syntax on them; I have to manually iterate through them (which isn't hard, but if it quacks like a duck, then ... ).

[1] Since I just started looking at it, I'm going to refrain from naming the product as I may end up liking it. Every component I've ever used from a vendor always has what I consider to be design quirks, but the product as a whole may work well.

> Published: 07.07.2004 01:15:18 PM CST