---
title: I Didn't Think This Was Possible
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# I Didn't Think This Was Possible

Yesterday and today (isn't that a band name? Anyway ... ) I was banging my head against the wall because a new Web setup project I added to our main .sln file was getting the output from another web project, not the one I wanted it to get output from. To make a long story short, somehow the project GUID for the two web projects were the same. Yeah, read that again: they were the same. That's why the deployment project was referencing the wrong project, because it wasn't - it just so happened to be first project in the .sln file with that GUID value.

Don't ask me how two projects got the same GUID value. But ... it happened! The only thing I can think of is that I somehow used the first project file as a "template" to create the second one and forgot to manually change the GUID value, and I'm guessing that's the problem.

> Published: 05.25.2005 11:58:56 AM CST