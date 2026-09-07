---
title: Lots of Fun with Office Add-Ins
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Lots of Fun with Office Add-Ins

Today I kept busy working with another developer getting an add-in working correctly in Office 2003. She has most of the code working, but the problem was that VB .NET wasn't playing very nice with the add-in object model. For some reason, at certain times were were getting an `ExecutionEngineException`, which screwed everything up. (Plus, I love the fact that the add-in model decided to name an interface `Exception`, which makes for oh-so-much-fun adding a generic exception handler, but I digress.) On a whim, we thought, hey, let's try this in C#. And, lo and behold, it worked.

Well, that's not telling the whole story. There's some really odd quirkly things you have to do to call properties and methods in the add-in model correctly that don't make any sense to me. I mean, why should I have to call `InvokeMember()` on a .NET object to get a collection back? Oh, well, it works now. It's an add-in that helps users at the client pick conference rooms that aren't being used at a particular time. For some reason, it's a real PITA in that there's no way to get open rooms in Outlook the way they have it set up, so this add-in makes it really easy to find open rooms. We have them grouped into a tree view control that puts the rooms into building and floor nodes. It kept me interested and occupied today, and that was a good thing. Plus, I can now say I wrote an Office add-in! [1] [2]

[1] Actually, I didn't "write it." The other developer had done most of the work, and she just ran into some really weird technical issues. I just came in and added the pair-programming view to it.

[2] In case my manager is reading my blog tonight (which I know he does), **I'M NOT SAYING THE ADD-IN IS DONE**. It still needs some polish and shine!

> Published: 06.01.2004 08:11:43 PM CST