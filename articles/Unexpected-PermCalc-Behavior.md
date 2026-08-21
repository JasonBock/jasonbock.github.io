---
title: Unexpected PermCalc Behavior
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Unexpected PermCalc Behavior

For my presentation on Thursday, I decided to dive into permcalc a bit. It's an interesting tool in that it can tell you the permissions that the client of your public API will need in order to successfully run your code. There's a key point in that statement, because permcalc doesn't tell your code will need to run; only what the client will need (which makes sense, but it tripped me up the first time). The odd thing is what it reports with a bare-bones API. Consider the following class:

```c#
public sealed class ClientAPI
{
  public void InnocentMethod() { }
}
```

Now, run permcalc:

```
permcalc -cleancache -stacks -show -sandbox CalculatePermissions.dll
```

This is what I got in my result:

```xml
<Sandbox>
  <PermissionSet version="1" class="System.Security.PermissionSet">
    <IPermission version="1" 
      class="System.Security.Permissions.SecurityPermission, mscorlib, ..." Flags="Execution" />
    <IPermission Window="SafeSubWindows" Clipboard="OwnClipboard" version="1" 
      class="System.Security.Permissions.UIPermission, mscorlib, ..." />
  </PermissionSet>
</Sandbox>
```

(Note that permcalc doesn't put the ellipses in the class name; I did that to save on some space). What threw me was the inclusion of `UIPermission`. Um ... just **why** would the client need that to invoke `InnocentMethod()`? If you take off the `-sandbox` switch, the output looks like this:

```xml
<Assembly>
  <Namespace Name="CalculatePermissions">
    <Type Name="ClientAPI">
      <Method Sig="instance void InnocentMethod()" />
      <Method Sig="instance void .ctor()" />
    </Type>
  </Namespace>
</Assembly
```

I don't see any UI needs here. So ... why is `UIPermission` being reported? Because here's the weird thing - if I add the following method to `ClientAPI`:

```c#
[ReflectionPermission(SecurityAction.Demand, 
  Flags = ReflectionPermissionFlag.AllFlags, MemberAccess = true,
  ReflectionEmit = true, TypeInformation = true, Unrestricted = true)]
public void DemandReflectionPermission()
{
}
```

The 2nd way of invoking permcalc gives this:

```xml
<Assembly>
  <Namespace Name="CalculatePermissions">
    <Type Name="ClientAPI">
      <Method Sig="instance void InnocentMethod()" />
      <Method Sig="instance void DemandReflectionPermission()">
        <Demand>
          <PermissionSet version="1" class="System.Security.PermissionSet">
            <IPermission version="1" 
                class="System.Security.Permissions.ReflectionPermission, mscorlib, ..." 
                Unrestricted="true" />
          </PermissionSet>
        </Demand>
        <Sandbox>
          <PermissionSet version="1" class="System.Security.PermissionSet">
            <IPermission version="1" 
                class="System.Security.Permissions.ReflectionPermission, mscorlib, ..." 
                Unrestricted="true" />
          </PermissionSet>
        </Sandbox>
        <Stacks>
          <CallStack>
            <IPermission version="1" 
                class="System.Security.Permissions.ReflectionPermission, mscorlib, ..." 
                Unrestricted="true" />
            <Method Type="CalculatePermissions.ClientAPI" 
                Sig="instance void DemandReflectionPermission()" Asm="CalculatePermissions" />
          </CallStack>
        </Stacks>
      </Method>
      <Method Sig="instance void .ctor()" />
    </Type>
  </Namespace>
</Assembly>
```

You can see that even though `DemandReflectionPermission()` doesn't do anything with Reflection, the analysis correctly picks up the `ReflectionPermissionAttribute` on that method.

But...still, why `UIPermission`? I found this article, but it didn't seem to shed any light on the issue. permcalc seems like an interesting tool to use - if it was around with the 1.0 release of .NET I wonder if people would've cared more about Code Access Security than what I've seen (this post nails it).

> Published: 05.15.2007 09:40:26 PM CST