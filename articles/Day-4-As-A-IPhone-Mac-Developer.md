---
title: Day 4 as a iPhone/Mac Developer
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Day 4 as a iPhone/Mac Developer

Today I realized that you can plug in a standard mouse to a Mac and the right button works! I know, it's a stupid n00b thing, but hey, I didn't know it would work. The only reason I tried is a guy here grabbed the Mac for the weekend and he forgot to bring the mouse with it. Rather than have him run back home, I took the mouse that I used with my laptop and plugged it it. Worked with no problems, and that's cool cuz I was really missing my context menus.

Now that I have the core view/controller stuff in place, things should start moving along. I'm getting used to working in Xcode and Objective-C so I should be able to get more code in place.

I also learned that making a `UITextField` secure is easy ... well, at least it hides the text. Just go into Interface Builder, and check the "Secure" box on the Text Input Trails section.

I ran into the issue where the keyboard wouldn't go away in the simulator. I found the answer here - the "Brandon #2" comment that talks about the "releaseKeyboard" empty action idea did the trick for me.

> Published: 09.15.2008 08:23:03 AM CST