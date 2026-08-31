---
title: Splitting Strings With Large Delimiters
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Splitting Strings With Large Delimiters

Yesterday I ran into an issue where I wanted to split a string that looked something like this:

```
Data@@@MoreData@@@EvenMoreData
```

In other words, the delimiter, `@@@`, was more than one character long, but `String.Split()` only takes a char data type. While I was IM-ing Cory about the problem, I found `Regex.Split()` in the SDK, but Cory also suggested importing the `Microsoft.VisualBasic` assembly and use `Strings.Split()` [1].

Well, this started to make me think ... which one is "better"? More specifically, which one performs faster? Looking at the code in Reflector, `Strings.Split()` uses a bunch of `IndexOf()` and `Substring()` calls to parse the string (which I expected). `Regex.Split()` does as well along with calls to `Match()`. Basically, the implementations are different, and the only way to see which one was faster was to write a test program and see which one wins. That's what I did last night - you can get the code here. Basically, I create a bunch of random strings with different words and delimiters, and use a high performance timer I found here to find out how long the splits took.

So, after splitting a thousand strings, the winner is ...

... wait for it ...

`Strings.Split()`!

Well, it's not by much. My numbers show that `Strings.Split()` is 0.000290633 seconds faster than `Regex.Split()`. That's almost 3 ten-thousandths of a second faster, and this is an average over 1000 splits.

Feel free to take a look at the code and trash it. Peformance tuning is not something I do a lot of, so my numbers may not be correct. In other words, I make no claim that my numbers are "true" - use them at your own risk. Moreover, I didn't look at memory issues, which is definitely another aspect that should be reviewed. But it was fun to use what's in the regular expressions library and compare it to the VB .NET runtime. Personally, I would just use `Regex.Split()` as it doesn't require me to add a new reference to another assembly in my project, and the difference between the two approaches is just not that substantial for the string parsing that I'm doing to warrant adding another assembly to the project.

[1] There's also a `split()` function in JScript, but I didn't try to mess with that one.

> Published: 03.30.2005 09:06:18 AM CST