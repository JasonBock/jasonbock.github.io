---
title: Using System.Reflection.Emit in a Windows 8 Metro style Application
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Using System.Reflection.Emit in a Windows 8 Metro style Application

I just got back from BUILD yesterday, and I wanted to spend some time going over some stuff that I worked on during the event. Essentially, I wanted to see if you use `System.Reflection.Emit` in a Metro style application. I know, that's probably not the big use case that Microsoft is looking at, but I've found throughout my career that trying to see what you can do at a lower-ish level in a given environment is a good way to understand what works (or what doesn't) and what's really going on.

So, I created a VS11 solution with a Metro style application and a .NET class library. The reason I did this is because there is no `System.Reflection.Emit` in WinRT. My personal guess is that this was done as you probably don't want a Metro style application you installed from the store (when that eventually becomes available) that can generate code that wasn't analyzed via the store submission process. Therefore, you need to throw all your `System.Reflection.Emit` code in a .NET class library.

Here's what my code does:

```c#
public static class Emitter
{
  public static string EmitThis()
  {
    var name = new AssemblyName("EmittedCode");
    var builder = AppDomain.CurrentDomain.DefineDynamicAssembly(
      name, AssemblyBuilderAccess.Run);
    var module = builder.DefineDynamicModule(name.Name);
    var type = module.DefineType("EmittedType", TypeAttributes.Public);
    var method = type.DefineMethod("EmittedMethod", 
      MethodAttributes.Public | MethodAttributes.Static,
      typeof(string), Type.EmptyTypes);
    var generator = method.GetILGenerator();
    generator.Emit(OpCodes.Ldstr, "I did it in emitted code.");
    generator.Emit(OpCodes.Ret);
    var createdType = type.CreateType();

    return (string)createdType.GetMethod(
      "EmittedMethod").Invoke(null, null);
  }
}
```

It's simple, but that's good enough for demo purposes.

Now, let's add it to the Metro style application:

![Referencing Metro.Emit](https://jasonbock.net/images/Reference-Manager-Metro-Emit.png "Referencing Metro.Emit")

Whoops! You can't add a reference to assemblies from .NET projects in a Metro style application. But you can reference the assembly directly. Note that this hardcodes the assembly reference to the bin\debug directory. You could add a post-build event to put the results of compilation in a directory and reference the assembly that way, but again, for demo purposes, this works.

Now I can run the code this way:

```c#
private void OnEmitClick(object sender, RoutedEventArgs e)
{
  this.EmitResult.Text = Emitter.EmitThis();
}
```

And when I run my Metro style application ... It works.

Now, remember, just because you can do something doesn't mean you should. As I stated above, an application like this will probably never get in the store. It's doing something that goes beyond what Microsoft wants (or doesn't want) a Metro style application to do. It does run locally, and I'm guessing you could ship the package around to others so they could run it locally as well, but I really don't know if that would work as well. There's Metro style applications from the store and there's Windows 7 desktop applications and they both run on Windows 8; it'll be interesting to see if there will be apps "in the middle", so to speak.

I do know that you would need functionality like this for libraries like NSubstitute. They rely on code being generated on the fly from `System.Reflection.Emit`; even though `System.Linq.Expressions` is in WinRT, that's not full-featured enough for proxies (at least not that I can tell right now). But doing that all locally in a unit test will probably be "OK"; you just can't ship applications like that into the store.

Note that this is only going to work with the VS11 Ultimate preview and Windows 8 (though you'll be able to look at the code file assets in any Windows OS). Enjoy!

> Published: 09.17.2011 06:12:13 PM CST