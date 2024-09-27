# Aims for the framework

- No dependencies.
- Standardization with frameworks in other languages with Java.
- Flexibility about where logs are written.
- Easy error capture.
- Extensibility.
- Ease of changing levels and locations for logs without amending code.

No dependencies is a key requirement. We want to be able to integrate logging and error handling into all our other VoltScript Library Modules. That won't work if we depend on VoltScript Library Modules like VoltScript Collection or VoltScript Extensions like JsonVSE, ZuluVSE, or StreamVSE. So there are no dependencies and we'll not be adding any features that require them. Additional libraries can add dependencies, and remember that dependency management will download downstream dependencies for consumers.

## LogSession and LogEntry

A **LogEntry** has six levels, equivalent to FATAL, ERROR, WARN, INFO, DEBUG and TRACE. These levels will be familiar to developers who have used logging frameworks in e.g. Java, Rust etc.

The **LogSession** holds multiple log entries. The expectation is that the code does not handle whether or not logs are added to a session. Instead log entries should *always* be added to the LogSession. The specific **LogWriters** will handle whether or not logs for a specific level should be outputted by a specific LogWriter, and where / how they should be written.

A sample scenario would be a LogSession with two LogWriters:

- LogWriter1 writing LOG_FATAL or LOG_ERROR logs to a file called "errors.log".
- LogWriter2 writing LOG_DEBUG and LOG_TRACE to a file called "debug.log" if an environment variable is set.

## ErrorSession and ErrorType

At this time no changes are being made to error handling in the core language. The same functions from LotusScript are available, with one addition:

- **Error()** gives the error message.
- **Err()** gives the error code.
- **Erl()** gives the line the Error was triggered on.
- **getThreadInfo(12)** is an addition to VoltScript and gives a safe-to-use stack trace.

The aims of the framework are to make it as easy as possible to track errors. Thus `getErrorSession().addError()` creates an ErrorType object and adds it to the session. See the [Error tracking how-to](../howto/errortracking.md) for more details.

Again, error logging is separated from error tracking. Some errors may be handled differently from others - some may required immediate end of processing, others may be handled within the code. So ErrorTypes are not automatically added to the LogSession, you add them if required and with the relevant log level.