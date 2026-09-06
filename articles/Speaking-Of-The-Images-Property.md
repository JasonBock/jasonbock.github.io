---
title: Speaking of the Images Property
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Speaking of the Images Property

Yesterday I mentioned some issues I had with WinRes. Well, I forgot to complain about the *Image Collection Editor* dialog box that pops up when you want to change the value of the *Images* property of an *ImageList* object. Here it is:

![Images Property](https://jasonbock.net/images/Images-Property.png "Images Property")

Now, can you tell me what those images are? I have to stare at it for a second or two before I realize they're a word balloon, a hat, and a bell.

Believe me, the image resolution becomes **very** important when you're trying to change the images for a different culture. Since the application expects the images to be at a certain index, I can't screw the order up (which also reminds me of another nit: I wish this dialog window had a *Replace* button so I could easily replace the image highlighted in the list with another). I have to go through the code and figure out, ah, the image at index 3 is the image for the *Enter* key, remove the current image, add the new one, and move it up at index 3 (not screwing anything up in this sequence, either).

It would be so nice if the dialog window would display the image full-sized if you double-clicked on it. Or, add a region under the Properties grid that would give a bigger preview of the image currently selected. But, unfortunately, this is where things stand today. I also checked VS .NET 2005 (Beta 1), and it's the same exact window.

Sigh ... oh, well ...

> Published: 12.22.2004 12:57:00 PM CST