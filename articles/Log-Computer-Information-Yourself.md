---
title: Log Computer Information Yourself
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Log Computer Information Yourself

I'm currently talking to the RSS Bandit folks - looks like I'm going to have to run the code myself in the debugger to see what the problem is. As I was adding the ZIP file to the e-mail RSS Bandit creates for you to send to them, I noticed that their e-mail text says this:

> ...replace this text with some more useful informations about:
> * Your system environment and OS version
> * Description of the issue to report
> * Any hints/links that may help,
> please!

I understand their point. It's helpful to know what OS the user is running, do they have a multi-core system, how much memory do they have, etc. But then I realized ... couldn't they add that to the log file themselves?

The `Environment` class has `ProcessorCount`, `OSVersion`, `WorkingSet`, etc. The `ComputerInfo` class (which is in microsoft.visualbasic.dll but C# can just ref that assembly) has a bunch of properties (like `OSFullName`, `AvailablePhysicalMemory`, etc.). So there's no reason a developer can't run code like this in their Main method:

```c#
ILog log = // ...create the logger;
log.Debug("Operation System: " + Environment.OSVersion.ToString());
log.Debug("Processor Count: " + Environment.ProcessorCount.ToString());
```

... and so on. I'm sure there are Win32 calls you can make to get other information about the user's system as well. My point is, if you **really** need system information in the log file that the user is going to send you anyway to resolve a bug, you might as well include it in the log file so you don't have to rely on the user to give you that information in a format you want.

> Published: 03.09.2007 09:51:34 AM CST