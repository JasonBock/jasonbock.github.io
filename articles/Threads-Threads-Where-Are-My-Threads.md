---
title: Threads, Threads ... Where ARE My Threads?!
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Threads, Threads ... Where ARE My Threads?!

## Introduction

This article is meant to save you pain, wasted effort, coffee, tears, etc. If you've heard of the term **multithreading** you may have wondered, "Can I do this in VB?" Let me tell you right now: You can't (at least, not in the Win32 way, which is what I'm addressing in this article). Please, stop wasting your time - it's better spent in other places. I'll keep this fairly short, and I hope I keep you from long nights at your computer.

## `CreateThread`

When VB5 came out, it added the `AddressOf` operator, which in my opinion is one of the best additions Microsoft added to VB. Suddenly, a whole slew of API calls were open directly for the VB developer to use without purchasing a third party component. Timers, window enumerations, subclassing ... it was now accessable right from VB. A lot of articles came out that showed a bunch of different applications using `AddressOf` (my book included), and for good reason. VB's low-level power shot up pretty high with that one keyword.

One other call that required a callback was `CreateThread`. This call allows you to spawn new threads in your applications, thereby making your application multithreaded. Of course, you can make COM components that are multithreaded, but unless you do some code trickery, the objects won't exist in different threads in VB. Even with the code trickery, you have COM's interface layer to deal with, which is why raw Win32 threads are appealing. So appealing, in fact, that it didn't take long for articles and newsgroup posts to come out that showed how you could use `CreateThread` in VB.

Well, I was hooked. I read the articles (even the bad ones that passed COM object pointers to different threads). I bought a **lot** of books on multithreading. I found out when it was good to use, and when it added way too much overhead. After a while, I started to become really comfortable with `CreateThread`. Sure, you couldn't use VB's IDE to debug your apps. Sure, all bets are off if you try to use any functionality that comes with VB in different threads. Sure, people warned me that VB wasn't meant to work with multiple threads. That didn't deter me. Whenever I crashed my app, I would find out where the problem was (like accessing the `Path` property from the `App` object in a different thread) and find a workaround.

In short, I was starting to really like having this multithreading tool in my toolkit. When appropriate or downright necessary, I would use `CreateThread` in VB5.

How things changed with VB6.

## `CreateThread` No More

After I was done writing my book for Wrox, I was originally going to follow it up with a complete book on multithreading in VB, both from a Win32 and COM perspective. Of course, I was mulling over this before I had the upgrade. Once I got VB6, however, I was so caught up in the book that I had to wait to investigate multithreading in VB6. Plus, it really wasn't an issue. I mean, hey, it works in VB5 - why would it change in VB6?

Yes, I assumed. And that's plain stupid.

When I tried to compile an app in VB6 that I wrote in VB5, it crashed horribly. I was kind of shocked. Why did it crash? I looked through my code ... nope, nothing out of the ordinary. Recompiled. Crashed again. Anxiety started to rise. I started to hear rumors that VB had changed its threading model, but what did that have to do with `CreateThread`?

Everything.

I've distilled the problem down into one sample application. All the application does is call `CreateThread`, waits for the thread to finish via `WaitForSingleObject`, and exits. No forms, no components, just a simple program that writes three lines of information to a file. If you compile it in VB5, it'll work just fine. Compiling it in VB6 will cause an exception. It doesn't matter what OS you have or what service packs you have installed. It'll crash no matter what.

Was I steamed? Hell, yes! I posted this to a bunch of newgroups, and I did get a fair amount of responses. Some people thought I was nuts to try `CreateThread` in the first place. Some people were as mad as I was. Some people asked, "How did you get this to work in VB5?"

But the moral of the story is, there is no definitive answer as of the writing of this article. Whatever Microsoft did with VB6, it will not let you call `CreateThread`, and I haven't a clue as to why.

> In fairness to Microsoft, they posted an article about this "feature," which you can view here. I've tried the type library approach, and although somebody may make it work in all circumstances, I couldn't get it to work consistently. The same app that you can download in this article works in VB6 via the type library, but other, more advanced, VB apps don't. Of course, MS didn't say it was full-proof, but it's a pretty worthless piece of advice as far as I'm concerned.

## Don't Give Up!

I don't give up easy. I was starting to resign myself to a long and fruitful study of C++ when I stumbled across a tool called PowerBASIC. I finally decided to purchase it, and it's worth it. By combining the multithreading capabilities of PowerBASIC with VB6, you can have multithreading in VB6.

So, you may ask, how the %^&$ do you do it? Well, I've submitted an article to PowerBASIC that will be their Part 3 of a series they started on multithreading. They haven't posted it yet on their site, but once they do I'll add it to this page. Once the general concept is down, you can thread to your heart's content in VB. Sure, you'll have to fork over $179 to get PowerBASIC, but if you need the advanced programming capabilities that VB doesn't have, it's worth the investment.

## Addendums

Matt Curland has figured out a way to get `CreateThread` to work. It involves using a type library (so it appears that it is a part of the solution...D'OH!) marshaling interface pointers around ... in short, it's pretty ugly, but it works. He presented this at a VBITS conference in 1998 and now he's presented his code in the June 1999 issue of VBPJ.

> Published: 08.02.1998 12:00:00 AM CST