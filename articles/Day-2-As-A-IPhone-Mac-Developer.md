---
title: Day 2 as a iPhone/Mac Developer
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Day 2 as a iPhone/Mac Developer

Today was a better day. I found this video which does a great job explaining how to create a multivew project - it's precisely what I needed. I'm going through the initial two views in my application and getting them set up the way he recommends and I'll see how that goes, but it makes a lot of sense.

I also found out that Objective-C doesn't like my C# stylings :). I'm used to doing something like this:

```c#
private string accountNumber;

public Transfer(string accountNumber)
{
  this.accountNumber = accountNumber;
}
```

But doing the same thing in Objective-C:

```objc
-(id) initWithAccountNumber:(NSString *)accountNumber
{
   // ... self initialization...
   self.accountNumber = accountNumber;
}
```

gave me the "local declaration of accountNumber hides instance variable" warning. I don't like warnings, and even though from what I able to glean from reading the docs the code should work the way I expect it to, I made it unambiguous (or, more to the point, it got rid of the warning):

```objc
-(id) initWithAccountNumber:(NSString *)theAccountNumber
{
   // ... self initialization...
   self.accountNumber = theAccountNumber;
}
```

Tomorrow I'll hopefully have the two views hooked up and then things should go smoothly from that point on ... I hope ...

> Published: 09.11.2008 03:19:52 PM CST