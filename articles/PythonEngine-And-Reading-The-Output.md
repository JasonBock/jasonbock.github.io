---
title: PythonEngine and Reading the Output
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# PythonEngine and Reading the Output

Here's my scenario. I have a script.py file that opens up a C# window I created as a dialog window (`ShowDialog()`). In that window, I'm trying to host the `PythonEngine` to run some script that's entered into a text box:

```c#
PythonEngine engine = new PythonEngine();

using (MemoryStream stream = new MemoryStream())
{
  engine.Sys.stdout = stream;
  engine.Sys.stderr = stream;

  //engine.SetStandardOutput(stream);
  //engine.SetStandardError(stream);

  engine.Execute(this.scriptValue.Text);

  stream.Position = 0;

  using (TextReader reader = new StreamReader(stream))
  {
    this.scriptResultsValue.Text = reader.ReadToEnd();
  }
}
```

As you can tell in the code, I'm trying to redirect the output and error streams from the engine. Using `SetStandardOutput()` and `SetStandardError()` didn't work, and setting the objects on the `Sys` property didn't do it either. The output ends up going to the console window that started the original Python script. (Yes, I know this is a weird setup - IronPython launches a C# dialog window that hosts IronPython - it is what it is :) ).

What am I missing? What do I need to do to get the output of the engine execution?

> Published: 10.06.2006 11:27:21 AM CST