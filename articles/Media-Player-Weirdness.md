---
title: Media Player Weirdness
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Media Player Weirdness

Yesterday was the first day we released our system on a beta site. Of course, there were a couple of hiccups - the typical stuff like a database script wasn't run. One problem was rather unexpected. We're embedding Windows MediaPlayer into the pages to show video clips, and up to yesterday it's been working with no problems. As soon as we viewed the beta site, we noticed that the player would automatically resize on the page. This whacked the placement of other elements on the page, so it was rather ugly.

I did a quick search of the control's properties, and I found two properties called AutoSize and AllowChangeDisplaySize. It appeared from the SDK that we didn't need to set these properties as the default values should've been false, but I decided to add them in anyway. Now the control doesn't resize and the page looks OK. The funny thing is, when we would load the page, we could see the control initially resize and then quickly go back to where we placed it on the page. Odd behavior.

> Published: 03.12.2003 09:09:13 AM CST