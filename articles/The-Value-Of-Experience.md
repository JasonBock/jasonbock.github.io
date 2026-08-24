---
title: The Value of Experience
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# The Value of Experience

I've been doing software development for over 10 years now, 8 of which has been as a consultant. There's a lot that I don't know about development, but I just ran into a situation where I was able to use experiences I had in the past to help them avoid potential problems. As you'll see, these are problems that are not so much technical issues as they are social issues.

One of my latest tasks is to implement URL friendliness. I spent a lot of time looking up how to do this in ASP.NET and all of the weird edge cases that go along with it. One of the things the client wants to do is try and make their URLs shorter. Right now, they have something that looks like this:

```
http://www.someclient.com/Info.com?GUID=9d8431c5-2959-47eb-8be6-27024fcda116
```

The "friendlier" version of this is as follows:

```
http://www.someclient.com/9d8431c5-2959-47eb-8be6-27024fcda116
```

However, they want the GUID to be smaller. This is where a past experience kicked in. A former client wanted to do the same thing, and by using a modified verison of the `BigInteger` class, we were able to encode the GUID in base 36:

```
http://www.someclient.com/9BPNL1X4LMVEMQO28LRXVPD06
```

With another slight modification (including lowercase letters as well), we can get down to base 62. This makes the URL smaller, which is what the client wants.

At the previous client, we only did a proof-of-concept with this - for many reasons, our GUID shortening idea was nixed for another approach. But when I started implementing it for the current client, another past experience came to light in my head. When you see a GUID in hexadecimal form, the only letters you'll see are "A", "B", "C", "D", "E", and "F". Therefore, you'll never see any questionable phrases with these letters in the GUID (unless you think "DEAD" is a questionable phrase). You can still generate a GUID with "666" in it, but there's little one can do about that. However, once you allow all of the characters in the alphabet to show up, you're now open to having any word or phrase show up in the URL. For example, the GUID "1046cd55-1a2b-f90d-5ca0-f2d8aa918bb1" turns into this:

```
http://www.someclient.com/YOUAREACOMPLETEBUTTMUNCH
```

This is the GUID encoded in base 36. Yikes! That's probably something you don't want a client to see. Granted, the chances of generating an entire phrase like this is very, very small, but generating a phrase with just one word like "DAMN" or "BITCH" is a bit more likely. Plus, I know that if something can happen, it will happen in production.

The reason I thought of this was a previous client has an automated password generator. Since they could generate any phrase, they had a class that would check the generated password for offensive phrases [1]. I raised this issue to my current client, and they hadn't even thought of that scenario. By mentioning this issue, I was able to save them potential fallout with generating a URL that seriously irked one of their users.

There's one more thing to this story. I was contemplating using the "bad word" dictionary I've seen at a previous client to filter our known bad words. This solution is problematic because you have to maintain this file for every "bad" phrase they could possibly think of. Furthermore, we don't have a choice to generate a new GUID for the content we're trying to find. So I was driving home when a possible solution hit me: just remove all the vowels from the enconding. This reduces the radix to base 52, but that's still much shorter than base 16, and then we have one simple encoding strategy for our identifier.

So what's my point with this post? It's not to rip on the young hotshots who start working at a software development company thinking they know everything there is to know about programming. Granted, people like that exist, but that's not what I'm getting at. You can find young people out of college who have excellent skills and get stuff done. What I'm getting at is that having experience can pay off in other situations, and sometimes these experiences don't always deal with simple implementation edge cases.

[1] This class was funny to look at, as it contains all sorts of offensive words in a string array. It was the only time the developers could get away with swearing in a code file.

> Published: 11.25.2005 11:46:24 AM CST