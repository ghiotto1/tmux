# Purpose
This C source code file provides a utility function, [`getprogname`](#getprogname), which returns the name of the currently executing program. It includes conditional compilation directives to handle different environments: if `HAVE_PROGRAM_INVOCATION_SHORT_NAME` is defined, it returns `program_invocation_short_name`; if `HAVE___PROGNAME` is defined, it returns the external variable `__progname`; otherwise, it defaults to returning the string "tmux". This approach ensures compatibility across various systems by adapting to the available program name retrieval methods. The file also includes a permissive license notice, allowing for broad use and modification of the code.
# Imports and Dependencies

---
- `sys/types.h`
- `errno.h`
- `compat.h`


# Functions

---
### getprogname<!-- {{#callable:getprogname}} -->
The `getprogname` function returns the name of the program as a string, using different methods based on available system features.
- **Inputs**:
    - None
- **Control Flow**:
    - The function checks if `HAVE_PROGRAM_INVOCATION_SHORT_NAME` is defined; if so, it returns `program_invocation_short_name`.
    - If `HAVE___PROGNAME` is defined instead, it returns the external variable `__progname`.
    - If neither of the above conditions are met, it defaults to returning the string "tmux".
- **Output**:
    - The function returns a constant character pointer to the program name.


