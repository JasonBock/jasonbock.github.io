---
title: WCF, NetMsmqBinding, and "The signature is invalid"
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# WCF, NetMsmqBinding, and "The signature is invalid"

Today I was trying to get my service to work in a test, and it used `NetMsmqBinding`. Here's the initialization:

```c#
public sealed class DemoTests
{
  private static string Address;
  private static string QueueName;

  [ClassInitialize]
  public static void ClassInitialize(TestContext context)
  {
    DemoTests.QueueName = Guid.NewGuid().ToString("N");
    MessageQueue queue = MessageQueue.Create(
      @".\Private$\" + DemoTests.QueueName);
    DemoTests.Address = 
      "net.msmq://localhost/private/" + DemoTests.QueueName;
  }
}
```

You can assume there's a `ClassCleanup` attributed method to delete the queue. Here's the test method:

```c#
[TestMethod]
public void ManageMessage()
{
  using (ServiceHost host = new ServiceHost(typeof(DemoService)))
  {
    NetMsmqBinding binding = new NetMsmqBinding();
    binding.Durable = false;
    binding.ExactlyOnce = false;

    host.AddServiceEndpoint(typeof(IDemoService),
      binding, DemoTests.Address);
    host.Open();

    using (ChannelFactory<IDemoService> factory =
      new ChannelFactory<IDemoService>(binding,
      new EndpointAddress(DemoTests.Address)))
    {
      IDemoService channel = factory.CreateChannel();
      DemoRequest request = new DemoRequest();
      request.StringData = "Data";
      channel.Manage(request);
    }
  }
}
```

Now, once `Manage()` was invoked, I would get a message in the dead-letter messages, saying "The signature is invalid". There was a link I found that mentioned this error, and it contains a subtle clue:

> However I had failed to create the MSMQ queue with the "Authenticated" option turned off! Doh.

That means I needed to do this:

```c#
[TestMethod]
public void ManageMessage()
{
  using (ServiceHost host = new ServiceHost(typeof(DemoService)))
  {
    NetMsmqBinding binding = new NetMsmqBinding();
    binding.Durable = false;
    binding.ExactlyOnce = false;
    binding.Security.Mode = NetMsmqSecurityMode.None;

    host.AddServiceEndpoint(typeof(IDemoService),
      binding, DemoTests.Address);
    host.Open();

    using (ChannelFactory<IDemoService> factory =
      new ChannelFactory<IDemoService>(binding,
      new EndpointAddress(DemoTests.Address)))
    {
      IDemoService channel = factory.CreateChannel();
      DemoRequest request = new DemoRequest();
      request.StringData = "Data";
      channel.Manage(request);
    }
  }
}
```

Just turn off security! Once I did this, all was well with the world.

> Published: 01.19.2007 01:22:59 PM CST