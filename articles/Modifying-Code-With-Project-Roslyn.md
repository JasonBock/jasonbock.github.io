---
title: Modifying Code with Project Roslyn
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Modifying Code with Project Roslyn

## Enforcing Consistency in Code

One principle I try to uphold on the projects I'm on is consistency. The more you can get developers to agree on how things should be done, the easier the code will be to maintain, even if some of your original choices end up being ones that weren't so good in retrospect. This also rings true with the way you write your code. If everyone writes their code using the same guidelines and rules, it's far easier for a developer to dive into any piece of code along the stacks and layers that usually exist in an application. However, most of the coding standards that I've seen have rules in place that don't affect what the compiled code will do. That is, they're codified text manipulation rules that should be handled by a tool so the developer really doesn't have to think about them when coding. A simple example is this:

```c#
public void MyMethod() {
         // ...
}
public void MyMethod()
{
         // ...
}
```

Some clients that I've worked for will insist that the opening curly brace show up on the same line with the method definition, while others want it on the next line. From an execution standpoint, it doesn't matter where it is, but developers should be consistent when they define a method. But really, do you need extensive code reviews to ensure that the curly brace shows up in the right spot?

What you really want is a tool to automatically do this for you. Visual Studio 2010 already has some formatting rules in place that you can use to repeatedly handle coding standards. For example, here's a screen shot of Visual Studio 2010's formatting options to handle the position of the opening curly brace:

![Modifying Code](https://jasonbock.net/images/Modifying-Code-1.png "Modifying Code")

You can access this dialog window via the *Tools -> Options ...* menu option. Once you specify the option, Visual Studio 2010 will format the code for you when you create a new method. You can also force formatting by selecting the *Edit -> Advanced -> Format Document* option. If you install the PowerCommands for Visual Studio 2010 extension, you can have the formatting rules automatically applied whenever you save your document:

![Modifying Code](https://jasonbock.net/images/Modifying-Code-2.png "Modifying Code")

This is all well and good, but we can't expect Microsoft to create options for every possible standard that we could come up with. Moreover, some coding standards may have rules that go beyond simple formatting rules. For example, there may be a standard that all `DateTime` values must be UTC-based, so any calls to the `DateTime.Now` property are disallowed. Rather, a developer should use `DateTime.UtcNow`. Right now, you can't easily create rules like this, but with Project Roslyn, you will be able to.

## What is Project Roslyn?

Project Roslyn is a framework created by Microsoft to give a developer deep access to the compilation process. Up until now, the compilers are a black box – that is, you don't get access to the various stages that a compiler goes through to create the result you desire. With Roslyn, you can give it a piece of source code, parse it, and see all the nodes and tokens in a tree structure. For example, here's how you can compile a code snippet with Roslyn:

```c#
var code = "public class MyClass { }";
var tree = Syntax.ParseCompilationUnit(code);
```

The resulting tree structure looks like this:

![Modifying Code](https://jasonbock.net/images/Modifying-Code-3.png "Modifying Code")

When you install the Roslyn CTP, you get a couple of visualizers that you can use to see the structure of the tree. As you can see in the screen shot, the trees represent everything that shows up in the code, including whitespace (referred to as trivia). While these trees are immutable, you can easily write code that creates a new tree based on the state of a given tree with the modifications you need. Let's see how you can modify a syntax tree to remove certain directives.

## Creating Code to Remove #region and #endregion Directives

One standard I personally like to enforce in my code is to have it region-free. That is, I like to discourage the usage of the `#region` and `#endregion` directives because they tend to hide classes and methods that may be hundreds, if not thousands, of lines long. That's a big warning flag that your code needs to be restructured such that the class or method doesn't have so many responsibilities. Seeing the structure of the code without the `#region` and `#endregion` noise is what I prefer. Making sure that all the code in the project follows this rule is another issue. Let's see how you can remove these directives with Roslyn.

The first thing you need to do is create the code that will actually transform the Roslyn tree such that it no longer contains the directives:

```c#
public static class SyntaxNodeExtensions
{
  public static SyntaxNode Deregionize(this SyntaxNode @this)
  {
    var nodesWithRegionDirectives =
      from node in @this.DescendentNodesAndTokens()
      where node.HasLeadingTrivia
      from leadingTrivia in node.GetLeadingTrivia()
      where (leadingTrivia.Kind == SyntaxKind.RegionDirective ||
        leadingTrivia.Kind == SyntaxKind.EndRegionDirective)
      select node;
 
    var triviaToRemove = new List<SyntaxTrivia>();
 
    foreach (var nodeWithRegionDirective in nodesWithRegionDirectives)
    {
      var triviaList = nodeWithRegionDirective.GetLeadingTrivia();
 
      for (var i = 0; i < triviaList.Count; i++)
      {
        var currentTrivia = triviaList[i];
 
        if (currentTrivia.Kind == SyntaxKind.RegionDirective ||
          currentTrivia.Kind == SyntaxKind.EndRegionDirective)
        {
          triviaToRemove.Add(currentTrivia);
 
          if (i > 0)
          {
            var previousTrivia = triviaList[i - 1];
 
            if (previousTrivia.Kind == SyntaxKind.WhitespaceTrivia)
            {
              triviaToRemove.Add(previousTrivia);
            }
          }
        }
      }
    }
 
    return triviaToRemove.Count > 0 ?
      @this.ReplaceTrivia(triviaToRemove,
        (_, __) => SyntaxTriviaList.Empty) :
      @this;
  }
}
```

Directive nodes are contained as trivia with other syntax nodes in the tree, like `MethodDeclarationSyntax` or `OperationDeclarationSyntax` nodes. The LINQ query finds all those nodes and tokens that have leading trivia that contain either `RegionDirectiveSyntax` or `EndRegionDirectiveSyntax` nodes. Then you look at all of the leading trivia for these nodes and tokens, and find those directives within the trivia. You also look at the previous trivia node to see if it's whitespace, because you'll want to remove that one as well for formatting purposes. These nodes are added to a list that's used at the end of `Deregionize()`. If you've found any trivia nodes to remove, then you create a new tree that replaces the trivia nodes with an empty trivia list, effectively removing that trivia. Otherwise, you return the tree that you got in the first place.

```c#
var code =
@"public class MyClass
{
  #region Constructors
  public MyClass()
    : base() { }
  #endregion

  public string Data { get; set; }
}";
 
var tree = Syntax.ParseCompilationUnit(code);
var newTree = tree.Deregionize();
```

The code in newTree looks like this:

```c#
public class MyClass
{
  public MyClass()
    : base() { }
 
  public string Data { get; set; }
}
```

Slick!

## Integrating Within Visual Studio 2010

At this point, you have code that will remove `#region` and `#endregion` directives from your code. The next step is to make it as seamless as possible to integrate into a developer's experience. That is, you want this reformatting to occur without user intervention. There are a couple of Roslyn project types that let you integrate your code within Visual Studio 2010, such as a *Code Refactoring* project, but that requires the user to perform an action to invoke the refactoring. There are other extensibility points that you can use to extend Visual Studio 2010, but for this example we'll create a console application that will use the Workspace class in Roslyn.Services.dll to automatically update code files during compilation.

First, you create a console application that has the necessary references to the Roslyn assemblies. Then it's a simple matter to traverse the code files within a given project file:

```c#
class Program
{
  static void Main(string[] args)
  {
    var workspace = Workspace.LoadStandAloneProject(args[0]);
    var solution = workspace.CurrentSolution;
    var newSolution = solution;
 
    foreach (var project in solution.Projects)
    {
      foreach (var document in project.Documents)
      {
        if (document.LanguageServices.Language == LanguageNames.CSharp)
        {
          var tree = document.GetSyntaxTree().Root as SyntaxNode;
          var newTree = tree.Deregionize();
 
          if (tree != newTree)
          {
            var newText = newTree.GetFullTextAsIText();
            newSolution = newSolution.UpdateDocument(document.Id, newText);
          }
        }
      }
    }
 
    if (newSolution != solution)
    {
      workspace.ApplyChanges(solution, newSolution);
    }
  }
}
```

You walk all the files in the project, and for those that are C# code files, you use the `Deregionizer()` extension method to change the tree. If any code files have been changed in the project, the changes are committed via the `ApplyChanges()` method.

To use this console application to modify code in a Visual Studio 2010 project, you include the following line in the *Pre-build event command line* section of a project:

```
"{PathForConsoleAppGoesHere}\Roslyn.Deregionizer.Client.Console.exe" "$(ProjectPath)"
```

This command can be found under the *Build Events* tab for the project properties:

![Modifying Code](https://jasonbock.net/images/Modifying-Code-4.png "Modifying Code")

Note that you'll have to change `{PathForConsoleAppGoesHere}` to match where you put the console application.

Once this is in place, any time you build the project, the code files will automatically have their `#region` and `#endregion` directives removed.

## Conclusion

While Roslyn has just been released for developers to use, it's only in a CTP state. Furthermore, the projected release date based on what's stated on the Roslyn download page slates it for after the Visual Studio 11 release. However, even with this mind, I think it's more than worth it to start spending time with this powerful API. There's so much you can do with Roslyn to make your code cleaner. Look at the samples that come with Roslyn and then see if you can codify the rules and recommendations you want to enforce on projects with Roslyn. The samples are good at demonstrating what you can do with Roslyn (in fact, the code in the Organizing Solution sample was used as guidance to create the Visual Studio 2010 project integration code). Chances are you'll be able to do it!

> Published: 01.19.2012 12:09:00 PM CST