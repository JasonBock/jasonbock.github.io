---
title: If At First You Can't Create It - Don't Try Again
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# If At First You Can't Create It - Don't Try Again

## What is Wrong With This Code?

Language/Tool: Visual C++ 

```c++
bool bEvent = false;
CComPtr pLogger;
HRESULT hr = CoCreateInstance(CLSID_Logger, 
  NULL, CLSCTX_ALL, IID_ILogger, (void**)&pLogger);

if (FAILED(hr))
{
  LogEvent(_T("CoCreateInstance of 
    ILogger failed: HR=0x%x"), hr);
}

//  Other code goes here.

if (pLogger.p)
{
  hr = pLogger->WriteLog(bstrMsg);
  if (FAILED(hr) && !bEvent)
  {
    bEvent = true;
    LogEvent(_T("WriteLog failed: hr=0X%X"), hr);
  }
}
else
{
  hr = CoCreateInstance(CLSID_Logger, 
    NULL, CLSCTX_ALL, IID_ILogger, (void**)&pLogger);
  if (SUCCEEDED(hr))
  {
    hr = pLogger->WriteLog(bstrMsg);
    if (FAILED(hr) && !bEvent)
    {
      bEvent = true;
      LogEvent(_T("WriteLog failed: hr=0X%X"), hr);
    }
  }
  else if (!bEvent)
  {
    LogEvent(_T("Failed to create ILogger in multiple
      attempts. HR=0x%x"), hr);
    bEvent = true;
  }
}
```

This piece of code existed in an NT service that would write log messages to a database via `WriteLog()` on `ILogger`. If something goes wrong during the execution of this code, information is logged to the NT event log via `LogEvent()`, Unfortunately, a rather trivial operation has been made very difficult and confusing. That operation is object creation. 

The biggest problem I have with this code is that if you couldn't create the logger object in the first place, why would you continue? If you noticed, there's a comment in the code where I took out a bunch of code that was performing other tasks before `pLogger` is checked. In fact, there's no reason to check if the interface pointer is `null`, as `CoCreateInstance()` would fail if the object creation was unsuccessful. Either way, though, a check is made to see if the creation worked, and it is critical to the service that it does; there's no point in continuing if the service can't log messages. However, the implementor decides to go on anyway. This code should be refactored. That is, stop executing any code if you can't create `ILogger`. 

Take a look at how many times `CoCreateInstance()` is called. Why would you try to create `ILogger` a second time if you couldn't create it the first time? I highly doubt that someone will suddenly register the COM server after the first attempt but before the second attempt. I really can see no logical reason why you'd want to try and create it a second time. This is a waste of time and effort. 

> Note: One other small problem: What's the point of `bEvent`? The only purpose I can see it having is to control when you call `LogEvent()`, but why would you not want to log the fact that a component critical to the service is not working correctly? This code should be removed.  

## Possible Solution

Given what we just went over, here's one possible way to improve the code: 

```c++
CComPtr pLogger;
HRESULT hr = CoCreateInstance(CLSID_Logger, 
  NULL, CLSCTX_ALL, IID_ILogger, (void**)&pLogger);

if (SUCCEEDED(hr))
{
  //  Other code goes here.
  hr = pLogger->WriteLog(bstrMsg);
  if(FAILED(hr))
  {
    LogEvent(_T("WriteLog failed: hr=0X%X"), hr);
  }
}
else
{
  LogEvent(_T("CoCreateInstance of 
    ILogger failed: hr=0X%X \n"), hr);
}
```

We try to create the object once. If it works, we continue on. Otherwise, we log the error to the event log. After we invoke `LogEvent()`, we check to see if it worked. If it didn't, we log that to the event log. This is much easier to maintain in the future as well as diagnose problems that may occur if COM servers aren't registered. The original implementation was trying to do too much when there wasn't a need for it. If the object isn't there the first time, it's highly unlikely that it'll be there 100 milliseconds in the future. 

## Summary

1. If a critical component isn't working right, stop the code flow immediately. There's no reason to continue.
2. Keep the code simple. If you try to be "clever," logically work through your "cleverness" to ensure that it's truly "clever."

> Published: 06.19.2001