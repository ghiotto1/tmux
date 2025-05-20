# Purpose
This C source code file is part of the tmux terminal multiplexer project and is responsible for managing the automatic renaming of window titles based on the activity within the window. The code provides a specific functionality that revolves around monitoring changes in the active pane of a window and updating the window's name accordingly. The main components of this file include functions for checking if the window name needs to be updated ([`check_window_name`](#check_window_name)), formatting the new window name ([`format_window_name`](#format_window_name)), and parsing the command or shell name to derive a default window name ([`parse_window_name`](#parse_window_name)). The code utilizes event-driven programming to handle timing for name updates, ensuring that the window name is only changed when necessary and at appropriate intervals.

The file defines internal functions and does not expose a public API, indicating that it is intended to be used within the tmux application rather than as a standalone library. The use of static functions like [`name_time_callback`](#name_time_callback) and [`format_window_name`](#format_window_name) suggests that these are helper functions meant to encapsulate specific tasks related to window name management. The code also interacts with other parts of the tmux system, such as options and event handling, to determine when and how to update window names. Overall, this file provides a focused and integral part of the tmux functionality, enhancing user experience by dynamically reflecting the current state of a window in its title.
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
- **Description**: The `format_window_name` is a static function that returns a pointer to a character string. It is used to generate a formatted name for a window based on its properties and a specified format string.
- **Use**: This function is used to create a formatted window name by expanding a format string with window and pane details.


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
    - This function does not return any value.


---
### name_time_expired<!-- {{#callable:name_time_expired}} -->
The `name_time_expired` function checks if the time since the last name change of a window has exceeded a predefined interval.
- **Inputs**:
    - `w`: A pointer to a `struct window` which contains the last name change time and other window-related data.
    - `tv`: A pointer to a `struct timeval` representing the current time.
- **Control Flow**:
    - Calculate the time difference between the current time (`tv`) and the last name change time (`w->name_time`) using `timersub` and store it in `offset`.
    - Check if the seconds part of `offset` is non-zero or if the microseconds part is greater than `NAME_INTERVAL`.
    - If either condition is true, return 0 indicating the name time has expired.
    - If neither condition is true, return the remaining time in microseconds until the name time expires (`NAME_INTERVAL - offset.tv_usec`).
- **Output**:
    - The function returns 0 if the name time has expired, otherwise it returns the remaining time in microseconds until the name time expires.


---
### check_window_name<!-- {{#callable:check_window_name}} -->
The `check_window_name` function checks and updates the name of a window if its active pane has changed and the automatic rename option is enabled.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window whose name is to be checked and potentially updated.
- **Control Flow**:
    - Check if the active pane of the window is NULL; if so, return immediately.
    - Check if the 'automatic-rename' option is enabled for the window; if not, return immediately.
    - Check if the active pane has the PANE_CHANGED flag set; if not, log a debug message and return.
    - Log a debug message indicating the active pane has changed.
    - Get the current time and check if the name change timer has expired using [`name_time_expired`](#name_time_expired); if not expired, set up or queue a timer event and return.
    - If the timer has expired, update the window's name time and delete any existing name event timer.
    - Clear the PANE_CHANGED flag from the active pane's flags.
    - Format the new window name using [`format_window_name`](#format_window_name) and compare it with the current name.
    - If the names differ, log a debug message, update the window's name, redraw the window borders, and update the window status.
    - If the names are the same, log a debug message indicating no change.
    - Free the memory allocated for the new name.
- **Output**:
    - The function does not return a value; it performs operations on the window structure, potentially updating its name and triggering UI updates.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`name_time_expired`](#name_time_expired)
    - [`format_window_name`](#format_window_name)
    - [`window_set_name`](window.c.driver.md#window_set_name)
    - [`server_redraw_window_borders`](server-fn.c.driver.md#server_redraw_window_borders)
    - [`server_status_window`](server-fn.c.driver.md#server_status_window)


---
### default_window_name<!-- {{#callable:default_window_name}} -->
The `default_window_name` function generates a default name for a window based on its active pane's command or shell.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window for which the default name is being generated.
- **Control Flow**:
    - Check if the active pane of the window `w` is `NULL`; if so, return an empty string.
    - Use [`cmd_stringify_argv`](cmd.c.driver.md#cmd_stringify_argv) to convert the active pane's command arguments into a single command string `cmd`.
    - If `cmd` is not `NULL` and not empty, parse the command string using [`parse_window_name`](#parse_window_name) to generate the window name `s`.
    - If `cmd` is `NULL` or empty, use the active pane's shell instead and parse it with [`parse_window_name`](#parse_window_name) to generate the window name `s`.
    - Free the memory allocated for `cmd`.
    - Return the generated window name `s`.
- **Output**:
    - Returns a dynamically allocated string containing the default name for the window, which should be freed by the caller.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`cmd_stringify_argv`](cmd.c.driver.md#cmd_stringify_argv)
    - [`parse_window_name`](#parse_window_name)


---
### format_window_name<!-- {{#callable:format_window_name}} -->
The `format_window_name` function generates a formatted name for a given window using a specified format string.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window for which the name is to be formatted.
- **Control Flow**:
    - Create a format tree `ft` using [`format_create`](format.c.driver.md#format_create) with the window's ID and the `FORMAT_WINDOW` flag.
    - Set default values in the format tree for the window and its active pane using [`format_defaults_window`](format.c.driver.md#format_defaults_window) and [`format_defaults_pane`](format.c.driver.md#format_defaults_pane).
    - Retrieve the format string from the window's options using [`options_get_string`](options.c.driver.md#options_get_string) with the key `automatic-rename-format`.
    - Expand the format string using [`format_expand`](format.c.driver.md#format_expand) to generate the window name.
    - Free the format tree using [`format_free`](format.c.driver.md#format_free).
    - Return the generated window name.
- **Output**:
    - A dynamically allocated string containing the formatted window name, which should be freed by the caller.
- **Functions called**:
    - [`format_create`](format.c.driver.md#format_create)
    - [`format_defaults_window`](format.c.driver.md#format_defaults_window)
    - [`format_defaults_pane`](format.c.driver.md#format_defaults_pane)
    - [`options_get_string`](options.c.driver.md#options_get_string)
    - [`format_expand`](format.c.driver.md#format_expand)
    - [`format_free`](format.c.driver.md#format_free)


---
### parse_window_name<!-- {{#callable:parse_window_name}} -->
The `parse_window_name` function processes a given string to extract and return a simplified window name by removing certain prefixes, trimming spaces, and handling special characters.
- **Inputs**:
    - `in`: A constant character pointer representing the input string to be parsed for a window name.
- **Control Flow**:
    - Duplicate the input string `in` to `copy` and `name` using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - If the first character of `name` is a double quote, increment the pointer to skip it.
    - Terminate the string at the first occurrence of a double quote using `strcspn`.
    - Check if the string starts with 'exec ' and skip this prefix if present.
    - Skip leading spaces and hyphens in the string.
    - Find the first space character in the string and terminate the string at that position if found.
    - If the string is not empty, trim trailing non-alphanumeric and non-punctuation characters.
    - If the string starts with a '/', use `basename` to extract the final component of the path.
    - Duplicate the processed name using [`xstrdup`](xmalloc.c.driver.md#xstrdup) and free the original `copy`.
    - Return the newly allocated and processed name.
- **Output**:
    - A newly allocated string containing the parsed and simplified window name.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


