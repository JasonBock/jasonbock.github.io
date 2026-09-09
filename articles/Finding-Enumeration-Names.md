---
title: Finding Enumeration Names
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Finding Enumeration Names

## Introduction

Recently I came across a problem that I thought was going to be somewhat tricky to solve, until I researched my own brain and remembered a nice component that comes with VB's installation. By using this component, I was able to retrieve the enumeration names for any enumeration value.

## What's The Problem?

When you make an enumeration in VB, you can easily find out what the name for a specific value is just by looking at the code. For example, if you had the following enumeration:

```vb
Private Enum MyEnum
  FirstValue = 1
  SecondValue
End Enum
```

you can do a simple search for `Private Enum MyEnum` and find out what the name is for the value equal to 1, which is `FirstValue`.

The problem becomes trickier with enumerations compiled into type libraries. Granted, VB's Object Library will display the names at design-time, but let's say that you were given the enumeration value at run-time. Just to show you that this does happen in the real world, here's the problem I ran into that motivates the whole discussion. I was recently playing around with the CDO Library to find out user information in Exchage. I knew that Exchange can store information like addresses, phone numbers, etc., but the way this is exposed is with a value called a property tag, which is stored in the `ID` property of each `Field` object that comes from the `Fields` collection of the `AddressEntry` object. Without getting into a long-winded discussion of CDO, the bottom line comes down to this: I'm given a property tag value that is a `CdoPropTags` enumeration value, and I'd like to be able to map that value to a descriptive string value. However, the `CdoPropTags` enumeration has 803 values. After futzing with OLEView and other tools, I couldn't find a easy way to get the name/value pairs to put them in a table format. Even if I could, that's a waste of time, because the type library has that information; why should I have to store it again?

## TypeLib Information Component to the Rescue

However, there's an easy way out of this. There's a component that comes with VB called the TypeLib Information component. The file is TLBINF32.DLL and installed in your System folder by default. This component can give you all sorts of information on any COM component, and can be used to make your own advanced Object Browser if you want. However, for our purposes, all we're going to use it to get the string name for an enumeration value.

Here's the function `GetEnumName`:

```vb
Private Function GetEnumName(EnumValue As Long, EnumName As String, TypeLibFile As String) As String

On Error GoTo Error_GetEnumName

Dim oTLIApp As New TLI.TLIApplication
Dim oTLI As TypeLibInfo
Dim oTL As TypeInfo
Dim oTLMember As MemberInfo

GetEnumName = "???"

Set oTLI = oTLIApp.TypeLibInfoFromFile(TypeLibFile)

For Each oTL In oTLI.TypeInfos
  If oTL.TypeKind = TKIND_ENUM And oTL.Name = EnumName Then
    '  Now we have to search each member to find the correct value.
    For Each oTLMember In oTL.Members
      If oTLMember.Value = EnumValue Then
        GetEnumName = oTLMember.Name
        Exit Function
      End If
    Next oTLMember
  End If
Next oTL

Exit Function

Error_GetEnumName:

GetEnumName = "Error"

End Function
```

Let's take a look at it in detail. `oTLIApp` is the root object that you use to get all the information out of a type library. There's a lot of ways to get type information, but I decided to use `TypeLibInfoFromFile`, which takes a file that is a TLB file or contains one (like COM DLLs and EXEs from VB do). Once that's done, you can go through all of the information in the type library. In this case, all I'm interested in is a specific enumeration, which is why I check the `TypeKind` and its' name. If we get a match, then I look through each `Member` in the `Members` collection and find the one with the value I'm looking for. If I find it, I return that enumeration name. Slick!

To show how valuable this is, take a look at some information I got out CDO pre-`GetEnumName`:

```
PROP TAG = 974061598 (???), PROP VALUE = "JBock"
PROP TAG = 805830720 (???), PROP VALUE = "10/01/1999 8:49:11 PM"
```

With the addition of a call to GetEnumName:

```vb
GetEnumName(oField.ID, "CdoPropTags", "cdo.dll")
```

I end up with:

```
PROP TAG = 974061598 (CdoPR_MHS_COMMON_NAME), PROP VALUE = "JBock"
PROP TAG = 805830720 (CdoPR_LAST_MODIFICATION_TIME), PROP VALUE = "10/01/1999 8:49:11 PM"
```

Now I know exactly what that date field corresponds to.

## Conclusion

I'd suggest playing with the TypeLib component. It can make you COM coding life much easier. In this case, I thought I'd be down a tricky registry path, but I ended up with a simple solution. There are all sorts of fun things you can do with TypeLib (like getting more information when you get the dreaded 429 error in VB).

> Published: 05.15.1998 12:00:00 AM CST