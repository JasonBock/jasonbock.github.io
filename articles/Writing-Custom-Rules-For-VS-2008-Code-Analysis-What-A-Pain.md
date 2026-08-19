---
title: Writing Custom Rules for VS 2008 Code Analysis - What a Pain!
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Writing Custom Rules for VS 2008 Code Analysis - What a Pain!

I am close to giving up.

With every single effin' upgrade to FxCop, the API breaks so badly that any former rules go haywire. So I have to spend hours of hacking and searching to try and figure out what the hell changed and how I can address it. The big problem now is that I have to use `RuleUtilities.GetAssembly()` to get a `TypeNode`, but that always throws a `NullReferenceException`, even though the file name I pass in is perfectly valid.

I really like Code Analysis, but I am so tired using an undocumented API.

Update: I'm guessing CommonUtilities is null ... which means that something has to load it.

Update: Ha! HA, HA, HA! Think custom rules can't be tested? I think this is a promising start:

```c#
[TestInitialize]
public void Initialize()
{
  AssemblyName commonAsmName = new AssemblyName(
    "FxCopCommon, Version=9.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a");
  Assembly commonAsm = Assembly.Load(commonAsmName);
    
  Type prov = commonAsm.GetType(
    "Microsoft.FxCop.Common.CommonUtilitiesProvider", false);
        
  if(prov != null)
  {
    object provInst = Activator.CreateInstance(prov, true);

    AssemblyName sdkAsmName = new AssemblyName(
      "FxCopSdk, Version=9.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a");
    Assembly sdkAsm = Assembly.Load(sdkAsmName);

    Type intUtil = sdkAsm.GetType("Microsoft.FxCop.Sdk.InternalUtilities", false);
        
    if(intUtil != null)
    {
      intUtil.GetField(
        "s_commonUtilities", BindingFlags.NonPublic | BindingFlags.Static)
        .SetValue(null, provInst);
      }
   }
}
```

We'll see if this holds up ...

Update: One step back ... getting an error in `ResolveAssemblyReference` in `CommonUtilitiesProvider` ...

> Published: 03.30.2008 03:28:53 PM CST