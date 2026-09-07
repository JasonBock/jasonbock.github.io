---
title: When Return Values Go Bad
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# When Return Values Go Bad

I mentioned a while ago that I was working on a theming architecture for WinForms. A couple of weeks ago, I added theme inheritance and I simplified the stylesheet structure, so now we have something that looks like this:

```xml
<Themes>
  <Theme name="DarkGreenControl">
    <BackColor value="17, 134, 23"/>
  </Theme>
  <Theme name="DarkGreenPanel" baseTheme="DarkGreenControl" 
    controlType="System.Windows.Forms.Panel, System.Windows.Forms, 
    Version=1.0.5000.0, Culture=neutral, PublicKeyToken=b77a5c561934e089">
    <ForeColor value="White"/>
    <Font value="Arial, 8.25pt"/>
  </Theme>
</Themes>
```

Note: Sorry, I can't post the code. This is an in-house project, and I can't release the source code as-is. But it's not too hard to recreate what I've done, and I'm trying to see if there is a way to release the code if enough people are interested in seeing it. But, back to the issue at hand ...

As you can see from the XML snippet, we have a theme called `DarkGreenControl`. This theme can be applied to any object that descends from `Control`. Contrast this with `DarkGreenPanel`, which can only be applied to `Panel`-based objects. Furthermore, `DarkGreenPanel` inherits the `BackColor` value when the properties are iterated from this theme. This is really nice because now I can define base themes and build up my themes accordingly, so if I want to change the dark green color to something that's a lighter shade of green, I can change `DarkGreenControl` and all of the other themes pick it up.

While there's a fair amount of code to parse the XML file and catch nasty issues like circular inheritance, the thing I want to share was a bug that took me by surprise when I started to apply some themes in my application. If I loaded a control where `DarkGreenControl` was applied, it had the dark green background. Then, if I loaded a control that used `DarkGreenPanel`, it had a dark green background along with white text and an Arial font. However, if loaded another control that used `DarkGreenControl`, its' text was now white! Essentially, base themes were picking up properties from themes that inherited from theme, which is completely wrong.

The theming architecture has unit tests, but unfortunately they weren't picking up this error. So, after some digging around, I finally found it. It was in the `Properties` property of our `Theme` class:

```vb
Public ReadOnly Property Properties(ByVal fullPropertyList As Boolean) As ThemePropertyCollection
  Get
    Dim retVal As ThemePropertyCollection

    If fullPropertyList = False Then
      retVal = _properties
    Else
      If IsNothing(_baseTheme) = True Then
        retVal = _properties
      Else
        retVal = _baseTheme.Properties(True)

        For Each subClassProperty As ThemeProperty In _properties.Values
          retVal.Item(subClassProperty.Name) = subClassProperty
        Next
      End If
    End If

    Return retVal
  End Get
End Property
```

Basically, what this getter does is return all of the properties for the given theme. The client can request to get only those properties defined on the theme itself (i.e. `fullPropertyList = False`), or, s/he can get the inherited properties as well (i.e. `fullPropertyList = True`). Unfortunately, I got too cute with my implementation. Basically, I get the list of properties from the base theme if there is one (i.e. `IsNothing(_baseTheme) = False`, where `_baseTheme` is an instance field of type `Theme`), and add the properties from the current them to this list. The problem is that the return value is a reference to the theme's internal field! So when I was adding the child theme's properties, I was adding them to the base theme's property list as well!

**D'OH!**

The problem was easy to fix:

```vb
Public ReadOnly Property Properties(ByVal fullPropertyList As Boolean) As ThemePropertyCollection
  Get
    Dim retVal As ThemePropertyCollection

    If fullPropertyList = False Then
      retVal = _properties
    Else
      If IsNothing(_baseTheme) = True Then
        retVal = _properties
      Else
        retVal = New ThemePropertyCollection

        Dim baseProperties As ThemePropertyCollection = _baseTheme.Properties(True)

        For Each baseProperty As ThemeProperty In baseProperties.Values
          retVal.Item(baseProperty.Name) = baseProperty
        Next

        For Each subClassProperty As ThemeProperty In _properties.Values
          retVal.Item(subClassProperty.Name) = subClassProperty
        Next
      End If
    End If

    Return retVal
  End Get
End Property
```

I basically copy the base theme's properties into a new list, and then add the child theme's properties to that list as well. Now the base theme is left unmolested.

The moral of the story is, you can break encapsulation by returning a reference to your private fields from a method call if you do something stupid like I did. And what's worse is that I knew this, but I still coded it wrong. Oh, well. Rest assured, I added a unit test so I'll know if I accidentally re-introduce this bug.

> Published: 10.15.2003 12:39:08 PM CST