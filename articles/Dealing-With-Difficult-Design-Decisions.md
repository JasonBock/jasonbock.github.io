---
title: Dealing With Difficult Design Decisions
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Dealing With Difficult Design Decisions

## What is Wrong With This Code?

Language/Tool: VB 

```vb
Option Explicit

Private Const XCEED_ZIP_PROG_ID As String = "XceedSoftware.XceedZip.4"

Private WithEvents m_oZip As XceedZipLib.XceedZip
Private m_oZipContents As Collection

Private Sub Form_Initialize()
  Set m_oZip = CreateObject(XCEED_ZIP_PROG_ID)
End Sub

Private Sub GetZipInfo()
  On Error GoTo Error_GetZipInfo

  Dim X As Long

  lstZIPFiles.Clear
  Set m_oZipContents = New Collection

  m_oZip.ZipFilename = App.Path & "\ziptest.zip"
  m_oZip.ListZipContents

  For X = 1 To m_oZipContents.Count
    lstZIPFiles.AddItem m_oZipContents.Item(X)
  Next X

  Exit Sub

  Error_GetZipInfo:

  MsgBox Err.Number & " - " & Err.Description, _
    vbOKOnly + vbExclamation, _
    "GetZipInfo Error"
End Sub

Private Sub m_oZip_ListingFile(ByVal sFilename As String, _
  ByVal sComment As String, ...)

  m_oZipContents.Add sFilename
End Sub
```

> Note: `ListingFile()` contains a lot more arguments than what I've shown. I have deleted them as they're not needed for the topic at hand.  

Just over a year ago, I was on a project that required me to zip a file on the server side, and unzip the contents on the client side. The server side was written in Java, and that part was fairly easy to implement due to the compression classes that exist within the JDK. However, there's no such component that comes with a VB installation that handles ZIP files, so I had to shop around. I came across Xceed's Zip Compression Library COM server, and after using the trial version for a while, I determined that this was the component I should use. The documentation was good, the support was great, and it did everything I wanted it to. 

However, there was a design decision that was made in the `XceedZip` class that made my code kind of awkward. One part of my project required me to go through all of the files in the ZIP file. The `XceedZip` class has a way to do this through `ListZipContents()`, but there's a snag. When I first saw this method, I thought that an array or collection would be returned to me. Not so! By calling `ListZipContents()`, the object actually raises the `ListingFile()` event every time it finds a file in the ZIP file. The arguments of `ListingFile()` contain all of the information you'll need on a file contained in the ZIP file. 

So as you can see in the code snippet, I had to run through some small hoops to get this information such that I could use it in `GetZipInfo()`. First, I created a collection called `m_oZipContents` at the form level, and then I dimensioned `m_oZip` `WithEvents` so I could trap the `ListingFile()` event. I made sure I had a new collection before I called `ListZipContents()`, and every time the event was raised, I added the file information to the collection. Since all of the events would be fired before the next line of code in `GetZipInfo()` was executed, I knew the collection would contain all of the ZIP file information. 

OK, that worked. But I had to do this information gathering in a handful of forms, so I had to repeat this code over and over again (there is a way around this - I'll show you how in a moment). The thing that really confused me with this design is I couldn't see why anyone would want to get a collection of items via events. Why not have `ListZipContents()` just return a collection of files? Maybe the developer who created this component was used to how some information in the Windows OS is collected in the API (e.g. `EnumObjects()` - see Chapter 7 in my API book for a complete description of how this is used) and decided that events were the way to go. But I find it cumbersome at best. And some languages don't support COM events (e.g. some scripting languages) - what do you do in that situation? 

I'm not saying that this design has no use whatsoever. But it breaks up the code flow, and is easy to mess up if you have to duplicate it. 

## Possible Solution

There is a way to get around this, and it's a solution that could easily be made into a COM server such that it could be used in languages that don't support COM events. You create a class that I call an event trapper. All it does is trap the event for you, and collect the information in that event. From the client's perspective, it ends up being one method call. 

Here's what I did. I created a class called `CZipEventTrapper`. In that class, I defined the following two variables as private class members: 

```vb
Private WithEvents m_oZip As XceedZipLib.XceedZip
Private m_oZipContents As Collection

Then, I added a method called GetZipContents(): 

Public Function GetZipContents _
  (ZipObject As XceedZipLib.XceedZip) As Collection

  '  No error trapper - caller handles it.

  If Not ZipObject Is Nothing Then
    If ZipObject.ZipFilename <> vbNullString Then
      Set m_oZip = ZipObject
      Set m_oZipContents = New Collection
      ZipObject.ListZipContents
      '  Now the collection should be populated
      Set GetZipContents = m_oZipContents
      Set m_oZipContents = Nothing
    End If
  End If
End Function
```

The method takes a `XceedZip` object type and calls `ListZipContents()` on it. The event is captured in the class, so you don't need to declare your `XceedZip` object `WithEvents` anymore. Once the method is done, the client gets a collection back filled with ZIP file information. 

Now the client code looks like this: 

```vb
Private Sub GetZipInfoWithTrapper()
  On Error GoTo Error_GetZipInfoWithTrapper

  Dim oZip As XceedZipLib.XceedZip
  Dim oZipContents As Collection
  Dim oZipTrapper As New CZipEventTrapper
  Dim X As Long

  Set oZip = CreateObject(XCEED_ZIP_PROG_ID)

  lstZIPFiles.Clear
  oZip.ZipFilename = App.Path & "\ziptest.zip"

  Set oZipContents = oZipTrapper.GetZipContents(oZip)

  If Not oZipContents Is Nothing Then
    For X = 1 To oZipContents.Count
      lstZIPFiles.AddItem oZipContents.Item(X)
    Next X
  End If

  Exit Sub

  Error_GetZipInfoWithTrapper:

  MsgBox Err.Number & " - " & Err.Description, _
    vbOKOnly + vbExclamation, _
    "Error_GetZipInfoWithTrapper Error"
End Sub
```

Granted, it's a bit more code, but it's all within the same method, and I find that much easier to work with. Plus, I can use `CZipEventTrapper` in other forms and classes; I no longer have to repeat the original code I first showed. 

## Current Status

I hope that my article doesn't discourage you from using Xceed's components. I never had a problem using this component, and I found it very easy to use. I've recommended it to other developers, and I wouldn't hesitate to use their other components in the future if need be. Plus, once I had my event trapper in place, things became easy to manage. 

It's interesting to note that Xceed added a method to `XceedZip` called `GetZipContents()` that gives you a collection. Here's how the code looks when using this new method: 

```vb
Private Sub GetZipWithNewMethod()
  On Error GoTo Error_GetZipWithNewMethod

  Dim eZipRet As XceedZipLib.xcdError
  Dim oZip As XceedZipLib.XceedZip
  Dim oZipItem As XceedZipLib.XceedZipItem
  Dim oZipItems As XceedZipLib.XceedZipItems

  Set oZip = CreateObject(XCEED_ZIP_PROG_ID)

  lstZIPFiles.Clear
  oZip.ZipFilename = App.Path & "\ziptest.zip"

  eZipRet = oZip.GetZipContents(oZipItems, xcfCollection)

  If eZipRet = xerSuccess Then
    For Each oZipItem In oZipItems
      lstZIPFiles.AddItem oZipItem.FileName
    Next oZipItem
  End If

  Exit Sub

  Error_GetZipWithNewMethod:

  MsgBox Err.Number & " - " & Err.Description, _
    vbOKOnly + vbExclamation, _
    "Error_GetZipWithNewMethod Error"
End Sub
```

In fact, if you look at the documentation, you'll notice that this was added in their 4.2 version for the following reason: 
"A listing of a zip file's contents can now be obtained in the form of a collection object for increased performance and easy access from within Active Server Page scripts" 

Obviously, someone must've tried to use this in an environment where using COM events just wasn't going to be feasible to get ZIP file information, and did some complaining. Or someone with Xceed caught this limitation. Either way, I'm glad that this method was added, as it makes sense to me to have the information given to you right from the method call itself. 

## Summary

1. Review your design decisions; Both on your own and with others Using an iterative approach gives you the ability to change your designs before a release.
2. If you're in a situation where you can't change the component, find ways to get around the design, and see if you can generalize the solution such that you can reuse it later.

> Published: 05.25.2001