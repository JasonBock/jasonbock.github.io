---
title: Why "But it Builds on My Machine!" Is a Complete Load of Crap
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Why "But it Builds on My Machine!" Is a Complete Load of Crap

For the last two days I've been in Nant\VSS\CruiseControl hell. It's not been that toasty, but it's been frustrating. Essentially, what I needed to do was to update the versions of all of our assemblies to match the current build number. Now, this isn't that hard from a logical standpoint: Find all of the AssemblyInfo.cs files, change the `AssemblyFileVersion` attribute value, and save the file. Of course, I also need to do the checkout/checkin dance with these files as well, and that caused me some pain to figure out how to get ss.exe [1] to understand which AssemblyInfo.cs file I wanted to dance with. But last night I got everything working as expected, and we had successful builds all last night and this morning.

Of course, when we made a turnover build at 12:00 today, the build would no longer work. (Funny how the gremlins always show up when you don't want them to!).

The issue was that our build process would make a directory with the build version (i.e. "1.0.2.2"). However, this was only being done on the build server. When the developers test their changes on their machines with the Nant script, it doesn't do any version changing as that's done on the server. So none of the development boxes have all of these build directories. But ... the build server does. And the recursive search to find the AssemblyInfo.cs files in the code directories was getting slower ... and slower ... and slower on the build server. Needless to say, this did work for approximately 14 hours until we needed it to work around lunch time! Once we realized this, we simply added Nant code to put the build version directory in a parent directory outside of the code directories, and now all is well.

The moral of the story is an old one: Just because it works on your machine means nothing because we're not **shipping** your machine! But, as this case shows, knowing that it works on one machine but not another is a key piece of information to keep around because you know there has to be something different between the two. By figuring out what the deltas are, you can make your product/project that much better by adapting to those changes.

[1] Yes, I know about NantContrib ... now :). I'll look at replacing our exec tasks that use ss.exe to these custom tasks.

> Published: 08.20.2004 02:50:10 PM CST