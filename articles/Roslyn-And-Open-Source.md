---
title: Roslyn and Open Source
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Roslyn and Open Source

At //build/ 2014, I was expecting a lot of exciting announcements around mobile and web development as well as cloud infrastructure. Microsoft's direction has been changing significantly in recent history to embrace more than just their creations and ecosystem – for example, Office 365 is now available on Apple devices.  But there was one announcement that I was hoping to hear at the event above all else, and it had to do with Roslyn.

In summary, Roslyn has been a long-running project within Microsoft. Roslyn is a framework to bring compiler features into the managed world. That is, the .NET compilers for C# and VB are now written in C# and VB. Furthermore, this information can be used through APIs for code analysis and refactoring tools. Needless to say, this is powerful stuff! But take note of the phrase, "long-running project". I don't know exactly when it started, but demos started to surface at the PDC around 2008. That's at least 6 years of a project in-flight before it moved into a RTM world. A couple of years ago, MS finally released CTP bits of the framework, but these were few and far between. Some people in the .NET developer community wondered if this was really going to become a released product, or if it was going to die on the vine.

Well, today, the world found out, and the news was quite unexpected. Anders Hejlsberg announced that, not only were new Roslyn bits available to the public, but the entire Roslyn code based was open-sourced. In fact, he pressed the "let everyone see the code" button right on stage! That took a lot of people by surprise, me included. He showed how you could add your own language feature to C# if you want, and have that work in the Xamarin world as well, as Miguel de Icaza demonstrated. While I don't expect a lot of .NET developers to customize the .NET compiler in this way, it was fascinating to watch this take place.

Then they also announced the [.NET Foundation](https://dotnetfoundation.org/), which, according to their site, "foster(s) open development, collaboration and community engagement on the .NET platform." As I perused all of the frameworks that fall under this category, it dawned on me just how different Microsoft is in 2014. They're releasing more and more to the open source community, something that would have never happened 10, or even arguably 5 years ago. One may still find faults with them, and arguably they still have their pain points. But I find it hard see them as the company they were years ago.

I think there are some very interesting times ahead for developers who have been in the Microsoft world. Universal apps, Azure improvements, and now, an open source compiler. I really need to find time to work in that "disposable" keyword into the C# compiler!

> Published: 04.03.2014 01:03:00 PM CST