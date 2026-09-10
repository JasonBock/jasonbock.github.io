---
title: Exploring Actors with Orleans – Getting Started
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Exploring Actors with Orleans – Getting Started

Synopsis: In this blog post, we'll create a simple actor in Orleans. We'll see what grains are and how they are defined. We'll get a grain hosted in Orleans and invoke it to see it in action. This will focus primarily on getting the actor to do something – subsequent articles will dive into specific aspects of Orleans.

## Grains and Silos in Orleans

Before I show any code, it's important to state how Orleans defines actors as the terminology is different:

* An actor is called a **grain**
* An actor system is called a **silo**

On the Orleans Gitter channel, I asked why these names were used, and Reuben Bond, a developer on the Orleans team, said this:

> It's related to the original analogue, where grains are hydrated (made active) and dehydrated (deactivated) and kept in silos. Calling them Actors is inaccurate, not because of the original Carl Hewitt formulation, but because of the Erlang (and subsequently Akka) interpretation of that formulation.

This differentiation will become clearer the more you use Orleans, especially if you compare it to other actor systems. I'll point out some of these aspects out in future articles in this series. In any event, keep this naming convention mind as you start to work with Orleans as you'll hear "actors" defined as "grains". In fact, as you're about to see, defining grains in Orleans requires you to inherit from specific types that have the word "grain" in them.

## Creating Echoes

Let's create a very simple grain and get it to do something. In this example, we'll create an "echo" grain that will get a name from the sender of the given message and print it to the console window a specified number of times. Note that you can find the complete example [here](https://github.com/JasonBock/ExploringActorsWithOrleans) (under the `Echo` folder). Keep this GitHub repo in mind as all of the source code examples for my Orleans articles will be found here. Each folder will relate to code used in a specific article. I didn't want to keep building on an example from article to article as that can get confusing and hard to do for no good reason. That said, you may see some duplication between folders, but that's OK.

To start, we'll create four different projects. Note that I'm using VS 2017, all of the projects are targeting .NET Core 2.0, and each project has `<LangVersion>` set to latest:

* `Echo.Contracts`
* `Echo.Grains`
* `Echo.Host`
* `Echo.Client`

> Note: At the time of writing this article, the 2.0.0-beta1 version of Orleans was available. A lot of changes were in flight with this release, primarily the effort to port Orleans to .NET Standard, but other API activity was in the works as well. What you see in this article may change when the final release is made public. 

The names should give strong hints as to what they're going to contain, and once we add code it'll make it crystal clear. Also, here's a diagram of the dependencies between the projects:

![Orleans - Getting Started](https://jasonbock.net/images/Orleans-GettingStarted-1.png "Orleans - Getting Started")

Now, it's not strictly necessary to have four projects just to do something in Orleans, but for now, we'll do it this way as it'll demonstrate that each project has a specific part to play in the overall system.

Let's start with `Echo.Contracts`. We'll create one grain interface, `IEchoGrain`:

```c#
public interface IEchoGrain
  : IGrainWithGuidKey
{
  Task SpeakAsync(EchoSpeakMessage message);
}
``` 

The provided message is shaped like this:

```c#
public sealed class EchoSpeakMessage
{
  public string Message { get; set; }
  public uint Repeat { get; set; }
}
``` 

Note that `IEchoGrain` inherits from the `IGrainWithGuidKey` interface and that our methods are asynchronous by definition. Don't worry about keys with grains right now; in a future article I'll talk about grain identity in greater detail. That interface comes from the `Microsoft.Orleans.Core` NuGet package.

Next, let's create an implementation of `IEchoGrain`. This class, `EchoGrain`, exists in `Echo.Grains`:

```c#
public sealed class EchoGrain
  : Grain, IEchoGrain
{
  public async Task SpeakAsync(EchoSpeakMessage message)
  {
    if (message == null) { throw new ArgumentNullException(nameof(message)); }

    for (var i = 0; i < message.Repeat; i++)
    {
      await Console.Out.WriteLineAsync(
        $"{i} - {message.Message}");
    }
  }
}
``` 

`EchoGrain` derives from both `IEchoGrain` and `Grain`. These base type definitions are critical in Orleans to have in place – if you don't, Orleans won't be able to manage your grain types correctly and bad things will happen at runtime. As you can see, our implementation is pretty simplistic. We repeat the given message the specified amount in the `EchoSpeakMessage` to the console window.

At this point, we need to get our host application up and running so we have a live silo that applications can send messages to our `EchoGrain`. Here's how it's done in `Main()` for the `Echo.Host` project:

```c#
public static async Task Main(string[] args)
{
  await Console.Out.WriteLineAsync($"Orleans silo is starting...");

  var configuration = ClusterConfiguration.LocalhostPrimarySilo();
  var builder = new SiloHostBuilder()
    .UseConfiguration(configuration)
    .AddApplicationPartsFromReferences(typeof(EchoGrain).Assembly);

  var host = builder.Build();
  await host.StartAsync();

  await Console.Out.WriteLineAsync($"Orleans silo is available.");
  await Console.Out.WriteLineAsync($"Press Enter to terminate...");
  await Console.In.ReadLineAsync();

  await host.StopAsync();
  await Console.Out.WriteLineAsync("Orleans silo is terminated.");
}
``` 

The `ClusterConfiguration` class is in the `Microsoft.Orleans.Server` NuGet package so you'll need to install that one. In a future article, I'll talk about different configuration options in Orleans, but for now, we'll keep it simple. As you can see, it doesn't take much to get a silo in place. Note that you use `AddApplicationPartsFromReference()` so Orleans knows where your grains are. This is different in 2.0.0; previous versions would essentially do directory scans to find the grains. This approach is much faster.

Finally, we need a client to invoke the grain:

```c#
public static async Task Main(string[] args)
{
  await Console.Out.WriteLineAsync(
    "Client is connecting...");

  var client = await Program.StartClientWithRetries();

  await Console.Out.WriteLineAsync(
    "Client has connected.");
  await Console.Out.WriteLineAsync(
    "Enter your message in the following format: {Repeat},{Message}.");
  await Console.Out.WriteLineAsync(
    "Example: 3,Hello");
  await Console.Out.WriteLineAsync(
    "Enter STOP to shutdown the client.");

  var grainId = Guid.NewGuid();

  while (true)
  {
    var content = await Console.In.ReadLineAsync();

    if(content == Program.TerminationMessage)
    {
      break;
    }

    var (success, message) = await Program.TryCreateMessageAsync(content);

    if (success)
    {
      var echoGrain = client.GetGrain<IEchoGrain>(grainId);
      await echoGrain.SpeakAsync(message);
    }
  }
}
``` 

The only NuGet package you'll need on the client to use `IClusterClient`, which is the return type from `StartClientWithRetries()` is in `Microsoft.Orleans.Core`.

Let's go through the client code in detail. First, we don't want to start calling the grain until we know the silo is ready to go. This is what `StartClientWithRetries()` does:

```c#
private static async Task<IClusterClient> StartClientWithRetries(
  int initializeAttemptsBeforeFailing = Program.RetryAttempts)
{
  var attempt = 0;
  IClusterClient client;

  while (true)
  {
    try
    {
      var configuration = ClientConfiguration.LocalhostSilo();
      client = new ClientBuilder()
        .UseConfiguration(configuration)
        .AddApplicationPartsFromReferences(typeof(IEchoGrain).Assembly)
        .Build();

      await client.Connect();
      Console.WriteLine("Client successfully connect to silo host");
      break;
    }
    catch (SiloUnavailableException)
    {
      attempt++;

      await Console.Out.WriteLineAsync(
        $"Attempt {attempt} of {initializeAttemptsBeforeFailing} failed to initialize the Orleans client.");

      if (attempt > initializeAttemptsBeforeFailing)
      {
        throw;
      }

      await Task.Delay(Program.Retry);
    }
  }
}
``` 

It uses the retry pattern to connect to the remote silo. Once the connection is established, we can start sending messages. The client app then takes comma-delimited input, parses it into an instance of `EchoSpeakMessage`, and then send it over the grain. If the input is "STOP", then the client application terminates.

The last part is to configure VS to launch both the host and the client applications:

![Orleans - Getting Started](https://jasonbock.net/images/Orleans-GettingStarted-2.png "Orleans - Getting Started")

If all goes well, you should see something like this on your computer:

![Orleans - Getting Started](https://jasonbock.net/images/Orleans-GettingStarted-3.png "Orleans - Getting Started")

## Conclusion

In this article, I went through the basics of creating a simple grain in an application. I demonstrated how that grain is hosted in a silo and how a client can call it. This wasn't a long, in-depth article per-se, and that's on purpose. Getting these core concepts in place will help as I explore other aspects of grains in subsequent articles. Stay tuned – in the next article, I'll cover failures and exceptions in Orleans. Until then, happy coding!

> Published: 11.14.2017