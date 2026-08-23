---
title: Sometimes, Log Files Are Helpful
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Sometimes, Log Files Are Helpful

This morning I installed the .NET 3.0 SDK. It was much more painful than I anticipated - I kept getting these weird errors (which were in the log file):

> Failed to download the file http://download.microsoft.com/download/a/7/7/a7767f09-0136-4a96-a1f8-276bf0ee31fa\dexplore_layout.cab, because the user cancelled the operation.

Uh ... I didn't cancel it! After rerunning Setup.exe a couple of times (because if at first you don't succeed, keep running the damn EXE over and over ... ), I decided to just manually download the CAB files. If you're seeing a message like this in the log file:

> 10:21:42 AM Wednesday, January 17, 2007: [SDKSetup:Info] Failed to download the file http://download.microsoft.com/download/a/7/7/a7767f09-0136-4a96-a1f8-276bf0ee31fa\dbg_x86.cab to C:\Documents and Settings\jasonb\blahblahblah\dbg_x86.cab. Warning - Failed to download the file http://download.microsoft.com/download/a/7/7/a7767f09-0136-4a96-a1f8-276bf0ee31fa\dbg_x86.cab, because the user cancelled the operation.

Copy the URL for the CAB file, paste it into your favorite browser, and save it into the directory given in the log file. Restart setup.exe, and all should be well.

I'm not sure if this completely fixes the problem, but it appears like setup.exe assumes that if the CAB file is in the local directory, it'll move on. Why it chokes on downloading the CAB file(s) itself is a mystery to me, but hey, I figured out a way to (sort-of) hack a solution.

> Published: 01.17.2007 11:29:42 AM CST