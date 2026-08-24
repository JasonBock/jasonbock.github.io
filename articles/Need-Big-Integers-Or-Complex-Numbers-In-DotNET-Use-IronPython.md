---
title: Need Big Integers or Complex Numbers in .NET? Use IronPython!
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Need Big Integers or Complex Numbers in .NET? Use IronPython!

Recently I noticed that IronPython has a separate assembly called IronMath.dll that contains two interesting classes, `BigInteger` and `Complex64`. The nice thing is, since they're just .NET classes, they can be used in other languages, like, oh, C#:

```c#
using IronMath;
using System;
using System.Collections.Generic;
using System.Text;

namespace UsingIronMath
{
  class Program
  {
    static void Main(string[] args)
    {
      // 99782067518340931
      BigInteger x = new BigInteger(0, 23232323, 23232323);
      Console.Out.WriteLine(x.ToString());

      // 207836985031958364
      BigInteger y = new BigInteger(0, 302940, 48390819);
      Console.Out.WriteLine(y.ToString());

      Console.Out.WriteLine("x + y = " + (x + y).ToString());
      Console.Out.WriteLine("x - y = " + (x - y).ToString());
      Console.Out.WriteLine("x * y = " + (x * y).ToString());
      Console.Out.WriteLine("x / y = " + (x / y).ToString());

      Complex64 z = new Complex64(3, 4);
      Console.Out.WriteLine(z.ToString());

      Complex64 q = new Complex64(2, 0);
      Console.Out.WriteLine(q.ToString());

      Console.Out.WriteLine("z + q = " + (z + q).ToString());
      Console.Out.WriteLine("z - q = " + (z - q).ToString());
      Console.Out.WriteLine("z * q = " + (z * q).ToString());
      Console.Out.WriteLine("z / q = " + (z / q).ToString());
    }
  }
}
```

This produces the following output:

```
99782067518340931
207836985031958364
x + y = 307619052550299295
x - y = 0
x * y = 20738404073267282935556751148996884
x / y = 0
(3+4j)
(2+0j)
z + q = (5+4j)
z - q = (1+4j)
z * q = (6+8j)
z / q = (1.5+2j)
```

It's interesting to note that the subtraction yields a "0", and not a negative number. That seems odd to me.

There are other implementation of arbitrary-sized integer arithmetic. I think that a future version of .NET will have support for large numerical calculations and complex arithmetic ... then again, I thought that by 2.0 .NET would have support for time zones too, and that didn't happen (although it seems like headway is being made for time zone support in 3.0). But if you need a big integer now, just use what the IronPython created for all of us out of the box.

> Published: 07.11.2006 12:13:26 PM CST