---
title: Unit Testing Custom MSBuild Tasks
layout: default
---
| [Home](https://jasonbock.net/index.html) | [Biography](https://jasonbock.net/Biography.html) | [Speaking](https://jasonbock.net/Speaking.html) | [Articles](https://jasonbock.net/Articles.html) | [Books](https://jasonbock.net/Books.html) | [Music](https://jasonbock.net/Music.html) |

# Unit Testing Custom MSBuild Tasks

I'm starting to really groove on the Team Edition of VS. I can now get rid of updating NAnt, NUnit, NCover, NProf, and FxCop (well, FxCop comes with Team Edition, and one can argue that MSTest is just NUnit, etc., etc.). I've got nothing against these tools - they're awesome open-source tools. But, if you can use Team Edition, you might as well use you get out of the box, right?

Anyway, one of my recent activities was writing a custom MSBuild task. This morning I thought of what it would take to unit test a custom task. I was worried because as soon as a class need some kind of contextural harness (like ASP.NET), things can get hairy. So I started by doing a Google search, and I found this. I understand his reasoning, but what I didn't like about this approach is that you either had to check to see if the build engine was set (which it really should be given that this is a MSBuild task) or you have to fire up msbuild.exe to ensure the correct context exists. I commented that it might be possible to mock a `IBuildEngine` class just so you could test your task, and sure enough, you can. First, here's an example of a simple logging task:

```vb
Imports Microsoft.Build.Framework
Imports Microsoft.Build.Utilities

Public Class LoggingTask
  Inherits Task

  Private _message As String

  <Required()> _
  Public Property Message() As String
    Get
      Return _message
    End Get
    Set(ByVal value As String)
      _message = value
    End Set
  End Property

  Public Overrides Function Execute() As Boolean
    Log.LogMessage(_message)
    Return True
  End Function
End Class
```

Now here's my test class to exercise that task:

```vb
Imports LoggingTask
Imports System
Imports System.Text
Imports System.Collections.Generic
Imports Microsoft.VisualStudio.TestTools.UnitTesting

<TestClass()> _
Public Class LoggingTaskTests
  Dim _task As LoggingTask

  <TestInitialize()> _
  Public Sub Initialize()
    _task = New LoggingTask()
    _task.BuildEngine = New MockBuildEngine()
  End Sub

  <TestMethod()> _
  Public Sub TestLogTask()
    _task.Message = "Log this!"
    Assert.IsTrue(_task.Execute())
  End Sub

  <TestMethod(), ExpectedException(GetType(ArgumentNullException))> _
  Public Sub TestLogTaskWithNoMessage()
    _task.Execute()
  End Sub
End Class
```

Note that I'm setting the `BuildEngine` to an instance of `MockBuildEngine` - here's what that class looks like:

```vb
Imports Microsoft.Build.Framework
Imports System.Collections

Public Class MockBuildEngine
  Implements IBuildEngine

  Public Function BuildProjectFile(ByVal projectFileName As String, _
    ByVal targetNames() As String, ByVal globalProperties As IDictionary, _
    ByVal targetOutputs As IDictionary) As Boolean _
    Implements IBuildEngine.BuildProjectFile
    Throw New NotImplementedException()
  End Function

  Public ReadOnly Property ColumnNumberOfTaskNode() As Integer _
    Implements IBuildEngine.ColumnNumberOfTaskNode
    Get
      Throw New NotImplementedException()
    End Get
  End Property

  Public ReadOnly Property ContinueOnError() As Boolean _
    Implements IBuildEngine.ContinueOnError
    Get
      Throw New NotImplementedException()
    End Get
  End Property

  Public ReadOnly Property LineNumberOfTaskNode() As Integer _
    Implements IBuildEngine.LineNumberOfTaskNode
    Get
      Throw New NotImplementedException()
    End Get
  End Property

  Public Sub LogCustomEvent(ByVal e As CustomBuildEventArgs) _
    Implements IBuildEngine.LogCustomEvent
  End Sub

  Public Sub LogErrorEvent(ByVal e As BuildErrorEventArgs) _
    Implements IBuildEngine.LogErrorEvent
  End Sub

  Public Sub LogMessageEvent(ByVal e As BuildMessageEventArgs) _
    Implements IBuildEngine.LogMessageEvent
  End Sub

  Public Sub LogWarningEvent(ByVal e As BuildWarningEventArgs) _
    Implements IBuildEngine.LogWarningEvent
  End Sub

  Public ReadOnly Property ProjectFileOfTaskNode() As String _
    Implements IBuildEngine.ProjectFileOfTaskNode
    Get
      Throw New NotImplementedException()
    End Get
  End Property
End Class
```

It's pretty bare-bones, but at least my tests will pass with this approach. A more "robust" mock of `IBuildEngine` may be needed for other tasks, but this is far easier than what I had to deal with to mock an `HttpContext` class!

> Published: 07.19.2006 01:34:44 PM CST