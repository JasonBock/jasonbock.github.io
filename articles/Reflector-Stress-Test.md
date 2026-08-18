---
title: Reflector Stress Test
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Reflector Stress Test

To iron out the bugs in my [ExceptionFinder](https://github.com/JasonBock/ExceptionFinder) add-in, I wrote a test method that examined every method in mscorlib. As it stands, it's not really a unit test, so I didn't want it running all the time (and I have more specific tests that address the errors I ended up finding), but I wanted to share the code with those who may be writing [Reflector](https://www.red-gate.com/products/reflector/) add-ins. If your code can run against everything in mscorlib that's another step in the process:

```c#
[TestMethod, Timeout(Int32.MaxValue)]
public void Stress()
{
  IAssemblyManager manager = // Get the manager from Reflector's IServiceProvider...
  var methods = from IAssembly assembly in manager.Assemblies
    where assembly.Name == "mscorlib"
    from IModule module in assembly.Modules
    from ITypeDeclaration type in module.Types
    from IMethodDeclaration method in type.Methods
    select method;

  int methodCount = 0;
  int analysisErrorCount = 0;
    
  foreach(IMethodDeclaration method in methods)
  {
    methodCount++;

    try
    {
      // Run your code against the method...
    }
    catch(Exception ex)
    {
      analysisErrorCount++;

      this.TestContext.WriteLine("Exception occurred: " + ex.GetType().FullName);
      this.TestContext.WriteLine("\tMessage: " + ex.Message);
      this.TestContext.WriteLine("\tStack Trace: " + ex.StackTrace);

      ITypeDeclaration type = method.DeclaringType as ITypeDeclaration;
      this.TestContext.WriteLine("\tType: " + type.Namespace + "." + type.Name);
      this.TestContext.WriteLine("\tMethod: " + method.Name +
        (method.Static ? " (static)" : " (instance)"));

      if(method.Parameters.Count > 0)
      {
        this.TestContext.WriteLine("\t\tParameters:");
        foreach(IParameterDeclaration parameter in method.Parameters)
        {
          this.TestContext.WriteLine("\t\t\t" + parameter.Name + ", " + parameter.ParameterType.ToString());
        }
      }

      this.TestContext.WriteLine(string.Empty);

      printedErrorCount++;
    }
  }

  this.TestContext.WriteLine("Total number of methods: " + methodCount);
  this.TestContext.WriteLine("Total number of errors: " + analysisErrorCount);
    
  if(analysisErrorCount > 0)
  {
    Assert.Fail("Too many errors! Method count: " + methodCount + 
      ", error count: " + analysisErrorCount);
  }
}
```

Note that this may produce a ton of errors, so you may want to throttle the number of printed errors. It's also higly ironic that I'm catching the Exception type here, when the point of my add-in is to try and prevent that! But in this case, I have no choice, because I have no idea what kind of exception may be thrown. I wouldn't intentionally write this code in a production application - in fact, this code was temporary, only around long enough to diagnose errors.

Happy coding!

> Published: 04.21.2008 07:50:50 AM CST