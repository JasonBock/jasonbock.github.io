---
title: Performance Tip: Dividing by the Reciprocal
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Performance Tip: Dividing by the Reciprocal

If you do a lot of floating-point division operations in VB, you may want to optimize these operations by multiplying by the reciprocal value. For example, instead of performing this calculation:

```
X/Y
```

do this:

```
X * (1/Y)
```

You can see how this works in VB by adding the following code to a form in a new project:

```vb
Private Declare Function GetTickCount Lib "kernel32" () As Long
Const NORMAL As Double = 1453
Const RECIPROCAL As Double = 1 / NORMAL
Const TOTAL_COUNT As Long = 10000000

Private Sub NormalDivide()

On Error GoTo Error_Normal

Dim dblRes As Double
Dim lngC As Long
Dim lngStart As Long

lngStart = GetTickCount

For lngC = 1 To TOTAL_COUNT
  dblRes = Rnd / NORMAL
Next lngC

MsgBox "Total Time:  " & GetTickCount - lngStart

Exit Sub

Error_Normal:

MsgBox Err.Number & " - " & Err.Description

End Sub
```

You'll also need a `DBR` function that is identical to `NormalDivide`, except that the one line of code in the `For ... Next` loop is:

```vb
dblRes = Rnd * RECIPROCAL
```

I have seen consistent performance gains of 15% with the `DBR` function. However, before you start changing all the code where you do division, there's a bit of a catch. There are some rounding issues to be concerned about (for example, 3/3 = 1, but 3 * (0.333333) = 0.999999). See http://www.cs.wisc.edu/~cs354-2/lec.notes/arith.flpt.html and http://science.nas.nasa.gov/Pubs/TechReports/RNRreports/dbailey/pi/pi.html for more detailed information.

> Published: 04.19.1999 12:00:00 AM CST