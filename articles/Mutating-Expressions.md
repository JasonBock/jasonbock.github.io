---
title: Mutating Expressions
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Mutating Expressions

One of the talks I've done in the recent past is called "Evolving .NET". The talk covers the basics of evolutionary computation and uses an example of evolving function implementations. I've only done the talk a couple of times but it was fun to put together and give (I hope I can do the talk more in the future). I use classes in the `System.Linq.Expressions` namespace to do this. Before 4.0, expressions were immutable, so I had to use some classes to get around that. Now there's an `ExpressionVisitor` class available to make this a little easier. The `ExpressionVisitor` class is just the framework - i.e. it's an abstract class. The developer is responsible for handling all of the necessary tree traversal. In this post I'll demonstrate how this works.

Let's say I have two functions that are used in the [Collatz Conjecture](https://en.wikipedia.org/wiki/Collatz_conjecture):

```c#
//Expression<Func<int, int>> evenExpression = x => x / 2;
var evenParameter = Expression.Parameter(typeof(int), "x");
var evenExpression = Expression.Lambda<Func<int, int>>(
  Expression.Divide(evenParameter, Expression.Constant(2, typeof(int))),
  evenParameter);

//Expression<Func<int, int>> oddExpression = x => 3 * x + 1;
var oddParameter = Expression.Parameter(typeof(int), "x");
var oddExpression = Expression.Lambda<Func<int, int>>(
  Expression.Add(Expression.Multiply(Expression.Constant(3, typeof(int)), oddParameter), 
    Expression.Constant(1, typeof(int))),
  oddParameter);
```

I've included comments so you can map the manual creation of the expression to the one-line approach. In my evolution example, I can't declare it the easier way; I have to manually create them, but as you can see, it's not that hard to create expressions.

Now that I have these expressions, I want to mutate them. I'm going to replace the parameters from one expression's body with the body of the other expression. In other words, I want this:

```
// x => ((3 * x) + 1) / 2
// x => 3 * (x / 2) + 1
```

I'll show you how I do the first one - the second mutation works the same way:

```c#
var newEvenExpression = Expression.Lambda<Func<int, int>>(
  new ReplacementVisitor().Transform(
    evenExpression.Body, evenExpression.Parameters, 
    evenParameter, oddExpression.Body),
  evenParameter);
```

Simple, right? So ... what's `ReplacementVisitor`? I'm glad you asked:

```c#
internal sealed class ReplacementVisitor 
  : ExpressionVisitor
{
  public Expression Transform(Expression source, 
    ReadOnlyCollection<ParameterExpression> sourceParameters,
    Expression find, Expression replace)
  {
    this.Find = find;
    this.SourceParameters = sourceParameters;
    this.Replace = this.Visit(replace);
    return this.Visit(source);
  }

  private Expression ReplaceNode(Expression node)
  {
    if(node == this.Find)
    {
      return this.Replace;
    }
    else
    {
      return node;
    }
  }

  protected override Expression VisitBinary(BinaryExpression node)
  {
    var result = this.ReplaceNode(node);

    if(result == node)
    {
      switch(node.NodeType)
      {
        case ExpressionType.Add:
          result = Expression.Add(
            this.Visit(node.Left), this.Visit(node.Right));
          break;
        case ExpressionType.Subtract:
          result = Expression.Subtract(
            this.Visit(node.Left), this.Visit(node.Right));
          break;
        case ExpressionType.Multiply:
          result = Expression.Multiply(
            this.Visit(node.Left), this.Visit(node.Right));
          break;
        case ExpressionType.Divide:
          result = Expression.Divide(
            this.Visit(node.Left), this.Visit(node.Right));
          break;
        default:
          throw new NotSupportedException(string.Format(
            "Unexpected binary type: {0}", node.NodeType));
      }
    }

    return result;
  }

  protected override Expression VisitParameter(ParameterExpression node)
  {
    Expression result = null;

    if(!this.SourceParameters.Contains(node))
    {
      result = (from sourceParameter in this.SourceParameters
        where sourceParameter.Name == node.Name
        select sourceParameter).First();
    }
    else
    {
      result = this.ReplaceNode(node);
    }

    return result;
  }

  protected override Expression VisitConstant(ConstantExpression node)
  {
    return this.ReplaceNode(node);
  }

  private Expression Find { get; set; }
  private Expression Replace { get; set; }
  private ReadOnlyCollection<ParameterExpression> SourceParameters { get; set; }
}
```

Here's what it does. When `Transform()` is called, the function stores the arguments as properties, but before it visits the source expression, it has to visit the replacement expression and swap out any of its parameters with the parameters from the first expression. If you don't, you'll get an exception that looks something like this:

```
Unhandled Exception: System.InvalidOperationException: variable 'x' of type 'System.Int32' referenced from scope '', it is not defined
```

Once the parameters are swapped, the source is visited. In my case, all I need to do is find parameter, binary and constant expressions. There are a slew of `VisitXYZ()` methods you can override from `ExpressionVisitor`, so you may need to override more than what I'm doing. Also, note that visiting one node may require you to do another visitation with child nodes (see how the `Left` and `Right` nodes are visited in `VisitBinary()` before the new `BinaryExpression` is created).

That's it. When I run this code:

```c#
Console.Out.WriteLine("New even expression: {0}", newEvenExpression.ToString());
Console.Out.WriteLine("New even expression: e(3) = {0}", newEvenExpression.Compile()(3));
```

I see this on the command line:

```
New even expression: x => (((3 * x) + 1) / 2)
New even expression: e(3) = 5
```

Perfect! That's exactly what I want. Now I can go back into my presentation code and use `ExpressionVisitor`.

I hope this has been useful. LINQ expressions are extremely powerful in 4.0, but unfortunately there's not a lot out there to look at. Hopefully this will get you steered down the right path if you want to know how to use `ExpressionVisitor`. Enjoy!

> Published: 05.14.2010 01:18:40 PM CST