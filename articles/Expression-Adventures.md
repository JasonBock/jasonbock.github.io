---
title: Expression Adventures
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Expression Adventures

Lately I've been diving into LINQ expressions pretty heavily. If you're going to the MVP Summit I'm going to give a brief description about what I've been doing at the MVP2MVP sessions on Sunday, but what I wanted to show now is how powerful these expression trees can be, and where the C# compiler seems to be doing some optimizations for me.

I've always been a fan of IL. Developers always seem to get hot and bothered by the latest language to target the CLR, but at the end of the day, opcodes have to be generated so the CLR can execute that wonderful OO or functional or dynamic code. Plus, there are some things you can do at the IL level that most languages don't support (for better or for worse). For example, you can overload methods by the return type only in IL. So the following code won't compile in C#:

```c#
public class ValueGenerator
{
  public int GetValue() { return 666; }
  public string GetValue() { return "666"; }
}
```

but it does in IL:

```
.class public auto ansi beforefieldinit Overloaded.ValueGenerator
  extends [mscorlib]System.Object
{
  .method public hidebysig specialname rtspecialname 
    instance void  .ctor() cil managed
  {
    .maxstack  8
    IL_0000:  ldarg.0
    IL_0001:  call instance void [mscorlib]System.Object::.ctor()
    IL_0006:  ret
  }

  .method public hidebysig instance int32 
    GetValue() cil managed
  {
    .maxstack  8
    IL_0000:  ldc.i4 0x29a
    IL_0005:  ret
  }

  .method public hidebysig instance string 
    GetValue() cil managed
  {
    .maxstack  8
    IL_0000:  ldstr "666"
    IL_0005:  ret
  }
}
```

Moreover, you can't call the `GetValue()` methods from C# or VB, so the only choice you have is to dynamically emit a piece of shim code to invoke the methods. Usually, I'd dive right into a `DynamicMethod` to accomplish this [1], but expression trees can make thing so much easier to read. So can we invoke the method from a LINQ expression tree? The answer is...yes!

First, here's the code to invoke `GetValue()` with a string return using a `DynamicMethod`:

```c#
private static void CallValueGeneratorViaDynamicMethod()
{
  var stringValueMethod = (from method in typeof(ValueGenerator).GetMethods()
    where method.Name == "GetValue"
    where method.ReturnType == typeof(string)
    select method).FirstOrDefault();
  var generatorMethodString = new DynamicMethod("CallGetValueString", null, Type.EmptyTypes);
  var generatorMethodStringIL = generatorMethodString.GetILGenerator();
  var generatorLocal = generatorMethodStringIL.DeclareLocal(typeof(ValueGenerator));
  generatorMethodStringIL.Emit(OpCodes.Newobj, typeof(ValueGenerator).GetConstructor(Type.EmptyTypes));
  generatorMethodStringIL.Emit(OpCodes.Stloc_0);
  generatorMethodStringIL.Emit(OpCodes.Call, typeof(Console).GetProperty("Out").GetGetMethod());
  generatorMethodStringIL.Emit(OpCodes.Ldloc_0);
  generatorMethodStringIL.Emit(OpCodes.Callvirt, stringValueMethod);
  generatorMethodStringIL.Emit(OpCodes.Callvirt, 
    typeof(TextWriter).GetMethod("WriteLine", new Type[] { typeof(string) }));
  generatorMethodStringIL.Emit(OpCodes.Ret);

  var compiledMethod = (Action)generatorMethodString.CreateDelegate(typeof(Action));
  compiledMethod();
}
```

Now take a look at it using an expression tree:

```c#
private static void CallValueGeneratorViaExpression()
{
  var stringValueMethod = (from method in typeof(ValueGenerator).GetMethods()
    where method.Name == "GetValue"
    where method.ReturnType == typeof(string)
    select method).FirstOrDefault();
  var expression = Expression.Call(typeof(Console).GetMethod("WriteLine", new Type[] { typeof(string) }),
    Expression.Call(
      Expression.New(typeof(ValueGenerator).GetConstructor(Type.EmptyTypes)), stringValueMethod));
  Expression.Lambda<Action>(expression).Compile()();
}
```

Personally, the second one makes a lot more sense to me. No opcodes, less code, it's all goodness. Technically, yes, they're not the exact same thing (the `DynamicMethod` version is calling `WriteLine()` on the `Out` property of `Console`, whereas the expression one uses `WriteLine()` directly on `Console`) but they both produce the right answer. Go LINQ expressions!

Another thing I've noticed is that if you write this:

```c#
Expression<Func<double, double>> optimizedLambda = a => a * (3.4 + 4.3);
Console.WriteLine(optimizedLambda.Body.ToString());
```

The console prints out "(a * 7.7)". But if you write the tree "by hand":

```c#
var parameter = Expression.Parameter(typeof(double), "a");
var optimizedExpression = Expression.Multiply(
  parameter, Expression.Add(
    Expression.Constant(3.4, typeof(double)),
    Expression.Constant(4.3, typeof(double))));
Console.WriteLine(optimizedExpression.ToString());
```

"(a * (3.4 + 4.3))" is printed to the console. The reason I bring this up is that with my `ExpressionEvolver`, I noticed I was starting to get really long expressions where binary operators were being performed on constants. So I wanted to write an expression reducer to make the evolved expressions smaller. When I did some initial prototyping, I noticed that the C# compiler seems to do some optimizations to the lambda expression - in fact, here's what it looks like (from Reflector):

```c#
ParameterExpression CS$0$0000;
Expression<Func<double, double>> optimizedLambda = Expression.Lambda<Func<double, double>>(
  Expression.Multiply(CS$0$0000 = Expression.Parameter(typeof(double), "a"), 
  Expression.Constant(7.6999999999999993, typeof(double))), new ParameterExpression[] { CS$0$0000 });
```

It's not a big deal - it was more of a curiosity that C# is "noticing" certain conditions and optimizing them away. In other words, it doesn't appear to be part of the Expression API. That would've been nice if that was the case, but it's not, and writing the reducer wasn't as hard as I thought it was going to be.

[1] This is something that you will probably never run into in the "real world".

> Published: 02.20.2009 10:44:45 AM CST