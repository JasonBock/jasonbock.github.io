---
title: Services Are The Exception
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Services Are The Exception

*Note: this is a somewhat rambling post. You have been warned ...*

I don't claim to be a SOA expert (in fact, I'll never claim to be an expert on anything. Maybe a guru ... but I need more grey hairs ;) ). But I'm coming to a conclusion that services should be fairly ... I don't want to say "rare", but "less common" than the entire system. As Rocky says, services cross a trust boundary. I'm beginning to agree with this. Since neither side "trusts" each other, the messaging between them needs to be clear and consise, and therefore it should not change as much. As anyone who has tried to write systems that are interface-based, it's not easy, because systems change frequently. But if I'm calling a service (like Amazon.com's stuff), I don't want it to change every day - it would drive me nuts. So a service should be very focused on one thing. Again, that's not easy.

What I'm getting at is that a service shouldn't have a rich, OO-like "look". In other words, a service layer shouldn't be the business object layer to your system (and yes, I've heard of such a thing being done - god help them when their system falls on its knees due to performance issues). Rather, they should (and this is where the messaging/service relationship is starting to become a blinding white light to me [1]) do a lot of things with one message. Like, an AddNewCustomer() service, that passes over a message that contains all sorts of information about a customer for a insurance system. The service will do all sorts of cool internal messages in the organization, but to the client, it chunks up a message, passes it over, and...done. Furthermore, the message can be crufted up through a smart application that can defer the full message creation until the employee gets on the network and can send all of the new customer stuff in one shot. Or a claim lookup service, that masks 3 different mainframe systems so it makes it easy for applications in the corporation (where there's Java and Delphi and VB6 apps floating all over the place) to find claim information without having to deal with all these internal systems.

At the end of the day, services to me should be "rare". They're well-focused, they hide lots of functionality behind one "call", and they don't evolve as much as internal corporate systems do.

Again, this is rambling. Feel free to rip it - I'd rather learn through criticism :).

[1] Sorry - sometimes I'm slow to catch on.

> Published: 10.21.2004 10:35:20 AM CST