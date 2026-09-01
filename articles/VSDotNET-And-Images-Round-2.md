---
title: VS .NET and Images - Round 2
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# VS .NET and Images - Round 2

Just when I think I've solved something, I run into another issue that I didn't anticipate. Fortunately, there's a solution to it, but now I'm paranoid - what else am I screwing with?

Here's the problem. My `DrumPicture` control works find at design-time and runtime when it's in another control, like `MainForm`. However, if `DrumPicture` is in another control like `HostingDrumPicture`, and then `HostingDrumPicture` is put within **another** control (`HostingHostingDrumPicture`), then all hell breaks loose. For some reason, the `Site` property is no longer being set in `DrumPicture` when its parent is no longer the one being designed, and I'm back to the issue of not being able to find where the project is located. There's a simple workaround to this, though:

```c#
protected override void OnLoad(EventArgs e)
{
  base.OnLoad (e);            

  if(this.Site == null)
  {
    ISite site = null;
        
    Control parent = this.Parent;

    while(parent != null)
    {
      if(parent.Site != null)
      {
        site = parent.Site;
        parent = null;
      }
      else
      {
        parent = parent.Parent;
      }
    }

    if(site != null)
    {
      ProjectItem projectItem = 
        (ProjectItem)site.GetService(typeof(EnvDTE.ProjectItem));

      if(projectItem != null)
      {
        this.loadingDirectory = 
          Path.GetDirectoryName(projectItem.ContainingProject.FullName);
      }
    }
  }

  this.drumPictureBox.Image = Image.FromFile(
    Path.Combine(this.loadingDirectory, "DrumsFront.jpg"));
  this.locationLabel.Text = this.loadingDirectory;
}
```

Basically, I check the `Site` property of all my parents in `OnLoad()`. This seems to fix the problem I found - I've update the code drop to reflect the fix.

By the way, if you really want to make the lives of your fellow WinForms developers a living hell, add the following code to your custom user control:

```c#
public override ISite Site
{
  get
  {
    return base.Site;
  }
  set
  {
    base.Site = value;

    if(base.Site != null)
    {
      foreach(Control childControl in this.Controls)
      {
        childControl.Site = value;
      }
    }
  }
}
```

When this control is dropped onto another custom control, you'll (eventually) see the following error pop up:

```
A circular control reference has been made. A control cannot be owned or parented to itself.
```

It's hard to get this error to pop up - you need to host the control into other controls that host it in other controls, but eventually it pops up. The code in `InitializeComponent()` is where the problem manifests itself:

```c#
private void InitializeComponent()
{
  this.drumPicture1 = new ImagesAndProjects.DrumPicture();
  this.drumPicture1.SuspendLayout();
  this.SuspendLayout();
  // 
  // drumPicture1
  // 
  this.drumPicture1.Controls.Add(this.drumPicture1);
  this.drumPicture1.Controls.Add(this.drumPicture1);
  this.drumPicture1.Location = new System.Drawing.Point(104, 160);
  this.drumPicture1.Name = "drumPicture1";
  this.drumPicture1.Size = new System.Drawing.Size(512, 408);
  this.drumPicture1.TabIndex = 0;
  // 
  // HostingDrumControl
  // 
  this.Controls.Add(this.drumPicture1);
  this.Name = "HostingDrumControl";
  this.Size = new System.Drawing.Size(544, 512);
  this.drumPicture1.ResumeLayout(false);
  this.ResumeLayout(false);
}
```

As you can see, the control is trying to contain itself, and that's not good. So don't try to set the `Site` value to child controls - the VS .NET code generation piece will slap you silly for that.

> Published: 02.04.2005 11:17:42 AM CST