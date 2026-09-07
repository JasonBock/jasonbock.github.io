---
title: UriViewer, or, How to Make a Quick Object Viewer in 5 Minutes
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# UriViewer, or, How to Make a Quick Object Viewer in 5 Minutes

The more I develop in .NET, the more I run into the `Uri` class. I really like `Uri` as it makes it easy to parse URLs and related items. However, I have to admit that I'm not very adept with its properties (e.g. I always forget what the `Authority` property stands for). A while ago, I wrote a console application that I used to peruse the contents of a `Uri` object, but I wanted to move that into a WinForms application. Today, after working with the `Url` property off of a `HttpRequest` object, I decided to put the WinForms application together.

Now, my initial worry was that I would have to lay out all of the controls to represent the properties that are in a `Uri` object. Then, for some bizarre reason a chain of neural connections in my brain woke up as I was thinking through this problem, and I remembered the `PropertyGrid` control in WinForms. "No ... ", I thought, "it can't be that easy! Or can it? Hmmm ... "

![UriViewer](https://jasonbock.net/images/UriViewer.png "UriViewer")

You can get the code [here](https://github.com/JasonBock/UriViewer). To me, this shows just how cool the .NET Framework can be. I was expecting to spend anywhere from 30 minutes to an hour to render all of the data in a `Uri` object. By using the `PropertyGrid` control, I got this done in 5 minutes!

> Published: 06.24.2004 01:47:50 PM CST