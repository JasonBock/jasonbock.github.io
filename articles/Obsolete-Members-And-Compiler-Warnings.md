---
title: Obsolete Members and Compiler Warnings
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Obsolete Members and Compiler Warnings

Yesterday at the Build Your Skills event, I got a question from an attendee about an issue he was having with the C# compiler. Take a look at the following code:

```c#
public static class UsingObsoleteStuff
{
  public static string Call()
  {
    return ConfigurationSettings.AppSettings["something"];
  }
}
```

His claim was that using VS 2005/C# 2.0, this would give a compiler warning because the `AppSettings` property is obsolete. However, when he upgraded to VS 2008/C# 3.0, the warning was never generated. But in VB, the warning still comes through. I didn't have a lot of time to look at the issue in detail, but I was intrigued, so when I got in to the office today, I dug into it a bit more. Here's what I found.

I created a new C# project in 2008. I turned the warning level to 4, had "treat warnings as errors" on, and put the `UsingObsoleteStuff` class in the project. I compiled it, and sure enough, no compiler error. Then I created an "identical" project in VB with this class:

```vb
Imports System.Configuration

Public Module UsingObsoleteStuff
  Public Function GetSetting() As String
    Return ConfigurationSettings.AppSettings("something")
  End Function
End Module
```

When I compiled this project ... the warning came up as an error.

```
'Public Shared ReadOnly Property AppSettings() As System.Collections.Specialized.NameValueCollection' is obsolete: 'This method is obsolete, it has been replaced by System.Configuration!System.Configuration.ConfigurationManager.AppSettings'.
```

This is error BC40000 from the VB compiler.

OK, so now I'm confused. The `AppSettings` property is definitely marked as `Obsolete`, so why isn't C# picking it up? I decided to create my own class with obsolete members:

```c#
public static class DeadCode
{
  [Obsolete("Dead!", false)]
  public static void DoNoCallMeAsWarning()
  {
  }

  [Obsolete("Dead!", true)]
  public static void DoNoCallMeAsError()
  {
  }
}
```

I changed `Call()` to this:

```c#
public static class UsingObsoleteStuff
{
  public static string Call()
  {
    DeadCode.DoNoCallMeAsError();
    DeadCode.DoNoCallMeAsWarning();
    return ConfigurationSettings.AppSettings["something"];
  }
}
```

And ran the compiler:

```
'CallingInCSharp.DeadCode.DoNoCallMeAsError()' is obsolete: 'Dead!

Warning as Error: 'CallingInCSharp.DeadCode.DoNoCallMeAsWarning()' is obsolete: 'Dead!'
```

In this case, I got two errors, as I was expecting. In C#, this error is CS0619.

Now I'm really confused. It seems like the C# compiler does notice when members are marked as obsolete, and it honors the `IsError` flag in the attribute, so why isn't it working with `AppSettings`?

On a whim, I changed the code to this:

```c#
public static class UsingObsoleteStuff
{
  public static string Call()
  {
    DeadCode.DoNoCallMeAsError();
    DeadCode.DoNoCallMeAsWarning();
    var settings = ConfigurationSettings.AppSettings;
    return settings["something"];
  }
}
```

And suddenly, C# sees the issue:

```
Warning as Error: 'System.Configuration.ConfigurationSettings.AppSettings' is obsolete: 'This method is obsolete, it has been replaced by System.Configuration!System.Configuration.ConfigurationManager.AppSettings'
```

Huh? This is really starting to befuddle me. My next though was to put the property in my own class:

```c#
public static class DeadCode
{
  [Obsolete("Dead!", false)]
  public static void DoNoCallMeAsWarning()
  {
  }

  [Obsolete("Dead!", true)]
  public static void DoNoCallMeAsError()
  {
  }

  [Obsolete("Don't use these settings!", true)]
  public static NameValueCollection Settings
  {
    get
    {
      return null;
    }
  }
}
```

And try to use it to see what would happen:

```c#
public static class UsingObsoleteStuff
{
  public static string Call()
  {
    DeadCode.DoNoCallMeAsError();
    DeadCode.DoNoCallMeAsWarning();
    return DeadCode.Settings["something"];
  }
}
```

Bzzt! Same as before - the compiler doesn't see it until I get the collection in one line and use it in another.

But let's try a different type for the property:

```c#
public static class DeadCode
{
  [Obsolete("Dead!", false)]
  public static void DoNoCallMeAsWarning()
  {
  }

  [Obsolete("Dead!", true)]
  public static void DoNoCallMeAsError()
  {
  }

  [Obsolete("Don't use this id!", true)]
  public static Guid Id
  {
    get
    {
      return Guid.NewGuid();
    }
  }

  [Obsolete("Don't use these settings!", true)]
  public static NameValueCollection Settings
  {
    get
    {
      return null;
    }
  }
}
```

And use that in the client code:

```c#
public static class UsingObsoleteStuff
{
  public static string Call()
  {
    DeadCode.DoNoCallMeAsError();
    DeadCode.DoNoCallMeAsWarning();
    return DeadCode.Id.ToString();
  }
}
```

Aha! Now the compiler catches it:

```
'CallingInCSharp.DeadCode.Id' is obsolete: 'Don't use this id!'
```

But why doesn't it work when the property type is a `NameValueCollection`?

I'm really not sure. But at this point, I remembered the guy said that this did work with C#'s 2.0 version of the compiler. So, when I got back from lunch, I decided to compile the code where it was using the indexer from `AppSettings` all in one line, and lo and behold, C# 2.0 does report it:

```
C:\Code>"C:\WINDOWS\Microsoft.NET\Framework\v2.0.50727\csc.exe" /target:library /out:Obsolete.dll /warn:4 /warnaserror+ *.cs
Microsoft (R) Visual C# 2005 Compiler version 8.00.50727.3053
for Microsoft (R) Windows (R) 2005 Framework version 2.0.50727
Copyright (C) Microsoft Corporation 2001-2005. All rights reserved.

UsingObsoleteStuff.cs(10,11): error CS0618: Warning as Error: 'System.Configuration.ConfigurationSettings.AppSettings'
        is obsolete: 'This method is obsolete, it has been replaced by
        System.Configuration!System.Configuration.ConfigurationManager.AppSettings'
```

Now, the last question I had is, is this fixed in the CTP for C# 4.0? I compiled the code against that build of 4.0, and unfortunately it isn't picking up the error either.

It's unfortunate that something seems to be amiss with C#'s 3.0 version (and still with the CTP 4.0 version) with respect to obsolete members. It's not completely borked as it does seem to find most of the cases, although I am curious why it doesn't work with the `NameValueCollection` case. Well, to be specific, it doesn't pick it up if all of the code is in one line. So that made me think, is there something different in the IL? I realize that the compiler is a black box and looking at the results may not tell me much but I thought it was worth looking at.

Here's what the IL looks like when I get the value all in one line (using a release build):

```
.method public hidebysig static string Call() cil managed
{
  .maxstack 8
  L_0000: call class [System]System.Collections.Specialized.NameValueCollection
    [System]System.Configuration.ConfigurationSettings::get_AppSettings()
  L_0005: ldstr "something"
  L_000a: callvirt instance string
    [System]System.Collections.Specialized.NameValueCollection::get_Item(string)
  L_000f: ret 
}
```

Here's what it looks like when I get the collection first, and then get the value:

```
.method public hidebysig static string Call() cil managed
{
  .maxstack 2
  .locals init (
    [0] class [System]System.Collections.Specialized.NameValueCollection settings)
  L_0000: call class [System]System.Collections.Specialized.NameValueCollection
    [System]System.Configuration.ConfigurationSettings::get_AppSettings()
  L_0005: stloc.0 
  L_0006: ldloc.0 
  L_0007: ldstr "something"
  L_000c: callvirt instance string
    [System]System.Collections.Specialized.NameValueCollection::get_Item(string)
  L_0011: ret 
}
```

There's definitely a difference, which wasn't too surprising as I'm using a local variable. But why this gives different compilation behaviors is beyond my sleuthing skills at the moment. I mean, in both cases, the code at line `L_0000` does the same exact thing and that's where it's using the obsolete member. There has to be something within the bowels of the compiler that treats these two cases differently, though.

At the end of the day, there really isn't a valid workaround that I can think of, other than writing a Code Analysis rule that goes through the entire assembly and finds any uses of obsolete stuff. Or write the code in VB!

> Published: 03.25.2009 10:32:07 AM CST