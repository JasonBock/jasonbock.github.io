---
title: Code Injection With CCI - Part 6
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Code Injection With CCI - Part 6

Previous Parts

* [Part 1](https://jasonbock/articles/Code-Injection-With-CCI-Part-1.html)
* [Part 2](https://jasonbock/articles/Code-Injection-With-CCI-Part-2.html)
* [Part 2.5](https://jasonbock/articles/Code-Injection-With-CCI-Part-25.html)
* [Part 3](https://jasonbock/articles/Code-Injection-With-CCI-Part-3.html)
* [Part 4](https://jasonbock/articles/Code-Injection-With-CCI-Part-4.html)
* [Part 5](https://jasonbock/articles/Code-Injection-With-CCI-Part-5.html)

Now that I'm done with my initial dive in [CCI](https://github.com/microsoft/cci)-land, I'd like to review what I've discovered and where I'd like to go next.

Changing my code over from [Cecil](https://github.com/jbevain/cecil) to CCI was interesting. It's great to see this framework pushed to CodePlex - I've been wanting something like this from Microsoft for a long, long time. That being said, just be aware of the state of the project, which is currently versioned at 0.2. It's not that easy to peruse the API. There's no documentation to speak of, and the samples that are there take some time to figure out just what they're trying to show you. As I was writing these articles (through the gracious help of Herman), I discovered that I was either not doing things the "right way", or there were other possibilities within the framework, such as the mutable code model. Granted, CCI is a framework that deals with low-level .NET aspects, so if you're going to wade in those waters, just be prepared to do some digging. Hopefully the articles I've created will help.

Now, where do I go from here? I have lots of ideas that I'd like to investigate with CCI:

* *Add PDB modification support.* I think it would be cool to change the PDB with my injections such that, when you debugged the code, it would highlight the `[ToString]` attribute when the injected `ToString()` method was invoked. CCI has a PDB reader-writer, so I'd like to dive into that area.
* *Look at using the code model.* Herman mentioned this in the post I put on the CCI discussion list - I'd like to see just how this all works. It may make assembly modification much easier than what I did in this series.
* *Create a MSBuild CCI task.* I know there's a tool called [PostSharp](https://postsharp.net/), but it would be nice to give developers a CCI-based way to modify the assembly after the compiler did its thing in a MSBuild file.
* *Change [`DynamicProxies`](https://github.com/JasonBock/DynamicProxies) to use CCI.* It would be interesting to make the proxy generation pluggable - in other words you could use CCI, or Reflection.Emit, or Cecil, or whatever you wanted.
* *Change [`EmitDebugger`](https://github.com/JasonBock/EmitDebugging) to use CCI.* Again, same idea as before. For whatever code you're dynamically generating, add the ability to create the .IL file with a .PDB on-the-fly.
* *Create a new PDB2XML.* Updating this tool to use the CCI version on CodePlex would (might?) be very cool.

Of course, the issue is finding the time to pull all of this off!

Anyway, it's been a fun dive into CCI that I hope to continue in other areas in the future.

> Published: 05.22.2009 05:25:06 PM CST