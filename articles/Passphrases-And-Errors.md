---
title: I Know I'm Getting Old
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Passphrases and Errors

[This article](https://blog.codinghorror.com/passwords-vs-pass-phrases/) does a really good job in explaining the benefits of using a passphrase over a password. This morning, however, as a co-worker and I were rambling on about biometric identification, security, smart cards, etc., I mentioned to him the idea of passphrases. We both thought it was a good idea, but the problem is, how do you know you're typing in the correct passphrase?

I'm sure you're all familiar with the following login window:

![Passphrases and Errors](https://jasonbock.net/images/Passphrase-1.png "Passphrases and Errors")

However, what happens when your passphrase is something like, "Begin the day with a friendly voice"? Now your passphrase ends up taking over the entire password region, and, furthermore, you have no idea if you forgot the space between the words "friendly" and "voice", which would make the passphrase invalid. Of course, one option is to allow the user to toggle the passphrase mask so the can actually see the text, but there are security risks in allowing a user to do this.

Another issue is correctness. Making sure I type 35 characters correctly without being able to see them on a screen is not an easy task. I'm a guy that always locks my machine when I walk away from it, so coming back to the box and trying to get in with an obscure password of about 10 characters can be an adventure at times. 35 characters may make me devolve to the chicken-pecking method of typing to ensure I'm getting each character correctly, and that's almost as bad as letting someone see the clear-text on the screen. What I mean is that I've sometimes watched people log into a machine and sometimes I can catch a couple of keystrokes by watching their hands (never for malicious purposes, of course, just curiosity ;) ). The faster they type, the harder it is to see the keys they're pressing, but passwords usually make people type slower because they don't want to screw it up. Passphrases may make it even harder.

Again, just to stress the point, I like passphrases. But from a UI standpoint, some changes may be needed to make it easier for a user to get the passpharse right. Here's an idea. Your password is masked except for n number of characters to the left and right of where the cursor is, something like this:

![Passphrases and Errors](https://jasonbock.net/images/Passphrase-2.png "Passphrases and Errors")

Again, if someone is watching you type the phrase in, they'd be able to pick up the passphrase. But maybe you could have a checkbox so that they could see a few characters to the left and right of the cursor:

![Passphrases and Errors](https://jasonbock.net/images/Passphrase-2.png "Passphrases and Errors")

Just some ideas on a Monday morning fraught with icy, **icy** roads ...

> Published: 12.20.2004 11:51:52 AM CST