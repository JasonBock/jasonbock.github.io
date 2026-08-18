---
title: Dealing With the Monster API
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Dealing With the Monster API

A while back I mentioned my current project uses an p/invoke API that has literally hundreds of parameters. I'm trying to put a friendly .NET wrapper around it to make it easier (for lack of a better term) but it's been slow-going. My original time estimate is still doable, but it's such grunt work. This puts the Office APIs to shame. The design I'm putting together puts data together in classes (novel concept :) ) and makes the call cleaner, but it's still going to have 50-60 parameters to make the call from the .NET API.

It's work ... but I feel kind of dirty coding against this. Seems like the p/invoke was written years and years ago, people just slogged new arguments into it, and no one took the time to really clean it up.

> Published: 05.08.2008 08:29:00 AM CST