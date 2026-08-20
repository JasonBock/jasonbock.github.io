---
title: Quixo3D - Part 1: Code Analysis
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Quixo3D - Part 1: Code Analysis

A couple of weeks ago Mike and I gave our talk on Quixo3D - a 3D variation of the game Quixo. I've written a 2D version of the game a while ago, and I thought that a 3D version would be an interesting problem to tackle. Mike knows WPF 3D very well, so he handled the UI while I worked on the framework. Sometime next week we'll publish our efforts as a project on CodePlex (probably), but until that time I want to talk about a couple of things I wanted to address and try as I was desiging the framework and engine assemblies. The first one is Code Analysis.

I realize that Code Analysis has some mixed feeling with .NET developers. Some love the fact that it helps them make their APIs cleaner, more manageable, etc., while others can't stand all the warnings that it generates (there's a reason why there's "anal" in "analysis" ;) ). Overall, I think it's worth integrating it into the build process and treat the warning as errors rather than not to use it. There's a slew of design issues that it finds that you may not be aware of, like:

* If you have an instance method that never uses the this parameter, it'll suggest making that method static.
* It does a (fairly) good job at figuring out if a private method is never used.
* It'll help enforce what it considers "good" naming conventions (camel- and Pascal-casing, etc.)

Of course, I also find some of the rules to be over-bearing. Usually I turn off the globalization rules, simply because I feel like they generate too much noise than they're worth (but if you're on a project with explicit i18n or l10n support then you probably want these on). I'll also turn off the "consider strong-naming your assembly" rule, as I don't run into many situations where I find strong-naming to be a good thing. But that still leaves a lot of things for Code Analysis to hunt down.

That last point really needs emphasis. The first time you turn on Code Analysis, you may be wondering what you got yourself into! You'll probably get tens, if not hundreds, of issues the first time you enable it. That's why I'm convinced that you must enable it right away with any new development you do. If you do, you'll be coming out of the gate with less issues to address and it'll be easier to keep that issue count low with every new build. Also, don't just suppress the issue! Take the time to look at what it's suggesting to you - I've found that more often than not, its recommendation is worth changing the code. This is especially true if you write a `catch` block that catches the general `Exception` type (which is usually not a good thing to do). But if you're deep into development and you have a lot of code in the code base, you should still consider turning it on. You may have to spend a day or two (or three or more) to get the code "up to snuff" but I still think it's worth it in the long run. If you want a code base that's clean, getting another set of eyes on it (even if they're electronic eyes) can't hurt.

Also, don't take Code Analysis issues as compiler errors. I still recommend having them as errors in your project because otherwise they'll be ignored if they're just warnings. But a Code Analysis error doesn't mean your code won't run. I've seen teams spend way too much time trying to fix Code Analysis issues that just weren't worth the effort put into fixing the problem (the strong-naming one comes to mind). Take the errors seriously, but if you think it's OK to suppress them, then either suppress it or turn off that rule.

Granted, with the two assemblies I created for Quixo3D, it's not a lot of code. But at the current client we have Code Analysis on for most of our assemblies (we'll be getting the rest of the code base in step very soon), and that's not a small code base. It's helped us scrub the code for bad implementations and questionable designs. It's not a silver bullet - that doesn't exist in software development - but if you want a solid code base, I think Code Analysis aids in achieving that goal.

Next up ... code coverage ...

> Published: 11.06.2007 08:22:51 PM CST