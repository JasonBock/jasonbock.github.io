---
title: Right-To-Left and Painting Issues - SOLVED
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Right-To-Left and Painting Issues - SOLVED

Richard Birkby sent me an e-mail on this issue, suggesting that I should flip the `ResizeRedraw` bit on for the control style. That made it "better" but it didn't make all of the painting problems go away. However, he got my brain going and I eventually came up with this:

```c#
this.SetStyle(ControlStyles.AllPaintingInWmPaint | 
  ControlStyles.DoubleBuffer | 
  ControlStyles.ResizeRedraw, true);
```

That fixes it. Now my image doesn't get smeared. I'm still a bit bothered that I don't understand why I don't have to do this when the control isn't mirrored, but at least I have a solution.

> Published: 01.06.2005 01:26:32 PM CST