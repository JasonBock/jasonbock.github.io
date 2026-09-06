---
title: Knowing if the `InstalledUICulture` is Right-To-Left
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Knowing if the `InstalledUICulture` is Right-To-Left

Right now, in my application I have a configuration property that tells the code whether or not it should display the UI as left-to-right or right-to-left. What I'd really like to do is look at the installed culture (i.e. the `InstalledUICulture` property on the `CultureInfo` class) and have it tell me if the application should read right-to-left or left-to-right. I'm thinking there's a way to do this, either through a .NET class or a p/invoke, but I can't find it. If you know of something in the Framework or in the Win32 layer, please let me know - thanks in advance.

> Published: 12.22.2004 01:16:24 PM CST