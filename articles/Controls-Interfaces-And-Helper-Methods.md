---
title: Controls, Interfaces, and Helper Methods
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Controls, Interfaces, and Helper Methods

I just ran into an interesting design decision today (at least it was interesting to me. You may end up going into a coma due to boredom after reading this post - you have been warned ;) ). Here's the situation. I wanted all of the common WinForms controls (like `TreeView`, `Panel`, and `Form`, just to name a few) to have some common "tweaking". Let's say this common behavior can be encapsulated in a property called `Tweak` [1]. Unfortunately, I can't "share" this code across these control because I have no access to a common ancestor in the inheritance tree, so what I did is create an interface called `ITweaked` that has the `Tweaked` property, and I create custom versions of the WinForms controls (like `TweakedPanel`) that implement `ITweaked`. It looks like this:

```c#
public interface ITweaked
{
  bool Tweaked
  {
    get;
    set;
  }
}

public TweakedPanel : Panel, ITweaked
{
  public bool Tweaked
  {
    // Tweak the panel...
  }
}
```

Now, what I do in `Tweaked` is a little different for each custom control, but whether I created a helper class that my custom controls all use or if I use the "copy-and-paste" technique is not important to discuss right now. What is important is that, when the tweaking is done (which happens in the setter), I have to tweak the control's child controls, and this code can be generalized into a helper method.

This is where it gets interesting, because how the child control is tweaked is dependent on the fact if the child control implement `ITweaked` or not. Furthermore, I only want objects passed into this method that are of type `Control` and implement `ITweaked`. Here's what I did - I'll explain why I did it this way in a moment:

```c#
private static void UpdateTweaking(ITweakedControl tweakedControl)
{
  Control control = tweakedControl as Control;

  if(control != null)
  {
    foreach(Control childControl in tweakedControl.Controls)
    {
      ITweakedControl childTweakedControl = childControl as ITweakedControl;

      if(childTweakedControl != null)
      {
        childTweakedControl.Tweaked = tweakedControl.Tweaked;
      }
      else
      {
        if(tweakedControl.Tweaked == true)
        {
          //  Handle the true case...
        }
        else
        {
          //  Handle the false case...
        }
      }
    }
  }
}
```

What I really want is a way to constrain the `tweakedControl` paramater such that I know it's of type `Control` and that it implements `ITweakedControl`. There isn't a way to do that in one shot, so I have to make a choice: is it more important that the incoming type is a `Control` or implements `ITweakedControl`? As you can see, I think it's the latter. The reason is that a developer will have to make a conscious choice to use this interface, so it's more likely that `UpdateTweaking` will be called by a developer who has created a tweaked control. However, I still need to make sure I have an object of type `Control`, which is what the first line of code does.

Again, this isn't an earth-shattering problem, but I found it interesting!

[1] As you can guess, code naming has been changed to protect the innocent.

> Published: 12.07.2004 02:58:44 PM CST