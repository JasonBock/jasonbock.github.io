---
title: Events, Tests, and Code Coverage
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Events, Tests, and Code Coverage

Last weekend I gave my "Writing Better Code" talk. While I was rehearsing the code coverage part, I was originally going to leave off the test assembly, but then I ran into something at the client last week that made me reconsider this proposition. Here's the scenario. Let's say you created a `Workflow` class in a `Workflows` assembly with an implementation that produces patriotic colors:

```c#
public abstract class Workflow
{
  public event EventHandler Completed;
    
  public abstract void Run();
    
  protected void FireCompleted()
  {
    if(this.Completed != null)
    {
      this.Completed(this, EventArgs.Empty);
    }
  }
}

public sealed class PatrioticColorWorkflow : Workflow
{
  public PatrioticColorWorkflow() : base()
  {
    this.Colors = new List<string>();
  }
    
  public override void Run()
  {
    this.Colors = new List<string> { "Red", "White", "Blue" };
    this.FireCompleted();
  }
    
  public List<string> Colors
  {
    get;
    private set;
  }
}
```

You create a test that hooks the `Completed` event:

```c#
[TestClass]
public sealed class PatrioticColorWorkflowTests
{
  [TestMethod]
  public void GetColors()
  {
    PatrioticColorWorkflow workflow = new PatrioticColorWorkflow();
        
    Assert.AreEqual(0, workflow.Colors.Count);
        
    workflow.Completed += (sender, args) => 
    {
      Assert.AreEqual(3, workflow.Colors.Count);
      Assert.IsTrue(workflow.Colors.Contains("Red"));
      Assert.IsTrue(workflow.Colors.Contains("White"));
      Assert.IsTrue(workflow.Colors.Contains("Blue"));
    };
        
    workflow.Run();
  }
}
```

OK, so far so good. When code coverage is enabled on the `Workflows` assembly, you'll see that the coverage is. But let's say a bug is introduced where the event is no longer fired:

```c#
public sealed class PatrioticColorWorkflow : Workflow
{
  public PatrioticColorWorkflow() : base()
  {
    this.Colors = new List<string>();
  }
    
  public override void Run()
  {
    this.Colors = new List<string> { "Red", "White", "Blue" };
    //this.FireCompleted();
  }
    
  public List<string> Colors
  {
    get;
    private set;
  }
}
```

Here's the problem. The test will still succeed! Now, I could fix the issue by introducing a boolean test:

```c#
[TestClass]
public sealed class PatrioticColorWorkflowTests
{
  [TestMethod]
  public void GetColors()
  {
    PatrioticColorWorkflow workflow = new PatrioticColorWorkflow();
        
    Assert.AreEqual(0, workflow.Colors.Count);
    
    bool wasCompletedHandled = false;
            
    workflow.Completed += (sender, args) => 
    {
      wasCompletedHandled = true;
      Assert.AreEqual(3, workflow.Colors.Count);
      Assert.IsTrue(workflow.Colors.Contains("Red"));
      Assert.IsTrue(workflow.Colors.Contains("White"));
      Assert.IsTrue(workflow.Colors.Contains("Blue"));
    };
        
    workflow.Run();
    Assert.IsTrue(wasCompletedHandled);
  }
}
```

But what I should also do is consider adding the test assembly in the code coverage processing:

![Events and Code Coverage](https://jasonbock.net/images/Events-Code-Coverage.png "Events and Code Coverage")

Now I would see that some of my testing code isn't running, which would be a red flag, because if all the tests are passing, all of the testing code should run as well, right?

As I said before, refactoring the test to check that the event was actually fired is something I should've done in the first place. But adding all of the code in code coverage is something to keep in mind. I used to think that testing code should just be "good enough" - that is, it doesn't need Code Analysis or code coverage, etc. I'm rethinking that notion every day. Even if the tests aren't being shipped with the product, they're a vital part of making a product successful and should be treated with the same amount of "respect" as the code it's testing.

> Published: 04.08.2008 08:54:44 AM CST