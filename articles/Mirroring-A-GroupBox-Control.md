---
title: Mirroring a GroupBox Control
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Mirroring a GroupBox Control

Yesterday afternoon I ran into a small quirk with the `GroupBox` control. Because I couldn't solve it before I left for the day, it put me in a somewhat sour mood (which is probably why I had little patience for technology). Anyway, I'm going to re-tackle the problem, and I'll add notes to this blog entry as I work through the issues. Hopefully someone out there has a solution.

Currently in .NET 1.x, there are a number of controls that do not mirror correctly when you set the `RightToLeft` property to `Yes`. This article explains the problem (along with a solution) in detail, and this link says that a `GroupBox` does not require mirroring. Really? Here's a screen shot of a `GroupBox` on a form that is mirrored:

![Mirroring a GroupBox](https://jasonbock.net/images/Mirroring-GroupBox-1.png "Mirroring a GroupBox")

As you can see, the form is mirrored correctly, the label and text box outside of the group box are mirrored, the text for the group box is mirrored correctly as well, and the label and text box are right-to-left, but the controls within the group box are not mirrored. Without getting into the guts of my code, whenever a control is mirrored, it traverses its child controls and either sets the `RightToLeft` property to match the current state of the parent's mirroring, or, if the child control supports mirroring, it will mirror the child control to match the parent control.

So, what I tried to do is create a `MirroredGroupBox` control that acts like the mirroring behavior of a mirrored `Panel` - that is, the `ExStyle` property of the `CreateParams` property is OR'ed with `WS_EX_LAYOUTRTL` and `WS_EX_NOINHERITLAYOUT`. Here's what happens:

![Mirroring a GroupBox](https://jasonbock.net/images/Mirroring-GroupBox-2.png "Mirroring a GroupBox")

Now the controls within the mirrored group box are mirrored correctly, but the text of the group box itself is backwards. Taking off `WS_EX_NOINHERITLAYOUT` makes no difference - the results are the same.

This is where I started to get frustrated, but then I thought of a way around this. The reason I found out about this problem in the first place is that a configuration tool being used at the client lays out their controls so the main application can be configured in all sorts of ways (it's really a cool thing to see, but it's not relevant to the conversation at hand). When the application is run in "mirrored" mode, controls with a group box were being mirrored correctly. But, when we added mirroring support to the configuration tool, we didn't see the contained controls being mirrored in design mode. What we figured out is that the configurators were putting a panel within the group box, and at run-time this was being rendered as a MirroredPanel, so everything worked out correctly. So, I though, couldn't I just add a hidden `MirroredPanel` contained within `MirroredGroupBox`, and then all would be well with the world?

It's not that easy.

The problem is in the VS .NET Designer for forms and controls (or, really, any designer for that matter). When the developer drops a control onto the mirrored group box, it wants to add the new control to the `MirroredGroupBox`'s `Controls` collection. Where it should **really** go is in the hidden `MirroredPanel`'s `Controls` collection. Otherwise, the added child controls and the hidden panel are in the same `Controls` collection and things end up looking goofy as hell. So, I thought, why don't I override `OnControlAdded` in `MirroredGroupBox` to remove the controls from its' collection and add it to the hidden `MirroredPanel`'s collection? Well, that kind of sucks too. Basically, the Designer gets all confused about where the controls should be layed out, and plus, once the control is added to the hidden panel, you can't "grab" it to move it around. Therefore, this option doesn't seem like a good one either.

So here I am, stuck with a `GroupBox` control that I can't seem to mirror properly. If you have any suggestions to get around this issue, please let me know. Thanks!

> Published: 12.21.2004 10:57:53 AM CST