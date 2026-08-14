---
title: Just How Fast Is That Code?
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Just How Fast Is That Code?

Recently I added a `CreateBigInteger()` method to my `SecureRandom` class in the `Spackle` library I have on CodePlex. You give it a number that defines how many digits you want, and that's what the method gives you, like this:

```c#
var number = new SecureRandom().CreateBigInteger(16);
// number will be something like this: 4796435982615820
```

My initial implementation was a very naive one:

```c#
public BigInteger GetBigInteger(ulong numberOfDigits)
{
  if(numberOfDigits == 0)
  {
    throw new ArgumentException(
      "The number of digits cannot be zero.", "numberOfDigits");
  }

  var digit = new StringBuilder();

  for(var i = 0u; i < numberOfDigits; i++)
  {
    if(i == 0)
    {
      digit.Append(this.Next(1, 10));
    }
    else
    {
      digit.Append(this.Next(10));
    }
  }

  return BigInteger.Parse(digit.ToString(), CultureInfo.CurrentCulture);
}
```

Using a `StringBuilder` like that wasn't the smartest thing I've ever done. But my first goal was to get a working implementation in place with tests, and then go back and improve it. I also created a performance test to ensure I could compare apples to apples:

```c#
[TestMethod]
public void TimeCreateBigInteger()
{
  const int iterations = 100;

  for(var i = 10u; i <= 100000u; i *= 10u)
  {
    this.TestContext.WriteLine("{0} size, total time is {1}", i, new Action(() =>
    {
      using(var random = new SecureRandom())
      {
        for(var j = 0; j < iterations; j++)
        {
          random.GetBigInteger(i);
        }
      }
    }).Time());
  }
}
```

With this test in place, my results would look like this:

```
10 size, total time is 00:00:00.0091400
100 size, total time is 00:00:00.0492115
1000 size, total time is 00:00:00.5380570
10000 size, total time is 00:00:09.1498312
100000 size, total time is 00:08:35.1854318
```

My second attempt was to try and reduce the number of times I was in the for loop:

```c#
public BigInteger GetBigInteger(ulong numberOfDigits)
{
  if(numberOfDigits == 0)
  {
    throw new ArgumentException(
      "The number of digits cannot be zero.", "numberOfDigits");
  }

  var digit = new StringBuilder();

  if(numberOfDigits == 1)
  {
    digit.Append(this.Next(1, 10));
  }
  else
  {
    var remainingDigits = numberOfDigits;

    while(remainingDigits > 0)
    {
      if(remainingDigits >= 9)
      {
        digit.Append(this.Next(100000000, 1000000000));
        remainingDigits -= 9;
      }
      else
      {
        digit.Append(this.Next((int)(Math.Pow(10, remainingDigits - 1)), 
          (int)(Math.Pow(10, remainingDigits))));
        remainingDigits = 0;
      }
    }
  }

  return BigInteger.Parse(digit.ToString(), CultureInfo.CurrentCulture);
}
```

I still didn't remove the `StringBuilder`, but I thought this would improve things. Was I ever wrong:

```
10 size, total time is 00:00:00.0076844
100 size, total time is 00:00:00.0107465
1000 size, total time is 00:00:00.1392907
10000 size, total time is 00:00:05.3649017
100000 size, total time is 00:07:36.3556192
```

This is a perfect illustration why you **must** have numbers to back up your investigations. I did improve things a bit, but obviously I need to do some more surgery to get things better. Having these numbers in place is a reality check to squash my blind hopes.

My 3rd (and so far final) attempt continues down the path in the 2nd version, but it removes the `StringBuilder` entirely:

```c#
public BigInteger GetBigInteger(ulong numberOfDigits)
{
  if(numberOfDigits == 0)
  {
    throw new ArgumentException(
      "The number of digits cannot be zero.", "numberOfDigits");
  }

  var digit = BigInteger.Zero;

  if(numberOfDigits == 1)
  {
    digit = new BigInteger(this.Next(0, 10));
  }
  else
  {
    var remainingDigits = numberOfDigits;

    while(remainingDigits > 0)
    {
      if(remainingDigits >= 9)
      {
        digit = (1000000000 * digit) + this.Next(100000000, 1000000000);
        remainingDigits -= 9;
      }
      else
      {
        digit = ((int)(Math.Pow(10, remainingDigits)) * digit) + 
          this.Next((int)(Math.Pow(10, remainingDigits - 1)), 
          (int)(Math.Pow(10, remainingDigits)));
        remainingDigits = 0;
      }
    }
  }
                    
  return digit;
}
```

Essentially, I'm just moving existing digits over with the multiplication, and inserting new numbers with the addition.

Now the results look much better:

```
10 size, total time is 00:00:00.0052317
100 size, total time is 00:00:00.0069716
1000 size, total time is 00:00:00.0539089
10000 size, total time is 00:00:01.0511939
100000 size, total time is 00:00:55.0149691
```

This is roughly an order of magnitude better. I might be able to make this even better (I already have some ideas in mind), but now I have a baseline test in place to make sure I'm making things better from a performance perspective. A deeper analysis with profiling tools would be advantageous, but this poor man's approach has already paid off.

> Published: 05.14.2010 09:56:00 AM CST