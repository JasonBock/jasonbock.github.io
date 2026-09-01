---
title: log4net Configuration Problem
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# log4net Configuration Problem

OK, I might figure this one out as soon as I post it, but I'm having good luck that way, so ...

Here's my log4net configuration:

```xml
<log4net>
  <appender name="EventLogAppender" type="log4net.Appender.EventLogAppender" >
    <applicationName value="AdministrationService" />
    <layout type="log4net.Layout.PatternLayout">
      <conversionPattern value="%d [%t thread] %-5p - %m%n" />
    </layout>
  </appender>
  <appender name="RollingLogFileAppender" 
    type="log4net.Appender.RollingFileAppender">
    <param name="File" value="AdministrationService.log" />
    <param name="AppendToFile" value="true" />
    <param name="MaxSizeRollBackups" value="10" />
    <param name="MaximumFileSize" value="1000000" />
    <param name="RollingStyle" value="Size" />
    <param name="StaticLogFileName" value="true" />
    <layout type="log4net.Layout.PatternLayout">
      <param name="Header" 
        value="[START Welcome to AdministrationService.log]\r\n" />
      <param name="Footer" 
        value="[END AdministrationService.log]\r\n" />
      <param name="ConversionPattern" 
        value="%d [%t thread] %-5p - %C.%M() - %m%n" />
    </layout>
  </appender>
  <root>
    <level value="DEBUG" />
    <appender-ref ref="EventLogAppender" />
    <appender-ref ref="RollingLogFileAppender" />
  </root>
</log4net>
```

Now, everything is working as planned. But what I want to do is only log information to the Event Log that is of `WARN` status or higher, and log everything (i.e. `DEBUG`) to the log file. But it seems like I'm stuck - I can only have one `<level>` element per `<root>` element, and you can only have one `<root>` element in your log4net configuration section. This isn't a big deal, but it would be nice to specify different logging levels for different appenders. If this is possible (or it it's definitely not possible), please let me know - TIA.

> Published: 02.14.2005 01:40:01 PM CST