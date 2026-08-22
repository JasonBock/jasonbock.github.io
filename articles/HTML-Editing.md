---
title: HTML Editing
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# HTML Editing

One of the things I added to my blog editing tool was an HTML editor. Previously, I was using a simple text box control, but that just wasn't cutting it. I did some searching on the Internet and I found a couple controls. I decided to using this one. It does the job, and it's better than writing HTML straight-up, but frankly I'm still not happy. Tim did a great job with it, but he's using the WebBrowser control underneath, and he has to do a lot of hoop-jumping to get some things to work. Inserting links is also not so much fun; I found out this weekend that it likes to put an "about:" string in front of the link if it's not absolute (which is just bizarre). In fact, this is the biggest missing feature (or at least I can't find is) with this control: there's no easy way to insert arbitray HTML where the cursor is, or to wrap text with my own tags, rather than just bolding text. I've also noticed that menuing has become quirky when this control is in view - i.e. menus don't want to go away once I'm done using them. I have to click somewhere else in the application to get rid of the menu.

It's still amazing to me that it's 2007 and we still don't have a decent HTML editor in .NET OOTB. Maybe I'm missing it, or maybe there's a really decent (and free!) solution out there that I haven't found. All I want is something that does basic HTML editing, and allows me to insert HTML where and when I want it. I may look at RichTextEditor or this editor or do something crazy like hosting this control (which I get free use of via OrcsWeb) in the WebBrowser control. Basically, I'm open to suggestions. If you know of a good HTML authoring component that's free and can be used in a WinForms app (in some way, shape or form), I'm all ears - leave a comment!

Update: I found Html Editor, which seems pretty decent. I'll dive into the guts later and I'll use it if it's more extensible.

> Published: 04.23.2007 07:56:25 AM CST