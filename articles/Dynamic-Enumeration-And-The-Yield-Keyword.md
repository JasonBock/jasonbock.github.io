---
title: Dynamic Enumeration and the `yield` Keyword
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Dynamic Enumeration and the `yield` Keyword

Ever since the new 2.0 C# features were announced, the one feature that puzzled me was `yield`. Well ... "puzzled" is the wrong word. I understood what it was doing, and I saw the examples where trivial 1.1 examples were reduced to small, elegant code bases. But what I didn't see was where it could really be useful. It was easy to see how generics were going to permeate themselves all over the place in 2.0 code, but `yield` didn't seem like anything special or cool.

That all changed on Friday.

The team lead and I were working on some service discovery code to make it asynchronous so it could tie into the asynchronous page architecture in ASP.NET 2.0. (It'll also be nice for 1.x applications as they'll be able to use this as well, but that's not important right now). I don't want to divulge the code base too much, but to distill the problem down to core, what the request-response code was doing was going through three stages of endpoint lookups:

* If an endpoint was provided by a developer, try that first.
* If that doesn't work (or no pre-defined endpoint was given), go through the cached endpoint list.
* If the cache doesn't exist or none of the cached endpoints worked, use UDDI discovery to get a list of endpoints and use those.

We also have to ignore endpoints that failed in a previous step.

Anyway, I wanted to tear out the endpoint enumeration into its own class so we could test it out and make sure it worked correctly. The current code had the endpoint enumeration tied directly into the current request-response method, and that's ugly. Now, my lead agreed with me, but we realized that making that an enumeration would be a bit ugly because we have to keep track of the state of the enumeration. We actually didn't finish this part on Friday (we got the asynchronous stuff working correctly, which was a big win in its own right), so I bashed out the code this weekend on my own, not only to solve it, but as soon as I mentioned the idea to the lead, I saw just how easy `yield` makes the enumeration. Here's the code for the 1.1 version:

```c#
public sealed class ServiceEndpointEnumeration : IEnumerator, IEnumerable
{
  private enum States
  {
    LastKnown,
    Cached,
    Lookup
  }

  private Uri currentEndpoint;
  private int currentIndex;
  private Uri lastKnownEndpoint;
  private ArrayList endPoints = new ArrayList();
  private States currentState = States.Cached;
  private bool visitedLastKnown, visitedCached, visitedLookup;

  public ServiceEndpointEnumeration() : base() {}

  public ServiceEndpointEnumeration(Uri lastKnownEndpoint) : this() 
  {
    this.lastKnownEndpoint = lastKnownEndpoint;
    this.currentState = States.LastKnown;
  }

  private static ArrayList GetCachedEndpoints()
  {
    ArrayList cachedEndpoints = new ArrayList();
    cachedEndpoints.AddRange(new Uri[] {new Uri("http://www.cached1.com"), 
      new Uri("http://www.cached2.com")});
    return cachedEndpoints;
  }

  private static ArrayList GetLookedUpEndpoints()
  {
    ArrayList lookedUpEndpoints = new ArrayList();
    lookedUpEndpoints.AddRange(new Uri[] {new Uri("http://www.lookup1.com"), 
      new Uri("http://www.lookup2.com"),
      new Uri("http://www.lookup3.com")});
    return lookedUpEndpoints;
  }

  private void AddNewEndpoints(ArrayList newEndpoints)
  {
    foreach(Uri newEndpoint in newEndpoints)
    {
      if(this.endPoints.Contains(newEndpoint) == false)
      {
        this.endPoints.Add(newEndpoint);
      }
    }
  }

  public IEnumerator GetEnumerator()
  {
    return this;
  }

  public bool MoveNext()
  {
    return this.ProcessState();
  }

  public bool ProcessState()
  {
    bool hasNext = false;

    switch(this.currentState)
    {
      case States.LastKnown:
        hasNext = this.ProcessLastKnownState();
        break;
      case States.Cached:
        hasNext = this.ProcessCachedState();
        break;
      case States.Lookup:
        hasNext = this.ProcessLookupState();
        break;
      default:
        break;
    }

    return hasNext;
  }

  public bool ProcessLastKnownState()
  {
    bool hasNext = false;

    if(this.visitedLastKnown == false)
    {
      this.visitedLastKnown = true;
      this.endPoints.Add(this.lastKnownEndpoint);

      if(this.currentIndex < this.endPoints.Count)
      {
        this.currentEndpoint = this.endPoints[this.currentIndex++] as Uri;
        hasNext = true;
      }
      else
      {
        hasNext = this.ProcessCachedState();
      }
    }
    else
    {
      hasNext = this.ProcessCachedState();
    }

    return hasNext;
  }

  public bool ProcessCachedState()
  {
    bool hasNext = false;

    if(this.visitedCached == false)
    {
      this.visitedCached = true;
      this.AddNewEndpoints(GetCachedEndpoints());
    }

    if(this.currentIndex < this.endPoints.Count)
    {
      this.currentEndpoint = this.endPoints[this.currentIndex++] as Uri;
      hasNext = true;
    }
    else
    {
      hasNext = this.ProcessLookupState();
    }

    return hasNext;
  }

  public bool ProcessLookupState()
  {
    bool hasNext = false;

    if(this.visitedLookup == false)
    {
      this.visitedLookup = true;
      this.AddNewEndpoints(GetLookedUpEndpoints());
    }

    if(this.currentIndex < this.endPoints.Count)
    {
      this.currentEndpoint = this.endPoints[this.currentIndex++] as Uri;
      hasNext = true;
    }

    return hasNext;
  }

  public void Reset()
  {
    this.endPoints.Clear();
    this.currentEndpoint = null;

    if(this.lastKnownEndpoint == null)
    {
      this.currentState = States.Cached;
    }
    else
    {
      this.currentState = States.LastKnown;
    }
  }

  public object Current
  {
    get
    {
      return this.currentEndpoint;
    }
  }
}
```

That's a lot of code, but it's necessary to implement `IEumerable` and `IEnumerator`. I have to keep track of `Reset()` calls, what the current value is, when to move next and if there are more values to return. Frankly, it's not hard, but it's yucky. When you look at the original rules, it's really not that hard, but when you tie them into enumeration in the 1.x world, you have to keep track of all this state.

Now look how this code changes when I jump to 2.0:

```c#
public sealed class ServiceEndpoints
{
  private Uri lastKnownEndpoint;

  public ServiceEndpoints() : base() {}

  public ServiceEndpoints(Uri lastKnownEndpoint) : base() 
  {
    this.lastKnownEndpoint = lastKnownEndpoint;
  }

  private static List<Uri> GetCachedEndpoints()
  {
    List<Uri> cachedEndpoints = new List<Uri>();
    cachedEndpoints.AddRange(new Uri[] {new Uri("http://www.cached1.com"), 
      new Uri("http://www.cached2.com")});
    return cachedEndpoints;
  }

  private static List<Uri> GetLookedUpEndpoints()
  {
    List<Uri> lookedUpEndpoints = new List<Uri>();
    lookedUpEndpoints.AddRange(new Uri[] {new Uri("http://www.lookup1.com"), 
      new Uri("http://www.lookup2.com"),
      new Uri("http://www.lookup3.com")});
    return lookedUpEndpoints;
  }

  public IEnumerator<Uri> GetEnumerator()
  {
    List<Uri> foundEndpoints = new List<Uri>();

    if(this.lastKnownEndpoint != null)
    {
      foundEndpoints.Add(this.lastKnownEndpoint);
      yield return this.lastKnownEndpoint;
    }

    List<Uri> cachedEndpoints = GetCachedEndpoints();
    this.PruneEndpoints(foundEndpoints, cachedEndpoints);

    foreach(Uri cachedEndpoint in cachedEndpoints)
    {
      yield return cachedEndpoint;
    }

    List<Uri> lookupEndpoints = GetLookedUpEndpoints();
    this.PruneEndpoints(foundEndpoints, lookupEndpoints);

    foreach(Uri lookupEndpoint in lookupEndpoints)
    {
      yield return lookupEndpoint;
    }
  }

  private void PruneEndpoints(List<Uri> currentEndpoints, List<Uri> newEndpoints)
  {
    for(int i = newEndpoints.Count - 1; i >= 0; i--)
    {
      if(currentEndpoints.Contains(newEndpoints[i]) == true)
      {
        newEndpoints.RemoveAt(i);
      }
      else
      {
        currentEndpoints.Add(newEndpoints[i]);
      }
    }
  }
}
```

From 183 lines of code to 77! I no longer have to keep track of resets and current values and moving to the next one. It's so much cleaner than what I can do in 1.x, especially when I have to change the source of the data during the enumeration. Having a type-safe `Uri` collection with no work is also nice too.

> Published: 10.16.2005 01:14:18 PM CST