---
title: Multiple Domains and log4net - No Resolution in Sight
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Multiple Domains and log4net - No Resolution in Sight

As I mentioned yesterday, I've run into a problem with log4net and multiple `AppDomain`s. Well, I now know why the problem is occuring, but I can't figure out a way to fix it. See, the main `AppDomain` gets a logging domain (why they reused the word "domain" is beyond me - it confuses the hell out of me: are they talking about an `AppDomain`, or a logging domain? Grrrr ... ), and the new `AppDomain` gets a new configuration, so it tries to open the logging file, which the first `AppDomain` has open, so the logging domain in the new `AppDomain` can't open it during configuration. This basically causes a `QuietTextWriter` field in an appender to stay uninitialized, so nothing is ever logged in the new `AppDomain`.

I'm not one to give up, but I think I've hit a point where I'd either have to change the log4net source code to do what I want (not a fun task), or switch over to something like EntLib or another logging library. Again, since the main logging domain [1] locks the appending file, no other `AppDomain`s can log to that file.

[1] And I've tried using `[assembly: Domain()]` in my server library; it doesn't help out at all. I'd love to be able to share the logging domain from the main `AppDomain`, but I can't figure out a way to do that. That's my last dying hope - if I can do that, then they could share the appenders and the code in the new `AppDomain` wouldn't try to re-open the logging file.

> Published: 02.17.2005 09:41:27 AM CST