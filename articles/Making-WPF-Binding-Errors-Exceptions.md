---
title: Making WPF Binding Errors Exceptions
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Making WPF Binding Errors Exceptions

For the last 7 months I've been on a project where we're using WPF for the front-end. WPF has really impressed me as a vehicle to develop Windows applications, but what has really floored me is how much I like binding in WPF. To me, binding has always conjured up images of "data binding" in VB6 (or earlier) where, frankly, it sucked. Binding in XAML is extremely powerful and once I (sort of) understood it, it has made development much easier to keep a better separation between presentation and logic.

That said, all is not roses and banana nut muffins [1] with XAML binding. One thing I found kind of annoying with it is that it doesn't treat binding errors as exceptions. Here's what I mean. Let's say I have the following class:

```c#
using System;

namespace WpfBindingExceptions
{
  public sealed class Customer
  {
    public Customer(Guid id, string name, DateTime birthDate)
      : base()
    {
      this.Id = id;
      this.Name = name;
      this.BirthDate = birthDate;
    }
        
    public DateTime BirthDate { get; private set; }
    public Guid Id { get; private set; }
    public string Name { get; private set; }
  }
}
```

I set it up as the `DataContext`:

```c#
using System;
using System.Diagnostics;
using System.Windows;

namespace WpfBindingExceptions
{
  public partial class CustomerView : Window
  {
    public CustomerView()
    {
      this.DataContext = new Customer(Guid.NewGuid(),
        "Joe Smith", new DateTime(1980, 3, 5));
      this.InitializeComponent();
    }
  }
}
```

I also have the XAML looking like this:

```xml
<Window x:Class="WpfBindingExceptions.CustomerView"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="CustomerView" Height="140" Width="340"
  WindowStartupLocation="CenterScreen">
  <Grid>
    <Grid.ColumnDefinitions>
      <ColumnDefinition Width="Auto" />
      <ColumnDefinition Width="*" />
    </Grid.ColumnDefinitions>
    <Grid.RowDefinitions>
      <RowDefinition Height="Auto" />
      <RowDefinition Height="*" />
      <RowDefinition Height="Auto" />
      <RowDefinition Height="*" />
      <RowDefinition Height="Auto" />
    </Grid.RowDefinitions>
    <TextBlock Grid.Row="0" Grid.Column="0" 
      Text="ID:" HorizontalAlignment="Right" Margin="8" />
    <TextBlock Grid.Row="0" Grid.Column="1" 
      Text="{Binding Path=Id}" Margin="8" />
    <TextBlock Grid.Row="2" Grid.Column="0" 
      Text="Name:" HorizontalAlignment="Right" Margin="8" />
    <TextBlock Grid.Row="2" Grid.Column="1" 
      Text="{Binding Path=Name}" Margin="8" />
    <TextBlock Grid.Row="4" Grid.Column="0" 
      Text="Birth Date:" HorizontalAlignment="Right" Margin="8" />
    <TextBlock Grid.Row="4" Grid.Column="1" 
      Text="{Binding Path=BirthDate}" Margin="8" />
  </Grid>
</Window>
```

With all this in place, the screen comes up as expected:

![Expected View](https://jasonbock.net/images/WPF-Binding-Expected.png "Expected View")

However, let's screw up our binding on purpose:

```xml
    <TextBlock Grid.Row="2" Grid.Column="0" 
      Text="Name:" HorizontalAlignment="Right" Margin="8" />
    <TextBlock Grid.Row="2" Grid.Column="1" 
      Text="{Binding Path=Bame}" Margin="8" />
```

Note that the change is that I'm not binding to the `Name` property; I'm binding to `Bame`, which doesn't exist.

Now, if I run the app, here's what I see:

![Silent Binding Fail](https://jasonbock.net/images/WPF-Silent-Fail-Binding.png "Silent Binding Fail")

Huh? The binding just silently fails. Granted, in this case I'd see the error pretty quickly, but in a complex application it can take some time to figure out what the problem is. I like failing fast, which means I'd rather have the application die a horrible and painful death right away on my laptop and make it brutually obvious as to what's wrong. As I found out, there's a way to make this happen.

If I run the app under the debugger and watch the *Output* window, I see this:

![Debugger Output](https://jasonbock.net/images/WPF-Debugger-Output.png "Debugger Output")

As I found out, WPF's binder will trace all binding errors. So, using what I found here, I created a listener class to watch for WPF binding error and package up all relevant data into a `WpfBindingException` object:

```c#
using System;
using System.Diagnostics;
using System.Text;
using System.Windows;
using System.Reflection;

namespace WpfBindingExceptions
{
  public sealed class BindingListener : DefaultTraceListener
  {
    private string Callstack { get; set; }
    private string DateTime { get; set; }
    private int InformationPropertyCount { get; set; }
    private bool IsFirstWrite { get; set; }
    private string LogicalOperationStack { get; set; }
    private string Message { get; set; }
    private string ProcessId { get; set; }
    private string ThreadId { get; set; }
    private string Timestamp { get; set; }

    public BindingListener(TraceOptions options)
      : base()
    {
      this.IsFirstWrite = true;
      PresentationTraceSources.Refresh();
      PresentationTraceSources.DataBindingSource.Listeners.Add(this);
      PresentationTraceSources.DataBindingSource.Switch.Level = SourceLevels.Error;
      this.TraceOutputOptions = options;
      this.DetermineInformationPropertyCount();
    }

    private void DetermineInformationPropertyCount()
    {
      foreach(TraceOptions traceOptionValue in Enum.GetValues(typeof(TraceOptions)))
      {
        if(traceOptionValue != TraceOptions.None)
        {
          this.InformationPropertyCount += this.GetTraceOptionEnabled(traceOptionValue);
        }
      }
    }

    private int GetTraceOptionEnabled(TraceOptions option)
    {
      return (this.TraceOutputOptions & option) == option ? 1 : 0;
    }

    public override void WriteLine(string message)
    {
      if(this.IsFirstWrite)
      {
        this.Message = message;
        this.IsFirstWrite = false;
      }
      else
      {
        var propertyInformation = message.Split(new string[] { "=" }, StringSplitOptions.None);
                
        if(propertyInformation.Length == 1)
        {
          this.LogicalOperationStack = propertyInformation[0];
        }
        else
        {
          this.GetType().GetProperty(propertyInformation[0], 
            BindingFlags.IgnoreCase | BindingFlags.NonPublic | BindingFlags.Instance)
            .SetValue(this, propertyInformation[1], null);
        }
                
        this.InformationPropertyCount--;
      }

      this.Flush();

      if(this.InformationPropertyCount == 0)
      {
        PresentationTraceSources.DataBindingSource.Listeners.Remove(this);
        throw new BindingException(this.Message,
          new BindingExceptionInformation(this.Callstack, 
            System.DateTime.Parse(this.DateTime),
            this.LogicalOperationStack, int.Parse(this.ProcessId), 
            int.Parse(this.ThreadId), long.Parse(this.Timestamp)));
      }
    }
  }
}
```

Now let's assume my application hooks the unhandled exception handler like so:

```c#
using System;
using System.Diagnostics;
using System.IO;
using System.Windows;
using System.Windows.Threading;

namespace WpfBindingExceptions
{
  public partial class App : Application
  {
    protected override void OnStartup(StartupEventArgs e)
    {
      base.OnStartup(e);

      this.DispatcherUnhandledException +=
        new DispatcherUnhandledExceptionEventHandler(
          this.OnAppDispatcherUnhandledException);
    }

    private void OnAppDispatcherUnhandledException(
      object sender, DispatcherUnhandledExceptionEventArgs e)
    {
      MessageBox.Show("UNHANDLED EXCEPTION: " + 
        Environment.NewLine + Environment.NewLine +
        e.Exception.GetType().Name +
        Environment.NewLine + Environment.NewLine +
        e.Exception.Message);
      e.Handled = true;
      this.Shutdown();
    }
  }
}
```

When I run the application with binding errors on:

```c#
using System;
using System.Diagnostics;
using System.Windows;

namespace WpfBindingExceptions
{
  public partial class CustomerView : Window
  {
    private BindingListener listener;
        
    public CustomerView()
    {
      this.listener = new BindingListener(TraceOptions.Callstack | TraceOptions.DateTime |
        TraceOptions.LogicalOperationStack | TraceOptions.ProcessId | TraceOptions.ThreadId |
        TraceOptions.Timestamp);
      this.DataContext = new Customer(Guid.NewGuid(),
        "Joe Smith", new DateTime(1980, 3, 5));
      this.InitializeComponent();
    }
  }
}
```

This is what happens:

![Detailed Binding Error](https://jasonbock.net/images/WPF-Detailed-Binding-Error.png "Detailed Binding Error")

Sweet!

Please let me know if you have any questions/problems/concerns - thanks!

[1] Sorry, I was eating a banana nut muffin as I was writing this blog post.

> Published: 03.13.2010 06:03:55 PM CST