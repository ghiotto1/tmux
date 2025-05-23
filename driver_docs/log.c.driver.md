# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it provides logging functionality. The code defines a set of functions to manage logging operations, including opening and closing log files, toggling logging states, and writing log messages. The logging system is designed to handle different levels of logging, with the ability to increment the log level and retrieve the current log level. It also integrates with the libevent library by setting a log callback function to handle event logging. The file includes functions for logging debug messages and handling critical errors, with the latter resulting in program termination.

The code is structured to facilitate logging in a controlled manner, allowing logs to be written to a file with a specific naming convention that includes the process ID. The use of variadic functions enables flexible message formatting, and the code ensures that log messages are timestamped with precise time information. The logging system is designed to be toggled on and off, providing flexibility in managing log verbosity. Additionally, the code includes mechanisms to handle fatal errors by logging the error message and terminating the program, ensuring that critical issues are recorded before the application exits.
# Imports and Dependencies

---
- `sys/types.h`
- `errno.h`
- `stdio.h`
- `stdlib.h`
- `string.h`
- `unistd.h`
- `tmux.h`


# Global Variables

---
### log_file
- **Type**: `FILE*`
- **Description**: The `log_file` variable is a static pointer to a `FILE` object, used for logging purposes in the program. It is initialized when logging is enabled and points to a file where log messages are written.
- **Use**: This variable is used to manage the file stream for logging messages, allowing the program to write log entries to a specified file.


---
### log_level
- **Type**: `int`
- **Description**: The `log_level` variable is a static integer that represents the current logging level of the application. It is used to control the verbosity of logging output, with higher values typically indicating more detailed logging.
- **Use**: This variable is used to determine whether logging should be enabled or disabled and to control the level of detail in the log messages.


# Functions

---
### log_event_cb<!-- {{#callable:log_event_cb}} -->
The `log_event_cb` function is a callback for logging events that writes a message to the debug log.
- **Inputs**:
    - `severity`: An integer representing the severity of the log event, which is unused in this function.
    - `msg`: A constant character pointer to the message string that needs to be logged.
- **Control Flow**:
    - The function takes two parameters: an unused integer 'severity' and a constant character pointer 'msg'.
    - It calls the [`log_debug`](#log_debug) function, passing the message 'msg' to it for logging.
- **Output**:
    - The function does not return any value.
- **Functions called**:
    - [`log_debug`](#log_debug)


---
### log_add_level<!-- {{#callable:log_add_level}} -->
The `log_add_level` function increments the global logging level by one.
- **Inputs**:
    - None
- **Control Flow**:
    - The function accesses the global variable `log_level`.
    - It increments the value of `log_level` by one.
- **Output**:
    - The function does not return any value.


---
### log_get_level<!-- {{#callable:log_get_level}} -->
The `log_get_level` function returns the current logging level.
- **Inputs**:
    - None
- **Control Flow**:
    - The function directly returns the value of the static variable `log_level`.
- **Output**:
    - The function returns an integer representing the current logging level.


---
### log_open<!-- {{#callable:log_open}} -->
The `log_open` function initializes logging by opening a log file with a specified name and setting up the logging environment if logging is enabled.
- **Inputs**:
    - `name`: A constant character pointer representing the base name for the log file to be created.
- **Control Flow**:
    - Check if the logging level is zero; if so, return immediately without doing anything.
    - Call `log_close()` to close any previously opened log file.
    - Use `xasprintf` to format the log file name as 'tmux-<name>-<pid>.log', where `<name>` is the input argument and `<pid>` is the process ID.
    - Attempt to open the log file in append mode using `fopen`; if unsuccessful, return immediately.
    - Set the log file's buffer to line-buffered mode using `setvbuf`.
    - Set the event log callback to `log_event_cb` using `event_set_log_callback`.
- **Output**:
    - The function does not return any value, but it sets up the global `log_file` for logging if successful.
- **Functions called**:
    - [`log_close`](#log_close)


---
### log_toggle<!-- {{#callable:log_toggle}} -->
The `log_toggle` function toggles the logging state by opening or closing the log file based on the current log level.
- **Inputs**:
    - `name`: A string representing the name used to create the log file name when opening the log.
- **Control Flow**:
    - Check if `log_level` is 0, indicating logging is currently off.
    - If logging is off, set `log_level` to 1, open the log file using `log_open(name)`, and log a debug message 'log opened'.
    - If logging is on (i.e., `log_level` is not 0), log a debug message 'log closed', set `log_level` to 0, and close the log file using `log_close()`.
- **Output**:
    - The function does not return any value; it modifies the logging state and performs side effects such as opening or closing the log file.
- **Functions called**:
    - [`log_open`](#log_open)
    - [`log_debug`](#log_debug)
    - [`log_close`](#log_close)


---
### log_close<!-- {{#callable:log_close}} -->
The `log_close` function closes the current log file and resets the logging callback to NULL.
- **Inputs**:
    - None
- **Control Flow**:
    - Check if `log_file` is not NULL, indicating an open log file.
    - If `log_file` is open, close it using `fclose`.
    - Set `log_file` to NULL to indicate no log file is open.
    - Reset the event logging callback by calling `event_set_log_callback` with NULL.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the logging system.


---
### printflike<!-- {{#callable:printflike}} -->
The `log_vwrite` function formats a log message with a timestamp and writes it to a log file if logging is enabled.
- **Inputs**:
    - `msg`: A format string for the log message.
    - `ap`: A `va_list` containing the arguments for the format string.
    - `prefix`: A string to prepend to the formatted log message.
- **Control Flow**:
    - Check if the global `log_file` is NULL, and return if it is, indicating logging is not enabled.
    - Use `vasprintf` to format the message string `msg` with the arguments in `ap`, storing the result in `s`. If `vasprintf` fails, return.
    - Use `stravis` to convert the formatted string `s` into a safe representation `out` using specified visual options. If `stravis` fails, free `s` and return.
    - Free the memory allocated for `s` after conversion.
    - Get the current time using `gettimeofday` and store it in `tv`.
    - Write the timestamp, prefix, and converted message `out` to the `log_file` using `fprintf`. If successful, flush the `log_file` buffer.
    - Free the memory allocated for `out`.
- **Output**:
    - The function does not return a value; it writes a formatted log message to a file if logging is enabled.


---
### log_debug<!-- {{#callable:log_debug}} -->
The `log_debug` function logs a debug message to a file if logging is enabled.
- **Inputs**:
    - `msg`: A constant character pointer representing the message format string to be logged.
    - `...`: A variable number of arguments that correspond to the format specifiers in the message string.
- **Control Flow**:
    - Check if the global `log_file` is `NULL`, and if so, return immediately without logging.
    - Initialize a `va_list` variable `ap` to handle the variable arguments passed to the function.
    - Call `va_start` to initialize `ap` with the variable arguments starting after `msg`.
    - Invoke `log_vwrite` with the message format string, the initialized `va_list`, and an empty string as the prefix to write the log message to the file.
    - Call `va_end` to clean up the `va_list` variable `ap`.
- **Output**:
    - The function does not return any value; it performs logging as a side effect if logging is enabled.


---
### fatal<!-- {{#callable:fatal}} -->
The `fatal` function logs a critical error message with the current error string and terminates the program.
- **Inputs**:
    - `msg`: A format string for the error message to be logged.
    - `...`: Additional arguments for the format string.
- **Control Flow**:
    - Declare a character array `tmp` of size 256 and a `va_list` variable `ap`.
    - Use `snprintf` to format a string into `tmp` that includes the current error string from `strerror(errno)` prefixed by 'fatal: '.
    - Check if `snprintf` fails (returns a negative value), and if so, terminate the program with `exit(1)`.
    - Initialize the variable argument list `ap` using `va_start`.
    - Call `log_vwrite` to log the formatted message with the prefix stored in `tmp`.
    - Clean up the variable argument list using `va_end`.
    - Terminate the program with `exit(1)`.
- **Output**:
    - The function does not return; it logs the error message and terminates the program execution.


---
### fatalx<!-- {{#callable:fatalx}} -->
The `fatalx` function logs a critical error message and terminates the program.
- **Inputs**:
    - `msg`: A constant character pointer representing the error message to be logged.
    - `...`: A variable number of arguments that can be used to format the error message.
- **Control Flow**:
    - Initialize a variable argument list `ap` using `va_start` with `msg` as the last fixed argument.
    - Call `log_vwrite` to log the formatted message with the prefix 'fatal: '.
    - Clean up the variable argument list using `va_end`.
    - Terminate the program by calling `exit(1)`.
- **Output**:
    - The function does not return as it terminates the program using `exit(1)`, indicating an abnormal termination.


