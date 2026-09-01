---
title: log4net and Multiple AppDomains Within an Application ... I Give Up
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# log4net and Multiple AppDomains Within an Application ... I Give Up

OK, I've tried for the past couple of hours trying to figure out just what magical incantation I need to perform with log4net to get it so another `AppDomain` will log to the configured log file from the EXE. I've uploaded the code that isolates the problem (you can get it here). Basically, I create a new `AppDomain`:

```c#
private void OnLogFromNewDomainButtonClick(object sender, System.EventArgs e)
{
  AppDomainSetup setup = AppDomain.CurrentDomain.SetupInformation;
  Evidence evidence = AppDomain.CurrentDomain.Evidence;

  AppDomain newDomain = AppDomain.CreateDomain("NewDomain", 
    evidence, setup);
    
  Assembly cultureAssembly = newDomain.Load("AppDomainServer");

  TestLogging testLogging = 
    (TestLogging)newDomain.CreateInstanceAndUnwrap(
    "AppDomainServer", "AppDomainServer.TestLogging");
  testLogging.Test("New Domain");
}
```

`TestLogging` does some pretty simple stuff for its logging tests:

```c#
public class TestLogging : MarshalByRefObject
{
  private ILog log = LogManager.GetLogger(typeof(TestLogging));

  public TestLogging() : base() 
  {
    this.log.Debug(string.Format(
      "Logging from AppDomain {0}", AppDomain.CurrentDomain.FriendlyName));
  }

  public void Test(string message)
  {
    this.log.Debug(message);
  }
}
```

I should see stuff in my log file, but nothing shows up when `TestLogging` is in a different `AppDomain`. I added `DOMConfigurator(Watch=true)` to my server assembly and then I noticed that the corresponding respository suddenly became configured but still, nothing in the log file. I noticed that one of the appenders didn't have its `QuietTextWriter` field set so that's why it wasn't logging, but the reason why the field wasn't set in the first place is beyond me (and I finally got tired of debugging log4net source code - it's not bad, it's just hard to follow at times). I'll keep plugging away and I'm sure it's just a simple thing that I need to do, but I can't figure it out. Please post a comment if you know what's going on - TIA.

**Update #1**: I turned on log4net's debugging and I noticed something interesting. When I log from the main AppDomain, I get this:

```
log4net: FileAppender: Opening file for writing [C:\jbock\Personal\.NET Projects\AppDomainsAndLog4Net\AppDomainClient\bin\AppDomainClient.log] append [True] log4net: DOMConfigurator: Logger [root] Level string is [DEBUG].
```

But this fails in the new AppDomain:

```
log4net: FileAppender: Opening file for writing [C:\jbock\Personal\.NET Projects\AppDomainsAndLog4Net\AppDomainClient\bin\AppDomainClient.log] append [True] log4net:ERROR [RollingFileAppender] OpenFile(C:\jbock\Personal\.NET Projects\AppDomainsAndLog4Net\AppDomainClient\bin\AppDomainClient.log,True) call failed. log4net:ERROR [RollingFileAppender] OpenFile(C:\jbock\Personal\.NET Projects\AppDomainsAndLog4Net\AppDomainClient\bin\AppDomainClient.log,True) call failed. System.IO.IOException: The process cannot access the file "C:\jbock\Personal\.NET Projects\AppDomainsAndLog4Net\AppDomainClient\bin\AppDomainClient.log" because it is being used by another process. at System.IO.__Error.WinIOError(Int32 errorCode, String str) at System.IO.FileStream..ctor(String path, FileMode mode, FileAccess access, FileShare share, Int32 bufferSize, Boolean useAsync, String msgPath, Boolean bFromProxy) at System.IO.FileStream..ctor(String path, FileMode mode, FileAccess access, FileShare share, Int32 bufferSize) at System.IO.StreamWriter.CreateFile(String path, Boolean append) at System.IO.StreamWriter..ctor(String path, Boolean append, Encoding encoding, Int32 bufferSize) at System.IO.StreamWriter..ctor(String path, Boolean append, Encoding encoding) at log4net.Appender.FileAppender.OpenFile(String fileName, Boolean append) at log4net.Appender.RollingFileAppender.OpenFile(String fileName, Boolean append) at log4net.Appender.FileAppender.ActivateOptions()
```

I'll keep digging ...

> Published: 02.16.2005 03:50:30 PM CST