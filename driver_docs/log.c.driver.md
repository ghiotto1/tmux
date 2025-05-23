# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it provides logging functionality. The code is designed to handle logging operations, including opening and closing log files, writing log messages, and managing log levels. It includes functions to increment and retrieve the current log level, toggle logging on and off, and write both debug and critical error messages to a log file. The file also defines a callback function for logging events from the libevent library, which is used for event notification. The logging messages are timestamped and formatted before being written to the log file, ensuring that they are both informative and easy to read.

The code is structured to be integrated into a larger application, as indicated by its use of external libraries and its inclusion of a header file, "tmux.h". It does not define a public API but rather provides internal logging capabilities for the tmux application. The use of static functions and variables, such as [`log_event_cb`](#log_event_cb) and `log_file`, suggests that these components are intended for use only within this file, maintaining encapsulation and reducing the potential for unintended interactions with other parts of the application. The file also includes error handling functions, [`fatal`](#fatal) and [`fatalx`](#fatalx), which log critical errors and terminate the program, ensuring that severe issues are promptly addressed.
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
- **Description**: The `log_file` variable is a static pointer to a `FILE` object, used for logging purposes in the program. It is initialized when logging is enabled and points to a log file where messages are written.
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
    - `msg`: A constant character pointer to the message that needs to be logged.
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
    - The function outputs an integer representing the current logging level.


---
### log_open<!-- {{#callable:log_open}} -->
The `log_open` function initializes logging by opening a log file with a specified name and setting up the log callback if logging is enabled.
- **Inputs**:
    - `name`: A constant character pointer representing the base name for the log file to be created.
- **Control Flow**:
    - Check if the global `log_level` is zero; if so, return immediately as logging is not enabled.
    - Call `log_close()` to ensure any previously opened log file is closed before opening a new one.
    - Use [`xasprintf`](xmalloc.c.driver.md#xasprintf) to format the log file name as 'tmux-<name>-<pid>.log', where `<name>` is the input argument and `<pid>` is the current process ID.
    - Attempt to open the log file in append mode using `fopen`; if unsuccessful, return immediately.
    - Set the buffer mode of the log file to line-buffered using `setvbuf`.
    - Set the event log callback to `log_event_cb` using `event_set_log_callback`.
- **Output**:
    - The function does not return a value but sets up the global `log_file` for logging if successful.
- **Functions called**:
    - [`log_close`](#log_close)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


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
    - The function does not return any value; it modifies the global logging state and performs side effects such as opening or closing the log file.
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
    - If `log_file` is open, call `fclose` to close the file.
    - Set `log_file` to NULL to indicate no log file is open.
    - Call `event_set_log_callback` with NULL to remove the current logging callback.
- **Output**:
    - The function does not return any value.


---
### printflike<!-- {{#callable:printflike}} -->
The `log_vwrite` function formats a log message with a timestamp and writes it to a log file if logging is enabled.
- **Inputs**:
    - `msg`: A format string for the log message, similar to printf.
    - `ap`: A va_list containing the arguments for the format string.
    - `prefix`: A string to prepend to the formatted log message, often used for indicating the severity or type of log.
- **Control Flow**:
    - Check if the log file is open; if not, return immediately.
    - Use `vasprintf` to format the message string with the provided arguments; if it fails, return.
    - Use `stravis` to convert the formatted string into a safe representation for logging; if it fails, free the formatted string and return.
    - Free the original formatted string after conversion.
    - Get the current time using `gettimeofday`.
    - Write the timestamp, prefix, and converted message to the log file using `fprintf`.
    - Flush the log file to ensure the message is written immediately.
    - Free the converted message string.
- **Output**:
    - The function does not return a value; it writes the formatted log message to the log file if successful.


---
### log_debug<!-- {{#callable:log_debug}} -->
The `log_debug` function logs a debug message to a file if logging is enabled.
- **Inputs**:
    - `msg`: A constant character pointer representing the format string for the debug message.
    - `...`: A variable number of arguments that correspond to the format specifiers in the `msg` string.
- **Control Flow**:
    - Check if `log_file` is NULL; if it is, return immediately without logging.
    - Initialize a `va_list` variable `ap` to handle the variable arguments.
    - Call `log_vwrite` with the message format string, the variable arguments list, and an empty string as the prefix to write the log message to the file.
    - Clean up the `va_list` variable `ap` using `va_end`.
- **Output**:
    - The function does not return any value; it performs logging as a side effect if logging is enabled.


---
### fatal<!-- {{#callable:fatal}} -->
The `fatal` function logs a critical error message with the current error string and terminates the program.
- **Inputs**:
    - `msg`: A format string for the error message to be logged.
    - `...`: Additional arguments for the format string.
- **Control Flow**:
    - Declare a character array `tmp` of size 256 to store the formatted error message prefix.
    - Declare a `va_list` variable `ap` to handle the variable argument list.
    - Use `snprintf` to format the error message prefix with the current error string from `strerror(errno)` into `tmp`. If `snprintf` fails, exit the program with status 1.
    - Initialize the variable argument list `ap` using `va_start`, with `msg` as the last fixed argument.
    - Call `log_vwrite` to log the formatted message using the variable argument list `ap` and the prefix stored in `tmp`.
    - Clean up the variable argument list using `va_end`.
    - Terminate the program with exit status 1.
- **Output**:
    - The function does not return; it logs the error message and terminates the program.


---
### fatalx<!-- {{#callable:fatalx}} -->
The `fatalx` function logs a critical error message and terminates the program.
- **Inputs**:
    - `msg`: A constant character pointer representing the error message to be logged.
    - `...`: A variable argument list that allows additional arguments to be passed for formatting the error message.
- **Control Flow**:
    - Initialize a variable argument list `ap` using `va_start` with `msg` as the last fixed argument.
    - Call `log_vwrite` to log the formatted message with the prefix 'fatal: '.
    - Clean up the variable argument list using `va_end`.
    - Terminate the program by calling `exit(1)`.
- **Output**:
    - This function does not return as it terminates the program execution with an exit status of 1.


