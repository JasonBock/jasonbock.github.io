---
title: Using Reflection to Find RGB Values for a Color
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Using Reflection to Find RGB Values for a Color

Last night I was working on a new web site that I hope to release within 2 to 4 weeks. Right now I'm working on the graphics, and I wanted to use silver. Unfortunately, the graphics editor I'm using doesn't have a list of colors via well-known names, so I had to enter the RGB value manually. Now, .NET defines the `Silver` property on the `Color` class, but the SDK doesn't tell you what the RGB values are. Well, I could've written a small snippet to show the RGB values for that specific color, but I decided to generalize it a bit and use Reflection to list all of the color properties on the `Color` structure:

```c#
using System;
using System.Drawing;
using System.Reflection;
class ColorList
{
  [STAThread]static void Main(string[] args)
  {
    Type colorType = typeof(Color);
    foreach(PropertyInfo colorProperty in colorType.GetProperties())
    {
      if(colorProperty.PropertyType == colorType)
      {
        Color colorPropertyValue = (Color)colorProperty.GetValue(null, null);
        Console.WriteLine(
          "{0}, R={1}, G={2}, B={3}",
          colorPropertyValue,colorPropertyValue.R,
          colorPropertyValue.G,colorPropertyValue.B);
      }
    }
  }
}
```

When you run this, the console window will spit back a bunch of colors and their corresponding RGB values [1]. I'm sure there's an easier way to do this [2], but to me this is a good example of how you can use Reflection to query metadata and find the values you're looking for in a relatively straightforward fashion.

[1] For the interested reader, silver is `Color [Silver], R=192, G=192, B=192`.

[2] If you know of one, I'm all ears!

> Published: 07.27.2004 09:44:07 AM CST