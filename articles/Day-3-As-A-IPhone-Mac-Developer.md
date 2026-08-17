---
title: Day 3 as a iPhone/Mac Developer
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Day 3 as a iPhone/Mac Developer

Today I've been able to make more progress on getting my multiview stuff to work. I'm still only getting the first view to show up, but that was a major achievement for me. Now that I know how to do it, things should start moving quicker.

I'm getting more familiar with Xcode's IDE. Deleting all breakpoints, getting used to the debugger, using Interface Builder more ... I don't feel as lost as I did a couple of days ago.

I also realized that this:

```objc
if(aString == @"SomeValue")
```

doesn't work, but this:

```objc
if([aString isEqualToString:@"SomeValue"] == YES)
```

does. Well, it works the way I'd expect it to.

I'm still getting used to the bracket syntax to send messages.

> Published: 09.12.2008 02:31:59 PM CST