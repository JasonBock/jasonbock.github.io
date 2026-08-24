---
title: ExceptionMessageBox
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# ExceptionMessageBox

As I was doing some spelunking this morning, I saw the `ExceptionMessageBox` assembly in the *References* dialog box in VS 2005. I have created my own UI components to display exception information, so I decided to check out what was in this assembly. It's pretty easy to use, as the following code demonstrates:

```c#
private void OnCreateExceptionClick(object sender, EventArgs e)
{
  try
  {
    this.BadArguments();
  }
  catch (ArgumentException exception)
  {
    ExceptionMessageBox box = new ExceptionMessageBox(exception);
    box.Show(this);
  }
}

private void BadArguments()
{
  throw new ArgumentException("Arguments?! We don't have no stinking arguments!!");
}
```

Here's what the dialog box looks like:

![ExceptionMessageBox](https://jasonbock.net/images/ExceptionMessageBox-1.png "ExceptionMessageBox")

There's a bunch of constructor overloads and properties you can set to customize the dialog box (e.g. change the title of the window, etc.). The two buttons at the lower-left allow you to copy the message text and show details of the exception. Here's what you get when you copy the message text.

```
TITLE: ExceptionTester
------------------------------

Arguments?! We don't have no stinking arguments!!

------------------------------
BUTTONS:

OK
------------------------------
```

Here's what the details window looks like:

![ExceptionMessageBox](https://jasonbock.net/images/ExceptionMessageBox-2.png "ExceptionMessageBox")

Frankly, I'm not sure what to make of this dialog box. You can only show it as a dialog box, and there are times where I've found it advantageous to show that information within the current UI. Showing a dialog box can disrupt the workflow of the user, so I like having the option of either showing a dialog box or embedding its UI content within my current control (my code lets you do both). The information you get when you copy the message, is, well, not that helpful. The UI isn't being misinformative in this case - it says it will copy the message, but I'd expect to get more information about the exception with a "copy to clipboard" mechanism. Finally, it's a little weird that this is in a `Microsoft.SqlServer` namespace. You'd think this is something that would be beneficial to any .NET WinForms application; why associate it with just SQL Server? It's not tied directly to SQL Server; I could use it just fine in a simple WinForms project. I just find it weird that it's buried in the namespace that it is. I'm not saying that I find this class useless. It has some benefits, just not as many as I was hoping for.

There so much in the .NET Framework and other Microsoft assemblies that it feels dizzying at times. I just came across this one by accident - there's a lot of other things in the *References* dialog box in VS 2005 that look appealing.

> Published: 01.21.2006 10:10:40 AM CST