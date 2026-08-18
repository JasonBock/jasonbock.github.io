---
title: Interesting Exception Results: Throwing Exceptions From the Finally Block
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Interesting Exception Results: Throwing Exceptions From the Finally Block

Note: I'm using .NET 3.5/VS 2008.

This code:

```c#
using System;

namespace ConsoleApplication1
{
  class Program
  {
    static void Main(string[] args)
    {
      try
      {
        Program.CallMe();
      }
      catch(Exception e)
      {
        Console.Out.WriteLine(e.GetType().FullName);
      }
    }

    private static void CallMe()
    {
      try
      {
        throw new NotSupportedException();
      }
      finally
      {
        throw new NotImplementedException();
      }
    }
  }
}
```

produces this:

```
System.NotImplementedException

Press any key to continue . . .
```

But this code:

```c#
using System;

namespace ConsoleApplication1
{
  class Program
  {
    static void Main(string[] args)
    {
      Program.CallMe();
    }

    private static void CallMe()
    {
      try
      {
        throw new NotSupportedException();
      }
      finally
      {
        throw new NotImplementedException();
      }
    }
  }
}
```

produces this:

```
Unhandled Exception: System.NotSupportedException: Specified method is not supported.
   at ConsoleApplication1.Program.CallMe() in C:\Documents and Settings\jasonb\My Documents\Visual Studio 2008\Projects\ConsoleApplication1\Program.cs:line 16
   at ConsoleApplication1.Program.Main(String[] args) in C:\Documents and Settings\jasonb\My Documents\Visual Studio 2008\Projects\ConsoleApplication1\Program.cs:line 9

Unhandled Exception: System.NotImplementedException: The method or operation is not implemented.
   at ConsoleApplication1.Program.CallMe() in C:\Documents and Settings\jasonb\My Documents\Visual Studio 2008\Projects\ConsoleApplication1\Program.cs:line 20
   at ConsoleApplication1.Program.Main(String[] args) in C:\Documents and Settings\jasonb\My Documents\Visual Studio 2008\Projects\ConsoleApplication1\Program.cs:line 9

Press any key to continue . . .
```

That was very ... odd (same behavior in VB). Just why is the console window showing 2 exceptions? I'm not saying that the caller is getting 2 exceptions, but ... that output is just strange. In a WinForms version of this:

```c#
namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      InitializeComponent();
      Application.ThreadException += (sender, e) =>
        this.textBox1.Text += e.Exception.GetType().FullName;
    }

    private void button1_Click(object sender, EventArgs e)
    {
      this.CallMe();
    }

    private void CallMe()
    {
      try
      {
        throw new NotSupportedException();
      }
      finally
      {
        throw new NotImplementedException();
      }
    }
  }
}
```

I only see one exception in the text box. So ... why does the console show 2? I'm sure there's an easy explanation but I have to go to bed.

> Published: 04.18.2008 12:44:28 AM CST