---
title: WinRes and ImageList Objects
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# WinRes and ImageList Objects

Let's say I have a form that has the `Localizable` property equal to true. I drop an `ImageList` on to the form, and then I add three images to the `Images` property. I save my project and all is well with the world.

Until I open the .resx file for the form in WinRes:

![WinRes and ImageList](https://jasonbock.net/images/WinRes-ImageList.png "WinRes and ImageList")

As you can see, there's no `Images` property for `applicationImageList`. The image data is definitely in the .resx file (encoded as base64, but it's there). I'm wondering if WinRes has problems showing collections, but this kind of stinks because I don't have an easy way to add culture-specific images to the .resx file. If you know of a workaround or if there's a way to do this without resorting to something analogous to open-heart surgery please let me know - thanks in advance.

> Published: 12.21.2004 03:26:02 PM CST