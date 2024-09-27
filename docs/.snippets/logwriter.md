| Method | Reason to extend |
|--------|-------------|
| initializeLog() | Action to be performed before writing each log. This is only triggered if there are any logs to be written. |
| convertLogEntryToMessage() | Use this if you do not intend to use a formatter. It should return a string to be written out. |
| outputLogEntryMessage() | This is where you put your custom logic to write to the relevant location. |
| terminateLog() | Action to be performed after writing all logs, such as closing a file or database connection. |