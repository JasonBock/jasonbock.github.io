---
title: VS .NET and Images
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# VS .NET and Images

I think I need to get a lot of sleep tonight. I'm just shot today, and in a few weeks it's not going to get any better. Anyway, here's the situation I ran into today - hopefully someone with a clearer head can suggest something constructive.

For a lot of reasons that I won't get into here, the application I'm working on can load images from disk. I know, that doesn't sound very earth-shattering. What I'm getting at is that we don't store images as resources; rather, we keep them in files so it's easier to replace an image (doesn't require a compilation of any kind). Well, so far, so good. When the application runs, we get the directory name of where the code is executing and load the image from there (it's a bit more complicated than that due to localization issues, but that explanation is sufficent for now). We have a post-build step that copies the image files from the project directory (note that the images are included as project items) into the appropriate *bin* directory, so that's taken care of as well.

The problem occurs when I have a control that loads an image from disk and puts it into a `PictureBox` at design time. In other words, I have a method called `LoadDrumPicture()` that's called from the constructor after `InitializeComponent()` does its magic, but that method fails in VS .NET. Why, may you ask? I'm glad you did! The issue is that VS .NET does some funky things with your referenced assemblies: it puts them into a directory under *Documents and Settings*, something like this:

> C:\Documents and Settings\jasonb\Application Data\Microsoft\VisualStudio\7.1\ProjectAssemblies\40q-cjfi01

YMMV as the folder names will be different on your machine, but you get the idea. Of course, my image files aren't there, so I get a `FileNotFoundException` and I'm SOL. I've been able to get around this for now by catching that exception, but that's not a good long-term solution. I've thought about using the Design-Time Environment objects, but from what I've seen I won't be able to determine if the executing code is running under the active project that DTE reports.

Essentially, I need some way to find out where my image files are at design-time, or I need to copy those images into that "funky" directory that VS .NET makes. An add-in is a possiblity, but then everyone would have to use that add-in and I'd like to avoid it if possible. I've posted a small solution that illustrates the problem, which you can get here. If you open `Form1` (that should show how tired I am - I didn't care that the new items had the default number scheme in their names), you won't see anything on it, and you'll have the following error in your Task List:

```
An exception occurred while trying to create an instance of ImagesAndProjects.UserControl1. The exception was "c:\windows\microsoft.net\framework\v1.1.4322\DrumsFront.jpg".
```

Note that the error message says that the executing directory is the Framework's directory and not in a folder under *Documents and Settings*, but that's because the user control is in the project itself; it's not a reference assembly. It's the same problem, though - the image is in the project's directory and at design-time I should go there for images. If you run the project, you'll see the user control with the image, but if you try to drop that user control on the form, it won't work. Basically, I commented out the code that loads the image, dropped the neutered control on the form, compiled, and then added the code back in. So it's still on the form, but it won't show up in the Designer.

It's not a big deal, but it is annoying. Any suggestions are appreciated.

> Published: 02.01.2005 03:05:29 PM CST