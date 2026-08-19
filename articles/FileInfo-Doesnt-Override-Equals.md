---
title: `FileInfo` Doesn't Override Equals
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# `FileInfo` Doesn't Override `Equals`

So if you write code like this:

```c#
FileInfo f1 = new FileInfo(@"C:\File1.txt");
FileInfo f2 = new FileInfo(@"C:\File1.txt");

bool areEqual = f1 == f2;
```

`areEqual` will be `false`. At least that's what I saw today.

Mrph.

Oh well, this "fixes" it (at least it makes sense to me!):

```c#
bool areEqual = f1.FullName == f2.FullName;
```

> Published: 02.25.2008 04:02:27 PM CST