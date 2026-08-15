---
title: Solving the Real Problem
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Solving the Real Problem

So I'm sitting at the Magenic office, and Chris IMs me:

> does C# have any kind of redim preserve for an array, like VB does

Now, I know C# doesn't, but I started to think, hey, how hard could this really be? So I started to cruft up an extension method:

```c#
public static T[] Redim<T>(this T[] @this, bool preserve)
```
Which would be called like this:

```c#
int[] array = {1, 2, 3};
var redimmed = array.Redim();
```

Of course, this extension method doesn't have the bounds listed as a params array at the end of the call, but I never got that far. Read on ...

So I started getting my tests to pass, but then I started to dive into jagged arrays and whatnot, and I thought ... hmmm, I wonder what VB is using anyway? So I wrote the following code:

```vb
Sub Main()
  Dim x(3, 5) As Integer

  Dim c = 0

  For i = 0 To 3
    For j = 0 To 5
      x(i, j) = c
      c += 1
    Next
  Next

  ReDim Preserve x(3, 10)
End Sub
```

And looked at it in Reflector:

```vb
<STAThread> _
Public Shared Sub Main()
  Dim VB$CG$t_i4$S0 As Integer
  Dim x(,) As Integer(0 To .,0 To .) = _
    New Integer(4  - 1, 6  - 1) {}
  Dim c As Integer = 0
  Dim i As Integer = 0
  Do
    Dim j As Integer = 0
    Do
      x(i, j) = c
      c += 1
      j += 1
      VB$CG$t_i4$S0 = 5
    Loop While (j <= VB$CG$t_i4$S0)
    i += 1
    VB$CG$t_i4$S0 = 3
  Loop While (i <= VB$CG$t_i4$S0)
  x = DirectCast(Utils.CopyArray(DirectCast(x, Array), _
    New Integer(4  - 1, 11  - 1) {}), Integer(0 To .,0 To .)(,))
End Sub
```

Seems to me like it's basically doing an array copy. If you don't preserve the information, VB just creates a new array.

So it seems like `Redim` is kind of useless ... why would I want to redo the functionality in C#?

Then I ask Chris why he wanted this in C#, and here was his answer:

> I created an array way too big (because I didnt know how much I would need) then wanted to chop it down after...the guy I was passing it to was expecting an array of string dictionaries

Aha! Sometimes you have to ask a couple of times to find out the real problem, because in this case, generic collections make the problem much easier to solve:

```c#
var data = new List<SortedDictionary<string, string>>();
// Add all the dictionaries here...
var dataAsArray = data.ToArray();
And there you go.
```

Now, I'm sure there are cases in VB where `Redim Preserve` is a little more compact. And I'm definitely not trying to get into a language war here. My point is that the problem you're trying to solve may not be the real problem at hand. In this case, reproducing what VB has isn't the correct solutions; there's a simpler way that works well in both languages.

> Published: 03.13.2009 01:53:09 PM CST