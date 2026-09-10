---
title: Eliminating Nulls in C#
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Eliminating Nulls in C#

Synopsis: One of the biggest features coming in C#8 is non-nullable reference types. In this article, I'll walk through what this feature is, how you can start using it in VS2019, and specific cases I ran into updating a NuGet project I maintain called [Rocks](https://github.com/JasonBock/Rocks).

## The Danger of Nulls

Have you ever had a `NullReferenceException` in C#? Fortunately, I haven't had a lot of them in my career, but it's not because I'm an exceptional programmer that never makes a mistake. Rather, it's because I learned a while ago how insidious bugs related to null references can be, and I started changing my programming style to reduce these occurrences. I'd check parameters in constructors for null and immediately throw `ArgumentNullException` if that was true. If a property was mutable, I'd be careful whenever I'd use that property's value, checking if it was null and handling that condition appropriately. Still, I'd occasionally run across a case where I'd make a false assumption about the output of a computation, assuming it would never be null, and I'd end up being sad.

The designers of C# understand the pains null bring to the table. In C#8, a new feature – non-nullable reference types – will help in removing this discomfort. This feature allows a developer to state when a reference type can be null. To be specific, a developer must explicitly declare the possibility of a variable being `null`; reference types are assumed to be non-nullable by default. It's quite a change from how C# has behaved since its inception, and frankly I wish C# would've done this from the start. Alas, we can't turn back the clock, so let's do the next best thing and learn how to use this feature to improve our code for the future.

## Enabling the Feature

As of the writing of this article, C#8 is still in preview mode, so it's possible this feature will be pulled, though I personally find that highly unlikely. Also, assuming the feature makes into the final release of C#8, some aspects of it may change. Therefore, the information you read in this article may end up becoming misaligned with where the feature finally lands.

If you want to try it now (where "now" is spring 2019), ensure that you've done the following things:

* Install VS 2019 (any SKU is fine)
* Install .NET Core 3.0, which is currently in preview mode.
* Enable preview versions of .NET Core 3.0. You can do this from *Tools -> Options -> Environment -> Preview Features*, and enable Use previews of the .NET Core SDK:

![Eliminating Nulls](https://jasonbock.net/images/Eliminating-Nulls-1.png "Eliminating Nulls")

* Set `<LangVersion>` in the .csproj file to 8.0, and `<Nullable>` to `enable`, like this:

```xml
<LangVersion>8.0</LangVersion>
<Nullable>enable</Nullable>
``` 

I also set `TreatWarningsAsErrors` to `true` in my projects. This isn't necessary, but I find having warnings as errors forces me to address issues that may be considered minor but won't be addressed if a compilation succeeds. By having this flag enabled, these warnings will fail a build.

This should be enough to light up the compiler such that it's now going to check if nulls are unexpectedly creeping into your code. Let's see how this will happen with my Rocks project.

## Updating Rocks

Rocks is a .NET mocking library, like what you'd see with [Moq](https://github.com/devlooped/moq) or [NSubstitute](https://nsubstitute.github.io/). To create a mock, you'd do something like this:

```c#
public interface IAmSimple
{
  void TargetAction();
}

var rock = Rock.Create<IAmSimple>();
rock.Handle(_ => _.TargetAction());

var chunk = rock.Make();
chunk.TargetAction();

rock.Verify();
```

My intention in creating Rocks wasn't to try to replace other frameworks; rather, I wanted to explore what it would take to build a mocking framework that used Roslyn to generate the mock types. It's been around for just over 3 years now, and it has nearly 5,000 lines of code. Are there any undiscovered null bugs in my code? Let's find out!

You can find all the work done to introduce non-nullable reference types to Rocks via [this GitHub issue](https://github.com/JasonBock/Rocks/issues/113).

## Handling Null Literals

Using the null keyword as a literal is a source of problems in C#8. For example, before C#8, setting a variable or a parameter to null was legal. Now, it's not. One example of this was discovered in the constructor to the `RockOptions` class:

```c#
public RockOptions(OptimizationSetting level = OptimizationSetting.Release, 
  CodeFileOptions codeFile = CodeFileOptions.None, 
  SerializationOptions serialization = SerializationOptions.NotSupported,
  CachingOptions caching = CachingOptions.UseCache, 
  AllowWarnings allowWarnings = AllowWarnings.No,
  string codeFileDirectory = null)
``` 

Notice that the optional `codeFileDirectory` parameter is non-nullable, so setting it to null if it wasn't provided is not legal. Fortunately, the assignment of the property that uses `codeFileDirectory` in the constructor is safe:

```c#
this.CodeFileDirectory = codeFileDirectory ?? Directory.GetCurrentDirectory();
``` 

In this case, the fix was simple:

```c#
string? codeFileDirectory = null
``` 

Users can still pass in `null` or ignore the argument entirely. In most cases, generating mocks will be done without generating a code file for that mock type, so having this parameter being `null` is fine. Also, having the current directory as the default location for code files if they've set `codeFile` to `CodeFileOptions.Create` is a reasonable choice. If they want to specify a directory, they can easily do that. This really was never an issue, but it's nice that we're being explicit now.

Another scenario of this error (CS8625) was in `Equals(CacheKey)` for `CacheKey`:

```c#
public bool Equals(CacheKey other)
{
  var areEqual = false;

  if (other != null)
  {
    areEqual = this.GetHashCode() == other.GetHashCode();
  }

  return areEqual;
}
```

The issue lies with the `if` statement. The problem is that `null` can't be converted to a non-nullable type. There are a couple of ways to address this – the easiest is to use the `is` keyword:

```c#
if (!(other is null))
``` 

As of C#7, this is a legal usage of is. Using the is keyword to compare against `null` is better over using the equality operator even if the non-nullable feature was disabled. A developer can change the behavior of the equality operator and subtly change how a `null` comparison would work. Using is makes the comparison unambiguous.

However, let's dig into the issue a bit further. This version of `Equals(CacheKey)` exists because `CacheKey` implements `IEquatable`. Here's how `Equals(object)` from `System.Object` is overridden:

```c#
public override bool Equals(object obj) => this.Equals(obj as CacheKey);
``` 

C#8 has an issue with this, because the `as` operator may actually yield a `null` value, and `Equals(CacheKey)` can't take a `null` value. But if we use a cast here:

```c#
public override bool Equals(object obj) => this.Equals((CacheKey)obj);
``` 

That would throw an `InvalidCastException` if the method is giving an object that can't be cast to `CacheKey`. But is that wrong? Why should anyone pass in a value here that would be anything but something that is a `CacheKey`-based object? The issue here is that the guidance around implementing `Equals(object)` is that it should not throw an exception. Therefore, the implementation of `IEquatable` should be changed to allow nulls:

```c#
internal sealed class CacheKey
  : IEquatable<CacheKey?>
``` 

Therefore, `Equals(CacheKey)` changes to this:

```c#
public bool Equals(CacheKey? other)
``` 

Now both versions of `Equals()` won't throw an exception, pass the C#8 compiler, and will behave as expected.

Is this a problem? No, because we know we're not implementing `==`. But it could be in another code base where `==` would act in an unexpected way. Again, being explicit about nullability and using is are good changes.

## Generics and Nullable Types

Generics work with nullable constraints just fine. Here's a case in my `Rock` class that was confusing to me at first, but the solution was straightforward:

```c#
public static (bool isSuccessful, T result) TryMake<T>(RockOptions options)
  where T : class
{
  var mappedOptions = Rock.MapForMake(options);
  var result = default(T);
  var isSuccessful = false;

  var tType = typeof(T);
  var message = tType.Validate(mappedOptions.Serialization,
    tType.IsSealed ? 
      new PersistenceNameGenerator(tType) as NameGenerator :
      new InMemoryNameGenerator() as NameGenerator);

  if (string.IsNullOrWhiteSpace(message))
  {
    result = Rock.NewRock<T>(mappedOptions, true).Make();
    isSuccessful = true;
  }

  return (isSuccessful, result);
}
``` 

The issue is with the return value. Specifically, that result can be `null` if a mock can't be made for the given type defined in `T`. I'm assuming users will look at the value of `isSuccessful` and only use result if `isSuccessful` is `true`, but using non-nullable types, I can make a clearer statement:

```c#
public static (bool isSuccessful, T? result) TryMake<T>(RockOptions options)
``` 

Notice that the return type is `T?`. A developer now knows that result could be `null` and should be defensive against that. However, one issue with this is that the C#8 compiler doesn't know that the values within the tuple have a conditional relationship – that is, result is only valid if `isSuccessful` is `true`. Therefore, a caller still needs to check result for null, or state that we know code using the value is safe because we know better than the compiler can. One way is to use the null-forgiving operator (`!`) whenever the variable is used:

```c#
var (isSuccessful, rock) = Rock.TryMake<IService>(new RockOptions());
if(isSuccessful)
{
  rock!.Handle(…);
}
``` 

Another way is to assign a local variable to result, explicitly stating it as not being `null`:

```c#
var (isSuccessful, result) = Rock.TryMake<IService>(new RockOptions());
if(isSuccessful)
{
  var rock = result!;
  rock.Handle(…);
}
``` 

This illustrates that there are cases where the compiler can't be smart enough to know when a value will not be `null`. Thankfully, we have a tool – the null-forgiving operator – that lets us tell the compiler we know things are OK. Keep in mind that you should use this operation sparingly. If you find that you're littering your code with null-forgiving operators, it's a sign that you're probably not taking advantage of the non-nullable reference type feature. In fact, you're probably fighting it, and you should re-evaluate why you're putting `!` everywhere.

## Initializing Fields in Constructors and Promoting Immutable Types

One nice aspect of immutable values is that you know exactly what the state of the value is. Specifically, if you have a mutable property, you always need to check if it's not `null` to use any of its members. However, there are cases in Rocks where I unfortunately have a field that isn't set on construction and could be `null` when I use it. Here's what that looks like in `PersistenceCompiler`:

```c#
internal sealed class PersistenceCompiler
  : Compiler<FileStream>
{
  private string assemblyFileName;
  private readonly string assemblyPath;

  // Note that the constructor doesn't set assemblyFileName.

  protected override void ProcessStreams(
    FileStream assemblyStream, FileStream pdbStream) =>
      this.assemblyFileName = assemblyStream.Name;

  protected override void Complete() =>
    this.Result = Assembly.LoadFile(this.assemblyFileName);
}
``` 

If I call `Complete()` before `assemblyFileName` is set in `ProcessStreams()`, I could get an error. Now, in Rocks, the way things are implemented, I don't call `Complete()` before `ProcessStreams()`. However, I may forget about this in the future and inadvertently forget about this call order dependency.

There's a subtle reason why I don't call `Complete()` before `ProcessStreams()` though. In `Compiler`'s `Compile()`, I grab the streams provided by `GetAssemblyStream()` and `GetPdbStream()` and put them within a `using` statement. If I call `Complete()` within the `using`, I'll get a file load error because the stream hasn't been closed yet in `PersistenceCompiler` (the `InMemoryCompiler` class that also derives from `Compiler` doesn't have this issue). What I ended up doing to remove the null issue and clean up the implementation a bit is to have one method all `Compiler` derivations must implement: `Emit()`:

```c#
protected override Assembly Emit(CSharpCompilation compilation)
{
  string assemblyFileName;

  using (FileStream assemblyStream = this.GetAssemblyStream(),
    pdbStream = this.GetPdbStream())
  {
    compilation.Emit(assemblyStream,
      pdbStream: pdbStream);
    assemblyFileName = assemblyStream.Name;
  }

  return Assembly.LoadFile(assemblyFileName);
}
``` 

I pushed the responsibility of creating and loading the assembly within this method. This eliminates the `null` issue and the stream problems.

This was a simple fix, but there was another instance of this situation relating to the implementation of `ArgumentExpectation<T>` that was harder to unravel. Feel free to review the history of the change to this class to see what I did to resolve it. Essentially, I split it up into distinct classes that handle the different states `ArgumentExpectation<T>` was doing all by itself, which feels like a cleaner design.

## Using Explicit Casts

Typically, I don't do explicit casts. That is, instead of doing something like this:

```c#
var x = (SomeClass)y;
``` 

I do this:

```c#
var x = y as SomeClass;
``` 

Oddly enough, I take this approach even when I know an explicit cast would be 100% safe. This starts to show up in C#8 as an issue, because as can return `null` if the cast would not be legal. Therefore, C#8 gave me null reference issues when I'd do something like what I'm doing with the return value of `GetGetterHandler()`:

```c#
internal static HandlerInformation GetGetterHandler(this PropertyInfo @this)
{
  var handlerType = typeof(HandlerInformation<>)
    .MakeGenericType(@this.PropertyType);
  return handlerType.GetConstructor(
    ReflectionValues.PublicNonPublicInstance, null,
    new[] { typeof(ReadOnlyDictionary<string, ArgumentExpectation>) }, null)
    .Invoke(new[] 
    { 
      PropertyInfoExtensions.CreateEmptyExpectations() 
    }) as HandlerInformation;
}
``` 

I know that `Invoke()` will return a value that can be safely cast to `HandlerInformation`. Therefore, I changed the code to explicitly casting the return value of `Invoke()` to `HandlerInformation`:

```c#
internal static HandlerInformation GetGetterHandler(this PropertyInfo @this)
{
  var handlerType = typeof(HandlerInformation<>)
    .MakeGenericType(@this.PropertyType);
  return (HandlerInformation)handlerType.GetConstructor(
    ReflectionValues.PublicNonPublicInstance, null,
    new[] { typeof(ReadOnlyDictionary<string, ArgumentExpectation>) }, null)
    .Invoke(new[] 
    { 
      PropertyInfoExtensions.CreateEmptyExpectations() 
    });
}
``` 

The more I thought about this, the more I realized this is the right way to handle the cast. If it would fail for some reason, something bad happened in an unexpected way.

## Ignoring Nullable Code

Sometimes, there are sections in code that you don't want to deal with non-nullable issues. There's an example of this with the `Arg` class:

```c#
public static class Arg
{
  public static T Is<T>(Func<T, bool> evaluation)
  {
    if (evaluation == null)
    {
      throw new ArgumentNullException(nameof(evaluation));
    }

    return default;
  }

  public static T IsAny<T>() => default;

  public static T IsDefault<T>() => default;
}
``` 

The return values for the methods are problematic as the default value for `T` is `null`, but that isn't allowed. However, `Arg` isn't used by Rocks at runtime. It's only used as an aid to help developers specify expectations in the `Handle()` methods via an expression, like this:

```c#
public interface IService
{
  void Use(int value);
}

var rock = Rock.Create<IService>();
rock.Handle(_ => _.Use(Arg.IsAny<int>()));
``` 

In this case, the mock is expecting that `Use()` will be called once, passing in any integer value. `IsAny()` isn't actually invoked; the expression passed into `Handle()` is parsed by Rocks to determine what the expectation is for the mock.

Therefore, what the methods on `Arg` do is pretty much immaterial. To turn off nullable checks, you can use the `#nullable` directive:

```c#
#nullable disable
public static class Arg
{
  public static T Is<T>(Func<T, bool> evaluation)
  {
    if (evaluation == null)
    {
      throw new ArgumentNullException(nameof(evaluation));
    }

    return default;
  }

  public static T IsAny<T>() => default;

  public static T IsDefault<T>() => default;
}
#nullable enable
``` 

With this in place, the three `CS8603` errors go away. While I'm OK with using this directive in this case, I'd recommend only using it sparingly. By disabling null checking, you're essentially lessening the worth of having the compiler feature in the first place. If you end up using this directive everywhere, you should probably turn the compiler feature off.

## Generating Code with Nulls

One aspect of Rocks is that it generates C# code that represents the mock class. Therefore, I needed to have the Compiler API configured such that it was targeting C#8 and it had the nullable feature enabled. The first part is done in `Builder.MakeTree()`. It requires the `languageVersion` of a `CSharpParseOption` set to `CSharp8`:

```c#
var options = new CSharpParseOptions(languageVersion: LanguageVersion.CSharp8)
``` 

This object is then passed to `SyntaxFactory.ParseSyntaxTree()`:

```c#
SyntaxFactory.ParseSyntaxTree(@class, options: options)
``` 

Enabling the nullable feature is done in `Compiler.Compile()`. The `CSharpCompilationOptions` passed to `CSharpCompilation.Create()` must have `nullableContextOption` set to `Enable`:

```c#
var options = new CSharpCompilationOptions(
  outputKind: OutputKind.DynamicallyLinkedLibrary,
  optimizationLevel: this.Optimization == OptimizationSetting.Release ?
    OptimizationLevel.Release : OptimizationLevel.Debug,
  allowUnsafe: this.AllowUnsafe,
  nullableContextOptions: NullableContextOptions.Enable)
``` 

Doing this uncovered some errors that were in the generated code. The hardest issue to overcome was overriding members that had parameters and/or return values that were nullable. Let's say you have an interface defined as follows:

```c#
public interface IHaveNullables
{ 
  void DoSomething(string? data);
}
``` 

You can implement the interface like this:

```c#
public sealed class HaveNullables : IHaveNullables
{
  public void DoSomething(string data) { }
}
``` 

However, you'll get a `CS8614` warning if you do. The parameter should really be `string?`. However, figuring out when parameters are nullable is not easy. There's no base "nullable" type for nullable references like there is for nullable value types. All the nullable information is stored in a `NullableAttribute` value associated with the parameter. They're essentially byte values that describe what "parts" of a type are nullable and which aren't. Thankfully, Jon Skeet has done some investigative work on his own describing how to map the values to the type parts. However, as of right now, `NullableAttribute` isn't a type that's exposed to you; it's created on the fly and injected into the assembly. Therefore, you can't easily use Reflection to find that attribute and those byte values, though it's doable. Hopefully before C#8 is released, `NullableAttribute` will be a type that exists that you can easily use.

All that said, once I figured out where the nullable metadata was stored, I created a `NullableContext` class that is used primarily in the `GetFullName()` extension method I created for types. This allows me to determine how to create a type name with the "?" in the right spots, even if generics come into play. This took a fair amount of time to wrestle with, but thankfully a solution is in place now.

## Testing and Verification

Once I got Rocks to compile, I ran all the tests to see what was failing. I expected tests to fail after doing this level of surgery to the code, and sure enough, some did. Some failures were easy to fix, but there were a couple that were harder to unravel. One had to do with `HandlerInformation<T>` and `MethodAdornments<T>`. Originally, the `ReturnValue` property was mutable, and the compiler was having an issue with this. My original change was to make a `WithReturnValue()` method that would return a new version of `HandlerInformation<T>` with the return value set, make `ReturnValue` read-only, and use `#nullable disable` around the entire class.

Unfortunately, this didn't work. `AsyncTests.RunAsyncSynchronously()` was failing because I was using `Returns()` to set the `Task` explicitly. What I didn't realize is I have shared state between the `HandlerInformation<T>` reference the mock knows about, and the reference that `MethodAdornments<T>` has. When I would change that field within `MethodAdornments<T>` when `WithReturnValue()` was called, the mock wouldn't know about it and still think `ReturnValue` was `null`, which would cause the test to fail.

To fix this would take a fair amount of time, and I made the call to go back to the original design. It's not ideal, and at some point, I'll figure out how to address this correctly. For now, the impact is minimal and is completely managed within Rocks. Such is life within software development. I strive to have clean, manageable code, but this must be balanced with the time to invest in making a change.

## Conclusion

In this article, I discussed the steps I took in converting a C# project such that it used the non-nullable reference type compiler feature in C#8. There were over 80 errors related to this feature that I had to address in some way to get the project to compile successfully. Unraveling all of them was not a trivial endeavor, and Rocks is not that large of a project. Code bases that have been around for a long time and/or have a significant amount of code may take a significant amount of effort to change the code.

My suggestion would be to have this feature on for any new projects in C#8 and consider turning it on for projects already in-flight. The feature has a lot of worth, but for projects that are well-tested, stable, have been used in production, etc., it may not be worth doing this for every line of code in the project. You may want to consider turning it on sparingly for specific sections of the code base using the #nullable directive and increase that scope over time.

I'm very happy non-nullable reference types are now in C#8, and I plan on using them as much as I can from this point on. I'm curious to get your take on this feature – send me your thoughts to jasonb@magenic.com. Try it out!

Thanks to the following Magenicons that reviewed this article and provided feedback and suggestions: Caleb Kapusta, Mike McCaughan, Jacob Maristany, and Rocky Lhotka.

## Update #1 (7/19/2019)

Recently there have been some changes around how nullability works in C#. This isn't unexpected, as C#8 is still in preview mode. One change that broke my code in Rocks was introducing `NullableContextAttribute`, which changed the byte flags that are stored in `NullableAttribute`. Fortunately, there's documentation on these attributes, which you can find here. Rocks has been updated to handle these changes, so head on over to the repository to see what's been done. 

## References
* [Introducing Nullable Reference Types in C#](https://devblogs.microsoft.com/dotnet/nullable-reference-types-in-csharp/)
* [Nullable reference types](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/null-safety/nullable-reference-types)

> Published: 05.28.2019