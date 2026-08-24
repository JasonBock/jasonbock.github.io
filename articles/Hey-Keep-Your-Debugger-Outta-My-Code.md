---
title: Hey! Keep Your Debugger Outta My Code!!
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Hey! Keep Your Debugger Outta My Code!!

Want to make someone's debugging life a little confusing? Do this:

```c#
[System.Diagnostics.DebuggerStepThroughAttribute]
public class NoDebuggersAllowed { }
```

Any methods added to `NoDebuggersAllowed` will not be debuggable because of that attribute. A co-worker was going nuts because his breakpoint on the constructor was never getting hit. It took me a while before I saw that the attribute was on the class. He didn't add it; the class was generated from some code generating tool and thought it would be a wonderful idea to confuse the hell out of someone by not letting them step into the internals of the generated class.

I can see where this attribute has some benefits, but the next time you're sure you've set a breakpoint that should be hit and it isn't, look out for `DebuggerStepThroughAttribute`!

> Published: 05.23.2006 10:49:23 AM CST