# Purpose
This C source code file provides a function to count the number of open file descriptors for the current process, utilizing the `/proc` filesystem if available. It includes necessary headers for system types and globbing, and defines two functions, `fatal` and `fatalx`, which are likely used for error handling, though their implementations are not provided here. The [`getdtablecount`](#getdtablecount) function is conditionally compiled based on the presence of the `HAVE_PROC_PID` macro, which indicates whether the `/proc` filesystem is available. If available, it constructs a path to the file descriptor directory of the current process and uses `glob` to count the entries, returning this count. If the `/proc` filesystem is not available, it simply returns zero. This code is likely part of a larger system where monitoring or managing file descriptors is necessary.
# Imports and Dependencies

---
- `sys/types.h`
- `glob.h`
- `unistd.h`
- `compat.h`


# Functions

---
### getdtablecount<!-- {{#callable:getdtablecount}} -->
The `getdtablecount` function returns the number of open file descriptors for the current process if the `/proc` filesystem is available, otherwise it returns 0.
- **Inputs**:
    - None
- **Control Flow**:
    - The function checks if the `HAVE_PROC_PID` macro is defined to determine if the `/proc` filesystem is available.
    - If `/proc` is available, it constructs a path to the file descriptor directory of the current process using `snprintf`.
    - It uses `glob` to count the number of entries in the file descriptor directory, which corresponds to the number of open file descriptors.
    - The function returns the count of open file descriptors if successful, or 0 if the `/proc` filesystem is not available.
- **Output**:
    - The function returns an integer representing the number of open file descriptors for the current process, or 0 if the `/proc` filesystem is not available.


