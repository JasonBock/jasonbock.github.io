---
title: Mirroring And Painting
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Mirroring And Painting

You know, just when I think I've solved a mirroring issue, I run into another one. This problem seems very insidious and tempermental ... it's feels like a real PITA. So here we go!

It all starts with a simple WinForms application:

![Mirroring](https://jasonbock.net/images/Mirroring-1.png "Mirroring")

Now I resize the form by grabbing the bottom portion of the window:

![Mirroring](https://jasonbock.net/images/Mirroring-2.png "Mirroring")

Everything still looks normal. The problem is when I shrink the window such that its Height value to 69:

![Mirroring](https://jasonbock.net/images/Mirroring-3.png "Mirroring")

It's hard to see, but the left-side of the label now has a black line. In fact, all of the labels have this:

![Mirroring](https://jasonbock.net/images/Mirroring-4.png "Mirroring")

The problem is that this doesn't happen when the application is not mirrored. In other words, if I set the `IsMirrored` configuration setting to `Off`, the application looks fine no matter how I resize the window.

Also, the initial size of the window seems to have some relation to when the black line manifests itself. Here's a table of some initial window size values along with the Height value when line problem occurs:

| Initial Width | Initial Height | Height Value That Causes Label Line |
|----:|----:|-----:|
| 376 | 312 | 69 |
| 600 | 200 | 53 |
| 248 | 360 | 91 |
| 248 | 160 | 91 |
| 376 | 160 | 69 |
| 600 | 800 | 53 |

See the trend? There seems to be a correlation between the initial width and the height at which the problem occurs. Why this is ... I have no idea. And it gets worse when you start to add images on to a form (via a `PictureBox` or a background image on a `Panel`). You start to see lines show up on the left-hand sides of the containers like you do with the labels. Also, I've noticed that the text painting in the label can get messed up with the problem occurs:

![Mirroring](https://jasonbock.net/images/Mirroring-5.png "Mirroring")

If you look closely, you'll see that the text in `Label2` is getting painted improperly.

This is starting to become a big issue with the client. I'm going to keep digging, but I'm starting to quickly reach the edge of my knowledge with GDI and window drawing. Again, the code is available here if you want to take a look at it. Any help/ideas are appreciated - thanks in advance.

> Published: 01.25.2005 10:55:26 AM CST