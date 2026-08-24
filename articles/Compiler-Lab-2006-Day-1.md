---
title: Compiler Lab 2006 - Day 1
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Compiler Lab 2006 - Day 1

![Compiler Lab](https://jasonbock.net/images/Compiler-Lab-Day-1.png "Compiler Lab")

See? I made it. I ran into Sam and Brad - I also saw Peter but I didn't talk to him. I also ran in a fellow Magenicon, Keenan - I had no idea he was going to be in Redmond this week (in the same building, no less).

Anyway, today was pretty much Linq-based. It was cool to see Anders and Erik talk about Linq and related technologies. It's amazing to see how much delegates (a 1.0 feature) permeate this space. During the lab time, I wrote a little program that loads all of the assemblies in a given directory, finds all the types that implement a worker interface, and then runs the worker. Here's what the code looks like without using Linq:

```c#
foreach (Assembly workerAssembly in 
  this.GetAssemblies(Path.GetFullPath("Workers")))
{
  foreach (Type workerCandidate in workerAssembly.GetTypes())
  {
    if (typeof(IWorker).IsAssignableFrom(workerCandidate))
    {
      IWorker worker = (IWorker)Activator.CreateInstance(workerCandidate);
      this.RunWorker(worker);
    }
  }
}
```

Ignore the helper functions `GetAssemblies()` and `RunWorker()` - just look at the Reflection-based code. Now, here's the Linq version:

```c#
var workers =
  from workerAssembly in this.GetAssemblies(Path.GetFullPath("Workers"))
  from workerCandidate in workerAssembly.GetTypes()
  where typeof(IWorker).IsAssignableFrom(workerCandidate)
  select Activator.CreateInstance(workerCandidate) as IWorker;

foreach (var worker in workers)
{
  this.RunWorker(worker);
}
```

Personally, I don't see a lot of code reduction, but it feels cleaner to me. I have a list of projects that I want to (eventually) work on, one of which was an ADO provider for Reflection. You know, write a query to find all the methods that have 5 arguments, the 3rd of which is a Guid...or something like that. Well, Linq pretty much makes that project null and void, and I'm very happy for that. I've seen Linq a while ago, but to finally sit down and try it is challenging (since there's no Intellisense support for it yet in C#) but it's really powerful.

What's funny is I copied the code + binaries to a USB drive and then ran the EXE on my laptop (which doesn't have the Linq preview). It worked perfectly. It's not surpising as all the 3.0 stuff does not require changes to the 2.0 CLR, but it was still cool.

Kathy was asking for presenters, so tomorrow I'll do a quick demo of my extensible compiler. There was a lot of offline talk about CCI, Reflection, code generation, attributes, etc., so hopefully this little presentation should generate more conversations (and I'll make sure to ask, "does Phoenix address this in any way?").

> Published: 03.13.2006 11:28:30 PM CST