---
title: Running Build Events for Testing Cleanup
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Running Build Events for Testing Cleanup

I was working on a utility application to synch up our resource files with culture-specific resource files. An issue came up when I was writing my tests as I wanted to use a bunch of .resx files that had to be in the same starting state for each test. I originally wrote a post-build event to move the test resource files into the correct target directory - it looked like this:

```
xcopy /y "$(ProjectDir)*.resx" "$(TargetDir)*.resx"
```

However, after writing more than one test I quickly realized that my "clean" resource files would be altered after the first test was done and therefore would be useless from that point on. Fortunately, I realized there was a way to work around this issue:

```c#
private DirectoryInfo executingDirectory = new DirectoryInfo(
  Path.GetDirectoryName(
  Assembly.GetExecutingAssembly().Location));

[SetUp()]
public void Setup()
{
  ProcessStartInfo postBuildEventProcessInfo = 
    new ProcessStartInfo(
    Path.Combine(executingDirectory.FullName, "PostBuildEvent.bat"));
  postBuildEventProcessInfo.UseShellExecute = false;
  postBuildEventProcessInfo.RedirectStandardInput = true;
  postBuildEventProcessInfo.RedirectStandardOutput = true;
  postBuildEventProcessInfo.RedirectStandardError = true;
  postBuildEventProcessInfo.CreateNoWindow = true;

  Process postBuildEventProcess = new Process();
  postBuildEventProcess.StartInfo = postBuildEventProcessInfo;
  postBuildEventProcess.Start();
  postBuildEventProcess.WaitForExit();
}
```

The point is that my test files will always be copied over to the target directory before a test is run, which is exactly what I wanted. Just thought I'd share this in case you ever want to duplicate a build event's behavior in your unit tests.

> Published: 12.15.2004 02:01:01 PM CST