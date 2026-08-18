---
title: 6 Hours for a Build
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# 6 Hours for a Build

Yes. It took me 6 hours to do our build today.

Yikes.

For the last 5 years, I've had it pretty good with the projects I've been on. I've learn new things, the projects have been (generally) managed well, etc. This one ... well, it seems like an old-school web project from the 90s. Lots of bad coding practices, very little unit tests (there were really none until I started adding some), poor distribution of concerns (i.e. lots of business logic in event handlers, code does not lend itself to being easily testible), and no automated build and deployment. The devs on the team know that it's not in a good state, but one thing I've learn throughout the years that the longer a code base stays the way it is, the harder it becomes to make it better.

Anyway, I haven't had a build like this go on for so long in ... well, I can't remember. Hell, the last major project I was on, pretty much everything was automated, and it was so sweet. Other than a move to production, we could just fire off a build and boom!, the QA box was updated. Again, we know we can do better, but I think it's going to be a challenge to rectify this without resources focused on making things better.

And yes, I know there are people who read this blog who are on this project :). I understand the history behind it ... it's a long story but I get how it got the way it did. But things need to change in the future.

At least I'm not scheduled to do another build for at least a month :)

> Published: 07.30.2008 07:01:45 PM CST