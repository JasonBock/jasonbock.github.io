---
title: Release Hell
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Release Hell

For the last two weeks, I've been in the about-to-release-the-system hell. I've received a couple of techie-based e-mail from readers, but I've been swamped. My apologies - I'll eventually get around to answering them. I really, really tried to get the system out to the testers in a very pristine state, but they still found bugs. A fair amount of them were the "oh-I-didn't-think-of-doing-that" variety, which is one of the reasons you have testers anyway. However, there were some bugs that just drove me batty. For example, I'm trying to embed the MediaPlayer control in the web page and drive it with some custom images (i.e. I hide WMP's control bar and hook some image `onClick()` events to javascript functions). This works just fine in IE/Windows, but Netscape and IE/Mac just would not listen to the javascript! I tried everything I could think of. I researched via Google and tried some other ideas. But when I called `Play()`, WMP wouldn't play. As "Monster Garage" would say, "Zip! Zero! Nada!!".

Damnit!

I was reading my aggregator on the plane this morning (which is where I'm making this entry) and somebody was ranting about how web programming still sucks in 2003. This project has had the most web programming I've ever done, and I have to agree. Standard CSS will not work the same in all browsers. I think there will eventually be a time where we have to say, "XHTML from now on. CSS version X.X from now on. Etc., etc." It is truly a pain in the ass to make sure your one little HTML change works on 50 different browser/OS configurations. It really should not be this hard. And I'm sure some will disagree with my statements. But, please, can't they (being web browsers) all just get along?

All in all, this project has been very educational. It's been a while since I worked on a system that going to be used by a lot of people, and that's a little bit more intimidating. I've also learned more about ASP.NET than I did before, and now I'd love to go back and change a lot of it because of this new-found knowledge. Unforunately, we weren't given time to do a couple of iterations, and while I still would change stuff when I know it would be very beneficial to do so, I can't make architecturally-sweeping changes at this stage of the game.

> Published: 03.28.2003 10:19:52 AM CST