---
title: Using ExtendedReflection To Manage Your .NET Code at Runtime
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Using ExtendedReflection To Manage Your .NET Code at Runtime

Consider the following (bad) code:

```c#
class Program
{
  static void Main(string[] args)
  {
    new SubClass("go");
  }
}

public abstract class BaseClass
{
  protected BaseClass()
  {
    this.Run();
  }

  protected abstract void Run();
}

public class SubClass
  : BaseClass
{
  private string data;

  public SubClass(string data)
    : base() 
  {
    this.data = data;
  }

  protected override void Run()
  {
    Console.Out.WriteLine(data.Length);    
  }
}
```

The issue of calling virtual members in constructor has been discussed in depth before, so I won't cover it here (see [this article](https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/constructor) for details). It's not that you can't do it right, but it can cause issues if you're not a little careful. This issue actually bit me years ago because I'm fond of designs where my base class tells the subclasses to do something when the constructor is called. However, it's too error-prone - all it takes is for someone to subclass my class that's a little careless and then all sorts of bad things happen.

A while ago I ran into [Pex](https://www.microsoft.com/en-us/research/project/pex-and-moles-isolation-and-white-box-unit-testing-for-net/), and I fell in love with its power and capabilities. It was kind of freaky-scary what Pex would uncover in my code - suddenly there were corner cases with a bright shining light on them and I realized that I have subtle errors that I really should fix! Pex is also interesting because of the technologies and frameworks that it uses that do all sorts of advanced, cool stuff. One in particular is called ExtendedReflection. This is basically a managed profiler - yes, you read that right. For the longest time I was under the impression that you can't write a profiler in .NET in managed code, but I was wrong, because ExtendedReflection does just that.

So what does this have to do with calling virtual members in constructors? Think of flipping around `IDisposable`. See, `IDisposable` works with the `using` statement in C# to automatically call the `Dispose()` method. Of course, this is all compiler magic; the runtime doesn't treat `IDisposable`-based objects any differently, so if you forget to call `Dispose()` in a timely manner, you could leak resources (depending on what the object used during its lifetime). Now, you could argue that the runtime should handle this for you, but that's not an easy problem to solve - in fact, it's downright difficult. But ... wouldn't it be interesting if you could have the runtime call a method on your object if it implemented a specific interface after the constructor was done but before anything else was done to the object?

Now I'm not suggesting that the .NET team do this. It's just a pie-in-the-sky idea: creating an `IInitializer` interface with an `Initialize()` method that would be called for you with the guarantees specified above. You could try and handle this with code weaving techniques, but the simple fact is, unless the runtime does this across the board, that guarantee may not be satisfied and bad things may happen. But let's try to get close by augmenting the runtime execution of code with ExtendedReflection.

The first thing you need to do is install Pex. Once you have that done, take some time to look at the ExtendedReflection examples - the code I'm about to show is just a slight modification of one of the samples. With that work out of the way, let's add the code to call `Initialize()` right after an object is created. The starting point is creating a console application that will set up the managed profiler and then run the target program:

```c#
public static class Program
{
  [LoaderOptimization(LoaderOptimization.MultiDomain)]
  static void Main(string[] args)
  {
    var application = args[0];

    Console.Out.WriteLine("Setting environment...");
    var startInfo = new ProcessStartInfo(application, null);
    startInfo.UseShellExecute = false;
    var environmentVariables = Program.GetMonitoringEnvironmentVariables();
    foreach (DictionaryEntry environmentVariable in environmentVariables)
    {
      startInfo.EnvironmentVariables[(string)environmentVariable.Key] =
        (string)environmentVariable.Value;
    }

    Console.Out.WriteLine("Starting target process, {0}...", application);
    using (var process = Process.Start(startInfo))
    {
      process.WaitForExit();
    }
  }
```

The profiler API needs you to set up some environment variables before you launch the real program. Here's what `GetMonitoringEnvironmentVariables()` does:

```c#
  private static StringDictionary GetMonitoringEnvironmentVariables()
  {
    var environmentVariables = new StringDictionary();
    var userAssembly = Metadata<ExecutionMonitor>.Assembly;
    var userType = Metadata<ExecutionMonitor>.Type;

    ControllerSetUp.SetMonitoringEnvironmentVariables(environmentVariables,
      MonitorInstrumentationFlags.All,
      false, userAssembly.Location, userType.FullName,
      null, null, null, null, null,
      new string[] { "*" },
      new string[] { 
        Metadata<Object>.Assembly.ShortName,
        Metadata<_ThreadContext>.Assembly.ShortName,
        userAssembly.ShortName },
      null, null, null, null, null, null,
      @"Modifier.txt",
      false, null, true, false, ProfilerInteraction.Fail, null);

    return environmentVariables;
  }
}
```

Don't worry too much about what `GetMonitoringEnvironmentVariables()` does. If you're really curious, read the docs, but for our purposes it's sufficient to know that this gets the profiler hooks in place.

The next is to create a custom execution monitor (which is what was used to define the `userType` variable above). This is what will get called whenever a thread is created in the application, which is where you'll set up your hook:

```c#
internal sealed class ExecutionMonitor
  : ExecutionMonitorBase
{
  protected override IThreadExecutionMonitor CreateThreadExecutionMonitor(int threadId)
  {
    return new ThreadExecutionMonitor(threadId);
  }
}
```

The `ThreadExecutionMonitor` class is the one that does all the heavy lifting:

```c#
internal sealed class ThreadExecutionMonitor
  : ThreadExecutionMonitorEmpty
{
  private MethodBase initializeMethod;

  internal ThreadExecutionMonitor(int threadId)
    : base(threadId)
  {
    this.initializeMethod = typeof(IInitializer).GetMethod("Initialize");
  }

  public override void AfterNewobjObject(object newObject)
  {
    var initializer = newObject as IInitializer;

    if (initializer != null)
    {
      initializer.Initialize();
    }
  }

  public override void Call(Method method)
  {
    this.HandleCall(method);
  }

  public override void Callvirt(Method method)
  {
    this.HandleCall(method);
  }

  private void HandleCall(Method method)
  {
    var initializerType = Metadata<IInitializer>.Type;
    var foundMethod = initializerType.GetMethod(this.initializeMethod);

    TypeEx declaredType = null;

    if (method.TryGetDeclaringType(out declaredType))
    {
      if ((from declaredInterface in declaredType.Interfaces
        where declaredInterface == initializerType
        select declaredInterface).FirstOrDefault() != null)
      {
        Method sourceMethod = null;
        if (method.TryReverseVTableLookup(Metadata<IInitializer>.Type, 
          out sourceMethod) && sourceMethod != null)
        {
          if (foundMethod.Signature == sourceMethod.Signature)
          {
            throw new NotSupportedException(
              "Modifier: This method can only be invoked once.");
          }
        }
      }
    }
  }
}
```

The first thing that's done is getting a reference to the `Initialize()` method via a `MethodBase` object - I'll get back to why this is needed in a moment. Next, `AfterNewobjObject()` is overriden to provide the key hook to know when the `newobj` IL instruction was finished. At that point, you know the object has been created, but nothing else has been done to it, so that's the perfect time to see if that object implements `IInitializer` so you can call `Initialize()`! Finally, two other method are overriden: `Call()` and `Callvirt()`. This is done to add a check to ensure that `Initialize()` is called once, and only once by the hook added via `AfternewobjObject()`. If code in the application would try to call `Initialize()`, that would be considered an error because we don't want that setup code to run more than one time. The code within `HandleCall()` may look a little odd, but it's all based on the ExtendedReflection API, and it'll catch if `Initialize()` is invoked by code within the profiled application.

So, once everything is in place, I can create this application:

```c#
class Program
{
  static void Main(string[] args)
  {
    Program.RunTester();
    Program.RunTester();
  }

  private static void RunTester()
  {
    var tester = new Tester() { Data = Guid.NewGuid().ToString("N") };
    Console.Out.WriteLine(tester.Data);

    try
    {
      tester.Initialize();
    }
    catch (NotSupportedException ex)
    {
      Console.Out.WriteLine(ex.Message);
    }
  }
}

public sealed class Tester : IInitializer
{
  public string Data { get; set; }

  public void Initialize()
  {
    Console.Out.WriteLine("I have been initialized.");
  }
}
```

When I run it under my managed profiler, this is what I get:

```
Setting environment...
Starting target process, Modifier.Example.exe...
I have been initialized.
7315d24e463446128897af9993703017
Modifier: This method can only be invoked once.
I have been initialized.
08c24f49a22a4bc7b720d0a4f40dd610
Modifier: This method can only be invoked once.
Press any key to continue . . .
```

Slick!

Again, this is just experimental code. I strongly recommend you don't just throw this on to a production machine and start adding all sorts of crazy hooks into your application via this managed profiler. But it is fun to play with code as it's executing at runtime. Enjoy!

> Published: 11.14.2011 08:57:00 PM CST