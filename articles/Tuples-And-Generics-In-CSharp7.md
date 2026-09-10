---
title: Tuples and Generics in C#7
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Tuples and Generics in C#7

## Synopsis

In C#7, tuples have been added to language to provide a concise way to group values together without having to explicitly define specific types. In this article, you'll see how you can combine tuples and generics together.

## Passing Messages

In modern applications, it's common for designs to provide a way for a class to specify which messages it will support and what it will do when it encounters that message. For example, in Caliburn.Micro, a MVVM framework for XAML-based applications, view models implement the `IHandle<>` interface for all of the events the view model will respond to:

```c#
public sealed class NewCustomerMessage { } 

public sealed class CustomerViewModel
  : IHandle<NewCustomerMessage>
{
  public CustomerViewModel(IEventAggregator events)
  {
    events.Subscribe(this);
  }

  public void Handle(NewCustomerMessage message)
  {
    // React to a new customer event...
  }
}
```

In this view model, the `CustomerViewModel` tells the `EventAggregator` that it wants to subscribe to events. Since it implements `IHandle<NewCustomerMessage>`, it will be notified whenever a `NewCustomerMessage` object is sent to the `EventAggregator`. The `EventAggregator` is an object that is typically shared across all view models and is injected when the view model is constructed.

Another way to do this is through function definitions. For example, in the actor framework Akka.Net, the actor states which messages it will respond to by specifying this information in the constructor:

```c#
public sealed class CreateCustomerMessage { }

public sealed class CustomersActor
  : ReceiveActor
{
  public CustomersActor()
  {
    this.ReceiveAsync<CreateCustomerMessage>(this.Create);
  }

  private Task Create(CreateCustomerMessage message)
  {
    // Do work to create a customer...
    return Task.CompletedTask;
  }
}
```

Because the parameter type to `ReceiveAsync()` is an `Func<T, Task>`, you can either specify a method defined on the actor class, or you can define an anonymous method – either one will work.

In both cases, you can see that generics are used judiciously to make it clear which message types the developer wants to support. Let's backtrack for a second and think about how we'd have to do this before generics were introduced in C# in version 2. Here's a simplified view model class done in C# 1.0 that would handle events similar to the Caliburn.Micro approach:

```c#
public interface IHandle
{
  void Handle(object message);
}

public sealed class CustomerViewModel
  : IHandle
{
  public CustomerViewModel(IEventAggregator events)
  {
    events.Subscribe(this);
  }

  public void Handle(object message)
  {
    var newCustomer = message as NewCustomerMessage;

    if(newCustomer != null)
    {
      // Do something with handling a new customer.
      return;
    }
  }
}
```

Since we have no way to specify the message type, we have to use switching logic to check the type of the message. A similar problem exists with methods – here's an example if we tried to create an actor framework like Akka.Net without generics:

```c#
public abstract class ReceiveActor
{
  public delegate Task ReceiveAsyncHandler(object message);
  protected void ReceiveAsync(ReceiveAsyncHandler handler) { }
}

public sealed class CustomersActor
  : ReceiveActor
{
  public CustomersActor()
  {
    this.ReceiveAsync(this.Create);
  }

  private Task Create(object message)
  {
    var createCustomer = message as CreateCustomerMessage;

    if(createCustomer != null)
    {
      // Do something to create a customer.
      return Task.CompletedTask;
    }

    throw new NotSupportedException();
  }
}
```

Adding generics helps by removing the need for casts and thereby increasing program safety. They also provide a clean mechanism for extensibility in frameworks along with adding clarity to a developer's intent when using that framework.

## Using Tuples

In both cases, notice that the messages were defined explicitly by a developer. For example, with the view model scenario, a `NewCustomerMessage` type was created, which the `CustomerViewModel` used to state that it could handle messages of that type. Having well-defined messages specified in a system is a desirable aspect of an application as it's clear which messages an object or a method can handle. However, in C#7, things get interesting because tuples have been added to the language, and you can use tuple definitions as types to generics!

Now, if you're an experienced C# developer, you may think, "weren't tuples added in the 4.0 version of the .NET Framework? How is this a new thing?" The correct response to these questions is, yes, the `System.Tuple` type has been in .NET since 4.0. For example, we could use this type and change our view model such that it handles events where the message is a `Tuple<string, int, Guid>`:

```c#
public sealed class CustomerUsingTupleViewModel
  : IHandle<Tuple<string, int, Guid>>
{
  public CustomerUsingTupleViewModel(IEventAggregator events)
  {
    events.Subscribe(this);
  }

  public void Handle(Tuple<string, int, Guid> message)
  {
    // React to a new customer event...
    // but, message.Item1, message.Item2 ... ewwwww!
  }
}
```

This will work, but notice the comment in `Handle()`. If you want to use any of the actual values passed by the message, you have to use `Item1`, `Item2` and `Item3`, and you have to remember that `Item1` is a `string`, `Item2` is an `int`, and `Item3` is a `Guid`. This code can be hard to maintain in the future, and frankly it just doesn't look very elegant.

In C#7, things change in a big way. There is a new type called `System.ValueTuple`, and the language added first-class syntax support for this type. Here's how you can create tuples:

```c#
var message = ("a", 2, Guid.NewGuid());
var namedMessage = (Name: "a", Age: 2, Id: Guid.NewGuid());
(var Name, var Age, var Id) = ("a", 2, Guid.NewGuid());
```

Note that for the message variable, there will still be properties like `Item1`, `Item2` and `Item3`. However, you can define the tuple with explicit names for the fields as shown in the second line of code. You can also assign the contents of a tuple into specific variables as the third line of code does – this is known as deconstruction. Furthermore, you can include names for the tuple so users don't have to deal with "Item" properties. For example, you can return multiple values from a method like this:

```c#
private static (string Name, int Age, Guid Id) CreateMessage()
{
  return ("a", 2, Guid.NewGuid());
}

var createdMessage = CreateMessage();
var name = createdMessage.Name;
```

So how does this work with generics? Let's update our view model so it takes a tuple type rather than an explicit message type:

```c#
public sealed class CustomerViewModel
  : IHandle<(string Name, int Age, Guid Id)>
{
  public CustomerViewModel(IEventAggregator events)
  {
    events.Subscribe(this);
  }

  public void Handle((string Name, int Age, Guid Id) message)
  {
    // Do something with handling a new customer.
  }
}
```

Note that if you name the members of the tuple in the generic definition, they must come along wherever that is used. In other words, you can't implement `Handle()` like this:

```c#
// This is an error
public void Handle((string, int, Guid) message)
```

Conversely, if you don't name the tuple members in the generic definition, you can't name them in the members where that tuple type is used.

This technique also works in Akka.Net:

```c#
public sealed class CustomersActor
  : ReceiveActor
{
  public CustomersActor()
  {
    this.ReceiveAsync<(string, int, Guid)>(this.Create);
  }

  private Task Create((string, int, Guid) message)
  {
    // Do work to create a customer... 
    return Task.CompletedTask;
  }
}
```

As you can see, you can easily use tuples in generic definitions and get all the benefits of tuples without having to generate specific types all the time. This doesn't mean you should do this in all cases. Arguably it's a good idea to have explicit message types so your intent is clear in terms of what kind of message you're passing around. For example, if I have a `NewCustomerMessage`, the name is a good indicator that I want to create a new customer. However, if I see `(string, int, Guid)`, it's rather unclear what the contents of that message should do.

Take care in using this approach. A good team may be able to work effectively with "nameless" messages as it's still clear that a particular message handler needs a `ValueTuple` of a specific shape, but it may also lead to maintenance issues in the future. Furthermore, there's no way to differentiate between tuples that have the same shape. It's possible I may want to handle two messages where both take a `(Guid, string)`, but based on the message sent I need to do two different things. With a tuple, you couldn't distinguish two messages of type `(Guid, string)`, even if you provided names to the tuple members.

## Serialization Concerns

One final aspect to tuples in C#7 that you should keep in mind: they are currently not serializable. For example, this code will fail with a `SerializationException` when `Serialize()` is called:

```c#
var formatter = new BinaryFormatter();

var message = ("a", 2, Guid.NewGuid());

using (var stream = new MemoryStream())
{
  formatter.Serialize(stream, message);
  stream.Position = 0;
  var newMessage = ((string, int, Guid))formatter.Deserialize(stream);
}
```

If you plan on passing tuples across any kind of boundary that would require its contents to be serialized (e.g. between `AppDomain`s or different machines) the tuple approach won't work. You'll have to use custom types that you've defined that are serializable, or package the `ValueTuple` into a `Tuple` type, and then deconstruct it back to the desired `ValueTuple` shape on deserialization.

> Note: There's an issue in GitHub for the CoreFx team to change the tuple type such that it is serializable – click on this link for details: https://github.com/dotnet/corefx/issues/15229.

## Conclusion

In this article, you've seen how modern applications and frameworks in C# use messages passing to handle workflows and events in a software system. In C#7, tuples can also be used in these frameworks even when generics come into play, thereby reducing the need to define messages explicitly. I hope that this has been helpful, and until next time – happy coding!

> Published: 01.19.2017