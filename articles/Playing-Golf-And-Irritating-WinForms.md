---
title: Playing Golf and Irritating WinForms
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Playing Golf and Irritating WinForms

I played golf on Thursday at CreeksBend Golf Course, where I shot a 45-49. The back nine has never treated me nice. I also played on Sunday at Ridges at Sand Creek - shot a 44-45. I played from the whites on the front 9, but switched to the blues on the back. It's a nice course, but they had a lot of problems last year with slow play. Both times I played it, I could only finish 9 holes as it took 3 hours to play the front nine and I would not have had enough sunlight to finish. By the way, I encourage you to take a look at their web site because it's a good example of what **not** to do for menu navigation. I hate that floating menu bar! As soon as you move it somewhere on the current page and then go to another page, the bar is back to its' default position.

I'm also finding some slight, yet irritating behaviors when desiging WinForm applications in VS .NET. If I change the icon in an `ImageList` control so a `ToolBar` control will have some new icons, the changes don't apply when I run the application. Sometimes, I have to go into `InitializeComponent()` to move the initialization code for the image list **before** the tool bar is initialized. That's important, as the list needs to have the icon information loaded before the tool bar tries to get the images. Sigh ...

> Published: 07.08.2002 12:00:00 PM CST