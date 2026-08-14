---
title: AutoPex
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# AutoPex

One of the talks I've been doing in recent months is called, "What Will Pex Do?" It's basically an overview of the Pex toolset - I demonstrate what it does and how you can use other frameworks that come with Pex to do interesting things (i.e. Moles). In the presentation I spend some time on a tool that I've been working on, and I've decided to put the project on CodePlex. It's called AutoPex.

So what is AutoPex? It's a way to automate Pex analysis. While it's interesting to right-click on a method in Visual Studio to see Pex go at it, Pex takes time to do the work it needs to do. So I thought, how could I make Pex do its magic in an offline way that provided meaning to a developer without having to wait for extended periods of time? That's what AutoPex does. It takes two versions of an assembly, and it looks for changes in public methods from the earlier version to the newer one. It does this by using the CCI framework to inspect the assemblies and parse method bodies (if needed). Once that list is obtained, AutoPex just shells out to pex.exe and it lets it take over from there. You can imagine this running on your build server as a special nightly build process. Since the generated method list only contains the "deltas", the resulting report every morning should be fairly easy to peruse - at least it would be smaller than having pex.exe process **all** of an assembly's public methods every night!

Right now AutoPex is truly a 0.1 version. I have no tests around it, and I know there needs to be a fair amount of tweaking, changes, and clean-up in the source code. Also, I need to play more with pex.exe's command-line options. Currently, once pex.exe is done, it opens the report in a web browser, so running this on a build server would cause a bunch of windows to pop up. It wouldn't freeze the build, but it would create clutter, so I want to see if there's a way to suppress that. Furthermore, I need to create a MSBuild task for this. All that said, if you're already diving into Pex, give AutoPex a drive and let me know what you think - thanks!

> Published: 02.17.2010 11:19:46 PM CST