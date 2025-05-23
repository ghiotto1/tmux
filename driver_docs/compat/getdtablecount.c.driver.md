# Purpose
This C source code file provides a function to count the number of open file descriptors for the current process, utilizing conditional compilation to support different environments. The function [`getdtablecount`](#getdtablecount) is defined to use the `/proc` filesystem, specifically the `/proc/[pid]/fd` directory, to count file descriptors if the `HAVE_PROC_PID` macro is defined, indicating the presence of a `/proc` filesystem (common in Unix-like systems). If this macro is not defined, the function simply returns zero, suggesting that the environment does not support this method of counting file descriptors. The file includes necessary headers for system types and globbing, and it also declares two external functions, [`fatal`](#fatal) and [`fatalx`](#fatalx), presumably for error handling. The code is wrapped in a permissive license, allowing for broad use and modification.
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
    - The `globfree` function is called to free the memory allocated by `glob`.
    - The function returns the count of open file descriptors.
    - If `/proc` is not available, the function simply returns 0.
- **Output**:
    - The function returns an integer representing the number of open file descriptors for the current process, or 0 if the `/proc` filesystem is not available.


# Function Declarations (Public API)

---
### fatal<!-- {{#callable_declaration:fatal}} -->
Logs a fatal error message and terminates the program.
- **Description**: Use this function to log a critical error message and immediately terminate the program. It is intended for situations where continuing execution is not possible or safe. The function formats the error message using a variable argument list, appends the current error string from errno, and logs it before exiting. This function does not return and will terminate the program with an exit status of 1.
- **Inputs**:
    - `msg`: A format string for the error message. It must not be null and should follow printf-style formatting conventions. The caller is responsible for ensuring the format string is valid and that the corresponding arguments match the format specifiers.
- **Output**: None
- **See also**: [`fatal`](../log.c.driver.md#fatal)  (Implementation)


---
### fatalx<!-- {{#callable_declaration:fatalx}} -->
Logs a fatal error message and terminates the program.
- **Description**: This function is used to log a critical error message and immediately terminate the program. It should be called when a non-recoverable error occurs, and the program cannot continue execution safely. The function takes a format string and a variable number of arguments, similar to printf, to construct the error message. The message is prefixed with 'fatal: ' before being logged. After logging the message, the function exits the program with a status code of 1. This function does not return, and its use should be limited to situations where program termination is the only viable option.
- **Inputs**:
    - `msg`: A format string for the error message, similar to printf. It must not be null and should be a valid format string. The caller retains ownership.
    - `...`: A variable number of arguments that correspond to the format specifiers in the format string. These arguments are used to construct the final error message.
- **Output**: None
- **See also**: [`fatalx`](../log.c.driver.md#fatalx)  (Implementation)


