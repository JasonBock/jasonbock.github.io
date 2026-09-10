---
title: Unit Testing Roslyn-Based Extensions
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Unit Testing Roslyn-Based Extensions

We should always test our code. But when you're dealing with a compiler API like [Roslyn](https://github.com/dotnet/roslyn), you may wonder if it's difficult to write tests. Turns out it's pretty straightforward, but there are a couple of things to keep in mind. In this article, I'll show you how I added unit tests to a diagnostic extension I wrote.

## Test All the Code

I've been writing code professionally – that is, getting paid for it – since 1995. But it wasn't until 2003 that I started including tests with my code. Over the last 10 years, I've become convinced that if you write code, you write tests. It's not a part of the process that you can cut to save money. In fact, you're probably hurting yourself in the long run if you don't write those tests. I've been on projects where the development team had thousands of tests in place. Having such a test suite in place led to a number of positive outcomes, such as having the confidence in making a change to the code base and knowing if the new code broke previous expectations.

That said, sometimes I've seen developers run into circumstances where there are legitimate cases where writing tests seems far too difficult to do, and therefore not worth the effort. The classic case is when a developer tries to add tests to a system that's been around for a long time that never had tests to begin with. Trying to write clean tests that don't require a substantial amount of dependency setup or have convoluted code paths ends up being a fallacy. Of course, one can lament that the developers "should have done it right" at the beginning, but to spend vast amounts of time to undo the mess may not be worth it just to get a couple of tests in place.

So, what happens when you start writing code that uses the new compiler APIs from Microsoft? Writing a compiler is no easy task, and Roslyn isn't just about writing a compiler. Refactoring, diagnostics, code analysis – these can all be built upon the Roslyn API. Such a large code base may make you wonder if it's possible to write tests with code that uses Roslyn, especially given that a fair amount of Roslyn is written with performance in mind and therefore doesn't lend itself well to unit testing techniques such as mocking. Specifically, some types in Roslyn are structs, or some methods are non-virtual, so using mock libraries such as [Moq](https://github.com/devlooped/moq) or [NSubstitute](https://nsubstitute.github.io/) won't help. But ... just how hard is it to write unit tests for code that uses Roslyn?

For the past couple of years, I've done a presentation called "Managing the .NET Compiler" for conferences and user groups that demonstrates how Roslyn works. One part of the talk is to show how you can write extensions that can find issues with code and potentially provide a fix for the user. The example I use is one that pops up in WCF with one way methods. If your operation is one way, but your method returns a value, the eventual execution of that code at runtime will cause an exception. However, the C# compiler doesn't know anything about WCF runtime semantics, so it'll happily compile code like that. But with a Roslyn diagnostic, I can inform the developer that there's an error with their code:

![Unit Testing Roslyn](https://jasonbock.net/images/Unit-Testing-Roslyn-1.png "Unit Testing Roslyn")

I can also provide a fix for them as well:

![Unit Testing Roslyn](https://jasonbock.net/images/Unit-Testing-Roslyn-2.png "Unit Testing Roslyn")

Having the ability to write diagnostics like this is one area I hope a lot of .NET developers take advantage of in the next version of Visual Studio and .NET. I think we've all run into cases where we want to enforce certain coding structures and rules in an application, and having the ability to detect that quickly is a good thing. Moreover, writing these extensions doesn't take as much code as one would think. For example, my diagnostic to find the `IsOneWay` issue is 89 lines of code, and the code fix is 58 lines. I don't think that's a lot of code given what's going on (and that's a literal count of lines in the class files, some of that ends up being using statements and whatnot).

Now, it's fun to run the extension in Visual Studio and see it work correctly. In fact, developers should always run their applications to ensure that when everything is tied together things work as expected. But we should also have a suite of automated tests in place for this code. Let's see what it takes to write tests for both the diagnostic analyzer and the code fix.

## Testing a Diagnostic

When you write a Roslyn diagnostic, your class inherits from `ISyntaxNodeAnalyzer<>`, which means your class must implement the following members:

* A property called `SupportedDiagnostics` that returns an `ImmutableArray<DiagnosticDescriptor>`
* A property called `SyntaxKindsOfInterest` that returns an `ImmutableArray<SyntaxKind>`
* A method called `AnalyzeNode` that takes 4 arguments and doesn't return a value

Writing tests for the first two members is pretty straightforward (note that for all of tests I'm using [xUnit](https://xunit.net/) but any test framework will work):

```c#
[Fact]
public void GetSupportedDiagnostics()
{
  var analyzer = new IsOneWayOperationAnalyzer();
  var supportedDiagnostics = analyzer.SupportedDiagnostics;

  Assert.Equal(1, supportedDiagnostics.Length);

  var supportedDiagnostic = supportedDiagnostics[0];

  Assert.Equal(IsOneWayOperationConstants.DiagnosticId, 
    supportedDiagnostic.Id);
  Assert.Equal(IsOneWayOperationConstants.Description, 
    supportedDiagnostic.Description);
  Assert.Equal(IsOneWayOperationConstants.Message, 
    supportedDiagnostic.MessageFormat);
  Assert.Equal(IsOneWayOperationConstants.Category, 
    supportedDiagnostic.Category);
  Assert.Equal(IsOneWayOperationConstants.DefaultSeverity, 
    supportedDiagnostic.DefaultSeverity);
}

[Fact]
public void GetSyntaxKindsOfInterest()
{
  var analyzer = new IsOneWayOperationAnalyzer();
  var syntaxKindsOfInterest = analyzer.SyntaxKindsOfInterest;

  Assert.Equal(1, syntaxKindsOfInterest.Length);
  Assert.Equal(SyntaxKind.MethodDeclaration, syntaxKindsOfInterest[0]);
}
```

All I'm doing is checking to see that the number of items in the collections are correct, and that the values within the collections have the right values.

The test for `AnalyzeNode()` is a little trickier. You need to provide 4 pieces of information:

* A `SytanxNode` object representing the parsed code
* A `SemanticModel` object that comes from the parsed code
* An `Action<Diagnostic>` object that will be called by the method implementation if it found something wrong
* A `CancellationToken` object

Here's the helper method I wrote in the test class to exercise the different cases the diagnostic code could run into:

```c#
private static void TestAnalyzeNode(string code, TextSpan span, 
  Action<IList<Diagnostic>> testDiagnostics)
{
  var tree = SyntaxFactory.ParseSyntaxTree(code);
  var methodNode = tree.GetRoot().FindNode(span);
  var compilation = CSharpCompilation.Create(null,
    syntaxTrees: new[] { tree },
    references: new[]
    {
      new MetadataFileReference(
	    typeof(object).Assembly.Location),
      new MetadataFileReference(
	    typeof(OperationContractAttribute).Assembly.Location)
    });
  var diagnostics = new List<Diagnostic>(); ;
  var addDiagnostic = new Action<Diagnostic>(_ => { diagnostics.Add(_); });

  var analyzer = new IsOneWayOperationAnalyzer();
  analyzer.AnalyzeNode(methodNode, compilation.GetSemanticModel(tree), 
    addDiagnostic, new CancellationToken(false));

  testDiagnostics(diagnostics);
}
```

The first thing I do is get a syntax tree of the given code via `ParseSyntaxTree()`. This is used to get a semantic model via a `CSharpCompilation` object. The method node is found based on a `TextSpan` provided by the caller (more on that in a moment). The `Action` object is created to keep track of the number of diagnostics passed to it. With all of that in place, we can call `AnalyzeNode()` on our analyzer. The last thing to do is to pass the captured `Diagnostic` objects to the caller.

So, how do I use this method in my tests? There are 5 tests I created to exercise the analyzer – I'll show you two of them, one that doesn't create a diagnostic, and one that does. Here's the test for when a method has an attribute on it, but it's not an `OperationContractAttribute`:

```c#
[Fact]
public void AnalzeNodeWhereMethodHasAttributesButNotOperationContractAttribute()
{
  var code =
@"public class AClass
{ 
	[Obsolete]
	public void AMethod() { }
}";
  IsOneWayOperationAnalyzerTests.TestAnalyzeNode(code, new TextSpan(25, 38),
    diagnostics => Assert.Equal(0, diagnostics.Count));
}
```

Here's the test that demonstrates when the analyzer finds the issue:

```c#
[Fact]
public void AnalzeNodeWhereMethodHasOperationContractAttributeAndIsOneWayIsTrueAndReturnTypeIsNotVoid()
{
  var code =
@"using System.ServiceModel;

public class AClass
{
	[OperationContract(IsOneWay = true)]
	public string AMethod() { return string.Empty; }
}";
  IsOneWayOperationAnalyzerTests.TestAnalyzeNode(code, new TextSpan(55, 87),
    diagnostics =>
    {
      Assert.Equal(1, diagnostics.Count);
      var diagnostic = diagnostics[0];
      Assert.Equal(0, diagnostic.AdditionalLocations.Count);
      Assert.Equal(IsOneWayOperationConstants.Category, diagnostic.Category);
      Assert.Equal(IsOneWayOperationConstants.Message, diagnostic.GetMessage());
      Assert.Equal(IsOneWayOperationConstants.DiagnosticId, diagnostic.Id);
      Assert.Equal(IsOneWayOperationConstants.DefaultSeverity, diagnostic.Severity);
      var span = diagnostic.Location.SourceSpan;
      Assert.Equal(74, span.Start);
      Assert.Equal(89, span.End);
    });
}
``` 

Now, you may be wondering how I got the values for all of the `TextSpan` objects. There's another extension you can install when you download Roslyn called the *Syntax Visualizer*. It's an extremely helpful tool that shows you exactly what the syntax tree looks like for the current code file. To find positions in my test code, I temporarily copy the test code to the top of the code file that contains the tests, and use the visualizer to find the span values:

![Unit Testing Roslyn](https://jasonbock.net/images/Unit-Testing-Roslyn-3.png "Unit Testing Roslyn")

As you can see in this figure, the `MethodDeclarationSytanx` node is what I'm looking for, and I see the span starting and ending values of 55 and 142, respectively, which is used to find the start and length values for the `TextSpan` constructor.

With all of this in place, I can test all the paths in my analyzer. The one minor issue I have is that I check the `CancellationToken` two times in `AnalyzeNode()`, and I'd like to make sure that these conditions are tested as well. But `CancellationToken` is a struct and therefore testing that out isn't possible as I have no way to mock or override any of the functionality with the token. It's not ideal, but I can live with that.

## Testing a Code Fix

Figuring out that code may have an issue is one thing. You can also provide the user with a fix by creating a class that inherits from `ICodeFixProvider`. This interface requires you to implement two members:

* A property called `GetFixableDiagnosticIds` that returns an `IEnumerable<string>`
* A method called `GetFixesAsync` that takes 4 arguments and returns a `Task<IEnumerable<CodeAction>>`

As with the analyzer, writing a test for the property is easy:

```c#
[Fact]
public void GetFixableDiagnosticIds()
{
  var fix = new IsOneWayOperationMakeIsOneWayFalseCodeFix();
  var diagnosticIds = fix.GetFixableDiagnosticIds().ToArray();

  Assert.Equal(1, diagnosticIds.Length);
  Assert.Equal(IsOneWayOperationConstants.DiagnosticId, diagnosticIds[0]);
}
```

The test for the method takes more work. The hardest part is the first argument, which is a `Document` type. Unfortunately, you can't just make an instance of the class because it doesn't have any public constructors. Getting a `Document` requires working with the Workspace API, which helps you work with solutions, projects, and the files they contain. Fortunately, there's a class called `CustomWorkspace` that you can use to create a temporary solution. Getting a document from that workspace ends up being a trivial matter. Here's the test in full – I'll explain other details after it:

```c#
[Fact]
public async Task GetFixes()
{
  var code =
@"using System.ServiceModel;

public class AClass
{
	[OperationContract(IsOneWay = true)]
	public string AMethod() { return string.Empty; }
}";
  var projectId = ProjectId.CreateNewId();
  var documentId = DocumentId.CreateNewId(projectId);

  var solution = new CustomWorkspace().CurrentSolution
    .AddProject(projectId, "MyProject", "MyProject", LanguageNames.CSharp)
    .AddMetadataReference(projectId, 
      new MetadataFileReference(typeof(object).Assembly.Location))
    .AddDocument(documentId, "MyFile.cs", code);
  var document = solution.GetDocument(documentId);
  var tree = await document.GetSyntaxTreeAsync();
  var span = new TextSpan(74, 15);

  var location = Location.Create(tree, span);
  var diagnostic = Diagnostic.Create(
    IsOneWayOperationAnalyzer.Descriptor, location);

  var codeFix = new IsOneWayOperationMakeIsOneWayFalseCodeFix();
  var fixes = (await codeFix.GetFixesAsync(document, span,
    new List<Diagnostic> { diagnostic }, 
    new CancellationToken(false))).ToArray();

  Assert.Equal(1, fixes.Length);
  var fix = fixes[0];
  Assert.Equal(
    IsOneWayOperationMakeIsOneWayFalseCodeFixConstants.Description,
    fix.Description);

  var operation = (await fix.GetOperationsAsync(
    new CancellationToken(false))).ToArray()[0] as ApplyChangesOperation;
  var newDoc = operation.UpdatedSolution.GetDocument(documentId);
  var newTree = await newDoc.GetSyntaxTreeAsync();
  var changes = newTree.GetChanges(tree);

  Assert.Equal(1, changes.Count);
  Assert.Equal("fals", changes[0].NewText);
}
```

To create a diagnostic (which is needed in the third argument), you create a `Location` (which uses a `TextSpan` that's passed as the second argument), and then you pass the `Location` into the `Diagnostic.Create()` call. Then you have all you need to call `GetFixesAsync()`.

Now, you can check the `CodeAction` objects returned by `GetFixesAsync()`, but that doesn't show if our fix has the right tree in place. To do that, you call `GetOperationsAsync()` on the `CodeAction`, and cast the `CodeActionOperation` to `ApplyChangesOperation`. This feels a little dirty – I personally don't like casting types like this as it can be fragile – but the end result is that we can get the tree changes from the updated solution. The main change is that "true" should have turned into "false", so the only change we should find is "fals".

## Test Performance

At this point I think there's a decent test suite in place for the diagnostic and the code fix. But you may be wondering what the performance of the test looks likes. Here's a screen shot of the tests and their respective execution times:

![Unit Testing Roslyn](https://jasonbock.net/images/Unit-Testing-Roslyn-4.png "Unit Testing Roslyn")

Michael Feathers, in his book, "Working Effectively with Legacy Code", says that a unit test should execute quickly. The average with these tests is 226 ms, which is slower than the typical average I try to shoot for, which is 10 ms. But I'm personally OK with this. For one thing, there's really no way to test code that uses the Roslyn API other than to directly use its members. And now I have tests that I can run in less than 3 seconds, which means I can refactor my class's implementations and have high confidence in knowing if I broke something expected or not.

## Conclusion

Testing code always pays off in the end. Even if you're working with a code base that may seem daunting due to its complexity, don't write off writing tests. I've been excited about Roslyn for a while now, and it's good to see that it's not hard to get tests in place for code that uses Roslyn. There are some things that are not ideal, but overall it's a straightforward process. I encourage you to start playing with the Roslyn bits as Microsoft has confirmed that the next version of Visual Studio will include the new compiler infrastructure. You may end up writing some cool extensions that you can easily write test for as well.

> Published: 05.13.2014 10:43:00 AM CST