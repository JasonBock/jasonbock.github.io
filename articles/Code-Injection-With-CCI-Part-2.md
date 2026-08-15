---
title: Code Injection With CCI - Part 2
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Code Injection With CCI - Part 2

Previous Parts

* [Part 1](https://jasonbock/articles/Code-Injection-With-CCI-Part-1.html)

In [Part 1](https://jasonbock/articles/Code-Injection-With-CCI-Part-1.html), I gave a high-level overview of what we needed to do to inject aspects into an assembly based on the existence of metadata. Let's take a look at what the main method of `CciInjector` looks like:

```c#
using Microsoft.Cci;
using System;
using System.IO;

namespace CciInjector
{
  class Program
  {
    static void Main(string[] args)
    {
      if(args != null && args.Length == 1)
      {
        var host = new HostEnvironment();

        var module = host.LoadUnitFrom(args[0]) as IModule;

        if(module != null && module != Dummy.Module && module != Dummy.Assembly)
        {
          var injector = new InjectorVisitor(host);

          var assembly = module as IAssembly;

          if(assembly != null)
          {
            module = injector.Visit(injector.GetMutableCopy(assembly));
                        
            PeWriter.WritePeToStream(module, host,
              File.Create(module.Location + ".pe"));
          }
        }
      }
    }

    private sealed class HostEnvironment : MetadataReaderHost
    {
      PeReader peReader;

      internal HostEnvironment()
        : base()
      {
        this.peReader = new PeReader(this);
      }

      public override IUnit LoadUnitFrom(string location)
      {
        var result = this.peReader.OpenModule(
          BinaryDocument.GetBinaryDocumentForFile(location, this));
        this.RegisterAsLatest(result);
        return result;
      }
    }
  }
}
```

That's a fair amount of code, so let's break it down a bit.

First, note the `Microsoft.Cci` namespace at the top. If you go to the CCI Metadata site, you can either get the source code and compile the latest version yourself, or you can get the binaries from the Downloads section. Either way, there are a number of assemblies you need to reference - here's a list of the ones I included in my project:

```c#
Microsoft.Cci.MetadataHelper
Microsoft.Cci.MetadataModel
Microsoft.Cci.MutableMetadataModel
Microsoft.Cci.PeReader
Microsoft.Cci.PeWriter
Microsoft.Cci.SourceModel
```

There are PDB reader/writer assemblies as well, but I won't be covering that in this series ... at least I don't plan to at the moment. Granted, when you modify assemblies, you should be aware if there are debug symbols lurking around as well and modify them as well. Otherwise, the sequence points may get all messed up and it'll give the developer and "odd" debugging experience. For now, though, I'm going to focus exclusively on the modification part.

The `Main()` method is pretty straightforward. I take the value from the command line and try to load it via `LoadFromUnit()`. `MetadataReaderHost` is an abstract class that requires you to implement `LoadUnitFrom()`. I saw this code somewhere in the CCI samples but basically it's just opening the file as a PE file. Note that if it couldn't interpret it as a .NET-based PE file, it'll give back one of those `Dummy` values.

Once I know I have a .NET assembly, I create an `InjectorVisitor` object. I'll dive into the details of this class in Part 3 - essentially it descends from `MetadataMutator`, which has numerous `Visit()` methods on it. These allow you to pick and choose which members of an assembly you want to investigate as the visitor class walks the member tree of an assembly. That's where all the assembly modification goes on.

The last thing I do is save the changes via `PeWriter.WritePeToStream()`. The only problem I have with this code is I can't overwrite the assembly file I'm given because CCI still has the file opened and won't let me overwrite it. This is different than Cecil were you can easily save the changes to the same file. I may be missing something here and there may be a way to do what I want here, but for now I have to create a new file that has the modifications.

That's it for now. Part 3 will dive into the real guts of assembly modification via CCI.

> Published: 04.23.2009 08:36:22 AM CST