---
title: How Far Should You Push VB?
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# How Far Should You Push VB?

I thought I'd digress from the technical side of things and move up a level or so in VB. Every now and then, I have this discussion/argument about what you should and should not do in VB, and I wanted to step on my soapbox for a brief moment and get my 2 cents in.

To start, here's some of the comments pertaining to this topic that I've heard (I'm sure you've heard some other winners as well):

* VB is good for user interfaces, but that's about it.
* Microsoft doesn't make device drivers or write their OS code in VB, do they?
* It's a toy language.
* They used VB to create Microsoft Bob, right?! (snicker, snicker)

To be honest, I agree with the intent of the comments. I would never even attempt to write a driver in VB - use C or assembly for that. Nor would I try to write VB code to create my own operating system. And I think a fair amount of VB programmers have dug a hole for all of us by writing terrible, twisted code that leaves a lot of us shaking our heads, wondering what they were smoking when they used that global boolean to keep track of changed data in multiple forms. (Yep, I was that person - don't ask!). But I think the question of how far VB can go doesn't address the underlying issue. There's one that I think exposes a much bigger problem and can also help new VB programmers start off on the right foot.

I really don't know how many times (myself included) a programmer has started off in the IS world with VB as their first language. Having the ability to get Windows programs out to the desktop much faster than a C++ project is a very appealing feature to many areas of the business. But the problematic result is a group of programmers who don't understand standard coding techniques (read the book, "Code Complete," for an in-depth look at solid coding practices). They throw global variables everywhere. They put business and data logic right in the control events. And they leave the programs for other unfortunate souls who have to dive into those murky waters to enhance the programs. (They also gave new meaning to the acronym RAD - Really Awful Designs).

Again, I was guilty of all these sins on my first project. And I'm still learning how to be a more effective programmer. But the problem still exists for a new VB programmer in particular: How are you approaching the language? In my opinion, this statement should always be in the forefront of the developer's mind:

> VB is a Windows development tool.

Many people look at VB the first time and say, "Hey, I don't have to worry about pointers and weird API calls anymore! This is great! Web classes? Wow! COM? It's easy!" But even though VB makes things like COM and Windows easy, this can hide too much for the first-time user of VB (or Windows, for that matter). I don't know everything there is to know about my car, but if I'm going to play around with the engine I better damn well know what all the parts are, where they fit, and what I can and cannot do with them. This holds true for Windows programming in VB. If you want to program VB, approach it from a Windows perspective. Learn what some of those API calls can do. Read a book on Windows. Investigate the intricacies of the OS. **Don't** approach Windows from a VB perspective. If you do, when you get to that point where VB can't do it all (and believe me, you will get there), you'll be stumbling around when those nasty API calls that require the `AddressOf` operator pop up in your face.

There's no doubt that VB isn't as powerful as C++ is. But VB allows you to use a whole bunch of API calls that scream just as loud in C++ as they do in VB. I think you can get a lot of mileage out of VB if you have a fairly good understanding of the OS. It's a continuous learning process, to be sure - Windows is constantly changing. However, VB allows you to open the doors to some very interesting features if you're willing to learn how they work. With that in mind, I'm all for pushing VB to the limits because you can do a lot within the environment. (Like another fellow programmer once told me, "A hammer does have a specific purpose, and you can't use it well for everything, but some people have come up with some really clever uses for it!")

Conversely, don't be suprised when you've reached that limit imposed by VB. VB by design is meant to make Windows programming easier. The closer and closer you get to more advanced issues in VB like API calls, the more the ground beneath you starts to shake and crumble. And if you try to call `CreateThread` in VB6, you're in for a really nasty suprise. If you're tried every trick you know of and VB simply won't do what you want it to do, just give up and give yourself a pat on the back. You've found the limit. Now go learn C++! (Or, look at a tool called PowerBASIC).

> Published: 08.13.1998 12:00:00 AM CST