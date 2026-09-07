---
title: Themes and WinForms
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Themes and WinForms

One of the more recent requirements of the application I'm working on is to change its' look dynamically. The primary reason for this requirement is that we're working with different form factors (e.g. desktop, TabletPC, etc.) and we may need to tweak the color schemes depending on the output's brightness, contrast, etc. Another reason is that we may want to simply change the look-n-feel of the application as the tool evolves without spending a lot of time changing control properties in VS .NET.

I ended up creating an architecture that feels a lot liks CSS for WinForms. Basically, you define an XML file that contains theme names that different controls can use. These controls can be changed at design or run time so you can see what the theme looks like. A tool was also created that made the generation of themes relatively straightforward. You can preview themes before you save them, you can pick the property you want to change for your theme, and so on. The only thing I didn't put it is theme inheritance, but that won't be too hard to include for a future revision.

The real cool thing about all of this is that it only took 2 or 3 days total to get all of this done. Things like .NET serialization (specifically the SOAP serializer), the `PropertyGrid` control, and attributes/metadata (among other cool things) made this pretty much painless. I wish I could show the results, but it's an in-house tool - sorry!. It's really not that hard to reproduce, thought. Basically you can run the application with the standard themes, put in the new theme file, and the next time you start the application, you'll get a completely different look.

My joy has been offset a little bit today by having to use Crystal Reports today. I have to admit that my frustrations with that tool have to do with my lack of knowledge about CR, but I've never had a wonderful experience with it. I don't know why - it's always driven me nuts. Granted, in .NET, things are better, but the pain factor is still a little high.

> Published: 07.21.2003 02:14:52 PM CST