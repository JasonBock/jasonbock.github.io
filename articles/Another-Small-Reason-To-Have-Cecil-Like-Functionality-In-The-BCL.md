---
title: Another (Small) Reason to Have Cecil-Like Functionality in the BCL
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Another (Small) Reason to Have Cecil-Like Functionality in the BCL

Today I'm trying to add some more tests to [ExceptionFinder](https://github.com/JasonBock/ExceptionFinder). One place I'm missing code is in the `Load()` method in `IAssemblyLocationExtensions`. That method is needed to load assemblies not currently in the `AppDomain`, but I don't run into it during my tests [1]. So...I tried to add some code to generate dynamic assemblies in my tests, and check to see if they load correctly via the extension method.

The problem is that saving a dynamic assembly causes it to be loaded into the `AppDomain`!

Now, there are a couple of ways I could solve this. One is to generate a couple of assemblies, add them as solution items, copy them to the test directory, and load them from disk. I don't like this because I'd like to have the flexibility to add code (it ends up being a one-liner) to create a new assembly whenever I want - this is much cleaner and easier to maintain. Another is to try and use `System.AddIn` to create the assembly in a different `AppDomain`, unload that `AppDomain`, and then try loading it into the current domain. That seems like too much friction for a unit test, and I'm not even sure if that would work anyway. If the BCL had an API like [Cecil](https://github.com/jbevain/cecil), then this problem would go away, because Cecil doesn't load your new assembly into the `AppDomain`. The issue is that I'm not sure what adding Cecil will do to my code licensing (probably nothing but I'm not a lawyer :) ).

Someday I'm hoping .NET will have a Reflection API that goes far beyond what it currently has. What is there is nice, but there's so much more that would make it that much better.

[1] The code needs to be there if the add-in runs into assemblies that have dependencies on other assemblies and those assemblies aren't loaded. I've seen this happen during some initial testing of my add-in and this code fixed the issue ... but I never got around to adding tests to it until today.

> Published: 10.20.2008 10:37:07 AM CST