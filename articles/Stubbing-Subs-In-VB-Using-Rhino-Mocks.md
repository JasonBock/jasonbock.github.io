---
title: Stubbing Subs in VB Using Rhino Mocks
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Stubbing Subs in VB Using Rhino Mocks

[As before](https://jasonbock.net/articles/Stubbing-Methods-Using-Rhino-Mocks-In-VB9), this post is primarily for my own benefit and hopefully others can use it too.

Let's say I have the following interface:

```vb
Public Interface IDataManager
  Sub Load(ByVal id as Integer, ByVal useReader as Action(Of IDataReader))
End Interface
```

I want to stub it using Rhino Mocks. Here's how I did it:

```vbnet
Dim manager = MockRepository.GenerateStub(Of IDataManager)()
manager.Stub(Of IDataManager)(AddressOf Me.LoadStub).WhenCalled(AddressOf Me.LoadCalled)

Private Sub LoadCalled(ByVal invocation as MethodInvocation)
  '  Do magic with the two arguments passed.
End Sub
 
Private Function LoadStub(ByVal e as IDataManager) As IDataManager 
  e.Load(Arg(Of Integer).Is.Anything, Arg(Of Action(Of IDataReader)).Is.Anything)
  Return Nothing
End Function
```

Frankly, doing it in C# feels much cleaner to me:

```c#
var manager = MockRepository.GenerateStub<IPredefinedTextManager>();
manager.Stub(e => e.Load(Arg<int>.Is.Anything, 
  Arg<Action<IDataReader>>.Is.Anything)).WhenCalled(
  (invocation) =>
  {
   ' Do magic with the arguments.
  });
```

YMMV :)

> Published: 09.29.2009 11:41:44 AM CST