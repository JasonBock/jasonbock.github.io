---
title: Gnashing Teeth With Existing Code
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Gnashing Teeth With Existing Code

I'm currently installing Eiffel Envision. It's supposed to integrate well with VS .NET - I'll see what it does. I'm really interested to dive into the guts of Eiffel, particularly with how it does multiple inheritance, DBC, catcalls and other wonderful Eiffel goodies at the CIL level.

The day gig has been frustrating and funny the last two days. My last project got put on hold because it was part of a bigger project that got frozen (really, it wasn't me!). So now I'm on an ASP.NET project. I've always avoided web programming because I've hated the model. ASP really turned me off to any kind of web programming - I figured if it was this much of a PITA, why bother? Oh, I know, corporations wanted it, and so I did a smattering of ASP and JSP, but I love rich UIs. I have to admit, ASP.NET is much more appealing to me - so much so that I might get motivated enough to finally implement the redesign of my site.

So I'm learning a lot about ASP.NET these days. Unfortunately, there's also been some personal gnashing of teeth. When someone tells you that your new project is the ASP.NET project that an intern converted over from ASP but didn't finish and didn't come back to the company, that sends up a lot of warning signs in my mind. You cannot imaging how bad this was butchered. The intern basically took all of the .asp files and stuck an "x" at the end, moving the VBScript code into script blocks and didn't put `Option Strict On`. So I have late-binding everywhere, which sucks - that was one of biggest beefs I had with ASP! Of course, CDONTS-type mail and old-fashioned ADO was all over the place as well. I spent the last two days frantically re-organizing the site so at least it had type-checking, all code-behinds, ADO.NET-only, and menial exception handling (among other things).

In a way, I'm not really happy because I don't have enough time to really give this a thorough refactoring like it needs. In reality, there's five mini-sites in the application - using inheritance to abstract out each page would make things much easier to maintain in the future. The intern basically used the copy-n-paste technique of code reuse and I doubt I'll have time to change what's there. What angers me most is that the intern supposedly said that he was moving things over to ASP.NET; in reality he made things much worse. He should've left it in ASP. Thankfully, the business understands that there needs to be more changes than what I'll be able to do in the time alloted (more functionality needs to be added, which takes precedence). I just don't like leaving a project with an architecture that isn't fully "there".

But overall I've kept my spirits up. The funny thing is, there's enough code bloopers in the site to write about 5 angryCoder article. I know there are very sharp coders out there. I've worked with developers that taught me a lot. But I've also been on enough projects where the code base absolutely stinks. True, bad code comes with age at times - if I would look at the first projects I was on I'd probably run away in horror. I'm just getting tired of getting on projects where the code is a mess.

By the way, the Eiffel install is done. I just opened up the calculator project - dear god, does it take forever to compile! But there's so much more to look at ...

> Published: 10.21.2002 10:50:00 PM CST