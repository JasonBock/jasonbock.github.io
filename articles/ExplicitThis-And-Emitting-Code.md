---
title: ExplicitThis and Emitting Code
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# ExplicitThis and Emitting Code

I'm diving back into my [EmitDebugger](https://github.com/JasonBock/EmitDebugging) library, primarily to add tests around the code and to remove the silly restriction that you must declare all your locals before the opcodes. I started adding tests that checks a method that generates the calling conventions of a method as a string. My "vararg" test worked, but my "explicitthis" test is acting weird. Basically, I generate the method like this:

```c#
private static void GenerateExplicitThisMethod(TypeBuilder typeBuilder)
{
  var methodBuilder = typeBuilder.DefineMethod(
    MethodBaseExtensionsGetCallingConventionsTests.MethodExplicitThisName,
    MethodAttributes.Public | MethodAttributes.HideBySig |
      MethodAttributes.NewSlot | MethodAttributes.Virtual, 
    CallingConventions.HasThis | CallingConventions.ExplicitThis, null, 
    new Type[] { typeBuilder.UnderlyingSystemType });

  var methodGenerator = methodBuilder.GetILGenerator();
  methodGenerator.Emit(OpCodes.Ret);
}
```

From what I've been able to gather about `explicitthis` methods, they define the "this" parameter explicitly as the first argument to the method (see Section 15.3 of Partition II for all the details). The docs for `CallingConventions` says I need to supply the `HasThis` value with `ExplicitThis`, which is why I'm doing that.

If I look at the method in ILDasm, though, the signature isn't correct:

```
.method public hidebysig newslot virtual 
  instance void ExplicitThis(
    class MethodBaseExtensionsGetCallingConventionsTests.MethodBaseExtensionsGetCallingConventions A_1) cil managed
{
  // Code size       1 (0x1)
  .maxstack  0
  IL_0000:  ret
} // end of method MethodBaseExtensionsGetCallingConventions::ExplicitThis
```

There should be an "explicit" word in the method signature, but there isn't.

Now, in Section 22.26 of Partition II, it says that `explicitthis` methods aren't verifiable, and that `explicitthis` can only be set in signatures for function pointers. I took my emitted assembly, loaded it in ILDasm, got the .il file, and tweaked it like this:

```
.method public hidebysig newslot virtual explicit
  instance void ExplicitThis(
    class MethodBaseExtensionsGetCallingConventionsTests.MethodBaseExtensionsGetCallingConventions A_1) cil managed
{
  // Code size       1 (0x1)
  .maxstack  0
  IL_0000:  ret
} // end of method MethodBaseExtensionsGetCallingConventions::ExplicitThis
```

I can compile this with ilasm, but when I run peverify on it, I get the following error:

```
[MD]: Error: Signature has invalid calling convention=0x00000060. [token:0x06000003]
```

So, it's no longer verifiable, but that doesn't mean I'm doing it right either.

After all this ... I still have no clue right now why my code doesn't generate that calling convention. Any ideas?

> Published: 10.21.2008 10:53:37 AM CST