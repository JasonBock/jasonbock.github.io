---
title: Small Complaint With Eiffel in .NET
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Small Complaint With Eiffel in .NET

This is a small nit with Eiffel - at least it doesn't make a lot of sense to me. Take a look at the following method:

```
.method public hidebysig newslot virtual
  instance void
  SetBinary([in] class EiffelSolarSystem.RootCluster.Star binary)
  cil managed
{
  .maxstack  8
  IL_0000:  ldarg.0
  IL_0001:  ldarg.1
  IL_0002:  castclass EiffelSolarSystem.RootCluster.Star
  IL_0007:  callvirt instance void
    EiffelSolarSystem.RootCluster.Star::_set_BinaryStar(
    class EiffelSolarSystem.RootCluster.Star)
  IL_000c:  ret
}
```

Now, the type of the argument is `EiffelSolarSystem.RootCluster.Star`. So **why** is a `castclass` showing up at `IL_0002`? Unless I'm missing some aspect about this opcode, all it's going to do is cast the given object to the desired type. But in this case, you're guaranteed to have the object as `EiffelSolarSystem.RootCluster.Star`! If you try to pass in a `System.Object` object, that won't even compile.

This happens when you declare the argument as `like Current`:

```
feature {STAR}
    set_binary(binary: like Current) is
        do
            binary_star := binary
        end
```

It seems like you could cut out the 2nd and 3rd opcodes altogether. Anybody see a reason why this may be beneficial?

Oh, if you want to have some fun, create a generic class in Eiffel and try to use it in C#. Something like this:

```
class
    GENERIC_HOLDER [T]

--  Eiffel code goes here...

end
```

Using it in Eiffel is painless. Using it in C# takes some exploring :).

> Published: 11.05.2002 11:46:00 PM CST