---
title: Painted Into a Corner By Generics
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Painted Into a Corner By Generics

Recently I thought I had a wonderful idea to refactor our code to use generics. At the end of the day, I still like it, but I got into a slight pickle when I had to deal with base types. Here's a slimmed down version of what happened.

The design is to have classes act like methods. They get one parameter (the request) and return a response. Nothing new about this - it's been done before. So I cruft up a bunch of requests and responses:

```c#
public class MessageRequest { }
public class MessageResponse { }
public class GetCustomerRequest : MessageRequest { }
public class GetCustomerResponse : MessageResponse { }
public class GetOrderRequest : MessageRequest { }
public class GetOrderResponse : MessageResponse { }
```

Now, in pre-generic days, I'd create a base interface that subclasses could inherit from:

```c#
public interface IMessageMethod
{
  MessageResponse Call(MessageRequest request);
}

public class GetCustomerMessageMethod : IMessageMethod
{
  public MessageResponse Call(MessageRequest request)
  {
    GetCustomerRequest realRequest = request as GetCustomerRequest;
    return new GetCustomerResponse();
  }
}
```

I could also create a helper method where I could pass in any object that implements `IMessageMethod` and it would invoke it for me:

```c#
static void CallMessageMethod(IMessageMethod method)
{
  // Do something with method...
}
```

Again, this is a stripped-down version of what the real code is doing. There's asynchronous invocations, JSON serialization, etc. Here's how I "genericized" it:

```c#
public interface IMethod<TRequest, TResponse>
  where TRequest : MessageRequest
  where TResponse : MessageResponse
{
  TResponse Call(TRequest request);
}

public class GetCustomerMethod : IMethod<GetCustomerRequest, GetCustomerResponse>
{
  public GetCustomerResponse Call(GetCustomerRequest request)
  {
    return new GetCustomerResponse();
  }
}
```

What's cute about this is that the argument to `Call()` is typed to `GetCustomerRequest` - I don't have to cast the argument to the type that I think I should get.

Unfortunately, I can no longer generically invoke a method like I can with `CallMessageMethod()`. In other words, I can't do this:

```c#
static void CallGenericMethod(IMethod<MessageRequest, MessageResponse> method) { }
```

Since `GetCustomerMethod` doesn't inherit from `IMethod<MessageRequest, MessageResponse>`, I can't pass in an instance of `GetCustomerMethod` to `CallGenericMethod()`. I could do something like this:

```c#
static void CallGenericMethod<T, TRequest, TResponse>(T method) where T : IMethod<TRequest, TResponse>
  where TRequest : MessageRequest
  where TResponse : MessageResponse
{ }
```

But that feels messy - look at how I have to call that:

```c#
GetCustomerMethod customerMethod = new GetCustomerMethod();
Program.CallGenericMethod<IMethod<GetCustomerRequest, GetCustomerResponse>, 
  GetCustomerRequest, GetCustomerResponse>(customerMethod);
```

Yuck! But what's even worse is that I don't know what the concrete `IMethod<TRequest, TResponse>`. In one case all I get is the full name of the type, like "GenericParameterDiscovery.GetCustomerMethod". So in that case I don't know what the request and response types are. Even if I do something quirky like this:

```c#
Type customerMethodType = customerMethod.GetType();
Type methodType = typeof(IMethod<MessageRequest, MessageResponse>);
string genericMethodType = methodType.FullName.Split('`')[0];

Type[] methodTypes = customerMethodType.FindInterfaces(new TypeFilter(delegate(Type m, object filterCriteria)
{
  return m.FullName.Contains(genericMethodType);
}), null);

if (methodTypes != null && methodTypes.Length > 0)
{
  Type interfaceType = methodTypes[0];
}
```

I still can't call `CallGenericMethod()`, because I can't pass in any type information - the following code is illegal:

```c#
// Invalid code!
CallGenericMethod<interfaceType, interfaceType.GetGenericArguments()[0], 
  interfaceType.GetGenericArguments()[1]>(customerMethod);
```

Plus, there's nothing I can do to generically reference the method object I'm passing in. Remember, all I have is the type name, so what I do is create the object via `Activator.CreateInstance()`. But ... since none of the method objects inherit from the same base type (since the interface is generic, one method object will in all likelihood not have the same request and response values) I can't pass in the object to `CallGenericMethod()` strongly-typed.

In the end, I've resorted to some Reflection techniques to invoke the method on the object. Although I'm not happy with it, overall I like the generic interface design - the gains outweight this problem. Maybe I'm missing something about generics, though, that I could create an instance of a method object and invoke `Call()` generically.

> Published: 04.27.2006 12:55:52 PM CST