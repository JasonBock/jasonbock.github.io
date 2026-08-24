---
title: JSON Bridges
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# JSON Bridges

John's post really unnerved me:

> ATLAS is the relative newcomer to the AJAX scene. I haven’t spent much time looking at the client side pieces, but I have taken a good look at the server side pieces. The one feature that really stands out is the XML to JSON marshalers that ship in the box. ATLAS introduces a nice (well, as nice as you can get with XML) syntax for declaratively describing the mapping between a Web Service (REST and SOAP are both supported among others) and your JSON schema ... The reason why I love this feature is that it’s just so natural to marshal data to browsers as JSON. It also nicely works around the 'I can only talk to the origin server' security restriction imposed by most browsers on XmlHttpRequest calls, since your origin server now acts as a proxy (or a bridge) to the web service that you want to call.

That's spooky, because that's what I've been doing internally at the client off and on for the last 6 months - writing .aspx bridge pages that take JSON requests and send back JSON responses (we're currently not using Atlas ... but we are using their JSON serialization classes!). Of course, we've also done a helluva lot with asynchronous method coordination on the server-side as well - not an easy problem to address. But ... now, it's a nice world to live in. We've got a lot of powerful tools in our system and it's really cool to see it coming together on the latest application.

John also made a nice comment about JavaScript here:

> JavaScript was really Scheme with semicolons.

How true. Those who still equate developers who use JavaScript as "script kiddies" just don't get it. It's not the best languages I've ever used, but I've grown to respect it because there's some powerful concepts within JavaScript.

> Published: 05.24.2006 09:13:29 PM CST