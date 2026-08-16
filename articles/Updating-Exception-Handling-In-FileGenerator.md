---
title: Updating Exception Handling in FileGenerator
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Updating Exception Handling in FileGenerator

I finished a small but important change in [FileGenerator](https://github.com/JasonBock/FileGenerator) last night. I got rid of all the crappy exception handling that existed in it. I also removed the ExceptionViews component in the UI. My philosophy on exception handling has crystalized over the last year or so. There are only a very few cases where you have no choice but to handle the general `Exception` type. The right approach is to handle the exceptions you know you can handle, and let the other ones go unhandled. My code was trying to catch unexpected situations when it would generate a file for a type or a resource, and that's just wrong.

I also found a bug with accessing UI component information on another thread. Yikes! I'm amazed that the add-in has worked at all up to this point. I need to clean up how the add-in does all of its work in an asynchronous fashion. It's pretty messy at this point, but it's based on code I wrote years ago, and I've learned a bit since then.

I'll push a 4.2.0.0 release very soon.

> Published: 12.18.2008 08:41:56 AM CST