---
title: Generating Exception Blocks
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Generating Exception Blocks

Recently I've been doing a lot of work generating dynamic types in .NET. I duplicated Java's dynamic proxy architecture, which forced me to start learning CIL and the `System.Reflection.Emit` classes. I hope to share what I did in detail in a published format soon, but right now I'll share an interesting side effect that the Emitter classes can have on your generated CIL.

During the creation of my `Proxy` class, I relied heavily on reverse-engineering techniques to help me understand how CIL works. I'd create my intended type in C#, load the assembly into ILDasm, and translate the CIL into a bunch of `Emit()` calls. I had done this a lot in the project, and while it was getting tedious, I was becoming comfortable with it.

I had to make a change to the code that generates a method called `InvokeMethod()`. This method had an if statement near the top of the method:

```c#
if(null == wrappedInfo.InnerMethod)
```

The generated CIL looked like this:

```
IL_0017:  ldloca.s   wrappedInfo
IL_0019:  ldfld      class [mscorlib]System.Reflection.MethodInfo       
  Viktor.TestProxy.InnerMethodInfo::InnerMethod
IL_001e:  brfalse.s  IL_009b
```

OK, no problem. Here's what the CIL looks like when I converted them into a bunch of `Emit()` calls:

```c#
invokeMethodIL.Emit(OpCodes.Ldloca_S, wrappedInfo);
invokeMethodIL.Emit(OpCodes.Ldfld, 
  innerMethodInfo.GetField(INNER_METHOD));
Label IL_009b = invokeMethodIL.DefineLabel();
invokeMethodIL.Emit(OpCodes.Brfalse_S, IL_009b);
```

I had done this so many times before, I didn't expect this to be a problem. But, lo and behold, when I ran my dynamic type through my tests, I got the following exception:

```
System.NotSupportedException: Illegal one-byte branch at position: 31. Requested branch was: 143.
   at System.Reflection.Emit.ILGenerator.BakeByteArray()
   at System.Reflection.Emit.MethodBuilder.CreateMethodBodyHelper(ILGenerator il)
   at System.Reflection.Emit.TypeBuilder.CreateType()
```

Well, this confused the hell out of me! I checked, double-checked, and triple-checked my C# code. Everything seemed fine - I had a couple of bugs, but when I fixed them I still got the error. So I made a post to a .NET list, hoping to get some help. Fortunately, I got a helpful response. Basically, the problem is this. `brfalse.s` is a "short" branch instruction. Using this opcode, you can only jump forward (or back) 127 bytes of CIL instructions. However, I'm trying to make a 143-byte branch in my dynamically-generated type, so when I try to "bake" the type, `BakeByteArray()` reports an error.

The quick fix was to change `Brfalse_S` to `Brfalse`. That eliminated the exception and everything worked, but I was bothered by the fact that the assembly I was using to reverse-engineer the CIL **generated** `brfalse.s`! Why wouldn't it work when I dynamically created it?

Finally, I realized (through the help of Joe Hummel) to simply look at the CIL in the base assembly and dynamically-generated one, just to make sure the CIL is identical. I initially thought this wasn't necessary as my C# code has the exact same CIL instuctions emitted by `Emit()`. However, when I compared the CIL, I was suprised. The dynamic type's CIL looks like this:

```
.try
{
  IL_003f:  ldarg.0
  IL_0040:  ldfld      class [DynamicProxiesClient]Viktor.DynamicProxiesClient.frmMain 
    Viktor.DynamicProxiesClient.SomeClass58168328::invokeHandler
  IL_0045:  ldloc.s    V_5
  IL_0047:  ldloc.s    V_4
  IL_0049:  ldarg.2
  IL_004a:  ldloca.s   V_1
  IL_004c:  ldloca.s   V_0
  IL_004e:  callvirt   instance void 
    [DynamicProxies]Viktor.DynamicProxies.InvocationHandler::BeforeMethodInvocation(string,
    string, object[]&, bool&, bool&)
  IL_0053:  leave.s    IL_0067
  IL_0055:  leave      IL_0067
}  // end .try
catch [mscorlib]System.Exception 
{
```

Now, my code is generated the last `leave.s` opcode, but I was sure I didn't generate the `leave` opcode. Well, I wasn't doing it directly with an `Emit()` call. It was happening in calls to `EndExceptionBlock()` and `BeginCatchBlock()`:

```c#
invokeMethodIL.BeginExceptionBlock();
invokeMethodIL.Emit(OpCodes.Ldarg_0);
invokeMethodIL.Emit(OpCodes.Ldfld, invokeHandler);
invokeMethodIL.Emit(OpCodes.Ldloc_S, interfaceName);
invokeMethodIL.Emit(OpCodes.Ldloc_S, methodName);
invokeMethodIL.Emit(OpCodes.Ldarg_2);
invokeMethodIL.Emit(OpCodes.Ldloca_S, doInvoke);
invokeMethodIL.Emit(OpCodes.Ldloca_S, raiseException);
invokeMethodIL.EmitCall(OpCodes.Callvirt, 
  typeof(Viktor.DynamicProxies.InvocationHandler).GetMethod("BeforeMethodInvocation"), null);
Label IL_005c = invokeMethodIL.DefineLabel();
invokeMethodIL.Emit(OpCodes.Leave_S, IL_005c);
//    }
//    catch(Exception beforeEx)	
invokeMethodIL.BeginCatchBlock(typeof(System.Exception));
```

If I took out the last call to `Emit()`, I only got the `leave` opcode. Of course, this is a bigger opcode than the one I wanted, so I still got the exception I had before. But at least I knew where the extra bits were coming from.

So when you're creating types in .NET and you're using ILDasm to help you generate the needed CIL, remember that the .NET Emitter classes may insert more opcodes than what you were expecting. Of course, if .NET supported design-by-contract, I would've seen the post-condition in `BeginCatchBlock()` and realized that `leave` is inserted into the CIL stream. The SDK does not document this, and technically DBC wouldn't require that this post-condition be stated either ... but I think if DBC was added to .NET, people would start to document these kinds of conditions and behaviors in their assemblies where they belong.

> Published: 03.02.2002 12:00:00 AM CST