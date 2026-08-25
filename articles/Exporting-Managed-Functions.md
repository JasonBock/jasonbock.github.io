---
title: Exporting Managed Functions
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Exporting Managed Functions

A question came up on an internal Magenic list, which basically went something like this: "Is there any way to export a function in .NET such that you could call it in C?" This means no COM interop; you have to export the function in your assembly. The only way I know to do is it is to use the `.vtfixup`, `.data`, `.vtentry`, and `.export` directives like this:

```
.vtfixup [1] int32 fromunmanaged at ENTRY_1

.class SimpleMethodDefines
{
  .method public static int32 UnmanagedInt32()
  {
    .vtentry 1:1
    .export [1] as UnmanagedInt32
    .maxstack 1
    ldc.i4.1
    ret
  }
}

.data ENTRY_1 = int32(0)
```

You can get the full source code example from my CIL book from this page - it's in the Methods` subfolder under the `Chapter02` directory. At the end of day, `UnmanagedInt32()` can be called like an exported function from VB6 [1] like so:

```vb
Private Declare Function UnmanagedInt32 Lib "Methods.dll" () As Integer

Private Sub cmdInvoke_Click()

MsgBox "UnmanagedInt32 = " & CStr(UnmanagedInt32())

End Sub
```

Can anyone else think of another way? If you allow COM interop, the C programmer could call the COM service (remember this article?). I'd lean towards this approach. Exporting the function is just too ugly.

[1] I realize the original question uses the C language. I'm not a C programmer, but the VB6 code example shows that the function is indeed exported.

> Published: 10.18.2005 08:28:35 PM CST