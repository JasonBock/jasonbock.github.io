---
title: Collection Madness
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Collection Madness

For the past week or so I feel like I've been down a never-ending refactoring road that deals with code that does all sorts of things with a list. To make a real long story short, it's not been fun. I feel like I'm only making little steps here and there when I'm doing a lot of refactoring. It has to be done, though, to get us to a point to have a (somewhat) more flexible architecture in place. Sometimes I feel like LINQ would've saved us a fair amount of pain because so much of what we do is list traversal, filtering, reducing, etc.

One thing that I don't like in the code is somebody has a method where the list is passed in and modified, along with a by-ref parameter to get another list, and the return value returns yet another list. All of this stuff is necessary, but I'm getting more a functional mind-set with programming. One of the mantras I'm trying to live by is side-effect-free programming. In this case, the original list should've been passed into the function (or a function should've been applied to the list :) ), and the return value should've been a structure that contains the "modified" list and the two other lists. In my mind, that would be a much cleaner design (and actually, as I write this down, I think I should try and revisit a portion of our API soon ... if I can find the time ... ).

> Published: 07.02.2007 01:51:18 PM CST