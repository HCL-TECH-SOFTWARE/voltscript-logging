# FAQs

## Why can't I create a LogSession or ErrorSession?

You should use the singletons returned by the global variable `globalLogSession` and global function `getErrorSession()`.

## Why is it different?

To make it logging as easy as possible, we use the `Delete` method of the `LogSession` object to write out the logs. However, if access is via a function, all properties of the LogSession get garbage collected by the start of Delete. Lists are still iterable, but all elements within the `logWriters` and `logEntries` list are garbage collected and `is Nothing` returns True.

However, if LogSession is a global variable **and** is instantiated during the `Initialize` **and** is accessed as a global variable, they are still available.

No functionality needs to run during the deletion of the `ErrorSession` object, it may not be used, and the `get...` paradigm is more familiar from other languages. So we retain the approach we intended originally for LogSession.

## Why is there only a LogWriter for printing to the terminal?

The framework provides tooling for managing errors and logs and is deliberately intended to minimize dependencies. The BaseLogWriter implements printing to the standard output, which can be done easily from the core language's `Print` function. Writing to a file or a database will require a VSE. To avoid polluting your project with unnecessary dependencies, we have decided not to include more sophisticated log writers. However, the tutorial explains how this can be done.

## What timezone are logs logged in?

We don't want to add dependencies, so only the core language is used. This means `Now()` core function, which uses the timezone of the operating system the VoltScript program running the code is deployed to.

If you wish to force UTC, you can extend LogEntry class with a class that uses ZuluVSE. We'll not be adding dependencies to this library.