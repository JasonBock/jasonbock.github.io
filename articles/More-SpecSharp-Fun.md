---
title: More Spec# Fun
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# More Spec# Fun

Another thing I'm looking at with Spec# are invariants. But ... they don't seem to be working as expected:

```c#
public sealed class Person
{
  invariant age >= 0;
    
  private string! firstName = string.Empty;
  private string! lastName = string.Empty;
  private int age;
    
  private Person() : base() { }
            
  public Person(string! firstName, string! lastName, int age) : this()
    requires firstName.Length > 0;
    requires lastName.Length > 0;
  {
    this.firstName = firstName;
    this.lastName = lastName;
    this.age = age;
  }

  public int Age
  {
    get
    {
      return this.age;
    }
  }
}
```

I've left out the getters for the first and last names - they're not important for the conversation at hand. Now, since I have an invariant on the `age` field that it must be non-negative [1], I expected the following test to fail:

```c#
[Test]
public void CreateSpecSharpPersonWithNegativeAge()
{
  SSP.Person person = new SSP.Person("Joe", "Smith", -50);
  Console.Out.WriteLine(person.Age);
}
```

But the test succeeds. Now, if I change the contract to this:

```c#
public sealed class Person
{
  private string! firstName = string.Empty;
  private string! lastName = string.Empty;
  private int age;
    
  private Person() : base() { }
            
  public Person(string! firstName, string! lastName, int age) : this()
    requires firstName.Length > 0;
    requires lastName.Length > 0;
    requires age >= 0;
  {
    this.firstName = firstName;
    this.lastName = lastName;
    this.age = age;
  }

  public int Age
  {
    get
    {
      return this.age;
    }
  }
}
```

The test will fail as expected. So, either I don't get how invariants work, or Spec# has a bug with respect to enforcing invariants. I'm leaning towards the former, but Spec# seems pretty "twichy" in its current beta release so who knows.

[1] I could've used a `uint` to enforce the contract, but I was more curious about how Spec# handles invariants.

> Published: 09.27.2006 08:39:34 AM CST