---
title: Mock `HttpContext` and `MapPath`
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Mock `HttpContext` and `MapPath`

Recently someone sent me an e-mail, informing me that `MapPath()` doesn't work on my mock `HttpContext`. Sure enough, he's right. This test:

```c#
[TestMethod]
public void MapPath()
{
  HttpContext context = (new MockHttpContext(true)).Context;
  string mappedPath = context.Request.MapPath("default.aspx");        
}
```

fails with the following stack trace:

```
Test method Web.Mocks.Tests.MockHttpContextTests.MapPath threw exception: System.ArgumentNullException: Value cannot be null.
Parameter name: str.
at System.Security.Permissions.FileIOPermission.HasIllegalCharacters(String[] str)
at System.Security.Permissions.FileIOPermission.AddPathList(FileIOPermissionAccess access, AccessControlActions control, String[] pathListOrig, Boolean checkForDuplicates, Boolean needFullPath, Boolean copyPathList)
at System.Web.InternalSecurityPermissions.PathDiscovery(String path)
at System.Web.HttpRequest.MapPath(VirtualPath virtualPath, VirtualPath baseVirtualDir, Boolean allowCrossAppMapping)
at System.Web.HttpRequest.MapPath(String virtualPath)
at Web.Mocks.Tests.MockHttpContextTests.MapPath() in C:\JasonBock\Personal\.NET Projects\MockHttpContext\2.0\MockHttpContext.Tests\MockHttpContextTests.cs:line 23
```

My initial thought was that I was missing something simple. So I opened up Reflector and started digging into the implementation of `MapPath()`.

Wow.

To make a `long` story somewhat short, I started to realize that to get `MapPath()` to work, I basically had to host ASP.NET! That sounds weird, but here's why I think that way. I started to realize that `MapPath()` has a subtle dependency on `HostingEnvironment`. So I added this code into the constructor for `MockHttpContext`:

```c#
if(MockHttpContext.environment == null)
{
  MockHttpContext.environment = new HostingEnvironment();
}
```

Well, that didn't fix things:

```
Test method Web.Mocks.Tests.MockHttpContextTests.MapPath threw exception: System.NullReferenceException: Object reference not set to an instance of an object..
at System.Web.Hosting.HostingEnvironment.MapPathActual(VirtualPath virtualPath, Boolean permitNull)
at System.Web.VirtualPath.MapPathInternal()
at System.Web.HttpRequest.MapPath(VirtualPath virtualPath, VirtualPath baseVirtualDir, Boolean allowCrossAppMapping)
at System.Web.HttpRequest.MapPath(String virtualPath)
at Web.Mocks.Tests.MockHttpContextTests.MapPath() in C:\JasonBock\Personal\.NET Projects\MockHttpContext\2.0\MockHttpContext.Tests\MockHttpContextTests.cs:line 22
```

The stack trace and the error message changes, but not there's something amiss with `HostingEnvironment`. The issue is that `HostingEnvironment` isn't being initialized correctly. There's an `Initialize()` method on `HostingEnvironment`, but it's `internal`, and I really didn't want to go down the road of creating objects via Reflection if I could avoid it. So I busted open my FileGenerator add-in to get the code for `System.Web` and find out that the only sane way to call `HostingEnvironment.Initialize()` was to call 02.19.2007 07:38:21 AM CST.

Well, at that point, I'm basically hosting ASP.NET, and using a mock `HttpContext` is almost useless. At this point, I see two options. If you have a Team edition of VS 2005, you can host your tests in ASP.NET. Or, you can host ASP.NET yourself. Either way, both approaches should provide an `HttpContext` that's fully-functional and can map paths correctly :).

Maybe I'm wrong with my analysis, but I spent a fair amount of time on this problem over the weekend, and I couldn't find a "simple" solution. If you have any insights into this, feel free to contact me.

> Published: 02.19.2007 07:38:21 AM CST