---
title: Frustrating Code
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Frustrating Code

Andrew Stopford sent me another Ruby/.NET link - I really need to check this language out more. Of course, there's a ton of languages I'd love to check out, but I've been hearing good things about Ruby.

I forgot to mention - I'm going to the Rush concert here in the Twin Cities on Nov. 2nd. I'll be sporting my lovely Apress glow-in-the-dark t-shirt - you won't miss me.

The ASP.NET project is going OK. I've accepted that it'll be in the state it's currently in and moved on. I also found it amusing that Sam Gentile picked up on my comments. Sometimes, I wonder, if my programming tales of woe are very familiar to others in the software development industry, do we really have things bass-ackwards? I see all this wonderful work and discussion on web services and AOP, yet I've far too often seen developers write code **after** a return statement, like this:

```c#
public int BadCodingStyle()
{
  return 1;
  //  More code here.
}
```

True, a good compiler will catch this and not include that code, but a developer should **never** write this code in the first place. Yes, there are outstanding developers out there. Yes, there are super-geniuses who can type circles around most of us mere mortals. And no, I'm not being sarcastic with those last two sentences. But my point is that I feel there isn't enough emphasis on good programming practices. Maybe it's just my own experiences and I've missed out on insightful code reviews and design meetings that occur elsewhere. But I'm starting to think that's not the case.

And it's not just IT development either. I've been on projects where purchased components from third-party vendors were used, and I couldn't believe the designs. The most recent blunder I saw was a COM server that **required** the caller to provide a window handle. The reality is, the method doesn't really need it, but for some reason it's designed that way, and it makes for a real PITA when you're trying to invoke it in a class library where you usually don't make a window (although in .NET it's rather straightforward to make a window anywhere you want, but who really wants to make a window just to get a god-forsaken window handle?).

Ultimately, it's onward and upwards. I really don't want progress to stop. I'm just hoping that everyone in the community keeps up.

> Published: 10.23.2002 10:50:00 PM CST