---
title: `foreach` And Skipping Objects
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# `foreach` And Skipping Objects

Hmmm. I always thought `foreach` in C# and `For Each` in VB .NET would return objects of the specified type and skip over ones that don't match. I should've read the documentation. Consider the following code:

```c#
ArrayList al = new ArrayList();
al.Add(new Guid());
al.Add(new Random());
al.Add(new Guid());
al.Add(new Random());
al.Add(new Guid());
al.Add(new Random());
al.Add(new Guid());
al.Add(new Random());

foreach(Guid g in al)
{
  Console.WriteLine("Found a Guid object.");
}
```

It will throw an `InvalidCastException`. I know how to get around this - here's some code in VB .NET to illustrate:

```vb
Dim oGUIDs As IEnumerator = al.GetEnumerator()

Do While oGUIDs.MoveNext()
  If TypeOf oGUIDs.Current Is Guid Then
    Dim aGUID as Guid = CType(oGUIDs.Current, Guid)
    '  Do something with aGuid...
  End If
Loop
```

So much for assuming!

> Published: 07.10.2002 10:30:00 AM CST