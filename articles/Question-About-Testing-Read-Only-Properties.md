---
title: Question About Testing Read-Only Properties
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Question About Testing Read-Only Properties

Peter asked me the following question via e-mail about this post:

> Why test the setter of a read only property? I'm sure in the real code it doesn't sound as silly as your posted sample suggests.

You're right, Peter, it's not as silly as my trivial code sample. Here's the deal. As my recent blog postings indicate, I've been doing a lot with right-to-left WinForms scenarios. I wanted my tests to check that I was setting the `ExStyle` property on a `CreateParams` object correctly when I mirrored a control, but to do that I had to be able to check the value of that property when mirroring was on or off. As I mentioned, though, mirroring should be driven at the application level; you wouldn't want a Panel not to be mirrored when the main form is. Therefore, at runtime, the property is read-only and cannot be changed. However, at design-time, the developers want to see what their custom controls will look like mirrored, so the property can be changed in the Designer.

There's where the problem is. When I'm testing, my controls are not hosted in a designer, so `DesignMode` is `false` - I'm in runtime mode. Therefore, the property's value cannot be changed, and I can't test certain conditions in NUnit. **That's** why I added the extra configuration switch. Maybe there's a better way and maybe it still sounds "silly", but ... well, that's the best I could come up with. I don't like it, but it allows me to test my classes.

As a side note, I thought, "why don't I try to fool the control by setting `DesignMode` to `true`?" No can do - the property is read-only (and I can see why - you wouldn't want anyone to be able to change that value). But, what I can do is create a class that implements `ISite` that always returns `true` for the `DesignMode` property, and set the control's `Site` property to an instance of that `ISite`-implementing class. Therefore, it would "think" that it's in design mode, and then I don't need the extra configuration parameter. Again, not a great solution either, but it works and it "feels" better.

> Published: 12.29.2004 03:55:14 PM CST