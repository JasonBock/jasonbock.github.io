---
title: The Annoying "Invalid postback or callback argument" Exception
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# The Annoying "Invalid postback or callback argument" Exception

Since the last web project I was on at the current client got pushed into production, we've been seeing the following exception occur (although it's a rare event):

> Invalid postback or callback argument. Event validation is enabled using <pages enableEventValidation="true"/> in configuration or <%@ Page EnableEventValidation="true" %> in a page. For security purposes, this feature verifies that arguments to postback or callback events originate from the server control that originally rendered them. If the data is valid and expected, use the ClientScriptManager.RegisterForEventValidation method in order to register the postback or callback data for validation.

I haven't been able to reproduce it on my local machine ... until today. Basically, what I think is happening is that the `__EVENTVALIDATION` form field isn't making it in the postback. It's odd that this field is rendered near the end of the page, and when I've been able to recreate it locally it's usually when I have to do a postback and the page hasn't finished rendering yet. In fact it looks like it's on of the last pieces of information that's added in the output during page processing. I've done a lot of searching for workarounds on this issue, and the only one that I've seen that squashes the exception is to put `<pages enableEventValidation="false" />` in the web.config file. But that doesn't feel right to me either - we want to validate our event data.

The only results I've been able to find from Microsoft are these, and all of them seem to hit a dead end. However, I definitely believe this is a bug or at least this should be handled more gracefully. I hope Microsoft has a fix for this problem in an upcoming service pack.

> Published: 05.12.2006 06:51:59 AM CST