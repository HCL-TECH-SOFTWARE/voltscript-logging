# Error tracking

!!! note
    This how to does not cover best practices around error handling. That is independent of a specific error tracking or logging library.

Error tracking is used for capturing VoltScript errors. These are errors triggered by VoltScript error codes (e.g. 13, Type mismatch) or custom errors thrown by `Error errCode, errMsg`. The ErrorEntity object will automatically parse the error code, message, line number and stack trace to store the relevant information.

!!! warning
    Remember that the error line and stack trace are based on where the Try/Catch is. If the error occurs in a sub or function being called, but that does not have error handling, the line number and stack trace will be based on the calling code. To get more granular information of precisely where the error was triggered, you will need to add error handling to the sub or function being called.

The error tracking uses two classes:

- **ErrorSession**, accessed via `getErrorSession()`, a container for an array of `ErrorEntity` objects.
- **ErrorEntity**, automatically created and added to the session by `getErrorSession().addError()` function.

ErrorEntitys do not have an error level, a level is only relevant when logging the entry. You may wish to do different things depending on whether a custom error code (typically between 1000 and 1999), but that should be handled with implementation code in your Try/Catch block.

## Get ErrorSession

Like the LogSession, the ErrorSession instance is a singleton, lazy-loaded on the first call to `getErrorSession()`. Always use this method to retrieve the current ErrorSession, trying to do `Dim errorSess as New ErrorSession()` will throw an error.

!!! info
    VoltScript objects cannot persist between script runs. So when the `Sub Initialize` is first triggered, the ErrorSession instance will be `Nothing`. When the `Sub Initialize` ends, the ErrorSession instance will be deleted.

## Add errors to the session

The expected way to create an ErrorEntity object is via `Call getErrorSession().addError()`. That function returns the `ErrorEntity` object, if you wish to do any additional processing with it like checking the error code for expected or unexpected errors.

```vbscript
Sub performFatalLoop

    Dim i as Integer
    Dim ee as ErrorEntity

    For i = 0 to 10
        Try
            ' Do stuff
            Error 1000, "Generic Error on loop " & i
        Catch
            Set ee = getErrorSession().addError(||)
        End Try
    Next

End Sub
```

## Logging errors

The `errorCount` can be used to check whether errors have been logged. This can avoid passing errors up the stack, as below:

``` vbscript
Sub Initialize
    Dim errors as Variant 
    Dim ee as ErrorEntity

    ' Additional code required here to add a LogWriter, or nothing gets written out!

    Call performFatalLoop()
    ' Print "Error count: " & getErrorSession().errorCount 

    If (getErrorSession().errorCount > 0) Then
        errors = getErrorSession().errors 
        Forall element in errors
            Set ee = element
            ' Call ee.printSummary() 
            Call globalLogSession.createLogEntry(LOG_FATAL, ee.getLogMessage(), ee.stackTrace, "")
        End Forall
    End If

End Sub
```

`ErrorEntry.getLogMessage()` formats the error string, error code and line number in a human-readable string. Additional information of method and script are in the stack trace, so we pass that to the LogEntry as the extended information.

## Clearing the error session

There may be a scenario where you are not interested in the actual errors, but just want to clear the ErrorSession to continue processing. `ErrorSession.reset()` will do this, clearing all ErrorEntity objects from the session and setting error count back to 0.

See [sample code](../assets/example_code/errors.txt)