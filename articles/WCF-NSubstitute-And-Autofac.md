---
title: WCF, NSubstitute, and Autofac
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# WCF, NSubstitute, and Autofac

Recently I've been working on the code samples for an upcoming 1/2-day course I'm giving at HDC. While reviewing some code, I stopped on this method:

```c#
public ReadOnlyCollection<string> GetGroups(IIdentity identity)
{
  var channel = new ChannelFactory<IGroupsService>().CreateChannel();

  return channel.Use<ReadOnlyCollection<string>>(() =>
  {
    return new ReadOnlyCollection<string>(
      (from response in channel.RetrieveGroups(
        new RetrieveGroupsRequest[] { new RetrieveGroupsRequest(identity.Name) })
      where response.Request.UserName == identity.Name
      select response.Groups).First());
  });
}
```

What I didn't like about it was the fact that within the method, I was getting a reference to a `IGroupsService`-based object via `ChannelFactory`. That smelled like a coupling I could break by adding some dependency injection (like I'm doing with the identity argument). But here's another problem. `Use()` is an hand-rolled extension method I use a lot in WCF-based code as it wraps code blocks that use a service and "properly" shuts down the service. Now, `ChannelFactory` will return a proxy based on the interface given, but it also has that proxy implement `ICommunicationObject`. So, my extension method is typed for `object` and then does a check to ensure that the given reference is of an `ICommunicationObject` type (if it doesn't, it throws an exception). Using typical mocking libraries like `RhinoMocks`, I can't give it a set of interfaces and have it cruft up a mock or stub that implements all those interfaces [1]. Sure, I could create a helper interface:

```c#
public interface IGroupsCommunicationObjectService : IGroupsService, ICommunicationObject { }
```

And feed that into my mocking framework of choice, but that feels klutzy.

So, could [NSubstitute](https://nsubstitute.github.io/) and [Autofac](https://github.com/autofac/Autofac) help out here?

See, part of the reason I'm revisiting the code samples is to swap out RhinoMocks and [Castle Windsor](https://www.castleproject.org/projects/windsor/) for NSubstitute and Autofac. I have nothing against my original frameworks of choice, but I've been reading good things about the new kids on the block, and I wanted to give them a test drive. Plus, I wanted my attendees to use a library that wasn't as well-known as the popular choices I originally made. As you'll see, NSubstitute and Autofac have features that made my changes easy to do.

First, let's change my method around a bit:

```c#
public ReadOnlyCollection<string> GetGroups(IIdentity identity, 
  IGroupsService service)
{
  return service.Use<ReadOnlyCollection<string>>(() =>
  {
    return new ReadOnlyCollection<string>(
      (from response in service.RetrieveGroups(
        new RetrieveGroupsRequest[] { new RetrieveGroupsRequest(identity.Name) })
      where response.Request.UserName == identity.Name
      select response.Groups).First());
  });
}
```

That's the easy part. Now let's see what the happy path test looks like for `GetGroups()`:

```c#
[TestMethod]
public void GetGroups()
{
  var name = Guid.NewGuid().ToString("N");
  var group = Guid.NewGuid().ToString("N");

  var identity = Substitute.For<IIdentity>();
  identity.Name.Returns(name);

  var channel = Substitute.For<IGroupsService, ICommunicationObject>();
  channel.RetrieveGroups(Arg.Any<RetrieveGroupsRequest[]>()).Returns(info =>
  {
    var request = (info[0] as RetrieveGroupsRequest[])[0];
    Assert.AreEqual(name, request.UserName);
    return new RetrieveGroupsResponse[] { 
      new RetrieveGroupsResponse(request, new string[] { group }) };
  });

  var repository = new GroupsRepository();
  var results = repository.GetGroups(identity, channel);
    
  Assert.AreEqual(1, results.Count);
  var result = results[0];
  Assert.AreEqual(group, result);
  channel.Received().RetrieveGroups(Arg.Any<RetrieveGroupsRequest[]>());
}
```

One thing I really like about NSubstitute is that it completely ditches the epic fakes/stubs/mocks/test-double war that's been tearing apart the software development community for centuries. OK, I kid, but honestly, I really never understood why there's been so much made over the terminology. I get the differences, but it seemed to make things more confusing and less pragmatic. NSubstitute just creates "substitutes", and I really like that term. You can create expectations if you want (that's what `Received()` does), or you don't have to. It's all up to you.

Furthermore, NSubstitute makes it brutally simple to make a substitute that implements multiple interfaces. The returned substitute is strongly-typed for the first interface listed in the generic definitions, but trust me, it implements `ICommunicationObject`, and that's exactly what I want. This also has a nice unexpected benefit in that this also act just like `CreateChannel()` does from `ChannelFactory` - i.e. it acts like the interface you give it, but it also is `ICommunicationObject`-based - you just don't "see" that.

OK, so I have my unit test in place. In the real world, how do I resolve my dependencies? In my test, I'm not using a container to resolve the dependencies, so let's see Autofac in action with the "real" service creation:

```c#
var builder = new ContainerBuilder();
builder.Register(_ => 
  (new ChannelFactory<IGroupsService>()).CreateChannel()).As<IGroupsService>();
var container = builder.Build();
```

Beautiful. Autofac allows you to define how the dependencies will be created via anonymous methods, so it's very simple to create the channel via normal WCF means. This doesn't even begin to show the power and flexibility that Autofac has - you really owe it to yourself to dive into Autofac's API.

By the way, if I wanted to use my channel substitute in the container, I'd just do something like this:

```c#
var builder = new ContainerBuilder();
builder.RegisterInstance<IGroupsService>(channel);
var container = builder.Build();
```

I hope this post has showed you some of the cool things you can do with NSubstitute and Autofac. I'm sold on them (until something cooler comes along :) ).

[1] Or at least I couldn't find the magic API to do it.

> Published: 08.24.2010 10:38:18 PM CST