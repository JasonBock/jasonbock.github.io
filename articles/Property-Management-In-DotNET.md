---
title: Property Management In .NET
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Property Management In .NET

## What is Wrong With This Code?

Language/Tool: C#/.NET 

```c#
if(this.Request.UrlReferrer != null)
{
    //Use the property's value.
}
```

My latest project has been an ASP.NET application. Since I've pretty much stayed away from the web/HTML development side of development up to this point in my career, this was a very educational experience. I've tried it before with ASP and it drove me nuts (it was only a 1-week project, but that's all it took to drive me over the edge). ASP.NET isn't perfect, but in my book it is definitely an improvement over the ASP way of coding. This project had its share of weird, off-the-wall UI requests as most projects I've been on have, and the code shown above was part of a feature that was particularly odd. Basically, the site was going to have two URLs (I'll call them http://www.showheader.com and http://www.hideheader.com for now) that could be typed in by the user to get access to the site. If the user came from the showheader URL, we would show a common header on all of the ASPX pages. If they came from the hideheader URL, the header would be hidden. (Don't ask me why I had to do this - I didn't design it). 

I was able to add this functionality by checking the `UrlReferrer` property of the `Request` object. Everything went fine during the test and production phases of the project. However, I noticed that our unhandled exception logger would occasionally send me an e-mail where we were getting an `UriFormatException`. Fortunately, our exception logging is fairly verbose (we pretty much grab everything we can without polluting the code with a bunch of tracing statements) so I was able to track down the offensive code, which is what you see at the beginning of this article. 

Now, I have to admit, I thought I was being fairly defensive in my coding approach. Granted, I could be really defensive and check to see if the `Request` property is also not null, but the real issue is that accessing the `UrlReferrer` property was causing an exception. When I looked at our exception log, I figured out why - the `HTTP_REFERER` value was being blocked by a site that had a link to our http://www.hideheader.com URL. But why would simply accessing a property cause an exception? 

## The Dangers of Accessing a Property

If you haven't work with .NET and you don't know how properties work, it's pretty easy. Properties are nothing more than metadata that covers up 1 or 2 methods that control how an internal field is accessed and/or mutated. In the Java world, you'd end up seeing methods like `getUrlReferrer()` and `setUrlReferrer()`. In fact, when you make a property in .NET, the compiler will actually make the underlying methods for you. In this case, `UrlReferrer` is a read-only property, so the `HttpRequest` class will have a method called `get_UrlReferrer()`. 

Of course, as properties are methods hidden with metadata, they can throw exceptions just like any other method. However, I have never seen a getter throw an exception in my career. Not in any of the Java classes I've used or in any of the .NET assemblies I've accessed. It's not that it can't be done, but it is unusual. I have seen the setters throw an exception if the given information is invalid in some way and I have also seen getters return null (hence my check for null, but accessing a property has always been tame. So I started to wonder what `get_UrlReferrer()` was actually doing to let an exception leak through to the caller? 

Fortuantely, Jay Saurik created a wonderful little tool called Anakrino that reverse-engineers a .NET assembly and translates it to C# code. There is a tool called ILDasm that ships with the .NET Framework, and while I'm comfortable with reading CIL, I'd much rather read C#. Well, I fired up Anakrino and dove into the guts of `get_UrlReferrer()` - here's what I found: 

```c#
public Uri get_UrlReferrer() {
  string local0;

  if (this._referrer == null && this._wr != null) {
    local0 = this._wr.GetKnownRequestHeader(36);
    if (local0 != null && local0.Length > 0) {
      try {
        if (local0.IndexOf("://") >= 0)
          this._referrer = new Uri(local0);
        else
          this._referrer = new Uri(this.Url, local0);
      }
      catch (HttpException) {
        this._referrer = null;
      }
    }
  }
  return this._referrer;
}
```

Basically, it's doing a lazy initialization of its `internal _referrer` field, which is a `Uri`-type class. That is, it's not created until it's needed. The point is that there are a couple of spots where the `UriFormatException` could leak through. One is in the call to `GetKnownRequestHeader()`. `_wr` is a field of type `HttpWorkerRequest`. 36 is the value of the `HeaderReferer` constant - since that's being blocked in this case, it may cause that exception to occur. However, `HttpWorkerRequest` is an abstract class, and it took a trip to the debugger to find out that `_wr` is set to a `System.Web.Hosting.ISAPIWorkerRequestOutOfProc` object. This descends from `System.Web.Hosting.ISAPIWorkerRequest`, and its implementation of `GetKnownRequestHeader()` didn't seem to be the source of the problem. 

The other spot is in the `Uri`'s constructors. In fact, I know it is as the stack trace in the log shows that it comes from a `.ctor()` method. It is explicitly stated in the SDK that the constructors can throw a `UriFormatException`. Yet the code is looking for `HttpException`! That seems like an error to me. According to the documentation, the constructors will never throw a `HttpException`, yet it could throw a `UriFormatException`. It seems like the `try ... catch` is a waste. 

## Possible Solution

What I ended up doing is writing a static method called `GetUrlReferrerSafe()`, which looked like this: 

```c#
public static Uri GetUriReferrerSafe(HttpRequest request)
{
  Uri retVal = null;
    
  try
  {
    retVal = request.UrlReferrer;
  }
  catch(Exception ex) {}
    
  return retVal;
}
```

My original code then changed into this: 

```c#
if(GetUriReferrerSafe(this.Request) != null)
{
  //Use the property's value.
}
```

I'm usually not one to create a catch block without handling the given exception, but in this case I think it makes it easier to manage. Furthermore, I could just check for `UriFormatException`, but all I wanted to do was to make sure I had a valid reference. I had to make this check in a lot of pages, so putting the safety code in one spot cleaned things up a bit and didn't affect my implementation. 

## Checked Exceptions and .NET

Again, while I found it odd that calling a getter would cause an exception, there's nothing to prevent it either. I still wouldn't have a getter throw an exception, though, as it can make for some clumsy code. I also think that the implementation of `get_UrlReferrer()` is buggy, but as I wasn't on the design team for ASP.NET there may be a valid reason for its implementation that I'm not seeing (note that my analysis was done using the 1.0 version. I'm curious to see if this has changed in the 1.1 version). However, one could also make the argument that if .NET had the notion of checked exceptions like Java does, this issue would've never came up. 

I won't take the time here to cover the idea of checked exception in detail along with the debate of their usefulness. That's been done before (if you're interested, click here for a Google search that will bring up a fair amount of links on the subject). I think that checked exceptions would've worked out well here, as the implementation of `get_UrlReferrer()` would've been forced to either handle the possibility of a `UriFormatException` from occurring, or it would have to explicitely defer the responsibility to its caller. I also think that implementing checked exceptions in .NET via attributes may be a viable approach, although given that the 1.0 release did not have this it may be an issue to retroactively fit it in. 

## Summary 

1. Make sure you code defensively, especially as it pertains to exceptions in .NET. You may end up getting one when you least expected it.
2. If something unexpectedly goes wrong, log whatever you can - it'll remove some of the pain during the debugging time.
3. There are lots of free tools in the .NET world (like Anakrino) to help you debug all sorts of weird and nasty issues - in fact this page is an excellent resource.

> Published: 04.15.2003