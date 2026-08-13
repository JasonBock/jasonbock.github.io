---
title: Throwing Yourself
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Throwing Yourself

I had a stupid idea a day ago: could I write code where "Throw Me" would be valid? Turns out it's not that hard:

```vbnet
Imports System.Runtime.Serialization

Public Class ACustomException
  Inherits Exception
    Public Sub New()
      MyBase.New()
    End Sub

  Private Sub New(ByVal info As SerializationInfo, ByVal context As StreamingContext)
    MyBase.New(info, context)
  End Sub

  Public Sub New(ByVal message As String)
    MyBase.New(message)
  End Sub

  Public Sub New(ByVal message As String, ByVal innerException As Exception)
    MyBase.New(message, innerException)
  End Sub

  Public Sub DoIt()
    Throw Me
  End Sub
End Class
```

You can do a similar thing in C# with "throw this" but "Throw Me" sounds funnier :).

> Published: 11.19.2010 12:22:09 AM