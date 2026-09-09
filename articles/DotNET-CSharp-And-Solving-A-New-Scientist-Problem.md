---
title: .NET, C#, and Solving a New Scientist Problem
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# .NET, C#, and Solving a New Scientist Problem

## Introduction

I've been doing more and more research into the .NET framework with every passing day. Recently, however, an odd "challenge" of sorts motivated me to actually create assemblies and use them. In this article, I'll talk about the "challenge" and I how used C# and .NET to solve it.

## New Scientist and Primes

I just started subscribing to New Scientist, a science magazine that's published across the pond (in England, that is). It's a great magazine and I highly recommend getting your own subscription. Anyway, I was flipping through the latest edition that I received in the mail today (27 January 2001), and I stumbled across their Enigma column. It's a puzzle that, if you solve it first, you receive a prize (i.e. money) for your efforts. Here's the puzzle:

> I invite you to select a three-digit prime number such that if you reverse the order of its digits you form a larger three-digit prime number. Furthermore the first two digits and the last two digits of these two three-digit prime numbers must themselves be four two-digit prime numbers, each one different from all the others.

I'm sure a mathematician skilled in the complexities of number theory could find a simple, yet elegant way to solve this. Me? I write a program to solve it for me. Brute force all the way!

Now, I could've easily cranked out a program in VB to solve this with little effort, but I wanted to try to write this in C#. Namely, I wanted to write a component that would give you a list of prime numbers, which the client could use to solve the problem. The component could be reused for other prime generation issues in the future (if need be). As I'm still learning the ins and out of C#, this was a good time to dive in and see what I could come up with.

## Designing and Implemeting the Component

As always, design comes before code, even if I'm dying to start developing. The first thing to tackle is, how do I generate a list of primes? That's easy - use the Sieve of Eratosthenes. When you don't need to reinvent the wheel, don't.

Now the next question I had is, how do I write the component? I decided to create a C# class library that would contain the classes I would need. All of the classes would reside in the `Viktor.Primes` namespace. I'm a firm believer in separating interfaces from implementation, so I decided to create an abstract class called `PrimeGenerator` so I could swap different prime generators if I ever needed to with little pain.

One problem that came to mind was, how should I get the data to the client? Fortunately, .NET has the `IEnumerable` interface that defines one method, `GetEnumerator`, which returns an `IEnumerator` interface. I thought that I would have to create my own implementation of `IEnumerator`, but I quickly realized that I could use an `ArrayList` in my implementation of `PrimeGenerator`, which has a `GetEnumerator` method already defined. Cool! This was going to be easy.

So here's the definition of `PrimeGenerator`:

```c#
public abstract class PrimeGenerator : IEnumerable; 
{ 
  public const uint DEFAULT_MAX_SIZE = 1000; 
  public const uint MAX_SIZE = 50000; 
 
  public PrimeGenerator() 
  { 
    this.Init(DEFAULT_MAX_SIZE); 
  } 
 
  public PrimeGenerator(uint SearchRange) 
  { 
    if(SearchRange > MAX_SIZE) 
    { 
      throw(new SizeTooLargeException("Search range is too large.")); 
    } 
    this.Init(SearchRange); 
  } 
 
  protected abstract void Init(uint SearchRange); 
  public abstract IEnumerator GetEnumerator(); 
  public abstract PrimeCheck IsPrime(uint CheckValue);
}
```

Note that `SizeTooLargeException` is an exception I defined - I just didn't show it here. Since `PrimeGenerator` is defined as an abstract class, I have to defer the implementation of `GetEnumerator` to its descendants. To me, it seems like an unnecessary piece of code - why should I have to type in that line? If the compiler would see that I haven't explicitly overridden `GetEnumerator`, it would know that I've deferred implementation. But so be it - that's the way it is in C#.

The `IsPrime` method allows a client to check to see if the object knows that `CheckValue` is prime. It returns one of three possible values, which are defined by the `PrimeCheck` enumeration:

```c#
public enum PrimeCheck
{
  Yes,
  No,
  Unknown
};
```

I doubt that any implementation could easily check any number given to see if it's prime. I know that I won't be able to. That's why the `Unknown` value is there. If the object doesn't know if it's prime or not, it can punt with this value.

I also created an `Init` method that is meant for subclasses to define. As `PrimeGenerator`'s constructors call `Init`, all the subclass needs to do is define `Init`. And that's exactly what I do in `VikPrimeGenerator`:

```c#
public class VikPrimeGenerator : PrimeGenerator
{
  private ArrayList primes = new ArrayList();
    
  public VikPrimeGenerator() : base()
  {
    
  }
    
  public VikPrimeGenerator(uint SearchRange) : base(SearchRange)
  {
    
  }
    
  public override IEnumerator GetEnumerator()
  {
    return primes.GetEnumerator();
  }
    
  protected override void Init(uint SearchRange)
  {
    //  Deleted.
  }
}
```

Check out the code for `Init`'s implementation in the downloadable code - it's pretty straightforward when you check the online reference I listed, so I've deleted it here. What's nice is that the client can now do this:

```c#
IEnumerator enumPrimes = (new VikPrimeGenerator(100)).GetEnumerator();
while(enumPrimes.MoveNext())
{
  uint prime = (uint)enumPrimes.Current;
}
```

If you look at the code to generate the primes, I add the primes to primes, so I don't have to worry about implementing `IEnumerator` on my own. It's done for me.

## Creating the Client to Solve the Problem, and ... The Answer, Please!

Once I created the assembly, I created a C# Windows application. I wanted it to do two things: list a bunch of primes given an upper limit, and then solve the New Scientist problem. Here's a screen shot of the application:

![Prime Generator](https://jasonbock.net/images/PrimeGenerator.png "Prime Generator")

The first part was easy:

```c#
protected void cmdSearch_Click (object sender, System.EventArgs e)
{
  try
  {
    lstPrimes.Items.Clear();
    IEnumerable enumPrimes = new Viktor.Primes.VikPrimeGenerator(this.txtLimit.Text.ToUInt32());		
    IEnumerator enumPrime = enumPrimes.GetEnumerator();
    while(enumPrime.MoveNext())
    {
      uint prime = (uint)enumPrime.Current;
      lstPrimes.Items.Add(prime);	
    }
  }
  catch(Viktor.Primes.SizeTooLargeException stle)
  {
    MessageBox.Show("The given upper limit is too large.");
  }
  catch(Exception ex)
  {
    MessageBox.Show("Unanticipated exception:  " + ex.Message);
  }
}
```

I get a list of primes from `VikPrimeGenerator`, and display them in the list box. If the user enters a value that goes beyond the upper limit of `PrimeGenerator` (i.e. `MAX_SIZE`), then a `Viktor.Primes.SizeTooLargeException` exception is thrown. Also, if the user doesn't enter a value that can be converted into a `uint`, I catch that exception as well in the second exception handler.

The second part isn't as easy - that is, finding the answer to the original problem. But it's not too hard; the biggest problem is taking a number in C#, flipping it, and extracting the four 2-digits numbers from the two 3-digit numbers. Here's the code:

```c#
protected void cmdNSSolver_Click (object sender, System.EventArgs e)
{
  try
  {
    PrimeGenerator pg = new VikPrimeGenerator((uint)999);
    //  OK, we have the primes from 1 to 999.
    //  Now go through the list from 100 to 999,
    //  and do the search
    uint i = 100;
    bool numberFound = false;
    do
    {
      if(pg.IsPrime(i) == PrimeCheck.Yes)
      {
        //  Now flip the digit around.
        char[] data = i.ToString().ToCharArray();
        Array.Reverse(data);
        String newStr = new String(data);
        uint uiRev = newStr.ToUInt32();
                
        //  First check:  The reversed digit must be bigger.
        if(uiRev > i)
        {
          //  The reversed digit must be prime.
          if(pg.IsPrime(uiRev) == PrimeCheck.Yes)
          {
            //  OK, now we need to extract the four digits.
            uint value1 = (new String(uiRev.ToString().ToCharArray(0, 2))).ToUInt32();
            uint value2 = (new String(uiRev.ToString().ToCharArray(1, 2))).ToUInt32();
            uint value3 = (new String(i.ToString().ToCharArray(0, 2))).ToUInt32();
            uint value4 = (new String(i.ToString().ToCharArray(1, 2))).ToUInt32();
                        
            //  They must all be unique...
            bool areUnique = true;
            Hashtable checker = new Hashtable();
            try
            {
              checker.Add(value1, value1);
              checker.Add(value2, value2);
              checker.Add(value3, value3);
              checker.Add(value4, value4);
            }
            catch(Exception ex)
            {
              areUnique = false;
            }
                        
            if(areUnique)
            {
              //  and they must all be prime.
              if(pg.IsPrime(value1) == PrimeCheck.Yes &&
                pg.IsPrime(value2) == PrimeCheck.Yes &&
                pg.IsPrime(value3) == PrimeCheck.Yes &&
                pg.IsPrime(value4) == PrimeCheck.Yes)
              {
                numberFound = true;
                lblAnswerInfo.Text = i.ToString();
                lblFlippedInfo.Text = uiRev.ToString();
                lblDigitAInfo.Text = value1.ToString();
                lblDigitBInfo.Text = value2.ToString();
                lblDigitCInfo.Text = value3.ToString();
                lblDigitDInfo.Text = value4.ToString();
              }
            }
          }
        }
      }
      i++;
    } while (i < 1000 && numberFound == false);
  }
  catch(Viktor.Primes.SizeTooLargeException stle)
  {
    MessageBox.Show("The given upper limit is too large.");
  }
  catch(Exception ex)
  {
    MessageBox.Show("Unanticipated exception:  " + ex.Message);
  }
}
```

The first thing I do is create a `PrimeGenerator`-based object with an upper limit of 999. Then, I iterate through the primes from 100 to 999. If the current value for `i` is prime, I flip the number and check to see if the flipped number is bigger than the original one. If it is, I have to check to see if it's prime. I then extract the four 2-digit numbers, and make sure they're all unique by storing them in a `Hashtable`. If they are, I see if they're prime, and if they are, I display the answer to the client.

OK, just in case you don't have .NET installed, I'll give you the answer. The number is 179 - I'll let you verify it on your own.

## Conclusion

In this brief example, I created a component in .NET that a client can use to generate a list of primes and solve a problem that can be easily done on a computer. Granted, I may be doing some inefficient code within the component and/or the client, but the nice thing is that the implementations can change independent of each other. Finally, I have to say this - coding in .NET is so much easier than anything I've ever used. And no, I don't get paid by Microsoft to say this either. It's just what I think. But for all the bad press that .NET is receiving (founded or unfounded), it's a vast improvement over their current toolset. I like C#, but nothing was preventing me from writing the client or component in VB.NET, or to mix and match them if wanted to. If you can, take a look at .NET - it's got a lot of cool things within.

> Published: 02.14.2001 12:00:00 AM CST