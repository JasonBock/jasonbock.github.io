---
title: Stubbing Methods Using Rhino Mocks in VB9
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Stubbing Methods Using Rhino Mocks in VB9

VB doesn't have multi-line lambdas (this is being fixed/corrected/changed in version 10), which makes it a bit problematic to use with APIs like Rhino Mocks. It's not hard, but for someone like me it took me a while to figure out how to get a stubbed method to do some custom action. So basically this post is here for my own benefit - i.e. in case I have to look it up again :). Hopefully this will be helpful to someone else as well.

Let's say you have an interface that gets and saves DTOs:

```vb
Public Interface IPersonRepository
  Function Load(ByVal id As Integer) As PersonTransferObject
  Function Save(ByVal data As PersonTransferObject) As PersonTransferObject
End Interface
```

Stubbing `Load()` isn't that hard; it's the `Save()` that took me some time. Here's what it looks like:

```vb
Dim repository = MockRepository.GenerateStub(Of IPersonRepository)()
repository.Stub(Function(e) e.Load(PersonTests.ExpectedId)).Return(data)
repository.Stub(Function(e) e.Save(Arg(Of PersonTransferObject).Is.Anything)).Return( _
  Nothing).WhenCalled(AddressOf Me.DoSave)
```

Where `DoSave()` looks like this:

```vb
Private Sub DoSave(ByVal invocation As MethodInvocation)
  Dim data = CType(invocation.Arguments(0), PersonTransferObject)
  invocation.ReturnValue = New PersonTransferObject( _
    If(data.Id = 0, PersonTests.NewId, data.Id), ...)
End Sub
```

The key is to use `WhenCalled()` and use `AddressOf` to give it the method that will handle the method under test. In that method, use the `MethodInvocation`-based argument to get argument values and set the return value.

Hope this helps!

> Published: 09.22.2009 12:27:59 PM CST