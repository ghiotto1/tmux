# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file provides functionality for automatically renaming tmux windows based on the active pane's activity. It includes functions to check and update the window name when the active pane changes, using a timer to manage the frequency of these updates. The code defines several static and non-static functions, such as [`check_window_name`](#check_window_name), [`format_window_name`](#format_window_name), and [`parse_window_name`](#parse_window_name), which collectively handle the logic for determining and setting the window's name based on the current command or shell running in the active pane.

The file is not an executable but rather a component of the tmux codebase, likely intended to be compiled and linked with other parts of the project. It does not define public APIs or external interfaces but instead focuses on internal functionality related to window naming. The code uses event-driven programming, leveraging libevent to manage timers and callbacks, ensuring that window names are updated efficiently without unnecessary computation. The use of options and format strings allows for customizable window naming, enhancing the flexibility and user configurability of tmux.
# Imports and Dependencies

---
- `sys/types.h`
- `ctype.h`
- `libgen.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### format_window_name
- **Type**: `static char*`
- **Description**: The `format_window_name` is a static function that returns a pointer to a character string. It is used to generate a formatted name for a given window based on its properties and a specified format string.
- **Use**: This function is used to create a formatted name for a window by utilizing the window's properties and a format string, which is then used for automatic renaming of windows.


# Functions

---
### name_time_callback<!-- {{#callable:name_time_callback}} -->
The `name_time_callback` function logs a debug message indicating that a window's name timer has expired.
- **Inputs**:
    - `fd`: An unused integer representing a file descriptor.
    - `events`: An unused short integer representing event flags.
    - `arg`: A pointer to a `struct window`, representing the window associated with the callback.
- **Control Flow**:
    - The function casts the `arg` parameter to a `struct window` pointer.
    - It logs a debug message with the window's ID to indicate that the name timer has expired.
- **Output**:
    - The function does not return any value.


---
### name_time_expired<!-- {{#callable:name_time_expired}} -->
The `name_time_expired` function checks if a specified time interval has passed since the last recorded name time for a window.
- **Inputs**:
    - `w`: A pointer to a `struct window` which contains the last recorded name time.
    - `tv`: A pointer to a `struct timeval` representing the current time.
- **Control Flow**:
    - Calculate the time difference between the current time (`tv`) and the window's last recorded name time (`w->name_time`) using `timersub` and store it in `offset`.
    - Check if the seconds part of `offset` is non-zero or if the microseconds part of `offset` is greater than `NAME_INTERVAL`.
    - If either condition is true, return 0 indicating the time interval has expired.
    - If neither condition is true, return the remaining microseconds until `NAME_INTERVAL` is reached.
- **Output**:
    - Returns 0 if the time interval has expired, otherwise returns the remaining microseconds until the interval expires.


---
### check_window_name<!-- {{#callable:check_window_name}} -->
The `check_window_name` function checks and updates the name of a window if its active pane has changed and the automatic rename option is enabled.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window whose name is to be checked and potentially updated.
- **Control Flow**:
    - Check if the active pane of the window is NULL; if so, return immediately.
    - Check if the 'automatic-rename' option is enabled for the window; if not, return immediately.
    - Check if the active pane has changed by examining the PANE_CHANGED flag; if not, log a debug message and return.
    - Get the current time and check if the name change interval has expired using [`name_time_expired`](#name_time_expired); if not expired, set up or queue a timer event and return.
    - If the timer event is initialized, delete it to prevent further callbacks.
    - Clear the PANE_CHANGED flag on the active pane to indicate the change has been processed.
    - Format the new window name using [`format_window_name`](#format_window_name) and compare it with the current name.
    - If the names differ, update the window's name, redraw the window borders, and update the window status; otherwise, log that the name has not changed.
    - Free the memory allocated for the new name.
- **Output**:
    - The function does not return a value; it performs operations on the window structure and may update its name, redraw its borders, and update its status.
- **Functions called**:
    - [`name_time_expired`](#name_time_expired)
    - [`format_window_name`](#format_window_name)


---
### default_window_name<!-- {{#callable:default_window_name}} -->
The `default_window_name` function generates a default name for a window based on its active pane's command or shell.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window for which the default name is being generated.
- **Control Flow**:
    - Check if the active pane of the window `w` is NULL; if so, return an empty string.
    - Use `cmd_stringify_argv` to convert the active pane's command arguments into a single string `cmd`.
    - If `cmd` is not NULL and not empty, parse it using [`parse_window_name`](#parse_window_name) to generate the window name `s`.
    - If `cmd` is NULL or empty, use the active pane's shell to generate the window name `s` using [`parse_window_name`](#parse_window_name).
    - Free the memory allocated for `cmd`.
    - Return the generated window name `s`.
- **Output**:
    - A dynamically allocated string containing the default name for the window, which should be freed by the caller.
- **Functions called**:
    - [`parse_window_name`](#parse_window_name)


---
### format_window_name<!-- {{#callable:format_window_name}} -->
The `format_window_name` function generates a formatted name for a given window using a specified format string.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window for which the name is to be formatted.
- **Control Flow**:
    - Create a format tree `ft` using `format_create` with the window's ID and the `FORMAT_WINDOW` flag.
    - Set default values in the format tree for the window and its active pane using `format_defaults_window` and `format_defaults_pane`.
    - Retrieve the format string from the window's options using `options_get_string` with the key `automatic-rename-format`.
    - Expand the format string using `format_expand` to generate the window name.
    - Free the format tree using `format_free`.
    - Return the generated window name.
- **Output**:
    - A dynamically allocated string containing the formatted window name.


---
### parse_window_name<!-- {{#callable:parse_window_name}} -->
The `parse_window_name` function processes a given string to extract and return a simplified window name by removing quotes, leading 'exec', spaces, and non-alphanumeric characters.
- **Inputs**:
    - `in`: A constant character pointer representing the input string to be parsed for a window name.
- **Control Flow**:
    - Duplicate the input string to a modifiable copy.
    - If the string starts with a double quote, skip it.
    - Terminate the string at the first occurrence of a double quote.
    - If the string starts with 'exec ', skip this prefix.
    - Skip leading spaces and hyphens in the string.
    - Terminate the string at the first space character, if any.
    - Trim trailing non-alphanumeric and non-punctuation characters.
    - If the string starts with a '/', use the basename of the string.
    - Duplicate the processed name to a new string.
    - Free the initial copy of the input string.
    - Return the newly allocated string containing the parsed window name.
- **Output**:
    - A newly allocated string containing the parsed window name, which must be freed by the caller.


# Function Declarations (Public API)

---
### name_time_callback<!-- {{#callable_declaration:name_time_callback}} -->
Handle the expiration of a window name timer.
- **Description**: This function is a callback intended to be used with an event loop to handle the expiration of a window name timer. It should be registered as a timer event callback and will be invoked when the timer expires. The function logs a debug message indicating that the timer for the specified window has expired. It is typically used in conjunction with other functions that manage window naming and renaming based on activity or other criteria.
- **Inputs**:
    - `fd`: This parameter is unused in the function and can be ignored.
    - `events`: This parameter is unused in the function and can be ignored.
    - `arg`: A pointer to a 'struct window' object. This must not be null, as it is used to identify the window whose name timer has expired. The caller retains ownership of this pointer.
- **Output**: None
- **See also**: [`name_time_callback`](#name_time_callback)  (Implementation)


---
### name_time_expired<!-- {{#callable_declaration:name_time_expired}} -->
Determine if the name time for a window has expired.
- **Description**: Use this function to check if the time since the last name update for a window has exceeded a predefined interval. It is typically called to decide whether a window's name should be updated based on elapsed time. The function requires a valid window structure and a current time value. It returns a non-zero value if the name time has not yet expired, indicating the remaining time until expiration, or zero if the name time has expired.
- **Inputs**:
    - `w`: A pointer to a 'struct window' representing the window whose name time is being checked. Must not be null.
    - `tv`: A pointer to a 'struct timeval' representing the current time. Must not be null.
- **Output**: Returns an integer: zero if the name time has expired, or the remaining microseconds until expiration if it has not.
- **See also**: [`name_time_expired`](#name_time_expired)  (Implementation)


