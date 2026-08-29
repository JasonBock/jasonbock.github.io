---
title: Securing Web Pages
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Securing Web Pages

One thing I wanted to do when I created my administration pages was to do whatever I could to prevent people from inadvertendly (or maliciously) adding posts. So, last night I went through the rigamarole of setting up my authentication mode correctly, adding a login page, using hashed passwords in the .config file, etc., etc. So far things seem good, but during my research I ran into two "interesting" things.

The first was trying to create the damn hash value! Great, I can SHA1 my password - is there a tool in .NET to do this? I couldn't find one. Trying to use classes from the `Cryptography` namespace didn't help either. Finally I ran into `FormsAuthentication.HashPasswordForStoringInConfigFile()`, which did the trick, but it sucks that I have to write a small console application just to create the password. Oh, well, maybe there's something in 2.0 that makes this a bit easier.

The second thing was something I didn't anticipate, but it was a post from someone that mentioned the concept of "defense in depth." Basically he says that you shouldn't rely upon the configuration settings - if a page needs to be secure check for authentication in the `OnInit()` and `OnLoad()` methods. I use a base class for all my pages so I creates a subclass of that class that does the checks automatically. This will make things easy in the future if I need to add new secure pages - all I have to do is use the secure class as the base class of my page. The funny thing is that during my testing last night I commented out my security settings in the configuration file, but then I noticed I still got prompted to log in to access my secure page, because ... the page itself didn't rely upon configuration settings.

Awesome.

> Published: 07.06.2005 01:27:00 PM CST