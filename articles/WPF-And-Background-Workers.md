---
title: WPF and Background Workers
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# WPF and Background Workers

For the Iowa Code Camp, I did my exceptions talk. I wanted to show that if an unhandled exception occurs in a thread launced within WPF, your application will crash. So I whipped this up (you can get all of the code here):

```c#
private void OnCreateViaBackgroundWorkerClick(object sender, RoutedEventArgs e)
{
  this.SetButtonEnabled(false);

  var worker = new BackgroundWorker();

  worker.DoWork += (doSender, doEventArgs) =>
  {
    Thread.Sleep(PersonWindow.WaitTime);
    doEventArgs.Result = this.FormatPerson(doEventArgs.Argument as UIInformation);
  };

  worker.RunWorkerCompleted += (runSender, runEventArgs) =>
  {
    this.SetButtonEnabled(true);
    this.nameResults.Content = runEventArgs.Result as string;
  };

  worker.RunWorkerAsync(new UIInformation
  {
    Age = this.ageValue.Text,
    FirstName = this.firstNameValue.Text,
    LastName = this.lastNameValue.Text
  });
}
```

Now, if `FormatPerson()` throws an exception, the application should crash ... at least, that's what I was hoping for. The funny thing is, it doesn't! Well, it doesn't crash if you override `OnStartup()` from your `Application`-based class:

```c#
protected override void OnStartup(StartupEventArgs e)
{
  base.OnStartup(e);

  this.DispatcherUnhandledException += 
    new DispatcherUnhandledExceptionEventHandler(
      this.OnAppDispatcherUnhandledException);
}
```

`BackgroundWorker` will catch an unhandled exception in the thread, and throw it in the main UI thread. So in this case To actually get an unhandled exception from another thread in WPF, do this:

```c#
private void OnCreateViaThreadClick(object sender, RoutedEventArgs e)
{
  this.SetButtonEnabled(false);
    
  ThreadPool.QueueUserWorkItem((state) =>
  {
    Thread.Sleep(PersonWindow.WaitTime);
    var information = state as UIInformation;
    var result = this.FormatPerson(information);
    Dispatcher.Invoke(new Action(() =>
    {
      this.nameResults.Content = result;
      this.SetButtonEnabled(true);
    }));
  },
  new UIInformation
  {
    Age = this.ageValue.Text,
    FirstName = this.firstNameValue.Text,
    LastName = this.lastNameValue.Text
  });
}
```

That'll bring your application to its knees if `FormatPerson()` fails.

I was actually happy to see `BackgroundWorker` work this way. I've always liked the asynchronous event programming model, especially since you can change the `AsyncOperationManager` in your host. In this case, the unhandled exception is automatically brought back into the main thread for me and I don't have to worry about that. Awesome.

> Published: 11.08.2008 03:51:43 PM CST