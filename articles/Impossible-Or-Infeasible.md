---
title: Impossible, or Infeasible?
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Impossible, or Infeasible?

I had a conversation a week ago with someone at the client's office about their service development. The conversation went well and I enjoyed it. During the conversation we got on the topic of implementation and he said all of their services were Remoting-based as they were internal and interoperability issues were not present. OK, that's fine, but then he said, "So if a Java client wanted to use our services, we'd need to put a Web Service front-end to it as Java talking to .NET is technologically impossible."

For some reason, that phrase didn't sit well with me, so I said, "You mean, technologically infeasible, not impossible, right?"

Granted, I was being a tad bit nitpicky. While there are bridging technologies that allow Java and .NET to talk without the WS-* "stuff" in the middle, it's usually easier just to "WS-ify" your service so other technologies can interoperate easier. I wouldn't want to write Java code to talk to a .NET Remoting service; it seems simpler to add the WS stack on top. But here's my point. There is a difference between "impossible" and "infeasible". As far as we know, it's impossible to travel faster than the speed of light. It's not impossible to get Java and .NET to talk outside of the WS world. But, the cost to do so may be prohibitively high compared to using WS standards. Sometimes, though, we have no choice, and we haul out our roll of duct tape and pliers to do something that's not impossible, just ugly.

> Published: 10.28.2004 10:57:22 AM CST