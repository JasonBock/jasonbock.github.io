---
title: Handling Events With Delegates in VB.NET
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Handling Events With Delegates in VB.NET

One of the reasons I like .NET is how delegates makes some tasks easier. Here's the code:

```vb
Private Sub InformationChanged(ByVal sender As Object, ByVal e As System.EventArgs) _
  Handles txtLastName.TextChanged, txtFirstName.TextChanged, _
  txtMiddleName.TextChanged, txtSuffix.TextChanged

  Me.m_HasChanged = True

End Sub
```

The reason this is so cool is that I can track UI changes with one method. In VB6 days, you'd have a bunch of `Change()` event handlers. No need for that anymore! It's a simple change, but I like the fact that I can do this with no pain whatsoever.

I guess it's the small things that sometimes make a difference.

> Published: 07.03.2002 01:31:00 PM CST