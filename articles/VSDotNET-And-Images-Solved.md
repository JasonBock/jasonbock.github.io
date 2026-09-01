---
title: VS .NET and Images - Solved
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# VS .NET and Images - Solved

After having a conversation with Jamie, we were able to figure out a way to get the project directory at runtime. I've updated the code example (I even updated the class names - I was so happy!) - it basically comes down to doing three things. First, get the location of the assembly in the constructor for a default directory value.

```c#
public DrumPicture()
{
  this.InitializeComponent();

  Assembly targetAssembly = (Assembly.GetEntryAssembly() != null ?
    Assembly.GetEntryAssembly() : Assembly.GetCallingAssembly());

  this.loadingDirectory = 
    Path.GetDirectoryName(targetAssembly.Location);
}
```

Next, override the `Site` property to determine if the control is in design-mode and, if it is, get the location of the project through the DTE:

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
      ProjectItem projectItem = 
        (ProjectItem)this.Site.GetService(
          typeof(EnvDTE.ProjectItem));

      if(projectItem != null)
      {
        this.loadingDirectory = 
          Path.GetDirectoryName(
            projectItem.ContainingProject.FullName);
      }
    }
  }
}
```

Finally, override `OnLoad()` and load the image there:

```c#
protected override void OnLoad(EventArgs e)
{
  base.OnLoad (e);
  this.drumPictureBox.Image = Image.FromFile(
    Path.Combine(this.loadingDirectory, "DrumsFront.jpg"));
  this.locationLabel.Text = this.loadingDirectory;
}
```

That's essentially it. Thanks again to Jamie for helping me out on this.

> Published: 02.01.2005 10:46:02 PM CST