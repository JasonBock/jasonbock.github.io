---
title: Back To VBA For an Hour
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Back To VBA For an Hour

Currently I have an Outlook rule that moves successful build notifications to a specific folder (Builds) so my Inbox doesn't get overwhelmed. However, failed build messages stay in the Inbox as I want to know when I have to ridicule the developers without mercy. The problem was when they fixed the build, I had to manually move those failed messages into the Builds folder so they weren't cluttering up the Inbox either (I know, life as a mouse mover or keyboard typer is so hard). What I really wanted was a rule to move successful messages to a folder and move the failed messages to the same folder. Then I'd know when to make the developers sweat and I didn't have to do any manual cleanup. Problem is, you can't invoke a rule from another rule, so I was forced to devolve into VBA-land for about a hour.

Dear god - how in the hell did I **ever** get anything done in VB before .NET? I'm so used to the .NET world I've lost all touch with the VB6/VBA/VBScript constructs. You don't realize how much of an improvement VB .NET is over previous versions until you've lived in the .NET land for a while and then you have to go back. Of course, not knowing the Office object model didn't help, but it was made worse by being in an IDE that, well, sucks compared to VS 2003.

After an hour of the "code-this-and-run-it-to-see-if-it-works" routine, I finally had a function that did what I wanted it to do. But ... man, if I could only write macros in a managed environment!

> Published: 08.25.2004 03:33:39 PM CST