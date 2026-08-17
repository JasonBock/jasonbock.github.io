---
title: Day 7 as a iPhone/Mac Developer
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Day 7 as a iPhone/Mac Developer

Thanks to Simon's help, I was able to get my class level method called from `NSThread`. Sweet!

I also found out about extension methods (sort of) in Objective-C through via categories. So I can declare something like this:

```objc
@interface UIView (MyCustomPackage)

- (void)update:(int)action;

@end
```

And now the one warning that I used to have...goes away, because it "looks like" `UIView` now supports the `update` method. Total awesomeness.

I've been getting annoyed with the mouse on the Mac. Feels like I'm moving it through molasses at times. Here's an interesting article on the issue. I never knew how complicated it was to get a mouse to work so smoothly.

> Published: 09.18.2008 01:29:03 PM CST