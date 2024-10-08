# Extend BaseLogWriter

The BaseLogWriter just prints to the console. This is unlikely to be sufficient for most scenarios. However, the class has been written to minimize and simplify code changes to output elsewhere. Specifically, you should not need to modify the `LogWriter.writeToLog()` function, which filters Log Entry objects and iterates them. You should only need to modify the following methods:

--8<-- "logwriter.md"

## InitializeLog()

A typical use case for this function would be to create / open a log file or authenticate to a database. It is important to consider whether the code, when implemented, could be called concurrently. If so, your implementation should handle it. This function will only be called if the LogWriter has been added to the LogSession, and only if there are any logs to write after filtering has been performed.

## convertLogEntryToMessage()

The base method uses a string formatter replacing mustache variables with properties extracted from the LogSession and LogEntry. If that is sufficient, you do not need to override the method. If you want something more sophisticated, override this method to produce a string from the LogSession and LogEntry being iterated.

!!! tip
    Remember: the formatter could be a JSON string, with variables inserted as appropriate. It could even be a string of YAML.

## outputLogEntryMessage()

This is the method you will always want to override. This is where you do the specifics of writing the relevant content to the log. It will take in the string generated by `convertLogEntryToMessage()`, but may perform transformation, e.g. converting it to JSON.

This method should minimize the amount of processing required. So if you need to write to a file, don't open it every time. Instead, open it in the `initializeLog()` and just write to it here.

## terminateLog()

Use this method to clean up processing, e.g. closing a log file.

!!! note
    If you want to collect everything into a single string and write that bulk string as a single operation, you can use `terminateLog()` to perform that. This may be more efficient. In this scenario, `outputLogEntryMessage()` would "output" it to a private variable that `terminateLog()` would use.

## Custom read-only properties

VoltScript classes only allow one method to be overloaded - the constructor. Use this approach if you want to pass other values into the constructor, e.g. ensuring that they can only be set once. The syntax is a little complex:

``` vbscript
Private myNewVariableHolder_ as Variant

Public Sub New(label As String, minLevel as Integer, maxLevel As Integer, formatter As String, myNewVariable as Variant), BaseLogWriter(label, minLevel, maxLevel, formatter)
    myNewVariableHolder_ = myNewVariable
End Sub
```

The method signature is followed by a comma, the base class name, and the variables to pass to its constructor. This will call the parent's constructor. It will then perform the additional code required, in this case passing `myNewVariable` argument to `myNewVariableHolder` variable.

If you wish to have fewer arguments, you would do something like this:

``` vbscript
Public Sub New(label as String, formatter as String), BaseLogWriter(label, LOG_TRACE, LOG_INFO, formatter)

End Sub
```

This prevents the developer using this class to set a minimum or maximum level. Instead, it always sets the minimum level to LOG_TRACE and the maximum level as LOG_INFO.