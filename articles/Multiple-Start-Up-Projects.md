---
title: Multiple Start-up Projects
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Multiple Start-up Projects

Today I'm finalizing some service work I've been banging out for the last two or three days. The last thing I needed to add was a simple WinForms application to test the service from a presentation layer. Sure, I have the necessary NUnit tests to do that, but I wanted something that would test the server-side code in a more "real" way. **Anyway**, after adding the project I ran into a minor annoyance because I had to launch the service host first (which can be either a console or WinService, but on my box I run it as a console application - it's simpler that way), and then I launched the WinForm application. After doing this enough to want to gouge my eyes out with my pencil, I wondered if there was a way to specify mutliple start-up project. Sure enough, there is - I've just never seen it before. When you right-click on a solution in VS .NET, you should see a *Set StartUp Projects...* option. This will bring up a dialog box that will look something like this:

![Multiple Start-up Projects](https://jasonbock.net/images/Start-Up-Projects.png "Multiple Start-up Projects")

Sorry for the deleted project names - just trying to keep the client anonymous. Anyway, all you do is select the *Multiple Startup Projects* radio button, and then set the *Action* of the projects you want to start up as *Start*. You can also put them in the order you want to have them launch in case you have some dependencies between the projects. Just highlight the project and use the *Move Up* or *Move Down* buttons as needed.

> Published: 05.16.2005 01:52:06 PM CST