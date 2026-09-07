---
title: Finding a Use for FileGenerator - Language Translation
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Finding a Use for FileGenerator - Language Translation

While I had a lot of fun writing/updating [FileGenerator](https://github.com/JasonBock/FileGenerator), the only use for it was to get the disassembled code into a file with ease. However, today I discovered a cute application of the add-in that developers may actually like: language translation.

Let's say you had the following class written in C#:

```c#
namespace CSharpTest
{
  public class CSharpExclusiveClass
  {
    public string CSharpOnlyData()
    {
      return "C# clients only, please!";    
    }
  }
}
```

Now you discover that your client [1] wants it in VB .NET. In this case, it's trivial to redo it by hand:

```vb
Namespace CSharpTest
  Public Class CSharpExclusiveClass
    Public Function CSharpOnlyData() As String
      Return "C# clients only, please!"
    End Function
  End Class
End Namespace
```

However, if you're trying to convert entire projects/assemblies over, it's too time-consuming to do it manually. Enter FileGenerator! Simply pick the assembly/module/type of your choice, then select the directory where you want the code to go, make sure your current language is Visual Basic, and voila! You now have VB .NET source code:

```vb
Imports System

Namespace CSharpTest
  Public Class CSharpExclusiveClass

  ' Methods
  Public Sub New()
  End Sub

  Public Function CSharpOnlyData() As String
    Return "C# clients only, please!"
  End Function

  End Class
End Namespace
```

In this case, the generated code is virtually identical to the manual production. In general, this may not always be the case as the example I give here is very easy, but at least I now have a straightforward way to generate code for my client in the language they want. Cool!

[1] This is not that uncommon these days in the consulting world for a client to insist that your .NET code is written in a certain language, and you have it implemented in another language.

> Published: 06.18.2004 02:46:21 PM CST