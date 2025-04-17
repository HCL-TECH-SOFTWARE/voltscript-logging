# Use LogWriters

LogWriters are designed to write logs to different locations. The LogWriter takes in four arguments:

- **label**:     a unique label which will be the key in the Map of LogWriters.
- **minLevel**:  a minimum level of a LogEntry to log.
- **maxLevel**:  a maximum level of a LogEntry to log.
- **formatter**: a string formatter to produce the message to log.

Logs will be outputted if they are at or above the minimum level and at or below the maximum level.

For example, if the *minimum* level is LOG_TRACE and *maximum* level is LOG_INFO then logs with LOG_TRACE, LOG_DEBUG, and LOG_INFO will be written out, but logs with LOG_WARNING, LOG_ERROR, and LOG_FATAL will be ignored.

A special level is also available for `maxLevel` in LogWriters - NO_LOGGING. If the max level is NO_LOGGING, no logs will be processed and nothing written.

!!! important
    --8<-- "nologs.md"

## Setting logging level

Best practice is to define or retrieve the maximum logging level before you create the LogWriter. This could be done in a variety of ways:

- Hard-code in a constant at the top of your script. This makes it easier to find and change.
- Retrieve from an environment variable or other configuration source.

## Dynamically loading loggers

A typical use case is wishing to define LogWriters in a configuration file. This code requires a dependency on JsonVSE and VoltScript JSON Converter. So the function to read LogWriters from a JSON configuration file can be found in [VoltScript JSON Converter](https://opensource.hcltechsw.com/voltscript-json-converter/howto/jsonlogwriters.md).

## Changing logging level

The maximum level can only be set in the constructor. If you have to change it afterwards, you will need to create a new LogWriter. But be aware whether the LogWriter has already been added to the LogSession. If so, the correct method of processing would be:

1. Remove the LogWriter from the LogSession (`globalLogSession.removeLogWriter(myLogWriter)`).
1. Delete the LogWriter. (`Set myLogWriter = Nothing`).
1. Re-instantiate the LogWriter. (`Set myLogWriter = New BaseLogWriter("stringFormat", LOG_DEBUG, LOG_FATAL, "{{LEVELNAME}}: {{MESSAGE}}")`).
1. Add the LogWriter back to the LogSession (`Call globalLogSession.addLogWriter(myLogWriter)`).

## Formatters

The formatter uses mustache syntax (`{{variable_name}}`) to replace content in the format. The following variables can be used:

| Variable           | Explanation                                                            |
|--------------------|------------------------------------------------------------------------|
| STACKTRACE         | The stack trace from the LogEntry                                      |
| STACKTRACE_COMPACT | The stack trace compacted with line feeds replaced by semi-colons      |
| LEVELNAME          | The name of the log level, e.g. FATAL                                  |
| ENTRYID            | The ID of the LogEntry                                                 |
| LEVEL              | The numeric value of the log level, e.g. 32                            |
| MESSAGE            | The log message                                                        |
| CLASSNAME          | The name of the class passed into the LogEntry                         |
| LINENUM            | The line number passed into the LogEntry                               |
| LIBRARYNAME        | The name of the library or module where the LogEntry was generated     |
| METHODNAME         | The name of the method or function where the LogEntry was generated    |
| TIMESTAMP          | The timestamp of the log entry                                         |
| EXTINFO            | Additional extended information passed into the log entry              |

!!! warning
    Variables are case-sensitive and **must** be entered in upper case in the formatter.

!!! info
    LINENUM is always the line your code triggered VoltScriptLogging, either by explicitly calling `globalLogSession.createLogEntry()` or calling `getErrorSession().createErrorEntry()` or `getErrorSession.createCustomErrorEntry()`. This is likely to be different to the error line referenced in the LogEntry message.

    For more details, see [Line numbers flowchart](../topicguides/approach.md#line-numbers).

### Sample Formatter

This code will write out the level name (e.g. DEBUG), a colon, a space and the message:

``` vbscript
Dim stringFormat as New BaseLogWriter("stringFormat", LOG_DEBUG, LOG_FATAL, "{{LEVELNAME}}: {{MESSAGE}}")
```

This code will write out a pretty-printed JSON object containing the level name, message, extended info and stack trace. **NOTE:** the stack trace tracks out from the `Sub New` of the LogEntry class, through `Function createLogEntry` of the LogSession class, up to the calling code.

``` vbscript
Dim jsonFormat  as New BaseLogWriter("jsonFormat", LOG_DEBUG, LOG_FATAL, |{
    "level": "{{LEVELNAME}}",
    "message": "{{MESSAGE}}",
    "extInfo": "{{EXTINFO}}",
    "stack": "{{STACKTRACE}}"
}|)
```

See [sample code](../assets/example_code/formats.txt)