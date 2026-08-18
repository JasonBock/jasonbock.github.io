---
title: Reflector, Add-Ins, and Unit Testing
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Reflector, Add-Ins, and Unit Testing

I'm currently looking at writing a new add-in for [Reflector](https://www.red-gate.com/products/reflector/). The problem is, I'd really like to get some unit tests around the non-UI portion of the code, and I can't find a way to "use" Reflector.exe in a unit testing context to load an assembly, find the method body, etc. I'm trying to IM Lutz about it but no response so far (but he's a busy guy :) ). I know writing add-ins for Reflector is not something devs do on a day-to-day basis but I'd thought I'd throw out a feeler before I go down a long and tedious time creating mock implementation of the interfaces in Reflector.exe.

UPDATE: There is a way! Go here for the answer. The current API is a bit different than what the article states - here's a preview of how to iterate over the types in an assembly:

```c#
[TestClass]
public sealed class PlaygroundTests
{
  [TestMethod]
  public void TryIt()
  {
    IServiceProvider provider = new ApplicationManager(null);

    IAssemblyManager manager = provider.GetService(
      typeof(IAssemblyManager)) as IAssemblyManager;
        
    IAssembly assembly = manager.LoadFile("YourAssembly.dll");
        
    foreach(IModule module in assembly.Modules)
    {
      this.TestContext.WriteLine("Module: " + module.Name);
            
      foreach(ITypeDeclaration type in module.Types)
      {
        this.TestContext.WriteLine("Type: " + type.Namespace + "." + type.Name);                
      }
    }
  }
    
  public TestContext TestContext
  {
    get;
    set;
  }
}
```

Nice!

> Published: 04.09.2008 09:02:41 AM CST