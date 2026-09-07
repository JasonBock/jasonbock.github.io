---
title: This is What I "Discussed" All Day at Work
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# This is What I "Discussed" All Day at Work

While people pontificate about Matrix-like corporate Programs, SOAs, VS .NET 2005, remember that crap like the following story happens all the time.

Even though my project has been cancelled, we are still developing code. (Go figure ... ) One of the things we did is create a "web service" that receives XML data and gives XML data back. For the past couple of days, the group that is calling this service has been having issues. We finally figured them out today, but once the other team could call our service correctly to get the data, the issue came up about empty XML elements and how they were physically represented in the XML stream.

Here was the problem. They wanted to see an empty element like this:

```xml
<priceCode/>
```

Well, one of our elements look like this:

```xml
<priceCode></priceCode>
```

Now, I think the spec is clear on this. That is, neither representation matters! You load up the XML into a parser, check the inner text, and, guess what, both will contain nothing. As a developer, you simply shouldn't care. Right? **RIGHT??**

Well, the other team apparently does. In fact, they care so much that they're almost insisting that we change our code so we produce the first representation. Sigh ... I can't even think of where to start with this. This is XML 101, correct? Maybe I'm missing something here, but I've reviewed this with enough people that they don't understand why that team cares at all about the XML is formatted. An empty element is an empty element. This is what makes me question whether I'm in the right industry. This is so petty - I can't believe a team can say with a straight face that, while they realize we're producing correct XML, they can't handle the 2nd version, so they expect us to "fix it." In my mind, they have the bug, not us. And, to be blunt, I know I'm out the door sometime in the very near future, so I'm more prone to stand my ground and fight this. I'm definitely not one to burn a bridge, and even if I smell smoke, I'll try to find water to put it out. But, hell, if the bridge is fully engulfed in flames via a fire I didn't start, and it's about to fall into a deep chasm, I start getting a little edgy. I figure, well, I'm screwed anyway, right? How much worse can I make it?

Seriously, I'm not about to sabotage systems or become an obnoxious asshole just for kicks. But, enough is enough. "Fixing" this problem is simply hiding an issue that the other team has, and they need to address it, not us. And issues like this just drive me nuts. If a group came to me and said, "look, you're not handling this right, fix it," and I can verify it, I'll fix it. This just seems like a stick-your-head-in-the-ground approach. But, in an odd sort of way, I'm glad I still experience moments like this. When I see people drooling over the new features in VS .NET or the cool stuff out at TechEd, I know that there's a ton of junk that goes on in real project development that people don't seem to focus on as much. Granted, I've seen some books come out recently that talk about horror stories in this industry, but I feel like it's not enough. True, it's not enough either to just complain about problems without giving solutions, but it's not enough either to fall in love with XAML when people still can't parse XML correctly.

> Published: 05.25.2004 07:53:59 PM CST