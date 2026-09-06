---
title: Mirroring in WinForms and Owner-Drawn String: SOLVED (With Code)
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Mirroring in WinForms and Owner-Drawn String: SOLVED (With Code)

When I stated that I solved this problem, I didn't show the code. The reason was that the code is rather hackish, but for the interested subscriber to this blog here's the code in all of its glory (just append `OnPaint()` from the original code sample with this code):

```c#
SizeF flippedMessageSize = e.Graphics.MeasureString(message, font);
RectangleF flippedMessageArea = new RectangleF(8, 24, this.Width - 24, 24);
Bitmap flippedImage = new Bitmap((int)flippedMessageSize.Width, 
  (int)flippedMessageSize.Height);

using(Graphics flippedGraphics = Graphics.FromImage(flippedImage))
{
  using(Brush brush = new SolidBrush(Color.DarkBlue), 
    fillBrush = new SolidBrush(Color.LightGreen))
  {
    flippedGraphics.FillRectangle(fillBrush, 0, 0, 
      flippedImage.Width, flippedImage.Height);
    flippedGraphics.DrawString(message, font, brush, 0, 0);
  }
}

using(Brush brush = new SolidBrush(Color.DarkBlue),
  fillBrush = new SolidBrush(Color.LightGreen))
{
  RectangleF messageArea = new RectangleF(8, 24, this.Width - 24, 24);

  e.Graphics.FillRectangle(fillBrush, messageArea);
  e.Graphics.DrawString(message, font, brush, messageArea);
}

flippedImage.RotateFlip(RotateFlipType.RotateNoneFlipX);
e.Graphics.DrawImage(flippedImage, 8, 24);
```

Again, ugly, but it does the job. I could clean it up, put it into a custom control and have people use that in their code, but to make a long story short, it's better for the developer who is currently drawing strings in one section of the code to use a Label control. However, I'm glad we caught this in our initial tests as we now know what happens when we try to mirror an application with owner-drawn code.

> Published: 12.09.2004 09:42:51 AM CST