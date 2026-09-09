---
title: Resizing List View Columns in .NET
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Resizing List View Columns in .NET

If you want to make your list view controls resize the columns so their widths will be long enough to show the column text along with any row information in .NET, it's pretty easy, but the way to do it is a little odd. Take a look at the following application:

(2026: sorry, the image was lost :( )

The screen shot was taken after the *Populate* button was pressed. As you can see, the last entry is too long given the current width values of the columns.

What I wanted to do was to make the view a bit more intelligent than that. I knew there was a way to do this with straight Win32 programming - check out Randy Birch's article, "Autosizing ListView Columns via API". It shows you everything you need to get this to work in VB6. Of course, I was hoping there would be a far easier way to do it in .NET than to rely on API calls, and there is. The code underneath the `Width` = -2 button shows you how it works:

```vb
Private Sub btnWidthMinusTwo_Click(ByVal sender As System.Object, ByVal e As System.EventArgs) Handles btnWidthMinusTwo.Click
  Dim oHeader As ColumnHeader

  For Each oHeader In Me.lvwTest.Columns
    oHeader.Width = -2
  Next
End Sub
```

After you run the code, the list view will look like this:

![Resize Columns](https://jasonbock.net/images/Resize-Columns-1.png "Resize Columns")

Perfect! The only issue I have with this is that it wasn't obvious to me to change the `Width` property to -2. If you look at the SDK, it'll tell you that you can only do this at run-time (try setting that property to -2 in the *Designer* and you'll see what I mean). OK, but this just feels awkward, like it's a holdover to the Win32 API days. And, sure enough, it is. If you look at Birch's article, you'll see that the constant you need to adjust the column widths is called `LVSCW_AUTOSIZE_USEHEADER` - its' value is (you guessed it) -2. By the way, the reason I have a `Width` = -1 button was just a test of mine - -1 is another valid value according to the SDK (it's the value for `LVSCW_AUTOSIZE`). After clicking that, the screen will look like this:

![Resize Columns](https://jasonbock.net/images/Resize-Columns-2.png "Resize Columns")

This wasn't quite what I had in mind. This will only resize according to the text in the columns, so if there's nothing in the column (as is the case with my *Middle Name* column) it'll get scrunched.

My original hope was that there would be something along the lines of a property called `ColumnResizing` that would take an enumeration called `ColumnResizingBehavior` with the following values: `None`, `AutoSize`, and `AutoSizeUseHeader`. That way, when you'd set `ColumnResizing` to `ColumnResizingBehavior.AutoSizeUseHeader`, the `Width` property would be able to tell you what the new column width was. The way it works now, after you set `Width` to -2, you have no idea what the true width is because you'll get a -2 back when you retrieve `Width`'s value.

It's a small nit, and it's nice that it only takes a couple of lines of .NET code to pull it off. But it would've been nice to make this feature a bit more .NET/OO-oriented. I mean, setting a `Width` property to a negative value to make the column auto-size is not semantically correct in my book. Using a property is a better approach.

> Published: 03.20.2002 12:00:00 AM CST