---
title: Seek And Ye Shall Find ... Eventually
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Seek And Ye Shall Find ... Eventually

## What is Wrong With This Code? 

Language: C# 

```c#
class Win32Test
{
  private const int FORMAT_MESSAGE_FROM_SYSTEM = 0x00001000;
  private const int FORMAT_MESSAGE_ALLOCATE_BUFFER = 0x00000100;

  [DllImport("kernel32.dll", CharSet=CharSet.Auto)]
  static extern int FormatMessage(int Flags,
    ref IntPtr MessageSource, 
    int MessageID, int LanguageID, ref IntPtr Buffer, 
    int BufferSize, int Arguments);

  [STAThread]
  static void Main(string[] args)
  {
    Console.WriteLine("Please enter in the Win32 error code.");

    try
    {
      int win32ErrorCode = Int32.Parse(Console.ReadLine());

      try
      {
        CreateAPIException(win32ErrorCode);
      }
      catch(Exception e)
      {
        Console.WriteLine(e.Message);
      }
    }
    catch(Exception e)
    {
      Console.WriteLine(e.Message);
    }
  }

  private static void CreateAPIException(int ErrorCode)
  {
    IntPtr bufferPtr = IntPtr.Zero;
    IntPtr messageSource = IntPtr.Zero;

    int retVal = FormatMessage(
      FORMAT_MESSAGE_FROM_SYSTEM | 
	    FORMAT_MESSAGE_ALLOCATE_BUFFER, 
      ref messageSource, 
      ErrorCode, 0, ref bufferPtr, 1, 0);

    if(retVal != 0)
    {
      throw new Exception(
	    Marshal.PtrToStringAuto(bufferPtr, retVal));
    }
  }
}
```

In Chapter 5 of my book, ".NET Security", I show the user how to do Windows impersonation through the `LogonUser()` API call. Now, if you've done Win32 programming before, you know that you should always check the return value as it will indicate if the method was successful or not. In the case of `LogonUser()`, a return value of zero indicates something went wrong. If that happens, you should call `GetLastError()` to get the error code, which can be translated into a more meaningful description with `FormatMessage()`. (Technically, in .NET you should call `GetLastWin32Error()` on the `Marshall` class instead of `GetLastError()` - look it up in the .NET SDK for the reason why). The code snippet above doesn't show how to use `LogonUser()` - I'm only interested in showing how the translation from an error code (which is given as input to the console in this example) to a description is done. 

First, you need to create a P/Invoke to `FormatMessage()`, using the `DllImport` attribute along with the `extern` keyword to tell the C# compiler you're going to wander off into unmanaged, native land for a while. You can look up the complete description for `FormatMessage()` in the Platform SDK (the function can get rather confusing as it's overly flexible with what it can do), but all I want to do is take an error code and get the description back. To do this, I set `Flags` equal to `FORMAT_MESSAGE_FROM_SYSTEM` and `FORMAT_MESSAGE_ALLOCATE_BUFFER` ORed together. The first flag states that I want to get a description based on the value of `MessageID`, and the second flag states that `FormatMessage()` should allocate the string buffer that `Buffer` points to. 

Now, once the method is done, I check the return value. If it's not zero, then the return value specifies how many characters are in the buffer pointed at by `bufferPtr`. Fortunately, .NET provides the `PtrToStringAuto()` method on the `Marshal` class to make the translation from a pointer to a string painless. Once the string is obtained, I create a new `Exception` object, passing in the description in its constructor, and throw it to the caller. 

## That's Nice, But ...

So what's wrong with the code? As the code stands, it works just fine (at least I hope you don't find anything wrong with it!). If you run the code and pass in 5, you should get back "Access is denied.". This matches up with what is shown in the "System Error Codes" article in the Platform SDK. In fact, I ended up using this technique on a couple of .NET projects where I was forced to use a P/Invoke and I wanted to report possible error conditions. 

The problem is that a P/Invoke to `FormatMessage()` is completely unnecessary. The .NET Framework already provides an `Exception`-based class called `Win32Exception` that does the code-to-description translation (I'll show you how it works in a moment). The sad thing is that it's not too hard to find in the .NET SDK. Just type "Win32" in the index box and you'll see it in the list 4 items down from the top. 

The reason I didn't find `Win32Exception` the first time is that I looked up `Exception` in the .NET Framework SDK, and clicked on the *Derived classes* link. After doing a quick search, I realized that it would take some time to find a class that handled Win32 error codes, so I gave up. In fact, trying to find `Win32Exception` using this searching technique is a bad idea, as its inheritance tree is fairly deep (`System.Object` to `System.Exception` to `System.SystemException` to `System.Runtime.InteropServices.ExternalException` to `System.ComponentModel.Win32Exception`). But if I had just typed in "Win32" I would have easily found `Win32Exception`. D'oh! 

I'm not sure why I didn't take that approach in the first place. Maybe my brain was thinking down the classic Win32 path and I had to get the chapter done, so once the "Derived classes" link approach didn't work, I gave up. Later on, I thought, "You know, I should create a class called `Win32Exception` that wraps the `FormatMessage()` junk." As soon as that idea came to mind, I wondered if the .NET Framework already had a class called that (or something similar to it). You can probably imagine how dumb I felt when I found it. 

## Possible Solution

This one's easy. Just dump the `FormatMessage()` P/Invoke along with the two related constants and replace it with the `Win32Exception` class: 

```c#
class Win32Test
{
  [STAThread]
  static void Main(string[] args)
  {
    Console.WriteLine("Please enter in the Win32 error code.");

    try
    {
      int win32ErrorCode = Int32.Parse(Console.ReadLine());

      try
      {
        throw new Win32Exception(win32ErrorCode);
      }
      catch(Exception e)
      {
        Console.WriteLine(e.Message);
      }
    }
    catch(Exception e)
    {
      Console.WriteLine(e.Message);
    }
  }
}
```

This is a much cleaner approach. I no longer have to explicitly deal with P/Invokes and pointers to strings; The `Win32Exception` class does everything for me. 

## Summary 

1. Remember, anytime you're dealing with a large API (like .NET or Java) spend some time to see if there's an easier way to get the job done. It's a simple phrase that everyone's heard before, but don't reinvent the wheel.
2. Even if base classes don't provide the necessary functionality, consider buying a component from a third-party vendor instead of creating it yourself. While it's educational to build it on your own, project time pressures sometime prohibit the necessary R&D.

> Published: 08.15.2002