---
title: A Split Clarification
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# A Split Clarification

I should clarify a bit what I see with the data I stated in [a previous post](https://jasonbock.net/articles/Splitting-Strings-With-Large-Delimiters.html). After digging into a bit (due to Cory not believing me :P), I found that for short strings (anywhere from 20 to 1000 characters), `Strings.Split()` blows away `Regex.Split()`. It's anywhere from 5 to 20 times better. But when you start generating large strings (between 1000 to 100000 characters), the benefit goes away as the string gets larger. With strings sizes greater than 100,000 characters, the difference goes away completely. My original test run was always generating strings at least 4000 characters long, so that's why my average showed no significant difference between the two methods. If you're curious, check the SplitResults.xls file in the code drop where I separate the averages out for different string sizes.

> Published: 03.30.2005 11:37:18 AM CST