---
title: It's Finally Released, and Some Thoughts on NewsCrawler
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# It's Finally Released, and Some Thoughts on NewsCrawler

Well, the project I've been working on for four months has finally been released to the public. Yesterday we got our ass kicked by some really weird Passport behavior. Throughout the development and test phase, we never got an error when we'd try to access the user's data through either direct calls to `GetProfileObject()` or indirect calls through the indexer. But when it went to production, we suddenly got a bunch of `COMException`-related errors. Getting the preferred e-mail value was OK, but anything else became verboten. I wish I knew why, but we had to stop all the exceptions from occurring, so we ended up taking out those calls. We also had some server permission issues as well that should've been set up by the admins, but that was resolved fairly quickly. Now I'm able to move onto the other phases of the project, which aren't that bad as the content of the site becomes smaller and smaller as each stage of the contest progresses. Plus, some of the information I've learned through these trial-by-fire issues will make the implementation a little bit easier.

After using NewsCrawler for the past week, I've decided to purchase it. I really like the RSS auto-discovery feature. However, the tool is not perfect. Sometimes, when I hover over items in the program and then move to other programs, the hover windows somehow stays **above** the program I just switched to. That's a little annoying. There are some other minor quirks with the tool, but what it offers in terms of news aggregation is very nice.

> Published: 04.03.2003 10:25:27 AM CST