---
title: Not Having FTP Access Stinks
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Not Having FTP Access Stinks

I usually post my entries via FTP to my site, but I don't have access to FTP at the company where I'm currently consulting. Of course, one could argue that I shouldn't be writing at the job, but ... well, never mind ;). I'll going to set up a file-posting page on my site so I can get my files up that way. Of course, I'll have to make this secure so my posts don't contain content that didn't come from me!

The current project is really cool, although I had to sign a NDA so I can't talk about what I'm doing. I always feel somewhat uncomfortable about it when I do this because it's sounds like, "ooooh, it's sooooo secretive that I can't talk about it." I know the reasons why companies have NDAs and I don't necessarily fault them for it, but it feels a little restrictive. In any event, the tech specs are that's it a .NET project in VB .NET on the TabletPC. Yeah, I'm having fun.

One minor issue that I had is that I need to display data in a grid-like format. What I really need to do is have the control act just like a HTML table. But, much as I tried to get 3rd-party grid controls to display my images and data the way I wanted to, they never quite cut it. Plus, it seemed like I had to write a fair amount of utility code when I had image data in your code for every grid control I tried (and please don't send me e-mails saying, "Did you try GridXYZ?" Believe me, I tried them all, and while some got close, I never got the look quite right, and I don't have the time to figure it out either.). Anyway, it finally dawned on me this morning that the `Panel` control has an `AutoScroll` property, and then I saw the light - .NET already has most of the functionality I need to make a Repeater control (of sorts) that can host controls that contain my data in the format I want. Whew! That's very cool.

I watched "The Believer" last night. It's a movie about a Jew who is also a Nazi. The guy who played the main character was pretty good - he made me believe that he was really, **really** conflicted. And, how could he not? I mean, in one scene he's giving the "Heil Hitler" arm-raising salute while he's saying something in Hebrew and pointing his pinky finger out (which I guess is done sometimes during a Jewish ceremony. At least that's what the director said in one of the extra features on the DVD.) As a person who is neither Jewish nor a Nazi, there were some points that were raised during scenes when arguments were being carried out that I didn't quite get, or at least I didn't understand the history behind them either. While I was a little confused during a couple scenes, I still liked the movie and would recommend it to anyone who wants to see a solid performance from the main actor.

> Published: 05.10.2003 11:24:00 AM CST