---
title: Mirroring And Painting - Round 2 (No Solution Yet)
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Mirroring And Painting - Round 2 (No Solution Yet)

I've made some progress, but what I've found isn't reassuring. It seems that if I take out the following code in my custom form and user control:

```c#
this.SetStyle(ControlStyles.AllPaintingInWmPaint | 
  ControlStyles.DoubleBuffer | 
  ControlStyles.ResizeRedraw, true);
```

I no longer get the line drawing problem I mentioned before. However, this isn't good because I need that code to prevent this problem! So it feels like I've hit a catch-22: either I get the line drawing problems or I end up with smeared, incorrect, background images. Neither one is acceptable.

I thought about creating a control that handles right-to-left semantics on its own. That is, it would not use `WS_EX_LAYOUTRTL`; rather, it would flip everything by itself. But I really do not want to do that, because I can think of so many things that I have to modify in the control's layout and how it shows child controls that it would basically suck rocks. And that's the point of `WS_EX_LAYOUTRTL` - it does all of that for you.

> Published: 01.25.2005 03:11:45 PM CST