---
title: Using MTS's Shared Property Group Manager Component to Maintain State in Transactional Objects - Part 2
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Using MTS's Shared Property Group Manager Component to Maintain State in Transactional Objects - Part 2

## Introduction

In Part 1 of this article series, I demonstrated a technique that kept the state of a COM object across a transactional boundary. This technique grabbed the pointer value of the context wrapper and used it to cache the object's state in the SPM. In the 2nd (and final) part, I'll show in detail why this technique doesn't work as presented in Part 1, and why there isn't a way to truly "fix" it.

## Live and Learn

There's a motto that I've tried to live by ever since I heard it in college: "Don't reinvent the wheel." Unfortunately, I forgot to do this when I wrote Part 1. I wanted to add more material to refine the techniques given in Part 1, but I soon found out that others have done what I was trying to do, and it just doesn't work. Why? Well, let's examine the process in more detail.

To start out, let's take a look at the performance issue. There's a way to make the persistence code slightly faster by storing the object state as a byte stream via a Property Bag. To do this, you make your class `Persistable` and you add two functions to read and write the object's state. For example, this is what the read method looks like:

```vb
Private Sub ReadPropertyBag(PropBag As PropertyBag)

m_dblX = CDbl(PropBag.ReadProperty("X"))
m_dblY = CDbl(PropBag.ReadProperty("Y"))

End Sub
```

I made this a `private` method so the `ReadProperties` method of the `CPointWithState` could call this method along with the `GetObjectState`. This is how you'd call this method in `GetObjectState`:

```vb
If Not objSharedProperty Is Nothing Then
  Dim oP As New PropertyBag
  '  Get the object's state from the SPM.
  oP.Contents = objSharedProperty.Value
  '  Read it in.
  ReadPropertyBag oP
  Set objSharedProperty = Nothing
End If
```

The whole process is reversed for the writing of the object state. This does improve the performance of persisting the object in the SPM, but there's still a cost to be paid. I've compared using the `CPointNoState` with the `CPointWithState`, and the "stateful" object is 2.5 times slower. Before I added the byte stream persistence, it was around 16.5 times worse, so you do gain something, but this technique is costly.

I predicted this in Part 1, which is why I warned the reader (you) to use the technique with caution. However, it gets worse.

## The Persistence of the Context Wrapper

One thing that we absolutely rely upon in this technique is that the context wrapper remain alive after our object is destroyed by MTS. However, there's a catch here. Look at the following code:

```vb
Dim objPoint As MTSState.CPointWithState

Set objPoint = CreateObject("MTSState.CPointWithState")

objPoint.X = 3
Set objPoint = Nothing

'  Now sleep for two seconds
Sleep 2000

'  You context wrapper should still be there,
'  and therefore X = 3
'  but this isn't what we want!
Set objPoint = CreateObject("MTSState.CPointWithState")

MsgBox "X should be 3:  X = " & objPoint.X
```

See what's going on here? Even though you explicitly released the object, when you come back 2 seconds later (that's what the `Sleep` API call is for) and "recreate" the object, the object's `X` value is set to 3! This is obviously wrong from the client's perspective, but the reason for this is that you context wrapper - more specifically, the mtx.exe MTS surrogate process - doesn't disappear for 3 minutes. That's the default value for the server process shutdown timeout, but it doesn't really matter what the value is, because the problem is that the context wrapper stays alive even when we've told COM that we no longer need the object. From the object's perspective, it's just grabbing the pointer value and looking in the SPM for any cached object state. Unfortunately, it finds the old one. What we want it to do is to create a new object, but there's no way for the MTS object to know this.

I thought of creating a `IHardPersist` interface that would take two methods, `Load` and `Save`. The `Load` method would take an object ID (OID) of a `GUID` type to uniquely identify the object we want, and the `Save` method would persist the object to permanent storage. However, this doesn't work, because once `SetComplete` or `SetAbort` is called in the object, it's gone and it forgets what the OID was - i.e. we're right back to where we started. You'd have to pass the OID into the object on every method call for this to work.

Therefore, this is even more evidence that you should have your state stored in an object that is "probably" outside of a transaction (more on this in a moment). This is the technique I hinted at in Part 1. Make your stateful objects outside of a transaction, and use MTS as an object persistence layer. However, there's even a better reason why this is a better method - storing state in the SPM (or anywhere else for that matter) using the context wrapper's pointer value results in memory leaks.

## Releasing Cached Memory

Take a look at the following code snippet:

```vb
Dim objPoint As MTSState.CPointWithState

Set objPoint = CreateObject("MTSState.CPointWithState")
objPoint.X = 3
Set objPoint = Nothing

'  Now sleep for 2 1/2 minutes (i.e. do other work).
Sleep 150000

'  You context wrapper will still be there
'  and so is the cached memory.
```

Do you see the leak? Because we have no way of knowing when the context wrapper is destroyed, we have some information stored in the SPM that we can't clean up. There's no way to know for sure when the context wrapper will go away, and that's not good for our technique.

Again, the solution is to move the state outside of MTS. Of course, this is something that somebody worked on a year ago (hence my self-berating for reinventing the wheel). Check out the following two posts - #1 and #2 - from Don Box. I won't repeat what he said there in detail, but in essence he states that it does suck that you loose identity with your object. Furthermore, he also makes the observation that you can't clean up your information when the client releases (at least deterministically). You only other viable option is to split the object into two different objects, and make the "outside" object support transactions. That way, it'll pass the transaction flow if it's called from a client in a transaction, but it won't make a new one if its' client wasn't in a transaction. He does state that it would be nice if we could store a OID in the wrapper and get some notification when the wrapper went away. This would make the transient state storage idea viable, but, alas, that's not the way things work in COM.

Well, at least I came to same conclusion, albeit 1 year late ;).

## Conclusion

I've demonstrated why it's not a good idea to use the SPM (or any other transient persistence mechanism) to store state across transactional boundaries. Your best bet is to make 2 objects, one that holds the state and one that persists the state. I'm still bothered that I've read in some articles that COM+ will have some mechanism to keep state across TX boundaries, but they never demonstrate in detail how this would work. My guess is that the IMDB had something to do with it, but since it's been dropped that's not an option anymore. Even if there is a way, it still will be slower performance-wise to implement, and when it comes down to it, users want fast systems.

> Published: 04.29.1998 12:00:00 AM CST