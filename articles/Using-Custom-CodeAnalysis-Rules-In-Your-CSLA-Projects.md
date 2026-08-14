---
title: Using Custom CodeAnalysis Rules in Your CSLA Projects
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Using Custom CodeAnalysis Rules in Your CSLA Projects

Over the last 7 months I've been on a project that used [CSLA](https://cslanet.com/). My opinion of CSLA has always been ... well, "meh". There seemed to be some interesting stuff in there, but it also appeared that you had to handle a lot on your own as well. At this point (with version 3.8), things are **much** better and you can basically focus on using CSLA to support what you want to do in your business objects, rather than thinking about all the details.

That said, there's still some conventions that you have to keep in mind, like marking your classes with `[Serializable]` or having a private no-argument constructor. Your code still compiles, but when you try to run it ... bad things can happen without those conventions in place. I wanted to try and write some custom CodeAnalysis rules to prevent running code without these conventions, but I had no time during the project. Well ... I now have some time, so I starting taking a look at it, with a couple of twists:

* I'm targeting CSLA 4.0
* This is all in .NET 4.0/VS 2010

The are a couple of reasons for this "latest-and-greatest" focus. One, I wanted to spent more time in 4.0/2010, something I really haven't had a lot of time to do lately. I also wanted to be able to look at what's new in CSLA 4.0 and write rules for those new features (if any are there!). Finally, I wanted to see what the unit testing story was like for custom CodeAnalysis rules in 2010. For whatever reason, I found testing custom rules in 2008 and before a real pain. It just ... never worked, so I had to resort to manually loading a project, testing the rules, etc. However, in 2010 ... I guess I got lucky, because unit testing rules is a breeze.

So, without further ado ... here's where you can get the 0.1 code drop for Csla.Rules. There's 2 projects: Csla.Rules, which contains the rules, and Csla.Rules.Tests, which contains the unit tests. Again, keep in mind the version targets I listed above; if you're using a previous version of CSLA or .NET my code may or may not work. Running the rules in 2010 is pretty simple - I'll refer you to [this web site](https://tatham.blog/2010/01/06/custom-code-analysis-rules-in-vs2010-and-how-to-make-them-run-in-fxcop-and-vs2008-too/) as it shows just how easy it is to create a custom .ruleset file that contains all the rules you want to run. Right now I have three rules in place:

* It must be marked with the `SerializableAttribute`.
* It must have a no-argument constructor.
* It must not expose the no-argument constructor.

Note that if you're running CSLA in Silverlight, some of these rules are not applicable (i.e. the no-argument constructor must be public) so you can just turn off that rule in your .ruleset file. I have one other rule in mind, but it's going to take some time - basically I want to check that you have the correct "DataPortal_XYZ" method if you're calling any of the "creator" methods on the DataPortal. So you'll see that 4th rule show up in your list, but it doesn't do anything ... yet!

As I stated above, this is definitely a 0.1 release. If you find any issues, cases that my rules don't pick up, or suggestions for other CSLA-based CodeAnalysis rules, please let me know. Enjoy!

> Published: 04.19.2010 10:26:19 AM CST