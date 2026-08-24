---
title: Join It, Baby!
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Join It, Baby!

I've run into coding situations where I have an array of strings and I want to put 'em all together, separated by a delimiter. For example, I have "A", "B", and "C", and I want to have a string that looks like "A,B,C". I still see .NET code written like this:

```c#
string[] letters = new string[] {"A", "B", "C"};
string letterList = string.Empty;

foreach(string letter in letters)
{
  letterList += letter + ",";
}

letterList = letterList.TrimEnd(',');
```

Another approach is to use a `for` loop, and test to see if you're at the last element of the array to determine if a comma should be included or not. But it's so much simpler than that:

```c#
string[] letters = new string[] {"A", "B", "C"};
string letterList = string.Join(",", letters);
```

Of course, there's probably a lot of things I'm still missing in the framework that would make me go, "d'oh! - I should be using that!".

> Published: 03.08.2006 11:42:10 AM CST