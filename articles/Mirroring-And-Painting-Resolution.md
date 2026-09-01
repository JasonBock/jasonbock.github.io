---
title: Mirroring and Painting - Resolution
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Mirroring and Painting - Resolution

I opened up a support call with Microsoft on this issue. Unfortunately, my fears were confirmed - there is a bug in the .NET Framework when you mirror a control via `WS_EX_LAYOUTRTL` and the control is double-buffered (via a call to `SetStyle()`). The support guy said this bug was fixed in 2.0 (which I confirmed), but that doesn't help me out right now. However, he was able to give us a workaround - I've updated the ZIP file so you can look at it. Basically, what I needed to do is subclass any WinForm controls that I was using and turn double-buffering off. In other words, the labels that are on the form are now replaced with a `CustomLabel` control:

```c#
class CustomLabel : System.Windows.Forms.Label
{
  public CustomLabel() : base()
  {
    this.SetStyle(ControlStyles.DoubleBuffer, false);
  }
}
```

This gets rid of the black line effect. I still have a problem if I drag another window across the form (which the guy at MS told me would happen) as I get a bunch of vertical lines being drawn, but I may be able to get of that effect as well (I haven't delved too deeply into that issue yet and the application is designed to work as a kiosk view so having other windows dragged across it would be unusual). At least I now have something to work with. It's not perfect and it would require the entire code base to be updated with new custom WinForm controls (not hard, just a bit time-consuming) and all of the developers would have to remember to use these custom controls in the future. The client hasn't decided if it's worth the effort just yet, but they have options to exercise, and that's a good thing.

> Published: 01.28.2005 10:56:22 AM CST