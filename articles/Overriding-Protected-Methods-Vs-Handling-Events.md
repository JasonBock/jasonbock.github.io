---
title: Overriding Protected Methods Vs. Handling Events
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Overriding Protected Methods Vs. Handling Events

Today I started at my new client. It's the typical "scamble-and-get-access-to-network-resources-and-setup-the-box" day. So far, it's going well, but I got to a point where I'm stuck because I don't have access to the source code in CVS, so while I'm waiting on that I thought I'd delve into something that I just noticed this morning.

I was reviewing some material on ASP .NET 2.0 yesterday, and I noticed something in a code example that I've seen before and, it kind of annoys me, but I never stopped and thought about the reasoning behind the design. Whenever you create a new page, VS .NET will generate code that hooks the `Load` event with a private `Page_Load` method:

```c#
/// <summary>
/// Required method for Designer support - do not modify
/// the contents of this method with the code editor.
/// </summary>
private void InitializeComponent()
{    
  this.Load += new System.EventHandler(this.Page_Load);
}

private void Page_Load(object sender, System.EventArgs e) {}
```

Now, there's a virtual method called `OnLoad()` that, when you take a look at its implementation via Reflector, it really isn't doing much:

```c#
protected virtual void OnLoad(EventArgs e)
{
  if (this._events != null)
  {
    EventHandler handler1 = this._events[Control.EventLoad] as EventHandler;
    if (handler1 != null)
    {
      handler1(this, e);
    }
  }
}
```

Basically, all it's doing is checking to see if anyone has registered for the `EventLoad` event, and if they have, raise the event. But the implementation of `OnLoad()` isn't what bothers me. What bothers me is, why doesn't the code generator just override `OnLoad()` instead of subscribing to an event?

Part of the problem may be that it's too easy for the developer to whack the call to the base class' implementation, which, in this case, is important, because if other objects have subscribed to the event, they wouldn't get the notification that the page has loaded. Of course, a developer can just go into the generated code and take out the event handling hook-up code line and mess up their lives harshly, but this code isn't generally "seen" by the developer as it's tucked away in a code region. It's still easy to find, though, but in 2.0 that's all put into a partial class anyway so it makes it harder for the developer to make their code uneventful (pun intended).

It just feels odd to catch an event that the object itself is raising. Events generally feel to me to be a way to notify consumers of the object [1]. If an action occurs in an object, it should have an easy way to let itself know that it occurred. In the case of the `Page` class, it does ... but for some reason it decides to go with the event approach. Of course, I'm sure I'm missing the obvious as to why things are done in ASP.NET they way they are. Just something I've noticed.

[1] I realize events can be static and then would be part of the class, not an object. I'm just talking in general terms here ...

> Published: 04.28.2005 11:24:15 AM CST