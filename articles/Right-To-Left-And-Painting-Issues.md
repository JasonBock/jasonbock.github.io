---
title: Right-To-Left and Painting Issues
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Right-To-Left and Painting Issues

The right-to-left/mirroring issues just keep on coming! Here's the latest thing I've ran into:

![Right-To-Left Painting](https://jasonbock.net/images/RTL-Painting-1.png "Right-To-Left Painting")

The previous picture looks "normal", but note that the application itself is in full right-to-left mode. Basically, I rotated the image before I loaded it in the Panel object because the image should not be mirrored, even if the application is right-to-left.

OK, everything is going as planned. But take a look at this screen shot after I make the window a little bit bigger:

![Right-To-Left Painting](https://jasonbock.net/images/RTL-Painting-2.png "Right-To-Left Painting")

Yikes! The painting is getting a bit messed up (although it goes away if I put another application in the front of it and then bring this application back to the forefront). The thing that's concerning me is that if I run the application without mirroring **I don't get the same painting problems**. Therefore, it feels like using `WS_EX_LAYOUTRTL` in `CreateParams()` does something "weird" to the painting code underneath the scenes.

In a way, it's fun being a client where I'm learning tons of stuff, but issues like this drive me slowly batty. As always, if you have any ideas please let me know.

> Published: 01.05.2005 04:47:18 PM CST