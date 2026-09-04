---
title: Owner-Drawn Text and Mirroring Revisited
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Owner-Drawn Text and Mirroring Revisited

Unfortunately, I have to revisit this problem once again, and, frankly, I'm stuck. I've included a quick demo project to illustrate the problem - I'm hoping someone can help me figure out how to solve this.

Basically, when you run the app, you should see this:

![Owner-Drawn Text and Mirroring](https://jasonbock.net/images/OwnerDrawn-Mirroring-1.png "Owner-Drawn Text and Mirroring")

Now compare that with the form when it is not mirrored (which can be toggled via the button on the form):

![Owner-Drawn Text and Mirroring](https://jasonbock.net/images/OwnerDrawn-Mirroring-2.png "Owner-Drawn Text and Mirroring")

The key difference is between the first line of text (".Draw this unformatted text") and the text in the label (".Reference purposes"). The first line is owner-drawn using a `StringFormat` object. It comes out correct when the form isn't mirrored, but when the form is mirrored, the text literally becomes mirrored. That's not what I want, and the other two lines of text are my unsuccessful attempts to correct the problem. And I need to solve this using owner-drawn text; I can't use labels or transparent labels due to how the resulting control in the real application is used and subsequentially sent to the printer. It's all done through the Graphics object. More to the point, I really don't want to do a major code change to their custom report printing process - if there's a small change to the text drawing code that's a much better option.

I've run out of ideas. If you have **any** suggestions, I'd greatly appreciate it. I'll give credit on this site (along with a big, warm virtual hug). Since I still do not have comments on my site (which I've started to work on - yes! it's true!), send your suggestions here. Thanks in advance!

> Published: 01.03.2005 01:49:49 PM CST