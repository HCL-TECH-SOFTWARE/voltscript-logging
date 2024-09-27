# Create a Stream LogWriter to Write Logs to a Text File

## Prerequisites
It's beneficial if you're familiar with how VoltScript Logging works and understand the basics of the **BasicLogWriter** class. Please review the following How-Tos for more information:

- [Integrate logging in code](../howto/logging.md)
- [Use LogWriters](../howto/uselogwriter.md)
- [Extend BaseLogWriter](../howto/logwriter.md)

## Introduction
This tutorial will help you understand the basics of extending the **BaseLogWriter** class by creating a LogWriter that uses `StreamVSE` extension to write your logs to a text file. It also takes advantage of the `OSUtilsVSE` extension to ensure the `logs` directory is created.

Before writing your code, you'll need to add these VSEs to your code by using dependency management - see [Use dependency management](../howto/archipelago.md) for more information.

## Create the StreamLogWriter Class
We'll begin by extending the **BaseLogWriter** class to add in support for writing our log entries to a file.

Now there are a few methods within the **BaseLogWriter** class that we will need to overwrite:

- **Sub New** - this is where we'll create our initial log directory path
- **Sub InitializeLog** - this is where we'll instantiate our **Stream** object with the appropriate path and file name
- **Sub outputLogEntryMessage** - this is where we'll add each log entry to our file stream each time one is added to the log
- **Sub terminateLog** - this is where we close the file stream, which commits all writes and closes the file on the disk

Let's start by setting up the basics of our `StreamLogWriter` class.

### StreamLogWriter - The Basics
Let's create the outline of our `StreamLogWriter` class and add in some necessary class properties that we'll need.

```vbscript
Class LogWriterStream as BaseLogWriter
    Private logDir_ as String
    Private logFilePath_ as String
    Private logStream_ as Stream

End Class
```

- `logDir_` - the full directory path to our `logs` directory
- `logFilePath_` - the full file path to the current log file being written
- `logStream` - the `Stream` object used to write our log file

Next we'll add in the various subroutines mentioned earlier. Note that the signatures for each method are the same as the corresponding subroutines found in the **BaseLogWriter** class.

### Sub New()
The `Sub New()` method is where we create the directory path for our `logs` directory using the `CurDir()` function that's a part of VoltScript.

```vbscript
Sub New(label as String, minLevel as Integer, maxLevel as Integer, formatter as String)
    ' this will write the log entries to a subdirectory called logs in the current directory
    logDir_ = CurDir() & "/logs"
End Sub
```

### Sub InitializeLog()
The `Sub InitializeLog()` method is where we create the full path for our log file and use it to instantiate our `Stream` object.

```vbscript
Sub initializeLog()
    Dim rbool as Boolean
    Dim myOS as New OSUtils

    ' make sure the logStream object is loaded, 
    ' and the file path is set appropriately
    If logStream_ is nothing Then
        Set logStream_ = New Stream

        rbool = myOS.makeDirectories(logDir_) ' ensures the log directory exists 
        If not rbool Then Error 1001, "Error creating log directory (" &_ 
        logDir_ & ")"

        logFilePath_ = logDir_ & "/" &_ 
        "log_" & Format(Now, "YYYYMMDD-hhmmss") &_ 
        "_" & createUUID() & ".log"

        Call logStream_.open(logFilePath_, "UTF-8")
    End If
End Sub
```

### Sub outputLogEntryMessage()
The `Sub outputLogEntryMessage()` is used to add each log entry to the log stream.

```vbscript
Public Sub outputLogEntryMessage(message as String)
    ' this writes the log entry to the stream 
    ' every time a log entry is added to the log
    Call logStream_.writeText(message, EOL_LF)
End Sub
```

### Sub terminateLog()
The `Sub terminateLog()` method is used to close the log file stream and ensure its properly written to disk.

```vbscript
Public Sub terminateLog()
    ' this closes the stream and writes the logs to the file on disk
    Call logStream_.close()
End Sub
```

When it's all put together, it looks like this...

```vbscript
Class LogWriterStream as BaseLogWriter
    Private logDir_ as String
    Private logFilePath_ as String
    Private logStream_ as Stream

    Sub New(label as String, minLevel as Integer, maxLevel as Integer, formatter as String)
    ' this will write the log entries to a subdirectory called logs in the current directory
        logDir_ = CurDir() & "/logs"
    End Sub
    
    Sub initializeLog()
        Dim rbool as Boolean
        Dim myOS as New OSUtils

        ' make sure the logStream object is loaded, and the file path is set appropriately
        If logStream_ is nothing Then
            Set logStream_ = New Stream
            rbool = myOS.makeDirectories(logDir_) ' ensures the log directory exists 
            If not rbool Then Error 1001, "Error creating log directory (" & logDir_ & ")"
            logFilePath_ = logDir_ & "/" & "log_" & Format(Now, "YYYYMMDD-hhmmss") & "_" & createUUID() & ".log"
            Call logStream_.open(logFilePath_, "UTF-8")
        End If
    End Sub

	Public Sub outputLogEntryMessage(message as String)
        ' this writes the log entry to the stream every time a log entry is added to the log
        Call logStream_.writeText(message, EOL_LF)
    End Sub

    Public Sub terminateLog()
        ' this closes the stream and writes the logs to the file on disk
        Call logStream_.close()
    End Sub

End Class
```

Now that we have our **StreamLogWriter** class, let's write a small script to test and make sure it works.

## Testing the StreamLogWriter Class
We can test our **StreamLogWriter** class by adding a small `Sub Initialize` routine to our script library. In this subroutine we'll:

- Instantiate our `StreamLogWriter` object by calling the `New` method
- Add our new `streamWriter` object to the `globalLogSession`
- Add a few log entries to the `globalLogSession` object

```vbscript
Sub Initialize
    Dim streamWriter as New LogWriterStream("StreamLogWriter", LOG_DEBUG, LOG_ERROR, "{{LEVELNAME}}: {{MESSAGE}}")
  
    Call globalLogSession.addLogWriter(streamWriter)
    Call globalLogSession.createLogEntry(LOG_INFO, "Here's a log entry", "", "")
    Call globalLogSession.createLogEntry(LOG_ERROR, "Error encountered", "Here's a logged error", "")
    Call globalLogSession.createLogEntry(LOG_DEBUG, "Debug message", "Here's a debug message", "")
End Sub
```
### Results
If your code runs successfully you'll find a new `logs` subdirectory under your current directory, and it will contain your log file. The contents of the log file should look like this:

```text linenums="1"
INFO: Here's a log entry
ERROR: Error encountered
DEBUG: Debug message
```