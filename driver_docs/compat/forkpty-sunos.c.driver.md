# Purpose
This C source code file provides a specific functionality related to pseudo-terminal (PTY) management, specifically implementing a [`forkpty`](#forkpty) function. The [`forkpty`](#forkpty) function is designed to create a new pseudo-terminal pair, fork a new process, and establish the child process's standard input, output, and error streams to the slave side of the PTY. This is a common requirement in terminal emulation and process management applications, where a program needs to interact with a terminal-like interface programmatically. The code includes system calls and library functions to handle PTY operations, such as `open`, `grantpt`, `unlockpt`, and `ptsname`, and it uses `ioctl` to configure terminal settings and window size.

The file is not a standalone executable but rather a component that can be integrated into larger applications requiring PTY functionality. It includes necessary headers for system-level operations and terminal control, indicating its reliance on low-level system interfaces. The presence of [`fatal`](#fatal) and [`fatalx`](#fatalx) function declarations suggests that error handling is managed through these functions, although their implementations are not provided in this snippet. The code is structured to handle errors gracefully by cleaning up resources before returning, ensuring robustness in resource management. This file is likely part of a library or a larger system that deals with terminal emulation or process control, providing a critical utility function for managing pseudo-terminals.
# Imports and Dependencies

---
- `sys/types.h`
- `sys/ioctl.h`
- `fcntl.h`
- `stdlib.h`
- `strings.h`
- `stropts.h`
- `termios.h`
- `unistd.h`
- `compat.h`


# Functions

---
### forkpty<!-- {{#callable:forkpty}} -->
The `forkpty` function creates a pseudo-terminal pair, forks a new process, and sets up the terminal environment for the child process.
- **Inputs**:
    - `master`: A pointer to an integer where the file descriptor for the master side of the pseudo-terminal will be stored.
    - `name`: A character array where the name of the slave pseudo-terminal device will be copied, if not NULL.
    - `tio`: A pointer to a `termios` structure containing terminal attributes to be set for the slave pseudo-terminal, if not NULL.
    - `ws`: A pointer to a `winsize` structure containing window size attributes to be set for the slave pseudo-terminal.
- **Control Flow**:
    - Open the master side of the pseudo-terminal using `/dev/ptmx` and store the file descriptor in `master`.
    - Grant access to the slave pseudo-terminal using `grantpt` and unlock it using `unlockpt`.
    - Retrieve the name of the slave pseudo-terminal using `ptsname` and copy it to `name` if `name` is not NULL.
    - Open the slave side of the pseudo-terminal using the retrieved name.
    - Fork the process; in the child process, close the master file descriptor, create a new session, and set the controlling terminal if `TIOCSCTTY` is defined.
    - Push terminal modules `ptem` and `ldterm` onto the slave file descriptor using `ioctl`.
    - Set terminal attributes using `tcsetattr` if `tio` is not NULL and set window size using `ioctl`.
    - Redirect standard input, output, and error to the slave file descriptor and close it if it is greater than 2.
    - Return 0 in the child process and the child's PID in the parent process.
    - On error, close any open file descriptors and return -1.
- **Output**:
    - The function returns 0 in the child process, the child's PID in the parent process, or -1 on error.


# Function Declarations (Public API)

---
### fatal<!-- {{#callable_declaration:fatal}} -->
Logs a fatal error message and terminates the program.
- **Description**: Use this function to log a critical error message and immediately terminate the program. It formats the error message using a printf-style format string and additional arguments, appending the system error message associated with the current value of errno. This function should be called when a non-recoverable error occurs, and the program cannot continue execution safely. It does not return, as it exits the program after logging the message.
- **Inputs**:
    - `msg`: A printf-style format string for the error message. Must not be null. The format string can include format specifiers that correspond to additional arguments.
    - `...`: Additional arguments to be formatted into the message string according to the format specifiers in 'msg'. These arguments must match the expected types for the format specifiers used in 'msg'.
- **Output**: None
- **See also**: [`fatal`](../log.c.driver.md#fatal)  (Implementation)


---
### fatalx<!-- {{#callable_declaration:fatalx}} -->
Logs a fatal error message and terminates the program.
- **Description**: This function is used to log a fatal error message and immediately terminate the program. It should be called when a non-recoverable error occurs, and the program cannot continue execution. The function takes a format string and a variable number of arguments, similar to printf, to construct the error message. The message is prefixed with 'fatal: ' before being logged. After logging the message, the function exits the program with a status code of 1. This function does not return, and its use should be limited to situations where program termination is the only option.
- **Inputs**:
    - `msg`: A format string for the error message, similar to printf. It must not be null and should be a valid format string. Additional arguments should match the format specifiers in the string.
- **Output**: None
- **See also**: [`fatalx`](../log.c.driver.md#fatalx)  (Implementation)


