---
title: Analyzing CSLA
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Analyzing CSLA

For the last couple of years, I've been writing on and speaking about the [Compiler API Platform from Microsoft](https://github.com/dotnet/roslyn), which, let's face it, will always be known as Roslyn. It's the new .NET bits that compiles your C# or VB code using C# or VB in VS 2015 and .NET 4.6, plus there's a large public API for everything that's going on within the compiler. Furthermore, it's all open-source, so you can submit pull requests to help make .NET work even better for you and your fellow .NET developers, whether they use Windows, Linux, or a Mac. This is a brave new world that's awesome for the .NET ecosystem.

One of the areas where the Compiler API really shines is allowing developers to write diagnostics to improve their code. Diagnostic plug into the compilation process so they can find areas in your code where you may be doing things that aren't desirable. Some examples that I've worked on the past are:

* Determining when `DateTime` objects are not UTC-based (https://github.com/JasonBock/FindingDateTimeNow)
* Finding where WCF operations are one-way but the method returns a value (https://github.com/JasonBock/FixingIsOneWay)
* Removing `#region` and `#endregion` directives from code (ok, this really isn't a "problem" but it's a fun one to demo) (https://github.com/JasonBock/Deregionizer)

Recently I started writing analyzers for [CSLA](https://github.com/MarimerLLC/csla). There are some cases in CSLA where you want to do (or not do) things in your code, but they can't be enforced by the compiler. Having the ability to write diagnostics will hopefully reduce, if not eliminate, common CSLA issues. The analyzers are not yet part of any CSLA Nuget distribution, but the code is in the repo, and once CSLA 4.6 is finalized, the analyzers should be a part of the package so they'll install into your CSLA-based projects. Here's a description of the analyzers that are currently in place.

# Serializable Business Objects

This one is so easy to forget, and so much can go wrong if you forget to do it. CSLA business objects should be serializable:

```c#
[Serializable]
public class Person
  : BusinessBase<Person> { }
```

If you don't make the class serializable, you'll experience errors in n-tier scenarios where the object is being passed over the wire from the server to the client and vice versa. You may not notice a problem in unit tests, though, and that's what makes this even more insidious, because all your tests will pass and you'll think you have everything correct. But all it takes is to move the server tier to its own physical tier, and you'll notice then!

The `IsBusinessObjectSerializableAnalyzer` will find cases where you've missed adding `SerializableAttribute`. Here's what it looks like when you've added the analyzer in VS2015:

![Analyzing CSLA](https://jasonbock.net/images/Analyzing-CSLA-1.png "Analyzing CSLA")

I've also added a code fix for this analyzer to add the attribute:

![Analyzing CSLA](https://jasonbock.net/images/Analyzing-CSLA-2.png "Analyzing CSLA")

If you don't have the proper using statement for `SerializableAttribute`, (either "using System;" or "using Csla.Serialization;") the code fix will add those for you:

![Analyzing CSLA](https://jasonbock.net/images/Analyzing-CSLA-3.png "Analyzing CSLA")

Now your CSLA business objects will always be serializable.

## Make Data Portal Operations Non-Public

This isn't an error, but rather a convention that you should follow with CSLA, and that is to keep your `DataPortal` operations (e.g. `DataPortal_Fetch`) non-public. That way, a user of a business object can't easily invoke the method. Sure, someone can use Reflection to invoke it, but it keeps accidental invocation from occurring and potentially messing up object metastate.

The analyzer looks like this:

![Analyzing CSLA](https://jasonbock.net/images/Analyzing-CSLA-4.png "Analyzing CSLA")

Just like the serialization analyzer, there's a code fix as well. This one is also contextual in that, if your class is `sealed`, it won't provide an option to make it protected:

![Analyzing CSLA](https://jasonbock.net/images/Analyzing-CSLA-5.png "Analyzing CSLA")

## Making Analyzers Work

That's a high-level view of what the analyzers do when you run them, but how do they work under the covers? I'll cover a couple of key details with the second analyzer – this may help you if you're interested in writing your own analyzers.

First, let's take a look at the code that finds public `DataPortal` operations:

```c#
private static void AnalyzeMethodDeclaration(
  SyntaxNodeAnalysisContext context)
{
  var methodNode = (MethodDeclarationSyntax)context.Node;
  var methodSymbol = context.SemanticModel.GetDeclaredSymbol(methodNode);
  var classSymbol = methodSymbol.ContainingType;

  if (classSymbol.IsStereotype() && 
    methodSymbol.IsDataPortalOperation() &&
    methodSymbol.DeclaredAccessibility == Accessibility.Public)
  {
    var properties = new Dictionary<string, string>()
    {
      [IsOperationMethodPublicAnalyzerConstants.IsSealed] = 
                    classSymbol.IsSealed.ToString()
    }.ToImmutableDictionary();

    context.ReportDiagnostic(Diagnostic.Create(
     IsOperationMethodPublicAnalyzer.makeNonPublicRule,
      methodNode.Identifier.GetLocation(), properties));
  }
}
```

The `IsStereotype()` extension method determines if the class that contains the method is a CSLA stereotype:

```c#
internal static bool IsStereotype(this ITypeSymbol @this)
{
  if (@this == null)
  {
    return false;
  }
  else
  {
    if (@this.Name == "IBusinessObject" &&
      @this.ContainingAssembly.Name == "Csla")
    {
      return true;
    }
    else
    {
      return @this.BaseType.IsStereotype() ||
        @this.Interfaces.Where(_ => _.IsStereotype()).Any();
    }
  }
}
```

Keep in mind that in Roslyn, you don't have full type information like you do in `System.Reflection`, so the best you can do is string comparisons on members like a `ITypeSymbol`'s `Name` property or `ContainingAssembly.Name`. However, for most analysis situations this is reasonable enough to ensure you've found the thing you're looking for.

Next, we need to see if the method is a DataPortal operation:

```c#
internal static bool IsDataPortalOperation(this IMethodSymbol @this)
{
  if (@this == null)
  {
    return false;
  }
  else
  {
    return @this.Name == "DataPortal_Create" ||
	@this.Name == "DataPortal_Fetch" ||
	@this.Name == "DataPortal_Insert" ||
	@this.Name == "DataPortal_Update" ||
	@this.Name == "DataPortal_Delete" ||
	@this.Name == "DataPortal_DeleteSelf" ||
	@this.Name == "DataPortal_Execute" ||
	@this.Name == "Child_Create" ||
	@this.Name == "Child_Fetch" ||
	@this.Name == "Child_Insert" ||
	@this.Name == "Child_Update" ||
	@this.Name == "Child_DeleteSelf";
  }
}
```

If those two conditions hold, and the method has been declared `public`, then I report an issue. Note that I stuff whether the class is sealed or not into a dictionary, as I use that in the code fix to provide (or not provide) options based on this condition. Here's what the code fix code looks like:

```c#
public override async Task RegisterCodeFixesAsync(CodeFixContext context)
{
  var root = await context.Document.GetSyntaxRootAsync(context.CancellationToken).ConfigureAwait(false);

  if (context.CancellationToken.IsCancellationRequested)
  {
    return;
  }

  var diagnostic = context.Diagnostics.First();
  var methodNode = root.FindNode(diagnostic.Location.SourceSpan) as MethodDeclarationSyntax;

  if (context.CancellationToken.IsCancellationRequested)
  {
    return;
  }

  IsOperationMethodPublicMakeNonPublicCodeFix.RegisterNewCodeFix(
    context, root, methodNode, SyntaxKind.PrivateKeyword,
    IsOperationMethodPublicAnalyzerMakeNonPublicCodeFixConstants.PrivateDescription, diagnostic);
  IsOperationMethodPublicMakeNonPublicCodeFix.RegisterNewCodeFix(
    context, root, methodNode, SyntaxKind.InternalKeyword,
    IsOperationMethodPublicAnalyzerMakeNonPublicCodeFixConstants.InternalDescription, diagnostic);

  var isSealed = bool.Parse(diagnostic.Properties[IsOperationMethodPublicAnalyzerConstants.IsSealed]);

  if (!isSealed)
  {
    IsOperationMethodPublicMakeNonPublicCodeFix.RegisterNewCodeFix(
      context, root, methodNode, SyntaxKind.ProtectedKeyword,
      IsOperationMethodPublicAnalyzerMakeNonPublicCodeFixConstants.ProtectedDescription, diagnostic);
  }
}
```

Note that I only provide a fix to change the visibility to protected if the class isn't `sealed`. Finally, here's what `RegisterNewCodeFix()` does:

```c#
private static void RegisterNewCodeFix(CodeFixContext context, SyntaxNode root, MethodDeclarationSyntax methodNode,
  SyntaxKind visibility, string description, Diagnostic diagnostic)
{
  var publicModifier = methodNode.Modifiers.Where(_ => _.Kind() == SyntaxKind.PublicKeyword).First();
  var visibilityNode = SyntaxFactory.Token(publicModifier.LeadingTrivia, visibility,
    publicModifier.TrailingTrivia);
  var modifiers = methodNode.Modifiers.Replace(publicModifier, visibilityNode);
  var newwMethodNode = methodNode.WithModifiers(modifiers);
  var newRoot = root.ReplaceNode(methodNode, newwMethodNode);

  context.RegisterCodeFix(
    CodeAction.Create(description,
      _ => Task.FromResult<Document>(context.Document.WithSyntaxRoot(newRoot))), diagnostic);
}
```

One thing I always stress with analyzers, code fixes and refactoring is to keep the original trivia when you're changing code – that is, all the whitespace before and after a node. You'll see with the call to `SyntaxFactory.Token()`, which creates a new node to change the visibility, that I keep the trivia that was in place before. Developers have strong opinions on their style and formatting choices, so don't change that on them!

## Future Work

There are two other analyzers I'm going to create for the 4.6 release of CSLA. They are:

* [Simplify Property Implementations](https://github.com/MarimerLLC/csla/issues/363) - If a property uses `Get/Set/Load/ReadProperty`, that's all the property should do. Any processing code should be handled as rules outside of the property getter and/or setter. This analyzer would flag properties that violate this idiom as a warning
* [Capture Result of `Save()` to a Variable](https://github.com/MarimerLLC/csla/issues/364) – If you call `Save()`, you don't want to ignore the return value, because that contains the updated state of the object. Usually you want to assign the result of `Save()` back to the variable reference, but it's easy to forget this. While this isn't an error, I know it's caused many a developer to go down roads of frustration – hopefully this diagnostic will find them for you before you go too far.

If you have other ideas, please add them to the [issues list](https://github.com/MarimerLLC/csla/issues/). And if you start to have ideas to create diagnostics for your own projects, go for it! Yes, the Compiler API is vast and may be a little intimidating at first glance. But once you dive in and go through a couple of samples, you'll start to see it usually doesn't take a lot of code to accomplish what you want to do. Let me know if you have any questions on these analyzers or if you're interested in writing your own and would like some guidance.

> Published: 06.11.2015 02:20:00 PM CST