---
title: Mocking Injected Types in Blazor
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Mocking Injected Types in Blazor

Synopsis: [Blazor](https://dotnet.microsoft.com/en-us/apps/aspnet/web-apps/blazor) is a new SPA framework from Microsoft, which allows a developer to write web applications in C#. In this article, we'll cover testing strategies for Blazor-based code, focusing on how to handle the core Blazor types that are set up for injection out of the box.

## Building Testable Code

Writing tests for code is a requirement in my book. The patterns, frameworks, and strategies developers use are important, but what I'm far more interested in is that there are good tests in place for a code base. If I need to do an assessment on an application, one of the things I look for is tests. Granted, just because tests are in place doesn't mean that the code base is healthy. The tests may be done poorly, or large areas of the code are not being tested. But having tests in place is a positive sign to me.

If you typically write code in C#, you may have heard of a new UI framework from Microsoft called Blazor. I've been watching its evolution ever since it started, and I'm excited about it. Of course, code written for Blazor needs to be tested just like everything else, so, how can you test Blazor code?

### Using End-to-End Testing

One way to test web applications is to take a front-end, end user perspective. That is, test the application the same way the user would – for example, by clicking on links and entering data into input elements. Frameworks like [Selenium](http://www.selenium.dev/) can be used to automate this process, and since Blazor is used for web applications, this just works. We won't dive into Selenium testing in this article.

### Using Component-Based Testing

Blazor is all about building components that can be used in an application. These components are either written in Razor syntax or directly in C#. Either way, the behavior of the components should be tested to ensure they work as expected. There's work being done by the Blazor team to make component testing in unit testing scenarios easier as well as other related implementations, like [bUnit](https://github.com/bUnit-dev/bUnit).

### Separating Logic into View Models

Blazor components have different directives a developer can use to specify binding expressions and supported routes. One is called `@code`, which embeds C# code directly into the component. Developers may choose to move this code into its own class, like a MVVM design where this code exists in a view model. Arguably, this provides a better separation of concerns, and enables code to be testable without specialized frameworks.

## Injectable Types in Blazor

Blazor provides dependency injection [out of the box](https://learn.microsoft.com/en-us/aspnet/core/blazor/fundamentals/dependency-injection?view=aspnetcore-10.0). Developers can specify their own dependencies such that they can be injected into code, but Blazor configures three dependencies for you: `HttpClient`, `IJSRuntime`, and `NavigationManager`. For example, if a developer needs to navigate to a new route in a component, they can add this to their component:

```c#
@inject NavigationManager manager
```

Then `manager` is a field that is automatically created and wired up for the developer to use anywhere within the component.

For front-end testing and component testing frameworks, these dependencies are set up automatically. However, if you write code that needs to use these types, you need to provide something that implements these types. In the next section, we'll talk about strategies you can use to write tests that use these types.

## Mocking Strategies
In this section, we'll cover the three main types that Blazor injects and how you can handle them in unit testing scenarios.

### `HttpClient`

It's a bit unfortunate that Blazor has `HttpClient` configured as a dependency, rather than a type that can be easily mocked such as `IHttpClientFactory`. `HttpClient` is configured as a singleton – for typical Blazor applications, that's a straightforward approach at runtime. Writing tests that use `HttpClient` can be challenging as it's a non-abstract class, but [well-known techniques exist](https://gingter.org/2018/07/26/how-to-mock-httpclient-in-your-net-c-unit-tests/) to address that, so I won't revisit them here.

### `IJSRuntime`

`IJSRuntime` is an interface, so it's easily mockable in tests. For example, if I had code that called `InvokeAsync()`, I could write a test that mocks `IJSRuntime` and verify that `InvokeAsync()` is used as expected. However, there's a catch if I want to call a method that doesn't return anything:

```c#
using Microsoft.JSInterop;
using System;
using System.Threading.Tasks;

namespace MockingBlazorDependencies
{
  public sealed class UsesIJSRuntime
  {
    private readonly IJSRuntime runtime;

    public UsesIJSRuntime(IJSRuntime runtime) =>
      this.runtime = runtime ?? throw new ArgumentNullException(nameof(runtime));

    public async Task UseAsync() =>
      await this.runtime.InvokeVoidAsync("method", 2, "value");
  }
}
```

In this case, `InvokeVoidAsync()` doesn't actually exist on `IJSRuntime`. It's an extension method that is defined on `JSRuntimeExtensions`:

![Mocking Blazor](https://jasonbock.net/images/Mocking-Blazor-1.png "Mocking Blazor")

So, is it possible to write a test in this case? It is:

```c#
[Test]
public static async Task CallUseAsync()
{
  var runtime = Rock.Create<IJSRuntime>();
  runtime.Handle(
    _ => _.InvokeAsync<object>("method", Arg.IsAny<object[]>()))
    .Returns(new ValueTask<object>());

  var uses = new UsesIJSRuntime(runtime.Make());
  await uses.UseAsync();

  runtime.Verify();
}
```

The key takeaway in this test is to notice that we expect `InvokeAsync()` to be called when we call `InvokeVoidAsync()`. This is somewhat unfortunate in that we're relying upon the behavior defined in `JSRuntimeExtensions` for this to work, and this might change in the future. However, at least we can write tests for code that relies on `IJSRuntime`.

## `NavigationManager`

The last type to discuss is somewhat problematic. It's an abstract class with only two virtual members: `EnsureInitialized()` and `NavigateToCore()`. It's not enough to use a mocking framework that allows you handle these two members. You need to look at how `NavigationManager` is implemented to ensure it's initialized properly. Here's what I did. It's not hard, but it took a while to get to this point:

```c#
public sealed class MockNavigationManager
  : NavigationManager
{
  public MockNavigationManager() : base() => 
    this.Initialize("http://localhost:2112/", "http://localhost:2112/test");

  protected override void NavigateToCore(string uri, bool forceLoad) => 
    this.WasNavigateInvoked = true;

  public bool WasNavigateInvoked { get; private set; }
}
```

The key is the call to `Initialize()` in the constructor. This sets up values in the base type such that things will work as expected, and yes, you must set the second argument, `uri`, such that it starts with the same value you pass into the first argument, `baseUri`. You'll get an exception if you don't do this. Also, if you don't call `Initialize()`, you'll get errors when you call `NavigateTo()`. Given that this work needs to happen within the constructor, I decided to hand-roll the mocked type, rather than trying to figure out how to get this work within a mocking framework.

With this setup in place, I can now pass in a `MockNavigationManager` instance to a method that needs `NavigationManager`. For example, if I have this code that needs a `NavigationManager` instance:

```c#
public sealed class UsesNavigationManager
{
  private readonly NavigationManager manager;

  public UsesNavigationManager(NavigationManager manager) =>
    this.manager = manager ?? throw new ArgumentNullException(nameof(manager));

  public void Use() =>
    this.manager.NavigateTo("/NewRoute");
}
```

I can write a test like this:

```c#
[Test]
public static void CallUse()
{
  var manager = new MockNavigationManager();
  var uses = new UsesNavigationManager(manager);
  uses.Use();

  Assert.That(manager.WasNavigateInvoked, Is.True);
}
```

This is a simple example, and you may want to do more validation on the expected method invocation and its parameter values, but it illustrates how you can use `NavigationManager` in your tests.

There's another way to handle this, which is to use a decorator (this was mentioned in the articles cited in the `HttpClient` section). Essentially, create an interface that you own, and wrap the implementation at runtime with the right type:

```c#
public interface INavigationManager
{
  void NavigateTo(string uri);
}

public sealed class NavigationManagerDecorator
  : INavigationManager
{
  private readonly NavigationManager manager;

  public NavigationManagerDecorator(NavigationManager manager) =>
    this.manager = manager ?? throw new ArgumentNullException(nameof(manager));

  public void NavigateTo(string uri) => 
    this.manager.NavigateTo(uri);
}
```

With this in place, I can change my code to this:

```c#
public sealed class UsesINavigationManager
{
  private readonly INavigationManager manager;

  public UsesINavigationManager(INavigationManager manager) =>
    this.manager = manager ?? throw new ArgumentNullException(nameof(manager));

  public void Use() =>
    this.manager.NavigateTo("/NewRoute");
}
```

Now I can use a mocking framework in a test with ease. I no longer need to know the implementation details of `NavigationManager`; I just need to register `NavigationManagerDecorator` with the IoC container. In a Blazor WebAssembly application, you'd register this dependency like this:

```c#
public class Program
{
  public static async Task Main(string[] args)
  {
    var builder = WebAssemblyHostBuilder.CreateDefault(args);
    builder.RootComponents.Add<App>("app");
    builder.Services.AddTransient<INavigationManager, NavigationManagerDecorator>();

    await builder.Build().RunAsync();
  }
}
```

For a Blazor Server application, it's done in Startup.cs:

```c#
public class Startup
{
  public void ConfigureServices(IServiceCollection services)
  {
    services.AddTransient<INavigationManager, NavigationManagerDecorator>();
  }
}
```

## Conclusion

In this article, I talked about testing Blazor code. I mentioned a couple of approaches you can use, and I went into detail on strategies you can use to unit test code that uses dependency types in Blazor. I hope you find this useful. Happy coding!

> Note: You can look at the code I mentioned in this article by going to [this repo](https://github.com/JasonBock/MockingBlazorDependencies).

> Published: 03.26.2020