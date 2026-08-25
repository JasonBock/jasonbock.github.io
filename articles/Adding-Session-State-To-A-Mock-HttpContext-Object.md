---
title: Adding Session State to a Mock `HttpContext` Object
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Adding Session State to a Mock `HttpContext` Object

A while back I ran into an issue where I was running some NUnit tests that required a `HttpContext` object to be in place, but I didn't want to host the entire ASP.NET runtime to do this. It would've been easy to do if `HttpContext` wasn't `sealed` or, even better, implemented an interface or inherited from a base class so I could create my own custom `HttpContext` class, but that's not the case. I did some searching and I found this solution. The guy did a good job in getting a mock `HttpContext` object in place, but as soon as I started using it I found it wanting in one area: session state. If you use the code as-is, you'll notice that the `Session` property is null, and for our tests we needed that property [1]. It wasn't a pressing problem but when I had some time I dug into the problem, and I eventually found a solution. It only takes a few lines of code, but I'm going to walk through the path that I took to arrive at the solution.

First, I opened `HttpContext` in Reflector. I've said it before and I'll say it again: if you're a .NET developer you **must** have this tool installed. It's much easier to use than ILDasm 98% of the time and it makes it very easy to dig around in code to see what's going on behind the scenes. Anyway, my first target was to look at the `Session` property to see what it does when the get is invoked:

```c#
public HttpSessionState get_Session()
{
  return (HttpSessionState) this.Items["AspSession"];
}
```

Seems simple enough, huh? The `Items` property is an `IDictonary`-based property. All I need to do is set the right value in the dictionary to a `HttpSessionState` object and I'm done. The problem is that `HttpSessionState` does not have a `public` constructor, so I can't make an instance of `HttpSessionState`. Well ... it can't be done easily. `HttpSessionState` has one constructor that's marked as `internal`:

```c#
internal HttpSessionState(string id, SessionDictionary dict, HttpStaticObjectsCollection staticObjects, 
  int timeout, bool newSession, bool isCookieless, SessionStateMode mode, bool isReadonly);
```

Now I have two problems. The first problem is I need to figure out the values of the parameters for this constructor. Second, I have to figure out how to call this constructor via Reflection.

The first part is a bit harder than it looks. `HttpContext` doesn't set the `HttpSessionState` value in the `Items` collection. I had to use my `FileGenerator` add-in to search all of the classes in the `System.Web` assembly. Once I did that, I found that `SessionStateModule` sets the session state value in its `CompleteAcquireState()` method. I won't bore you with the details on how I figured out what the parameter values are, but here's what I came up with:

```c#
HttpSessionState state = new HttpSessionState(Guid.NewGuid().ToString("N"), sessionDictionary, 
  new HttpStaticObjectsCollection(), 5, true, true, SessionStateMode.InProc, false);
```

Again, I need to call that constructor via Reflection - I'll get to that later. Most of the values are easy to see why they are what they are, but the kicker is that `sessionDictionary`. The second argument to `HttpSessionState` is a `SessionDictionary`. Unfortunately, now I have two more problems (it just keeps getting better, doesn't it?). `SessionDictionary` has no `public` constructors - in fact, the class itself is `internal`. So, I have to create a type via `Activator.CreateInstance()`:

```c#
private const string TypeSessionDictionary = 
  "System.Web.SessionState.SessionDictionary, " + 
  "System.Web, Version=1.0.5000.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a";

// ...

Type sessionDictionaryType = Type.GetType(TypeSessionDictionary);
object sessionDictionary = Activator.CreateInstance(sessionDictionaryType, true);
```

Cool! Now, I'm suddenly in business. I have all the values I need to pass to `HttpSessionState`'s constructor. This time, I have to use a more complex version of `Activator.CreateInstance()`, but at the end I have a valid `HttpSessionState` object:

```c#
HttpStaticObjectsCollection staticObjects = new HttpStaticObjectsCollection();
Type sessionDictionaryType = Type.GetType(TypeSessionDictionary);
object sessionDictionary = Activator.CreateInstance(sessionDictionaryType, true);

HttpSessionState state = Activator.CreateInstance(
  typeof(HttpSessionState),
  BindingFlags.Public | BindingFlags.NonPublic | 
  BindingFlags.Instance | BindingFlags.CreateInstance, 
  null,  
  new object[] {Guid.NewGuid().ToString("N"), sessionDictionary, 
    staticObjects, 5, true, true, SessionStateMode.InProc, false}, 
  CultureInfo.CurrentCulture) as HttpSessionState;
this.context.Items[ContextKeyAspSession] = state;
```

All it takes is five lines of code. It took some time to figure it out, but now I have a mocked version of `HttpContext` that has session state. If you want to see the code in full, you can get it here. If you have any issues, please me know. Note that this code is very prone to problems. I've tested it against the .NET 1.1 version, but I have no guarantees that this code will always work. I'm doing things that require a high permission level. Also, I'm calling constructors and using types that are not public by design, so there's no guarantee that future versions will have the same signatures. But showing how I found out the session state object was created in the first place, hopefully you can fix the code if a future .NET version changes things.

[1] I realize in the comment section that someone posted a solution to the session state problem, but I'm not too fond of it. The solution I propose is very similar to what happens at runtime and you can use an HttpContext object directly.

> Published: 09.06.2005 09:02:49 PM CST