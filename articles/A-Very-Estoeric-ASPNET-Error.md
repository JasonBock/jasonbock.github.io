---
title: A Very Estoeric ASP.NET Error
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# A Very Estoeric ASP.NET Error

I have some free time tonight, so I'm working on updating the main site. Well, I just ran across one of the most bizarre errors I've ever seen, and I wanted to share it for historical purposes.

Up until tonight, my assembly was called `OldName.dll`. After some thought, I realized this wasn't the "right" name, so I change it to NewName.dll. But once I did this, my default.aspx page wouldn't load, and I'd get some really bizarre error about my `Default` class being defined twice. Huh? I checked my code base 15 times over, and ... nope, I only had one definition of `Default`. So what the hell was going on?

Here's the problem. Since I still had `OldName.dll` and `NewName.dll` in my `bin` directory, the ASP.NET engine was getting confused since there were **two** `Default` classes, and it decided to puke on me. Now, why ASP.NET was still referring to `OldName.dll` when I definitely had no more use for it is a mystery, but once I cleaned out my `bin` directory, all was well with the world.

Whew! No back to coding while the live Transatlantic concert plays on my TV. I also have a wonderful glass of merlot wine by my side (which did not cause the problem in the first place, honest!).

> Published: 09.17.2004 09:53:19 PM CST