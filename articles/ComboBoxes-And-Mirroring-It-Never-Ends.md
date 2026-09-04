---
title: ComboBoxes and Mirroring ... It Never Ends!
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# ComboBoxes and Mirroring ... It Never Ends!

I was trying to debug a problem today with Arabic text being "reversed" in a printout. After fiddling and futzing with this, I finally narrowed down the problem. I realized it had nothing to do with printing - it's actually a problem with the `ComboBox` class. Here's a screen shot to illustrate the issue:

![Mirrored ComboBox](https://jasonbock.net/images/ComboBox-Mirroring-1.png "Mirrored ComboBox")

Yeah, I can't read Arabic either (I grabbed this text from an article - I have no idea what it says, although someone at the client who can read Arabic told me that it's equivalent to "print test". Sounds good to me!). So here's the test app in English to illustrate the issue further:

![Mirrored ComboBox](https://jasonbock.net/images/ComboBox-Mirroring-2.png "Mirrored ComboBox")

Notice that the `Label` has the period character on the left-hand side, which is "correct" for English text going right-to-left. The `ComboBox` doesn't do that. Now, if you go back to the first screenshot, you'll see that the text has been reversed. Then it dawned on me. What happened was that someone noticed that the Arabic text was showing up in reverse when it was in a `ComboBox`. So they ended up storing the text in reverse! This fixed it in terms of what the user sees in the UI, but during a print job the text looks wrong, even though it's printing out the Arabic text exactly as it is in the string.

I tried to create a custom `ComboBox` class that did all the mirroring stuff I've done on other controls to support mirroring, but that did nothing. So ... I'm kind of stuck now. I'm thinking that I could just reverse the string, but that's not the same thing as having the text go right-to-left. Otherwise you'd see ".txet tseT" in the English screenshot, and that's obviously not right. I'll keep banging my head at this. What would really be nice is to have something that works like `Graphics.DrawString()`, except that it would return a string reference of the text that it drew, because `DrawString()` can take a `StringFormat` object, which has a `FormatFlags` property that I set to `DirectionRightToLeft` when I draw text for right-to-left scenarios. If anyone has ideas on how to fix this just leave a comment - thanks!

> Published: 01.20.2005 02:33:46 PM CST