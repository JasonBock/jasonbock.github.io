---
title: Rant: OnXXX() .NET SDK "Help"
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Rant: OnXXX() .NET SDK "Help"

So I'm looking up `OnActivate()` so I can get a better understanding of when this method is invoked. This is what the docs tell me:

> Called when the form is activated.

Oh, gee, that's helpful. Maybe the help for `OnLoad()` is better:

> Raises the Load event.

Man, that's a boatload of assistance! I think I'm on to something - what does `OnResize()` say? Wait for it ...

> This member overrides Control.OnResize.

Fine, so I jump to `Control.OnResize`:

> Raises the Resize event.

**AAAAAAAAAARRRRRRRRRRRGGGGGGGGGGGGHHHHHHHHHHHH!!!!!!!!!!**

There are some sections in the SDK that are very helpful, but I find this section lacking. Even a one- or two-sentence description on when `OnLoad()` is invoked would be so much better than, "Raises the Load event." And please don't post comments about what these events do - my rant is targeted towards the quality of the help for all of the `OnXXX()` methods in WinForms controls, rather than what they actually do.

> Published: 01.12.2005 09:59:37 AM CST