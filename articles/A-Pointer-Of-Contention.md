---
title: A Pointer Of Contention
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# A Pointer Of Contention

## What is Wrong With This Code?

Language/Tool: Visual C++ 

```c++
CeGetUserNotification((HANDLE)NotificationHandle, 
  dwSize,	&dwNeeded, (LPBYTE)pHeader);

if (CNS_SIGNALLED == pHeader->dwStatus)
{
  m_Signaled = true;
}
else
{
  m_Signaled = false;
}

m_UserNotify.ActionFlags = pHeader->pceun->ActionFlags;

if(NULL != pHeader->pceun->pwszDialogTitle)
{
  m_UserNotify.pwszDialogTitle = 
    SysAllocString(pHeader->pceun->pwszDialogTitle);
}

if(NULL != pHeader->pceun->pwszDialogText)
{
  m_UserNotify.pwszDialogText = 
    SysAllocString(pHeader->pceun->pwszDialogText);
}

if(NULL != pHeader->pceun->pwszSound)
{
  m_UserNotify.pwszSound = 
    SysAllocString(pHeader->pceun->pwszSound);
}

m_UserNotify.nMaxSound = pHeader->pceun->nMaxSound;
m_UserNotify.dwReserved = pHeader->pceun->dwReserved;
	
m_UserTrigger.dwSize = pHeader->pcent->dwSize;
m_UserTrigger.dwType = pHeader->pcent->dwType;
m_UserTrigger.dwEvent = pHeader->pcent->dwEvent;

if(NULL != pHeader->pcent->lpszApplication)
{
  m_UserTrigger.lpszApplication = 
    SysAllocString(pHeader->pcent->lpszApplication);
}

if(NULL != pHeader->pcent->lpszArguments)
{
  m_UserTrigger.lpszArguments = 
    SysAllocString(pHeader->pcent->lpszArguments);
}

m_UserTrigger.stStartTime = pHeader->pcent->stStartTime;
m_UserTrigger.stEndTime = pHeader->pcent->stEndTime;
	
delete[] (BYTE*)pHeader;
```

This code snippet is from a COM server that I wrote to wrap the Notification API for the Windows eMbedded OS (also known as Win CE and Windows Powered - it's the OS that runs your PocketPC devices). It's part of a class called `CNotification` - specifically, it's found in the `Load()` method. I wrote this class because the APIs cannot be easily used by eMbedded Visual Basic, so I used the ATL wizard to set up the COM server framework code for me, and then I simply implemented the methods. 

At least, that was the plan. After I compiled my server, I wrote an eVB client to test it out. Unfortunately, nothing worked. My server had a class called `CNotifications`, which had one method called `GetNotification()`. It was supposed to create a bunch of `CNotification`-based objects all loaded up and ready to go, and return them in a `SAFEARRAY` that was stuffed into a `VARIANT`. Problem was, `GetNotifications()` always died. 

To debug this problem, I took a look at my implementation of `GetNotifications()`. I didn't see anything wrong with it right away, but my skills are not that strong in C++, so I posted the problem to my company's internal mailing list to see if any experts would find something. Sure enough, I had a couple of problems with my usage of a `SAFEARRAY`. Confident that my changes would fix everything, I re-ran my eVB application. 

Nothing. 

OK, now I was getting mad. Granted, I fixed a couple of problems that needed to be fixed, but that wasn't the source of the error. So, I had to go one level deeper. I tried calling `Load()`, and I noticed that I got the same behaviour. That's when I started taking a long and hard look at my code, and I eventually figured it out (with the help of my fellow coworkers, of course!). 

Now, if you've done a lot of C++ code, you've probably already spotted the error. But, if you're like me - that is, someone who used VB for most of their career - it may take a bit of time to determine what's going on. pHeader is defined to be of a `UserNotificationInfoHeader` structure, and it looks like this: 

```
typedef struct UserNotificationInfoHeader {
    HANDLE 				hNotification;
    DWORD 				dwStatus;
    CE_NOTIFICATION_TRIGGER		*pcent;
    CE_USER_NOTIFICATION		*pceun;
} CE_NOTIFICATION_INFO_HEADER, *PCE_NOTIFICATION_INFO_HEADER;
```

The last two members are defined as follows: 

```
typedef struct UserNotificationTrigger {
	DWORD		dwSize;
	DWORD		dwType;
	DWORD		dwEvent;
	TCHAR		*lpszApplication;
	TCHAR		*lpszArguments;
	SYSTEMTIME	stStartTime;
	SYSTEMTIME	stEndTime;
} CE_NOTIFICATION_TRIGGER, *PCE_NOTIFICATION_TRIGGER;

typedef struct UserNotificationType {
    DWORD ActionFlags;
    TCHAR *pwszDialogTitle;
    TCHAR *pwszDialogText;
    TCHAR *pwszSound;
    DWORD nMaxSound;
    DWORD dwReserved;
} CE_USER_NOTIFICATION, *PCE_USER_NOTIFICATION;
```

`CeGetUserNotification()` will return information about a specific notification (or a new notification if you set the first argument to `NULL`), which I then use to populate private member-level variables that can be read via COM properties. Now, you'll notice that I'm being a good C++ citizen in that I'm checking the pointers found in the two structures to make sure that they're not `NULL`. But how good of a citizen am I being? As it turns out, I'm committing two crimes. While I'm checking the pointers in the two structures, I'm assuming that `CeGetUserNotification()` contains two valid pointers in `pcent` and `pceun`. 

It's bad when you assume. Very bad. 

It's quite possible that one or both of the two pointer values could be `NULL`. All kinds of things can go wrong if this happens. As it turns out, `pceun` was `NULL` for one of the notifications. Yikes! 

## Possible Solution

Fortunately, the solution was easy: Make sure the pointer values are checked: 

```c++
CeGetUserNotification((HANDLE)NotificationHandle, 
  dwSize, &dwNeeded, (LPBYTE)pHeader);

if (CNS_SIGNALLED == pHeader->dwStatus)
{
  m_Signaled = true;
}
else
{
  m_Signaled = false;
}

if (NULL != pHeader->pceun)
{
  m_UserNotify.ActionFlags = pHeader->pceun->ActionFlags;
    
  if(NULL != pHeader->pceun->pwszDialogTitle)
  {
    m_UserNotify.pwszDialogTitle = 
      SysAllocString(pHeader->pceun->pwszDialogTitle);
  }
    
  if(NULL != pHeader->pceun->pwszDialogText)
  {
    m_UserNotify.pwszDialogText = 
      SysAllocString(pHeader->pceun->pwszDialogText);
  }
  
  if(NULL != pHeader->pceun->pwszSound)
  {
    m_UserNotify.pwszSound = 
	  SysAllocString(pHeader->pceun->pwszSound);
  }
    
  m_UserNotify.nMaxSound = pHeader->pceun->nMaxSound;
  m_UserNotify.dwReserved = pHeader->pceun->dwReserved;
}

if (NULL != pHeader->pcent)
{	
  m_UserTrigger.dwSize = pHeader->pcent->dwSize;
  m_UserTrigger.dwType = pHeader->pcent->dwType;
  m_UserTrigger.dwEvent = pHeader->pcent->dwEvent;
  if(NULL != pHeader->pcent->lpszApplication)
  {
    m_UserTrigger.lpszApplication = 
	  SysAllocString(pHeader->pcent->lpszApplication);
  }
  if(NULL != pHeader->pcent->lpszArguments)
  {
    m_UserTrigger.lpszArguments = 
	SysAllocString(pHeader->pcent->lpszArguments);
  }
  m_UserTrigger.stStartTime = pHeader->pcent->stStartTime;
  m_UserTrigger.stEndTime = pHeader->pcent->stEndTime;
}
	
delete[] (BYTE*)pHeader;
```

> Note: Of course, if I would have used the debugger, I probably would have caught the null pointer right away. If you've never debugged a live PocketPC application, though, you don't know what you're missing. If your device is connected via a USB or COM port, your debugging session will be very slow, and I was too lazy to deal with the delays. In retrospect, though, I should have created a debug version of my COM server.  

This was an easy problem to fix. But in my book, it's an interesting code blooper, because it demonstrates how accustomed you can get with one language. Most of my work has been in VB with some Java. Only in the last six months, have I needed to seriously use C++. Basically, I wasn't used to working with pointers. And, although I knew enough about pointers to check that they were valid, I obviously wasn't diligent enough to catch them all. 

## Summary

1. Even when you've found a bug, don't assume that you've fixed it. Re-run your tests to ensure correctness.
2. Make sure your variables are always initialized properly. Defensive coding will result in robust systems.

> Published: 06.27.2001