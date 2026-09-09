---
title: Collection Abuse
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Collection Abuse

## What is Wrong With This Code?

Language/Tool: VB 

```vb
Private Function ParseByCollection(ParseString As String, _
  FindString As String) As Boolean

Dim oCol As New Collection
Dim x As Long

ParseByCollection = False

For x = 1 To Len(ParseString)
  oCol.Add Mid$(ParseString, x, 1)
Next x

For x = 1 To oCol.Count
  If oCol.Item(x) = FindString Then
    ParseByCollection = True
    Exit For
  End If
Next x

End Function
```

One day about three years ago, I was talking to a manager/developer about a small project I was working on with him. I say, "manager/developer," because he was primarily a manager, but he wanted to start doing some development. He had taken a course on VB, and was ready to dive into development. He was having some problems with his code, and we ended up looking at some code that I've put into a method called `ParseByCollection()`. 

What he wanted to do is this. He had data in a `String` variable, and he wanted to find the first occurrence of a character in that string. He had just read about the `Collection` class, and he decided that using a collection was the best way to handle this task. As you can see, he takes each character out of the string, puts it into the collection, and then goes through the collection again to find the string. 

Now, there are definitely some things wrong with this method. For example, if `FindString` contains more than one character, `ParseByCollection()` won't work - at least, it won't find the first occurrence of `FindString`. But the biggest issue is the use of a collection to find a character in a string. As you can see, the collection has N number of items, where N is the length of the string given. Then it goes through each item in the collection and sees if it's the same as `FindString`. The overhead needed to do this is quite high - in fact, I'll show you how bad it can get later on. What amazed me is that he didn't even see in the first For loop that he was getting every character out of the string at that time. Why not make the comparison there? 

This problem was similar to the "Golden Hammer" antipattern, described in the book, "AntiPatterns: Refactoring Software, ... ". He had fallen in love with collections, and he reasoned that a collection would make his life easier for this string parsing exercise. Maybe he also thought that, since he had the string broken up in the collection, he could use the collection later on to do all sorts of interesting things (maybe to multiply two numbers together - who knows?). The reality was, he just needed to find the first occurence of a character, and you don't need to use a collection to do that. 

## Possible Solutions

There are many ways to refactor this. I'm going to go for a simplistic approach (remember, the audience was a developer who thought collections and string parsing go hand in hand) rather than getting into regular expressions and the like. So let's go with two implementations. The first one is the one I hinted at above - that is, just use the `Mid$()` function and go through the string: 

```vb
Private Function ParseByMid(ParseString As String, _
  FindString As String) As Boolean

ParseByMid = False

Dim x As Long

For x = 1 To Len(ParseString)
  If Mid$(ParseString, x, 1) = FindString Then
    ParseByMid = True
    Exit For
  End If
Next x

End Function
```

However, we can do better. VB has the `Instr()` method, which does exactly what we are looking for: 

```vb
Private Function ParseByInstr(ParseString As String, _
  FindString As String) As Boolean

ParseByInstr = False

If InStr(1, ParseString, FindString) > 0 Then
  ParseByInstr = True
End If

End Function
```

`Instr()` handles the string searching for you, so you don't have to worry about searching the string yourself. 

Now, let's compare these three methods and see what kind of performance gains we get out of last two. I set up a test program in VB that called each method with the same `ParseString` and `FindString` values. I ran six tests: 

1. Set `ParseString` equal to a string with 1000 zeros plus a 1 at the end, and search for a 1.
2. Set `ParseString` equal to a string with 1000 zeros, put a 1 right in the middle, and search for a 1.
3. Set `ParseString` equal to a string with 1000 zeros, stick a 1 in front, and search for 1.
4. Set `ParseString` equal to a string with 100 zeros plus a 1 at the end, and search for a 1.
5. Set `ParseString` equal to a string with 100 zeros, put a 1 right in the middle, and search for a 1.
6. Set `ParseString` equal to a string with 100 zeros, stick a 1 in front, and search for 1.

After I ran some tests, I found out that `ParseByMid()` was anywhere from 6 to 16 times faster than `ParseByCollection()`. However, `ParseByInstr()` was hundreds of times faster than `ParseByCollection()`. Even if you use a method like `ParseByMid()`, you'll still trounce the collection-based algorithm. But the answer is clear: use `Instr()`. 

In a way, I felt bad that this guy used the collection the way he did - he felt pretty stupid after he realized what he had done. He really wasn't trying to make his life harder than it needed to be; he simply didn't know that VB had methods that did what he wanted. But I've learned as a developer that I should always see what the tool has in its' libraries before I try to implement a piece of functionality myself. That's one of the fundamental rules of software development: reuse when possible and applicable. Most tools will have a lot of basic algorithms (such as string parsing and searching) implemented, so search the help files before you start to code the algorithm yourself. And if you do need to implement something on your own, remember that you may need to learn how to use some new tools to solve it. 

## Summary

1. Keep the blinders off. Continue to learn new things, but don't get trapped into solving problems the same way over and over again.
2. Make reuse your mantra. Unless the algorithm has been horribly implemented, use what has already been given to you.

> Published: 05.18.2001