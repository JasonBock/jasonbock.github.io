---
title: Testing Windows Forms
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Testing Windows Forms

One thing that's always bugged me about my add-in for Reflector is that I have no tests. The issue is that the add-in is tied into Reflector's add-in architecture - it pretty much requires being run under Reflector to function correctly. I know there's NUnitForms, but the problem that I see (at least from its' documentation) is that their Recorder only works with controls in DLLs. I need to script it out when Reflector is running.

I know there are tools like WinRunner, but it's not free. I'm wondering if there's a tool that I can use to script Reflector so I can ensure that my add-in works before I publish an update and is free (like NUnit). Any suggestions?

> Published: 09.09.2005 08:04:14 AM CST