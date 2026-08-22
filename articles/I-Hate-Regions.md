---
title: I Hate Regions
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# I Hate Regions

I know that some coders live and die by regions. You know, those areas with the "#" character surrounding them?

```c#
#region Massive Code Section
// Lots of code is usually hidden here...
#endregion
```

I think my little code snippet illustrates my distaste of regions. No matter what code base I've seen, they're always used to hide a ton of code. So, when you open up the class file in VS, you only see 10 lines, but the class usually has thousands of lines of code.

I'm not saying regions are evil in and of themselves. But the only time I've seen them used is to hide huge quantities of code. Classes and methods should be small and lightweight. If you're using regions to cover that up, that's a sign that you need to refactor the code base to increase its maintainability.

> Published: 04.11.2007 07:26:40 AM CST