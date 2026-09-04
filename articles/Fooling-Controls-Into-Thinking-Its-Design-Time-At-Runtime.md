---
title: Fooling Controls Into Thinking it's Design-Time at Runtime
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Fooling Controls Into Thinking it's Design-Time at Runtime

Yesterday I hinted at a way to get my controls to work in my unit tests with a `ISite` implementation that would fool the control into thinking it was in design mode. That worked for most of my unit tests, but the `TabControl` barked loudly at me. Basically, I needed more of an implementation of `ISite` than what I was doing, but as this post states writing a designer isn't the easiest thing in the world to do. It's doable, just not as simple as adding two numbers together. It would be nice, though, to have some kind of `ISite` faux implementation for unit tests to make controls behave as they would at design time in a unit test [1].

Along these same lines, I'm finding that writing controls to behave in certain ways when they are in design mode can be tough because I can't find a way to debug the control when it's in the designer. It's easy to add a breakpoint in code, run the application, and stop at that breakpoint to see what's going on. But I can't find a similar approach with controls in the Designer. It's not a big deal, but it is a bit annoying. I know that some would argue that a debugger should be the last step a developer does to fixing a problem, but tests and log files can only go so far - sometimes you need the debugger to figure out what's going on.

[1] Yes, I am familiar with NUnitForms, but that wouldn't help me out in this case as I'm trying to test what a control would do at design-time, not runtime.

> Published: 12.30.2004 10:53:25 AM CST