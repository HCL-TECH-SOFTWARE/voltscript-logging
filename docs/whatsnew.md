---
hide:
    - navigation
---

# What's new

The section provides information on the latest features, improvements, and resolved issues related to VoltScript Logging.

!!! note "Important"

    Items marked in red are API changes that may impact your applications and should be reviewed before upgrading.

???+ info "v1.0.1 - What's new or changed"
    ## v1.0.1

    - A warning is now printed when instantiating a LogWriter, if maxLevel is lower than minLevel.
    - Marketplace URLs updated.
    - Better parsing of stack trace.
    - <span style="color:red">ErrorType renamed ErrorEntry, consistent with LogEntry.</span>
    - <span style="color:red">ErrorType methods have been changed. Review documentation.</span>

    !!! warning
        
        - v1.0.1 contains a major refactoring of the API. 1.0.0 code will need updating.
        - 1.0.1 requires VoltScript runtime EA4. Errors will be thrown with the EA3 runtime.

??? info "v1.0.0 - What's new or changed"
    ## v1.0.0

    - First release version of VoltScript Logging.