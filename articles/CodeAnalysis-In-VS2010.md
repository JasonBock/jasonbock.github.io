---
title: CodeAnalysis in VS2010
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# CodeAnalysis in VS2010

One of the tools I love to use on project is CodeAnalysis, but one key feature sorely lacking in its current incantation is custom rules. They're there and you can write them...but nothing's publicly documented and they're pretty much near impossible to test from a unit testing framework perspective. Recently I saw a post about CodeAnalysis in VS2010, and I chimed in with my questions:

> OK, 2 questions :) -
>
>1) Are the rules APIs public? IOW is it now much easier to create custom rules?
>
>2) Are custom rules testable under unit testing frameworks? Right now it's basically "impossible" (from what I've been able to gather) - the best way to test custom rules is to run them under VS or FxCop manually, which is less than ideal. Has this changed?
>
>Thanks,
>
>Jason Bock

Here's the response: 

> The rule APIs are not available as a public SDK in the CTP release. We are still working on these and hoping to get them in for RTM, but it's going to be tight. Having a good unit testing story for rules is also part of our goal with the public SDK.
>
>Tom

I don't want to read too much into this ... but I was hoping for a better answer. No matter what framework/toolset/library/parser you're writing to wow .NET developers, having automated code reviews via tools is important, if not key, and not having a clear, simple way to do this with the tool that come in VS is somewhat disheartening [1]. I truly hope the SDK finally becomes public and unit testing custom rules becomes a nice story.

[1] And yes, I know about [NDepend](https://www.ndepend.com/). Great tool. Not putting it down in the slightest. This post is just about the state of CodeAnalysis.

> Published: 11.03.2008 09:31:30 AM CST