---
title: Tuples and Serialization
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Tuples and Serialization

Synopsis: With .NET Standard 2.0, .NET developers now have a much easier time writing cross-platform code. However, it's still important to read the fine print and understand what the difference is between a specification and an implementation.

## Ask, and You Shall Receive ( ... Sort Of)

If you're a .NET developer that wants their code to run on different platforms, you know there's effort that you have to put forth. Just saying "target .NET Core!" isn't sufficient. There's the .NET Framework, Mono, Xamarin and all of its platforms (iOS, Android, etc.), and so on. Figuring out which APIs will work and which ones won't and trying to finagle your code to work on all these systems can be quite painful.

I think we've all wanted to see some unification in the .NET universe, broadening its reach in the process. Fortunately, now we have ".NET Standard", which, as the documentation says:

> ... is a formal specification of .NET APIs that are intended to be available on all .NET implementations. The motivation behind the .NET Standard is establishing greater uniformity in the .NET ecosystem.

With .NET Standard 2.0, the API surface has increased substantially, making the deltas between platforms significantly smaller. The strife in writing C# code that can target multiple platforms has lessened. This is a good thing.

Now, let me digress a bit, I'll come back to .NET Standard in a moment.

A while back, I wrote about tuples and how you could use them to pass messages to objects in your application. Unfortunately, at that time, tuples, or more specifically, the types defined in System.ValueTuple were not serializable. Long story short, a GitHub issue was created, and (eventually) it was marked as "Closed". This meant that value tuples were now serializable. Yay!

Why am I happy about this? There's a project I've been meaning to work on for a while now, and with .NET Standard 2.0 in place, I feel like I can finally start on it and put significant effort behind it. I've done a fair amount of design work on it, and one piece hinged on tuples, potentially on them being serializable. While I could do that in a customized way, I was really hoping that serializing tuples would just work, and required no more effort on my part than any other serializable type in .NET.

So, I thought, let's test out tuples and serialization in a .NET Standard project. The issue was closed, right? It should just work, right? What could possibly go wrong?

Well, things did go wrong, and it was my fault. Essentially, I didn't read the docs, and I assumed too much. That's usually a recipe for disaster most of the time. But it's also a good illustration showing what .NET Standard is and what you should code for. Let's take a closer look.

## Serializing Value Tuples

To test the serialization of value types, I created a .NET Standard 2.0 class library project, and added this code:

```c#
public static (int, int) Roundtrip((int, int) value)
{
  var formatter = new BinaryFormatter();

  using (var stream = new MemoryStream())
  {
    formatter.Serialize(stream, value);
    stream.Position = 0;
    return ((int, int))formatter.Deserialize(stream);
  }
}
```

Next, I created two console application projects: one that targeted .NET Core 2.0, and one that targeted .NET 4.7. Both referenced my class library project and called `Roundtrip()` like this:

```c#
static void Main(string[] args) =>
  Console.Out.WriteLine(
    ValueTupleSerializer.Roundtrip((1, 2)));
```

With the .NET Core console application, everything worked out as I wanted it to:

![Tuples And Serialization](https://jasonbock.net/images/Tuples-And-Serialization-1.png "Tuples And Serialization")

But with the .NET framework project, things didn't go as planned:

![Tuples And Serialization](https://jasonbock.net/images/Tuples-And-Serialization-2.png "Tuples And Serialization")

What happened? Why didn't this work? I thought it was fixed!

## Read the Contract!

The problem was I had foolishly made some incorrect assumptions. When I saw that the GitHub issue was closed, I naturally assumed it was fixed for .NET Standard 2.0. This is completely false. In fact, in .NET Standard, types are not defined with the `[Serializable]` attribute. This is because the actual type definition used at runtime is type-forwarded to the runtime being used by the host. In other words, .NET Standard doesn't specify if a type is serializable (unless it implements ISerializable, but that's a slightly different case).

So why does it work for .NET Core 2.0 and not .NET Framework 4.7? In .NET Core, the serialization addition made it through, so at runtime my serialization code runs without issue. In other words, I got lucky! With .NET Framework 4.7, that change hasn't made it through, though in 4.7.1 it should be there. Here's the list of all serializable types in .NET, though it's interesting that in this case, ValueTuple is considered serializable in .NET Core and not the full .NET Framework.

This is the main takeaway for .NET Standard: it's a specification, not an implementation. The implementations are targets like .NET Core, Xamarin.iOS and UWP. These implementations can add features to these types, but they can't take anything away from them. This is especially true with serialization as none of the types are defined as such in the .NET Standard. It's up to the implementations to mark them that way. Given that in .NET Framework 4.7 and previous versions ValueTuple wasn't defined as serializable yet, it's just a timing issue that one must wait for 4.7.1 to assume that this type can be serialized without having to create custom code for that specific type.

## Conclusion

In my mind, .NET Standard is the way to go when creating packages that need to run in many different environments. Over time, I foresee most, if not all, of the packages in NuGet supporting some version of .NET Standard. By defining a large API surface, developers can focus on creating the features they want for customers across a wide range of platforms. The key is to remember that .NET Standard only provides the surface; the platforms need to support that structure. Developers targeting .NET Standard should always be aware of what this surface looks like and code for that view. In cases where platform-specific code is needed, this can be handled in a variety of ways (e.g. dependency injection).

> Published: 09.08.2017