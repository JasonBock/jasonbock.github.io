---
title: Owner-Drawn Text and Right-To-Left Woes Progress
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Owner-Drawn Text and Right-To-Left Woes Progress

Thanks to the help of Bill Wagner, I was able to come close to a solution. I can now get the text to draw correctly (and after Bill nudged me in the right direction, I had a "duh-that-was-easy!" moment), but there's another problem. I've uploaded a new version of the sample project - here's what's going on now.

Since I'm now drawing the text to a temporary image, there are some issues going on with anti-aliasing and other GDI+-tomfoolery that I have no clue about. Here's what happens when the text is drawn on the form with no background image (note that it no longer matters if the form is mirrored - the problems show up either way):

![Owner-Drawn Text](https://jasonbock.net/images/OwnerDrawnText-1.png "Owner-Drawn Text")

Note that the second line of text (the one that is drawn to a bitmap first) isn't as sharp as the first line. It gets worse if I don't set the `SmoothingMode` and `TextRenderingHint` properties on the `Graphics` object as I am in `DrawRtlAndFlipped()`:

![Owner-Drawn Text](https://jasonbock.net/images/OwnerDrawnText-2.png "Owner-Drawn Text")

Bill found a way to clean it up if there is a background image:

![Owner-Drawn Text](https://jasonbock.net/images/OwnerDrawnText-3.png "Owner-Drawn Text")

So, while I can now owner-draw text correctly in mirroring scenarios, the text doesn't look quite right when there isn't a background image. Note that this is no longer a problem with mirroring - it's just a GDI issue, so I'm hoping there's a better chance that someone can solve this problem. Let me know if you have any ideas.

> Published: 01.04.2005 05:05:32 PM CST