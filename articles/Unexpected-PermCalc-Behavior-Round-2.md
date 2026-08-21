---
title: Unexpected PermCalc Behavior - Round 2
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Unexpected PermCalc Behavior - Round 2

Last night I was trying to figure out why a .dll-based assembly (i.e. one that doesn't have an entry point defined) would automatically require `UIPermission`. As I was driving into work I thought that maybe the compiler was adding some kind of attribute to the assembly that would require it. The only thing that I found was buried in the manifest using ILDasm:

```
.subsystem 0x0003       // WINDOWS_CUI
```

When I wrote my CIL book I briefly touched upon flags like `.subsystem. 0x0003` is for console applications, and [this article](https://web.archive.org/web/20150704121609/http://blogs.msdn.com/b/shawnfa/archive/2005/06/06/425804.aspx) states that console applications require `UIPermission`. That definitely sheds some light on what I'm seeing, but what doesn't make sense is why would a .dll-based assembly set that flag to a value used for console applications?

Here's a list of the values for `IMAGE_SUBSYSTEM` in winnt.h (which are used to define `.subsystem`):

```
#define IMAGE_SUBSYSTEM_UNKNOWN              0   // Unknown subsystem.
#define IMAGE_SUBSYSTEM_NATIVE               1   // Image doesn't require a subsystem.
#define IMAGE_SUBSYSTEM_WINDOWS_GUI          2   // Image runs in the Windows GUI subsystem.
#define IMAGE_SUBSYSTEM_WINDOWS_CUI          3   // Image runs in the Windows character subsystem.
#define IMAGE_SUBSYSTEM_OS2_CUI              5   // image runs in the OS/2 character subsystem.
#define IMAGE_SUBSYSTEM_POSIX_CUI            7   // image runs in the Posix character subsystem.
#define IMAGE_SUBSYSTEM_NATIVE_WINDOWS       8   // image is a native Win9x driver.
#define IMAGE_SUBSYSTEM_WINDOWS_CE_GUI       9   // Image runs in the Windows CE subsystem.
#define IMAGE_SUBSYSTEM_EFI_APPLICATION      10  //
#define IMAGE_SUBSYSTEM_EFI_BOOT_SERVICE_DRIVER  11   //
#define IMAGE_SUBSYSTEM_EFI_RUNTIME_DRIVER   12  //
#define IMAGE_SUBSYSTEM_EFI_ROM              13
#define IMAGE_SUBSYSTEM_XBOX                 14
```

(By the way, I really like the XBox entry!). Just by reading this list, it seems like `IMAGE_SUBSYSTEM_NATIVE` would be a better choice for .dll-based assemblies. So I exported the IL from ILDasm, changed `.subsytem` to `0x0001`, and ran ILasm on that .il file. Unfortunately, I still get the `UIPermission` showing up from permcalc.

The thing is, the permission is only for using the clipboard and only for sub-windows. I wonder if this is somehow automatically added in permcalc's analysis because it would be an extrememly common occurrence for public APIs. That still feels like a hokey explanation ...

> Published: 05.16.2007 06:47:17 AM CST