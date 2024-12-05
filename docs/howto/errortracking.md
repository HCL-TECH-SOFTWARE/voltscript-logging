# Error tracking

!!! note
    This how to does not cover best practices around error handling. That is independent of a specific error tracking or logging library.

Error tracking is used for capturing VoltScript errors. These are errors triggered by VoltScript error codes (e.g. 13, Type mismatch) or custom errors thrown by `Error errCode, errMsg`. The ErrorEntry object will automatically parse the error code, message, line number and stack trace to store the relevant information.

!!! warning
    Remember that the error line and stack trace are based on where the Try/Catch is. If the error occurs in a sub or function being called, but that does not have error handling, the line number and stack trace will be based on the calling code. To get more granular information of precisely where the error was triggered, you will need to add error handling to the sub or function being called.

The error tracking uses two classes:

- **ErrorEntry**: Instances of this class carry the aforementioned information about an error.  
- **ErrorSession**, accessed via `getErrorSession()`, is a container of `ErrorEntry` object instances.  These instances are automatically created and added to the `ErrorSession` via the `createErrorEntry()` function.  Custom error information can also be added to the `ErrorSession` by calling the `createCustomErrorEntry()` method. 

ErrorEntry instances carry the error code, but do not have an associated error or log level: level is only relevant when logging the information.  Implementation code for differing handling behavior based upon the value of the error code will typically be contained inside of a Try/Catch block.

## Get ErrorSession

Like the LogSession, the ErrorSession instance is a singleton, lazy-loaded on the first call to `getErrorSession()`. Always use this method to retrieve the current ErrorSession; attempting to call `Dim variable_name as New ErrorSession()` will throw an error.

!!! info
    VoltScript objects cannot persist between script runs. So when the `Sub Initialize` is first triggered, the ErrorSession instance will be `Nothing`. When the `Sub Initialize` ends, the ErrorSession instance will be deleted.

## Add errors to the session

The expected way to create an ErrorEntry object is via `Call getErrorSession().createErrorEntry()`. That function returns the `ErrorEntry` object. Use this instance if you wish to do any additional processing such as checking the error code for expected or unexpected errors.

```vbscript
Sub performFatalLoop()

    Dim i as Integer
    Dim ee as ErrorEntry

    Call globalLogSession.createLogEntry(LOG_INFO, "Performing a Fatal Loop", "")

    For i = 0 to 10
        Try
            ' Do stuff
            Error 1000, "Generic Error on loop " & i
        Catch
            Set ee = getErrorSession().createErrorEntry(NO_LOGGING)
        End Try
    Next
    
    Call globalLogSession.createLogEntry(LOG_INFO, "Finished Performing a Fatal Loop", "")

End Sub
```

## Logging errors

The `errorCount` can be used to check whether errors have been logged. This can avoid passing errors up the stack, as below:

``` vbscript
Sub Initialize()
    Dim errors as Variant 
    Dim ee as ErrorEntry
    Dim writer as BaseLogWriter 

    ' Additional code required here to add a LogWriter, or nothing gets written out!

    Call performFatalLoop()

    If (getErrorSession().errorCount > 0) Then
        errors = getErrorSession().errors 
        Forall element in errors
            Set ee = element
            Call globalLogSession.createLogEntry(LOG_FATAL, ee.getLogMessage(), ee.stackTrace)
        End Forall
    End If

End Sub
```

`ErrorEntry.getLogMessage()` formats the error string, error code and line number in a human-readable string. Additional information about the library, class, and method are in contained int the stack trace.  When spawning a LogEntry we pass the stack trace as extended information.

## Clearing the error session

There may be a scenario where you are not interested in the actual errors, but just want to clear the ErrorSession to continue processing. `ErrorSession.reset()` will do this, clearing all ErrorEntry objects from the session and setting error count back to 0.

## Custom Errors 

Custom ErrorEntry instances can be created at any time, they do **not** require an Error to be thrown and caught.  A common programming pattern of the past would be to intentionally throw an exception, then catch or trap any thrown errors, gather and log information about the error, and then throw a new error with additional contextual information.  This pattern is messy and should be avoided; being able to create an ErrorEntry instance independent of throwing an exception allows a developer to do just that. The method `createCustomErrorEntry()` has four arguments:

 - *message*  Error message describing the error.  If code is less than 1 the value of Error$() will be used. 
 - *code*  Numeric code of the error. 	If less than 1 the value of Err() will be used.  
 - *lineNum*  Line number in the source code where the error occurred. If code is less than 1 the value of Erl() will be used.  
 - *levelNum*  The Logging Level for conditionally spawning a LogEntry 

Consider the following example: 

``` vbscript 
Sub performCreateCustomErrorEntryInstances() 

    ' Create custom ErrorEntry instances without throw / catch.
    Call getErrorSession().createCustomErrorEntry("The email address requires an '@' character", 2021, 101, NO_LOGGING)

    Call getErrorSession().createCustomErrorEntry("The passed in argument is invalid", 1195, 201, LOG_WARN)
    
    Call getErrorSession().createCustomErrorEntry("The file cannot be found", 1065, 351, LOG_ERROR)

End Sub 
``` 


## Logging Errors with context 

The sample code in the above `Sub Initialize()` will create LogEntries for ALL errors, using log level of LOG_FATAL.  While this may be useful in some instances, this all-or-nothing approach is not the only way to add error information to the log.  

The `levelNum` argument of the `createErrorEntry()` method specifies the logging level.  If set to `NO_LOGGING` then the ErrorEntry will simply be created and added to the ErrorSession.  Passing any other valid log level will cause a new LogEntry object to be immediately created by the LogSession, using information from the ErrorEntry.   

``` vbscript 
Call getErrorSession().createErrorEntry(LOG_ERROR)
``` 

This capability allows the developer to add information to the log based upon the context of the error (`LOG_TRACE`, `LOG_DEBUG`, `LOG_INFO`... `LOG_FATAL`). 

Additionally, there is no need for the developer to subsequently process the ErrorEntry instances to add information to the LogSession. 

``` vbscript
Sub performFatalLoopWithContext()

    Dim i as Integer

    Call globalLogSession.createLogEntry(LOG_INFO, "Performing a Fatal Loop with context", "")

    For i = 0 to 10
        Try
            ' Do stuff
            If i < 5 Then Error 1000, "Generic Error on loop " & i

            Call getErrorSession().createCustomErrorEntry("Iteration " & i, 1000 + i, 85, LOG_INFO)
        Catch
            Call getErrorSession().createErrorEntry(LOG_ERROR)
        End Try
    Next
    
    Call globalLogSession.createLogEntry(LOG_INFO, "Finished Performing a Fatal Loop with context", "")

End Sub
``` 


See [sample code](../assets/example_code/errors.txt)