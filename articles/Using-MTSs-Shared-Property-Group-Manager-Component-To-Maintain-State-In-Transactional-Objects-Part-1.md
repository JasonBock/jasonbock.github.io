---
title: Using MTS's Shared Property Group Manager Component to Maintain State in Transactional Objects - Part 1
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Using MTS's Shared Property Group Manager Component to Maintain State in Transactional Objects - Part 1

## Introduction

As many COM developers know, MTS destroys transactional objects when either `SetComplete` or `SetAbort` is called. This is done to enforce semantic correctness within a transactional boundary (as Don Box eloquently put it in his ActiveX Q&A; article in the March 1998 issue of MSJ). However, this brings up the issue of keeping state between method calls. Either the client must pass the state to the object on every method call or the object must find a way to retrieve and save its' state on its' own. In this article, I'll demonstrate how you can use the **Shared Property Manager** components to achieve this transparency, and I'll also discuss if this is a viable option under MTS.

## A Vexing Point Problem in MTS

To start, let's take a look at a very basic class definition of a point called `CPointNoState` coded in a VB IDE (note that all of the code is done in BASIC through VB):

```vb
Private m_dblX As Double
Private m_dblY As Double

Public Property Let X(ByVal Val As Double)

m_dblX = Val

End Property

Public Property Get X() As Double

X = m_dblX

End Property

Public Property Let Y(ByVal Val As Double)

m_dblY = Val

End Property

Public Property Get Y() As Double

Y = m_dblY

End Property
```

As you can see, there's nothing really interesting here. The class is saving the state of a given point via its' `X` and `Y` properties. A client could create a point via the following pseudo-code:

```vb
Dim objP As New CPointNoState

objP.X = 3.4
objP.Y = 2.3
```

Also, the class isn't "MTS-aware," meaning that its' `MTSTransactionMode` is set to `NotAnMTSObject`. But let's say we wanted to make it part of a transaction. So we change the transaction mode to `RequiresTransaction`, and add the following code to the `Declarations` section:

```vb
Private m_objContext As ObjectContext
Implements ObjectControl
```

Again, this is pretty standard stuff for MTS programming. We can get the object context via a call to `GetObjectContext` and we can also catch when the object is activated and deactivated via the `ObjectControl` interface. Now let's take a look at how the properties change (we'll look at changing the `X` coordinate - you can assume that similar changes have occurred in the other methods):

```vb
Public Property Let X(ByVal Val As Double)

On Error GoTo Error_LetX

m_dblX = Val

If Not m_objContext Is Nothing Then
  m_objContext.SetComplete
End If

Exit Property

Error_LetX:

If Not m_objContext Is Nothing Then
  m_objContext.SetAbort
End If

End Property
```

Notice the changes? We're now telling the current transaction that if we encountered no errors, our work is complete and the transaction can commit; if we encountered errors we tell the transaction to abort (of course, this depends on what the other objects "vote" on during the transaction). Herein lies the issue of state. Because we've marked our object as transactional, our object will be destroyed once the entire transaction is finished. This is to preserve transactional semantics and has nothing to do with scalability issues. However, a result of this model is that the component (more or less) becomes stateless. For example, consider what happens in our previous client pseudo-code. When X is set to 3.4, the object calls `SetComplete`, which destroys the object. Then, `Y` is set to 2.3 - again, the object is destroyed. So, if we added another line of code in our pseudo-code example like so:

```vb
Dim objP As New CPointNoState

objP.X = 3.4
objP.Y = 2.3
MsgBox objP.X
```

We would see 0 in the message box and not 3.4.

> I realize that you don't have to call `SetComplete` for every method call. Technically, by not calling `SetComplete` or `SetAbort` you can maintain state in your object, because the object will remain until one of the methods is called. However, I strongly recommend it, even if your object is running under MTS but is not a transactional object. MTS can reclaim resources that the non-transactional object has used after a call to `SetComplete` or `SetAbort`, so that's why I've done it everywhere. Plus, it illustrates the whole issue of state management within a transactional object explicitly.

So how do you maintain state? There are a couple of ways to do this, but most of them boil down to this: Maintain the state in the client. Use a `SetData` method to pass the state through in one shot, and commit the state to permanent storage. Create a `GetData` method that allows the client to retrieve the state data via some object identifier.

> Another take on this practice is to use objects with state that do not exist under MTS. The `GetData` method would return an object to the client that s/he could change at will. When the state should be saved, the object with state is passed into the `SetData` method. Therefore, your MTS layer becomes nothing more than a persistence layer with a bunch of reader and writer objects. I don't mind this approach - in fact, on one project I used Property Bag byte arrays to pass object state from the MTS server to the client and it worked very well. Keep this technique in mind - I'll come back to it near the end of this article.

Therefore, we would now have two new methods in our `CPointNoState` class:

```vb
Public Sub SetPoint(ByVal X As Double, ByVal Y As Double)

'  Add code here to save the point.

End Sub

Public Sub GetPoint(ByVal PointID As Long)

'  Add code here to get the point.

End Sub
```

Here's where I'll be a little blunt. This technique feels weird. I understand why MTS does what it does, and I don't argue with that. It's just that this looks awkward from a traditional OO perspective. As a client, I shouldn't have to do what the object could do, which is to maintain the state. It's not a bad idea to pass all of the initialization data over in one shot to save on network traffic, but it leaves the client holding the bag.

So how can you make the state issue transparent? As I've found out, you can hide this issue from a client fairly easily using two things: the Shared Property Manager and an undocumented function in VB. Let's take a look at these tools in more detail.

## The Shared Property Manager

This component is usually mentioned in MTS books but it doesn't get a lot of attention. Its' main purpose is to give a MTS developer a way to share information between objects. Note that this information is located in memory, so if a crash occurs it's lost. Since the component maintains state across transactional boundaries, it's a good candidate to solve our problem of state transparency.

To use this component, you create an instance of the shared property group manager like this:

```vb
Set objPropertyGroupManager = m_objContext.CreateInstance("MTxSpm.SharedPropertyGroupManager.1")
```

This object allows you to create new shared property groups via the `CreatePropertyGroup` method:

```vb
Function CreatePropertyGroup(Name As String, dwIsoMode As Long, dwRelMode As Long, fExists As Boolean) As SharedPropertyGroup
```

The `Name` argument identifies the new group. `dwIsoMode` is an enumeration with two values: `LockSetGet`, (the default) which states that each get or set operation on a property is atomic, or `LockMethod`, which locks all of the properties of the group to the caller. `dwRelMode` is another enumeration with two values: `Standard` (the default) which destroys the group when all references to the group have been released, and `Process`, which keeps the group alive until the process that created the group in the first place has terminated. Finally, `fExists` tells us if the group was already there before we tried to create it.

Once we have a property group, we can create properties via the (you guessed it) `CreateProperty` method:

```vb
Function CreateProperty(Name As String, fExists As Boolean) As SharedProperty
```

The arguments have the same meaning as in `CreatePropertyGroup` so I won't discuss them again (there's also a `CreatePropertyByPosition` but I won't talk about it here). Once you have a `SharedProperty` object, you can get and set the `Value` property to whatever you want - it's dimensioned as a `Variant` (although I wouldn't recommend storing object references).

## Undocumented VB Functions

We now know of a mechanism that can maintain information even when an object dies (I'll discuss how we'll use it in the next section). However, there's still one problem to overcome. Here's a diagram to illustrate what's going on when a client calls an object running under MTS:

(2026 - sorry, image is gone)

You can see that the context wrapper sits between the actual object and the client. When a transaction completes, the MTS object is destroyed. When a new one begins, a new object is created. However, at no time does the context wrapper disappear. That's a good thing for us, because we can exploit that fact to name our shared property group - namely, through its' pointer value.

Yes, I said "pointer" and this is an article that uses VB! There are three undocumented functions in VB - `VarPtr`, `StrPtr`, and `ObjPtr` - that return the pointer value of the variable under inspection. In our case, we'll use `ObjPtr` to get the pointer value of our context wrapper reference and stringify the pointer value in the name of our property group.

Remember, the context wrapper doesn't go away even when the underlying object is destroyed, so this pointer value does give us the uniqueness we're looking for. Well, almost. We'll also combine it with the process ID to ensure that our property group name is unique.

> **Don't** confuse the context wrapper object and the object context object! They are two separate things, and using the object context object's pointer value doesn't work. Trust me. When I started writing this article, part of my brain was asleep and I was mixing up the two. As I created the test client, I found out that having more than one object with state in memory wasn't working. That's because I was using the object context object's pointer value, and each MTS object was referring to the same object context! So much for unique values. After realizing my mistake, I switched over to using the context wrapper's pointer value.

## Tying the Two Tools Together

Let's get back to our point class. We're actually going to create a new class called `CPointWithState` just to keep things separate between the two. It's defined exactly the same as `CPointNoState` except that our `Activate` and `Deactivate` events from the `ObjectControl` interface are different:

```vb
Private Sub ObjectControl_Activate()

Set m_objContext = GetObjectContext
GetObjectState

End Sub

Private Sub ObjectControl_Deactivate()

SetObjectState
Set m_objContext = Nothing

End Sub
```

When we receive the `Activate` event, we call `GetObjectState` to get the object's current state. Conversely, when `Deactivate` is raised, we call `SetObjectState` to persist the state. Here's what `GetObjectState` looks like:

```vb
Private Sub GetObjectState()

On Error Resume Next

Dim bolExists As Boolean
Dim enmIsolationMode As LockModes
Dim enmReleaseMode As ReleaseModes
Dim objPropertyGroup As SharedPropertyGroup
Dim objPropertyGroupManager As MTxSpm.SharedPropertyGroupManager
Dim objSharedProperty As SharedProperty

m_lngWrapperPtr = ObjPtr(SafeRef(Me))

Set objPropertyGroupManager = m_objContext.CreateInstance("MTxSpm.SharedPropertyGroupManager.1")

If Not objPropertyGroupManager Is Nothing Then
  enmReleaseMode = Process
  enmIsolationMode = LockSetGet
  Set objPropertyGroup = objPropertyGroupManager.CreatePropertyGroup("SP" & CStr(m_lngWrapperPtr) & "_" & CStr(GetCurrentProcessId), enmIsolationMode, enmReleaseMode, bolExists)
  If Not objPropertyGroup Is Nothing Then
    If bolExists = True Then
      '  We can read the previous state.
      Set objSharedProperty = objPropertyGroup.CreateProperty("X", bolExists)
      If Not objSharedProperty Is Nothing Then
        m_dblX = CDbl(objSharedProperty.Value)
        Set objSharedProperty = Nothing
      End If
      Set objSharedProperty = objPropertyGroup.CreateProperty("Y", bolExists)
      If Not objSharedProperty Is Nothing Then
        m_dblY = CDbl(objSharedProperty.Value)
        Set objSharedProperty = Nothing
      End If
      Set objPropertyGroup = Nothing
    End If
  End If
  Set objPropertyGroupManager = Nothing
End If

End Sub
```

The first thing we do is try to create a property group based off of the context wrapper's pointer value and the process ID. Note that I pass in a reference to the current object to the `SafeRef` function to get the value. This will actually grab the context wrapper's pointer; if we did this:

```vb
m_lngWrapperPtr = ObjPtr(Me)
```

then we'd be getting a raw reference to the object without the context wrapper. The `GetCurrentProcessId` call is an API function that looks like this:

```vb
Private Declare Function GetCurrentProcessId Lib "kernel32" () As Long
```

If the property group was just created (we find this out by checking the value of `fExists`), we don't do anything. That's because there isn't any state to read from the property group - this is the first time that the object was created in this process. However, if it's already there, we read the values for `X` and `Y` out of the property group.

I won't show the `SetObjectState` method, because it's virtually identical to `GetObjectState`. There are two subtle differences, though. The first issue is related to storing the context wrapper's pointer value in `GetObjectState`. By the time `Deactivate` is raised, you don't have access to the context wrapper anymore. So if you try to get the pointer in `SetObjectState`, MTS suddenly goes quiet. A quick trip to the NT Event Log shows this:

(2026 - sorry, image is gone)

Informative, ain't it? After reading up a bit more on context wrappers and object deactivation, I realized that I needed to cache the pointer value in `GetObjectState`. Calling `SafeRef` on the object during deactivation wasn't making MTS very happy. The second issue is pretty mundane: During the save operation, you don't care about the preexistence of the property group. You always write the property values to the property group, so `fExists` isn't checked.

> One thing that occurred to me as I wrote this is, "Hey, you could use a database instead of the property group!" In fact, you could use any persistence mechanism you wanted to - flat files, relational databases, and registry entries - as long as the information remained after the object died. Feel free to experiment - it's not that difficult to create database tables that can store object state.

## Testing

For the sake of completeness, I created a test client to ensure that my design worked. Here's a snapshot of the form:

(2026 - sorry, image is gone)

It's not pretty, but it works. I won't go through the code here because it really doesn't do much other than assign and display the values of each object. However, you'll notice that the stateless object will always show 0 values in its' `X` and `Y` labels, while the objects with state will always show the assigned values. Furthermore, the *Get 2X* button retrieves the `X` value from the second object. This is to demonstrate that the two objects with state are storing their values in separate shared property groups.

## Conclusion

I'm going to end this article with a warning: Use this technique with caution. I demonstrated that you **could** make state management transparent to the client. That doesn't mean that you should. I mentioned that I've used objects that are passed by value over the network via byte arrays and are saved via a MTS persistence layer. I've found that technique to be very successful and rather snappy in terms of performance. In this article, our `CPointWithState` objects don't have a single method to get and set the object state. This is not a performance winner by any means. And if you decide to add the single `GetData` and `SetData` methods, you might as well succumb to what MTS really wants you to do in the first place.

Also, another issue to consider is performance. Using a shared property group is not cheap. In our example, having data from one point object sitting in memory isn't a big deal, but what happens if you have 20, or 200, or 2000 clients trying to create points? It starts to add up in a hurry. Furthermore, your class definitions will probably be far more complex that defining a point in Cartesian space, so the performance impact is even greater. Again, by using MTS as a persistence layer your clients can have objects with state and only store the state when desired.

OO purists don't like what MTS does with your objects. It turns them into reader/writer objects. The persistence is divorced from the object. Unfortunately, you don't have much of a choice in MTS. However, I don't see this as too big of a deal. As I have stated, using MTS as a persistence layer for your objects works very well in most circumstances, and you can still have non-transactional objects with state to work with. But if you have to hide the state management, you now know how you can accomplish this with the Shared Property Group components and context wrapper pointers.

> Published: 04.19.1998 12:00:00 AM CST