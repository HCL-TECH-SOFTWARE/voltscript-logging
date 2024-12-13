# Error Management

There are several options when managing errors:

- Throw the error, with standard VoltScript functionality `Error <code>,<message>`.
- Add a thrown error to the error session, with `getErrorSession().createErrorEntry()`.
- Add a custom error to the error session, with `getErrorSession().createCustomErrorEntry()`.

## But which should you use and when?

``` mermaid
flowchart TD
    AA[Processing Malfunction] --> A{Can you handle it?}
    A -- No --> B(Throw the error)
    B --> Z([EXIT SUB OR FUNCTION])
    A -- Yes --> C{Should subsequent<BR>code continue?}
    C -- No --> D{"Does Err() have a value?"}
    D -- Yes --> E("Call getErrorSession().<BR>createErrorEntry()<BR>to extract from core functions")
    E --> Z
    D -- No --> F("Call getErrorSession().<BR>createCustomErrorEntry()")
    F --> Z
    C -- Yes --> G{"Does Err() have a value?"}
    G -- Yes --> H("Call getErrorSession().<BR>createErrorEntry()<BR>to extract from core functions")
    H --> L{Are we in<BR>a logical loop?}
    L -- No --> Y([Continue Code])
    L -- Yes --> M("Call getErrorSession().<BR>createCustomErrorEntry() for which loop")
    M --> Y
    G -- No --> K("Call getErrorSession().<BR>createCustomErrorEntry()")
    K --> L
```

Calling `getErrorSession().createErrorEntry()` is a quick way to add an error to logs without manually parsing it. But it increments the error count in the ErrorSession. That's fine if you're immediately aborting the code, but could be problematic elsewhere.

Calling `getErrorSession.createCustomErrorEntry()` is **only** useful if you want to track the error in the ErrorSession later. If you *definitely don't* need to track it later, you should just create a LogEntry.

Adding errors to the ErrorSession is useful when you're performing a loop. You can then:

- process each element in the loop.
- add a try / catch block.
- throw an error if it occurs.
- catch the error and add to the global ErrorSession.
- also add an error for which element you hit the error on - remember the error may not know it's within a loop!
- after processing, check the `getErrorSession().errorCount > 0` and act accordingly.

Obviously this only works if the global ErrorSession is clean before starting the loop. So you may wish to track incoming error count before the loop. As a consumer of other people's code, it's best practice to ensure you do not add errors to the global ErrorSession unless you are immediately aborting.

!!! warning
    Instead of tracking incoming error count, you could call `getErrorSession().reset()`. However, the current loop could be called from within another loop, in which case you would be clearing all errors from the previous loops!

## Clearing the error stack

A good use case for clearing the error stack would be code that runs through a specific set of steps. The traditional approach would be to have functions that return a boolean: `True` means we're safe to continue, `False` means abort.

Logging provides an alternative approach.

``` mermaid
sequenceDiagram
participant User
participant Runtime
participant Sub as Sub Initialize
User ->> Runtime: Trigger process
Runtime ->> Sub: Trigger code
Sub ->> Function: Call subordinate function 1
Function ->> Sub: Complete
Sub ->> Sub: Errors in session?
Sub -->> User: Notify of error
Sub ->> Function: Call subordinate function 2
Function ->> Sub: Complete
Sub ->> Sub: Errors in session?
alt Errors found
    Sub ->> Function: Call subordinate function 3
    Function ->> Sub: Complete
else No errors
    Sub ->> Function: Call subordinate function 4
    Function ->> Sub: Complete
    Sub ->> Sub: Errors in session?
end
Sub ->> User: Notify of success / failure
```