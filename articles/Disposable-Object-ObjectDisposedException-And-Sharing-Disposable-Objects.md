---
title: Disposable Object, ObjectDisposedException, and Sharing Disposable Objects
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Disposable Object, ObjectDisposedException, and Sharing Disposable Objects

If you've coded in .NET for a while, you've probably run across the `IDisposable` interface. I won't go into the details of how you should implement that interface (that's been done numerous times before); what I wanted to cover was `ObjectDisposedException`.

When I first heard about `IDisposable`, I learned why it was introduced and how it should be implemented. I think what got lost over the years is the fact that you shouldn't let anyone use an object after it's been disposed. The docs on `IDisposable` don't mention `ObjectDisposedException`, which is a shame. The docs on implementing `IDisposable` do, but only in the code sample, not in the general discussion. Basically, you should do something like this:

```c#
public void DoSomething()
{
  this.CheckForDisposed();
  // ...
}

private void CheckForDisposed()
{
  if(this.disposed)
  {
    throw new ObjectDisposedException("MyClass");
  }
}
```

Now, I admit that I haven't been doing this until fairly recently, which is a bad thing. It really became evident in my [EmitDebugging](https://github.com/JasonBock/EmitDebugging) code (which is now fixed!). There are a couple of classes that implement `IDisposable` because they have fields that implement `IDisposable`. Furthermore, they all share the same resource: a `CodeDocument`. That is, the `AssemblyDebugging` object has a `CodeDocument` field, and it has a creational method (`GetTypeDebugging()`) to create a `TypeDebugging` object. It passes the `CodeDocument` reference to that new object. The reason this is done is because they're all writing to the same stream to produce an .il file. But the problem is that I was disposing the `CodeDocument` reference in the `TypeDebugger` when it was disposed.

D'oh!

That's wrong, because that `TypeDebugger` reference didn't create the disposable resource in the first place. Only when the `AssemblyDebugger` reference is disposed should the `CodeDocument` reference be disposed. I had this bug in my code for a while but I didn't notice it until last night when I started to add checks for disposal in all of the methods. That's when I started to get lots of `ObjectDisposedExceptions` in my tests!

The `IDisposable` interface seems very simple at first glance - it has only one method, `Dispose()`. But there's lots of implementation details to keep in mind when you implement it.

> Published: 02.25.2008 07:52:42 AM CST