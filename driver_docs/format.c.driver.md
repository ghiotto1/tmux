# Purpose
The provided C code is part of the `tmux` project, specifically handling the formatting and expansion of strings with embedded variables and expressions. This code is responsible for parsing and expanding format strings that contain placeholders for various tmux-related data, such as session names, window indices, pane IDs, and more. The format strings can include complex expressions, conditional logic, and loops over sessions, windows, panes, and clients. The code supports a wide range of format modifiers that allow users to manipulate the output, such as quoting, truncating, padding, and applying mathematical operations.

Key components of this code include the `format_tree` structure, which holds the context for the format expansion, and the [`format_expand1`](#format_expand1) function, which performs the actual expansion of the format string. The code also defines a set of callback functions that retrieve specific pieces of information from the tmux environment, such as the current session name or the number of attached clients. Additionally, the code includes mechanisms for handling format jobs, which are asynchronous commands that can be run and their output included in the format string. This file is integral to the customization and dynamic display capabilities of tmux, allowing users to tailor the information shown in their status lines and other outputs.
# Imports and Dependencies

---
- `sys/types.h`
- `sys/wait.h`
- `ctype.h`
- `errno.h`
- `fnmatch.h`
- `libgen.h`
- `math.h`
- `pwd.h`
- `regex.h`
- `stdarg.h`
- `stdlib.h`
- `string.h`
- `time.h`
- `unistd.h`
- `tmux.h`


# Global Variables

---
### format_job_get
- **Type**: ``static char *``
- **Description**: The `format_job_get` function is a static function that returns a pointer to a character string. It is used to find or create a job in a format job tree based on a given command string and format expand state.
- **Use**: This function is used to retrieve or initiate a job for a specific command, expanding the command as necessary and managing the job's lifecycle within the format job tree.


---
### format_expand1
- **Type**: ``char *``
- **Description**: The `format_expand1` variable is a static function pointer that takes a `struct format_expand_state` pointer and a `const char *` as arguments, and returns a `char *`. It is used to expand format strings by replacing placeholders with their corresponding values.
- **Use**: This variable is used to perform the expansion of format strings within the context of a format expansion state.


---
### format_jobs
- **Type**: ``struct format_job_tree``
- **Description**: The `format_jobs` variable is a static instance of a red-black tree data structure, specifically a `struct format_job_tree`, which is used to manage `format_job` entries. Each `format_job` represents a job related to formatting operations, such as expanding key-value pairs in strings.
- **Use**: This variable is used to store and manage `format_job` entries in a red-black tree for efficient insertion, deletion, and lookup operations.


---
### format_upper
- **Type**: ``const char *` array`
- **Description**: The `format_upper` variable is a static constant array of strings, where each index corresponds to an uppercase letter of the alphabet. The array contains string values for certain indices, representing specific format keys, while other indices are set to `NULL`. For example, index 3 ('D') maps to "pane_id", and index 18 ('S') maps to "session_name".
- **Use**: This array is used to map single-character uppercase format keys to their corresponding string representations in the context of formatting operations.


---
### format_lower
- **Type**: ``const char *` array`
- **Description**: The `format_lower` variable is a static constant array of pointers to strings, with each element corresponding to a lowercase letter of the alphabet. Most elements are initialized to `NULL`, except for the element at index 7 (corresponding to 'h'), which is initialized to the string "host_short".
- **Use**: This array is used to map single-character lowercase format specifiers to their corresponding string representations in a formatting system.


---
### format_table
- **Type**: ``static const struct format_table_entry[]``
- **Description**: The `format_table` is a static constant array of `struct format_table_entry` that defines a list of key-value pairs used for expanding format strings in the application. Each entry in the array consists of a key, a type (either `FORMAT_TABLE_STRING` or `FORMAT_TABLE_TIME`), and a callback function that provides the value for the key. This table is used to map format keys to their corresponding values or functions that generate those values.
- **Use**: This variable is used to store and retrieve format keys and their associated values or callback functions for expanding format strings.


# Data Structures

---
### format_job
- **Type**: `struct`
- **Members**:
    - `client`: Pointer to a client structure associated with the format job.
    - `tag`: Unsigned integer tag used to identify the format job.
    - `cmd`: Pointer to a constant character string representing the command to be executed.
    - `expanded`: Pointer to a constant character string representing the expanded command.
    - `last`: Time of the last update or execution of the format job.
    - `out`: Pointer to a character string containing the output of the format job.
    - `updated`: Integer flag indicating if the format job has been updated.
    - `job`: Pointer to a job structure associated with the format job.
    - `status`: Integer status of the format job.
    - `entry`: Red-Black tree entry for managing format jobs in a tree structure.
- **Description**: The `format_job` structure is used to manage and execute format jobs within a system, typically involving the execution of commands and handling their output. It contains pointers to associated client and job structures, as well as fields for managing command strings, output, and status. The structure is designed to be part of a Red-Black tree, allowing efficient management and retrieval of format jobs. The `tag` field uniquely identifies each job, while the `last` and `updated` fields help track the timing and state of the job's execution.


---
### format_entry
- **Type**: `struct`
- **Members**:
    - `key`: A pointer to a character string representing the key in the key-value pair.
    - `value`: A pointer to a character string representing the value in the key-value pair.
    - `time`: A time_t value representing the time associated with the format entry.
    - `cb`: A callback function of type format_cb associated with the format entry.
    - `entry`: An RB_ENTRY macro for integrating the format_entry into a red-black tree.
- **Description**: The `format_entry` structure is used to represent an entry in a format tree, which is part of a system for managing key-value pairs and their associated metadata in a red-black tree. Each entry consists of a key and a value, both represented as strings, a timestamp indicating when the entry was created or last modified, a callback function for additional processing, and a red-black tree entry for efficient storage and retrieval. This structure is integral to the management of dynamic formatting and expansion of strings within the system.


---
### format_type
- **Type**: `enum`
- **Members**:
    - `FORMAT_TYPE_UNKNOWN`: Represents an unknown format type.
    - `FORMAT_TYPE_SESSION`: Represents a session format type.
    - `FORMAT_TYPE_WINDOW`: Represents a window format type.
    - `FORMAT_TYPE_PANE`: Represents a pane format type.
- **Description**: The `format_type` enumeration defines different types of formats that can be used within the application. It includes four possible values: `FORMAT_TYPE_UNKNOWN` for unknown formats, `FORMAT_TYPE_SESSION` for session-related formats, `FORMAT_TYPE_WINDOW` for window-related formats, and `FORMAT_TYPE_PANE` for pane-related formats. This enumeration is likely used to categorize or identify the context in which a particular format is applied.


---
### format_tree
- **Type**: `struct`
- **Members**:
    - `type`: An enum indicating the format type.
    - `c`: A pointer to a client structure.
    - `s`: A pointer to a session structure.
    - `wl`: A pointer to a winlink structure.
    - `w`: A pointer to a window structure.
    - `wp`: A pointer to a window pane structure.
    - `pb`: A pointer to a paste buffer structure.
    - `item`: A pointer to a command queue item structure.
    - `client`: A pointer to a client structure, possibly redundant with 'c'.
    - `flags`: An integer representing various flags.
    - `tag`: An unsigned integer used as a tag.
    - `m`: A mouse event structure.
    - `tree`: A red-black tree of format entries.
- **Description**: The `format_tree` structure is used to manage and store information about various entities in a terminal multiplexer, such as clients, sessions, windows, and panes. It holds pointers to these entities and additional metadata like flags and tags. The structure also includes a red-black tree to manage format entries, which are key-value pairs used for formatting output strings. This structure is central to the formatting system, allowing dynamic and context-sensitive string expansion based on the current state of the multiplexer.


---
### format_expand_state
- **Type**: `struct`
- **Members**:
    - `ft`: A pointer to a `format_tree` structure, representing the format tree associated with this state.
    - `loop`: An unsigned integer representing the current recursion depth or loop count.
    - `time`: A `time_t` value representing the current time for time-based formatting.
    - `tm`: A `struct tm` representing the broken-down time for time-based formatting.
    - `flags`: An integer representing various flags that control the behavior of the format expansion.
- **Description**: The `format_expand_state` structure is used to maintain the state during the expansion of format strings in the tmux application. It holds a reference to the format tree being used, tracks the recursion depth to prevent infinite loops, and stores time-related information for formatting time-based variables. The flags field is used to control specific behaviors during the format expansion process.


---
### format_modifier
- **Type**: `struct`
- **Members**:
    - `modifier`: A character array of size 3 to store the format modifier.
    - `size`: An unsigned integer to store the size of the modifier.
    - `argv`: A pointer to an array of strings representing arguments.
    - `argc`: An integer to store the count of arguments.
- **Description**: The `format_modifier` structure is used to represent a format modifier in the context of string formatting operations. It includes a character array `modifier` to store the modifier itself, a `size` to indicate the length of the modifier, and an array of strings `argv` with its count `argc` to hold any additional arguments associated with the modifier. This structure is likely used in parsing and applying format modifiers to strings in a flexible and extensible manner.


---
### format_table_type
- **Type**: `enum`
- **Members**:
    - `FORMAT_TABLE_STRING`: Represents a string type in the format table.
    - `FORMAT_TABLE_TIME`: Represents a time type in the format table.
- **Description**: The `format_table_type` is an enumeration that defines two possible types for entries in a format table: `FORMAT_TABLE_STRING` and `FORMAT_TABLE_TIME`. These types are used to specify whether a format table entry is expected to be a string or a time value, respectively. This enumeration is part of a larger system for managing and expanding format strings, which are used to dynamically generate output based on various data sources.


---
### format_table_entry
- **Type**: `struct`
- **Members**:
    - `key`: A constant character pointer representing the key in the format table.
    - `type`: An enumeration value indicating the type of the format table entry.
    - `cb`: A callback function pointer for processing the format table entry.
- **Description**: The `format_table_entry` structure is used to define entries in a format table, which are key-value pairs used for expanding format strings. Each entry consists of a key, a type indicating whether the entry is a string or a time, and a callback function that is used to process the entry when expanding format strings. This structure is part of a system that allows dynamic string formatting and expansion based on predefined keys and their associated processing logic.


# Functions

---
### format_job_cmp<!-- {{#callable:format_job_cmp}} -->
Compares two `format_job` structures based on their `tag` and `cmd` values.
- **Inputs**:
    - `fj1`: A pointer to the first `format_job` structure to compare.
    - `fj2`: A pointer to the second `format_job` structure to compare.
- **Control Flow**:
    - First, it checks if the `tag` of `fj1` is less than that of `fj2`, returning -1 if true.
    - Next, it checks if the `tag` of `fj1` is greater than that of `fj2`, returning 1 if true.
    - If both tags are equal, it compares the `cmd` strings of both `format_job` structures using `strcmp`.
- **Output**:
    - Returns -1 if `fj1` is less than `fj2`, 1 if greater, or the result of `strcmp` on their `cmd` values if equal.


---
### format_entry_cmp<!-- {{#callable:format_entry_cmp}} -->
Compares two `format_entry` structures based on their `key` values.
- **Inputs**:
    - `fe1`: A pointer to the first `format_entry` structure to compare.
    - `fe2`: A pointer to the second `format_entry` structure to compare.
- **Control Flow**:
    - The function uses `strcmp` to compare the `key` fields of the two `format_entry` structures.
    - It returns a negative value if `fe1->key` is less than `fe2->key`, a positive value if greater, and zero if they are equal.
- **Output**:
    - An integer indicating the result of the comparison: negative if `fe1->key` is less than `fe2->key`, positive if greater, and zero if they are equal.


---
### format_logging<!-- {{#callable:format_logging}} -->
The `format_logging` function checks if logging is enabled based on the log level and verbosity flags.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains flags indicating the logging verbosity.
- **Control Flow**:
    - The function retrieves the current log level using `log_get_level()`.
    - It checks if the log level is not equal to 0 or if the `FORMAT_VERBOSE` flag is set in the `ft->flags`.
    - The function returns 1 (true) if either condition is met, indicating that logging is enabled; otherwise, it returns 0 (false).
- **Output**:
    - Returns an integer: 1 if logging is enabled, 0 otherwise.


---
### printflike<!-- {{#callable:printflike}} -->
The `printflike` function formats a string using a variable argument list and returns the resulting string.
- **Inputs**:
    - `fmt`: A format string that specifies how to format the output.
    - `ap`: A variable argument list that contains the values to be formatted according to the format string.
- **Control Flow**:
    - The function starts by initializing a variable argument list using `va_start` with the provided format string.
    - It then calls `xvasprintf`, which formats the string according to the format and the variable arguments, storing the result in a dynamically allocated string.
    - Finally, it cleans up the variable argument list with `va_end` and returns the formatted string.
- **Output**:
    - The function returns a pointer to a dynamically allocated string containing the formatted output.


---
### format_job_update<!-- {{#callable:format_job_update}} -->
The `format_job_update` function updates the output of a job based on the input received from an event buffer.
- **Inputs**:
    - `job`: A pointer to a `struct job` that represents the job to be updated.
- **Control Flow**:
    - The function retrieves the job's associated data and the input event buffer.
    - It reads lines from the event buffer until there are no more lines.
    - If a line is read, it updates the job's output and marks it as updated.
    - It logs the updated output for debugging purposes.
    - If the job's status is set and the last update time has changed, it notifies the client of the job's status.
- **Output**:
    - The function does not return a value; it modifies the job's output and internal state directly.


---
### format_job_complete<!-- {{#callable:format_job_complete}} -->
The `format_job_complete` function processes the completion of a job by reading its output from an event buffer and updating the job's status.
- **Inputs**:
    - `job`: A pointer to a `struct job` that represents the job whose completion is being processed.
- **Control Flow**:
    - The function retrieves the associated `format_job` data from the job.
    - It reads a line from the event buffer; if no line is available, it copies the entire buffer content into a new string.
    - It logs the command and the output buffer.
    - If the output buffer is not empty or the job has not been updated, it updates the job's output with the new buffer.
    - If the job's status is set, it notifies the client of the job's status and resets the job's status.
- **Output**:
    - The function does not return a value; it modifies the state of the job and its associated data.


---
### format_job_get<!-- {{#callable:format_job_get}} -->
The `format_job_get` function retrieves or creates a job associated with a specific command in a format tree.
- **Inputs**:
    - `es`: A pointer to a `format_expand_state` structure that holds the current state of format expansion.
    - `cmd`: A string representing the command associated with the job to be retrieved or created.
- **Control Flow**:
    - Checks if the `client` in the `format_tree` is NULL; if so, it uses a global job tree.
    - If the `client` has no jobs, it allocates memory for a new job tree.
    - Creates a `format_job` structure and checks if a job with the same tag and command already exists.
    - If the job does not exist, it allocates memory for a new job and initializes it.
    - Expands the command using [`format_expand1`](#format_expand1) and checks if the expanded command differs from the stored one.
    - If the job needs to be updated or created, it runs the job using `job_run`.
    - Handles job output and status updates based on the job's execution.
- **Output**:
    - Returns a dynamically allocated string containing the output of the job if it exists; otherwise, it returns an empty string.
- **Functions called**:
    - [`format_expand1`](#format_expand1)


---
### format_job_tidy<!-- {{#callable:format_job_tidy}} -->
The `format_job_tidy` function cleans up and removes old jobs from a job tree.
- **Inputs**:
    - `jobs`: A pointer to a `format_job_tree` structure representing the collection of jobs to be tidied.
    - `force`: An integer flag indicating whether to force the removal of jobs regardless of their last execution time.
- **Control Flow**:
    - The current time is retrieved and stored in the variable `now`.
    - A safe iteration is performed over the `format_job_tree` using `RB_FOREACH_SAFE` to allow for safe removal of jobs during iteration.
    - For each job, if `force` is not set and the job's last execution time is within the last hour, the job is skipped.
    - If the job is to be removed, it is logged for debugging purposes.
    - The job's associated resources (like command strings and output) are freed.
    - Finally, the job itself is freed from memory.
- **Output**:
    - The function does not return a value; it modifies the job tree in place by removing old jobs.


---
### format_tidy_jobs<!-- {{#callable:format_tidy_jobs}} -->
The `format_tidy_jobs` function cleans up old format jobs for all clients.
- **Inputs**:
    - None
- **Control Flow**:
    - Calls [`format_job_tidy`](#format_job_tidy) to clean up the global `format_jobs` tree.
    - Iterates over each `client` in the `clients` list using `TAILQ_FOREACH`.
    - For each `client`, if it has jobs, calls [`format_job_tidy`](#format_job_tidy) to clean up its jobs.
- **Output**:
    - The function does not return a value; it performs cleanup operations on format jobs.
- **Functions called**:
    - [`format_job_tidy`](#format_job_tidy)


---
### format_lost_client<!-- {{#callable:format_lost_client}} -->
The `format_lost_client` function cleans up resources associated with a lost client.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client whose resources are to be cleaned up.
- **Control Flow**:
    - The function first checks if the `jobs` member of the `client` structure is not NULL.
    - If `jobs` is not NULL, it calls the [`format_job_tidy`](#format_job_tidy) function to clean up the jobs associated with the client.
    - After handling the jobs, it frees the memory allocated for the `jobs` member of the `client` structure.
- **Output**:
    - The function does not return a value; it performs cleanup operations on the client structure.
- **Functions called**:
    - [`format_job_tidy`](#format_job_tidy)


---
### format_cb_host<!-- {{#callable:format_cb_host}} -->
The `format_cb_host` function retrieves the hostname of the current machine.
- **Inputs**:
    - None
- **Control Flow**:
    - The function declares a character array `host` with a size of `HOST_NAME_MAX + 1` to store the hostname.
    - It calls the `gethostname` function to fill the `host` array with the current hostname.
    - If `gethostname` fails (returns a non-zero value), the function returns an empty string by calling `xstrdup` with an empty string.
    - If `gethostname` succeeds, it returns a duplicate of the hostname string using `xstrdup`.
- **Output**:
    - The function returns a dynamically allocated string containing the hostname of the machine, or an empty string if the hostname could not be retrieved.


---
### format_cb_host_short<!-- {{#callable:format_cb_host_short}} -->
The `format_cb_host_short` function retrieves the hostname of the current machine and returns it without any domain suffix.
- **Inputs**:
    - None
- **Control Flow**:
    - The function starts by declaring a character array `host` to store the hostname.
    - It calls `gethostname` to populate `host` with the current machine's hostname.
    - If `gethostname` fails, it returns an empty string by calling `xstrdup` with an empty string.
    - It then checks if there is a '.' in the hostname using `strchr`.
    - If a '.' is found, it replaces it with a null terminator to truncate the hostname to just the first part.
    - Finally, it returns the hostname (or the truncated version) by calling `xstrdup`.
- **Output**:
    - The function returns a dynamically allocated string containing the short hostname (the part before any domain suffix) or an empty string if the hostname could not be retrieved.


---
### format_cb_pid<!-- {{#callable:format_cb_pid}} -->
The `format_cb_pid` function retrieves the current process ID and formats it as a string.
- **Inputs**:
    - None
- **Control Flow**:
    - The function calls `getpid()` to obtain the current process ID.
    - It then uses `xasprintf` to format the process ID as a string.
    - Finally, it returns the formatted string.
- **Output**:
    - The function returns a pointer to a string containing the current process ID formatted as a decimal number.


---
### format_cb_session_attached_list<!-- {{#callable:format_cb_session_attached_list}} -->
The `format_cb_session_attached_list` function generates a comma-separated list of client names attached to a specific session.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains the session information.
- **Control Flow**:
    - Checks if the session pointer in the `format_tree` is NULL; if so, returns NULL.
    - Creates a new `evbuffer` to hold the client names.
    - Iterates over all clients in the `clients` list using `TAILQ_FOREACH`.
    - For each client, checks if its session matches the session in the `format_tree`.
    - If a match is found, appends the client's name to the `evbuffer`, separating names with a comma.
    - After processing all clients, checks the length of the `evbuffer` and allocates memory for the output string.
    - Copies the contents of the `evbuffer` into the output string and frees the `evbuffer`.
- **Output**:
    - Returns a dynamically allocated string containing the comma-separated list of client names attached to the session, or NULL if no clients are attached.


---
### format_cb_session_alerts<!-- {{#callable:format_cb_session_alerts}} -->
The `format_cb_session_alerts` function generates a formatted string of session alerts based on the active windows.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains the session and window information.
- **Control Flow**:
    - Check if the session pointer in the `format_tree` is NULL; if so, return NULL.
    - Initialize an empty string `alerts` to accumulate alert information.
    - Iterate over each `winlink` in the session's windows using `RB_FOREACH`.
    - For each `winlink`, check if it has alert flags set; if not, continue to the next.
    - Format the index of the `winlink` and append it to the `alerts` string.
    - If the `winlink` has activity, bell, or silence flags, append corresponding symbols to the `alerts` string.
    - Return a duplicated string of the accumulated `alerts`.
- **Output**:
    - Returns a dynamically allocated string containing the formatted alerts for the session's windows, or NULL if the session is not valid.


---
### format_cb_session_stack<!-- {{#callable:format_cb_session_stack}} -->
The `format_cb_session_stack` function generates a string representation of the indices of the current and last windows in a session.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains session and window information.
- **Control Flow**:
    - Check if the session pointer in the `format_tree` is NULL; if so, return NULL.
    - Use `xsnprintf` to format the index of the current window into the `result` string.
    - Iterate over the last windows in the session using `TAILQ_FOREACH`.
    - For each window, format its index into a temporary string `tmp`.
    - If `result` is not empty, append a comma before adding the `tmp` index to `result`.
    - After processing all windows, return a duplicate of the `result` string.
- **Output**:
    - Returns a dynamically allocated string containing the indices of the current window followed by the indices of the last windows, separated by commas.


---
### format_cb_window_stack_index<!-- {{#callable:format_cb_window_stack_index}} -->
The `format_cb_window_stack_index` function calculates and returns the index of a specified `winlink` in the stack of windows for a given session.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current window link and session.
- **Control Flow**:
    - Checks if the `winlink` in the `format_tree` is NULL; if so, returns NULL.
    - Retrieves the session associated with the `winlink`.
    - Initializes an index counter to zero.
    - Iterates through the list of `winlink` structures in the session's last window list, incrementing the index until the specified `winlink` is found.
    - If the `winlink` is not found, returns a string '0'.
    - If found, formats the index as a string and returns it.
- **Output**:
    - Returns a dynamically allocated string representing the index of the specified `winlink` in the session's last window list, or '0' if the `winlink` is not found.


---
### format_cb_window_linked_sessions_list<!-- {{#callable:format_cb_window_linked_sessions_list}} -->
Formats a list of session names linked to a specific window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context, including the window link.
- **Control Flow**:
    - Checks if the `winlink` pointer in the `format_tree` is NULL; if so, returns NULL.
    - Retrieves the `window` associated with the `winlink`.
    - Creates a new `evbuffer` to hold the formatted session names.
    - Iterates over each `winlink` in the `window`'s list of winlinks.
    - If the buffer is not empty, adds a comma before appending the session name.
    - Appends the session name to the buffer.
    - After processing all winlinks, checks the buffer length; if not zero, formats the buffer data into a string.
    - Frees the `evbuffer` and returns the formatted string of session names.
- **Output**:
    - Returns a dynamically allocated string containing a comma-separated list of session names linked to the specified window, or NULL if there are no linked sessions.


---
### format_cb_window_active_sessions<!-- {{#callable:format_cb_window_active_sessions}} -->
The `format_cb_window_active_sessions` function counts and returns the number of active sessions in a specified window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current window and its associated sessions.
- **Control Flow**:
    - Check if the `winlink` pointer in the `format_tree` structure is NULL; if it is, return NULL.
    - Retrieve the `window` associated with the `winlink` from the `format_tree` structure.
    - Iterate over each `winlink` in the `window`'s list of `winlinks` using `TAILQ_FOREACH`.
    - For each `winlink`, check if its `session`'s current window (`curw`) is the same as the current `winlink`; if so, increment the count.
    - After iterating through all `winlinks`, use `xasprintf` to format the count as a string.
    - Return the formatted string containing the count of active sessions.
- **Output**:
    - Returns a dynamically allocated string representing the number of active sessions in the window, or NULL if there are no active sessions.


---
### format_cb_window_active_sessions_list<!-- {{#callable:format_cb_window_active_sessions_list}} -->
The `format_cb_window_active_sessions_list` function generates a comma-separated list of names of active sessions associated with the current window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context, including the current window link.
- **Control Flow**:
    - The function first checks if the `wl` (window link) in the `format_tree` is NULL; if it is, the function returns NULL.
    - If `wl` is not NULL, it retrieves the associated `window` from the `format_tree`.
    - A new `evbuffer` is created to hold the session names; if memory allocation fails, the program exits with an error.
    - The function iterates over each `winlink` in the `window`'s list of links using `TAILQ_FOREACH`.
    - For each `winlink`, it checks if the current window (`curw`) is the same as the `winlink`; if so, it adds the session name to the `evbuffer`, prefixed by a comma if it's not the first entry.
    - After processing all `winlinks`, if the `evbuffer` has content, it allocates memory for the output string and copies the data from the `evbuffer` into it.
    - Finally, the `evbuffer` is freed, and the function returns the constructed string of session names.
- **Output**:
    - The function returns a dynamically allocated string containing a comma-separated list of active session names, or NULL if there are no active sessions or if an error occurs.


---
### format_cb_window_active_clients<!-- {{#callable:format_cb_window_active_clients}} -->
The `format_cb_window_active_clients` function counts the number of active clients associated with the currently focused window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context, including the window link.
- **Control Flow**:
    - Check if the `wl` (window link) in the `format_tree` is NULL; if it is, return NULL.
    - Retrieve the `window` associated with the `wl` from the `format_tree`.
    - Iterate over all `clients` using `TAILQ_FOREACH`.
    - For each `client`, check if it has an associated `session` and if the current window of that session matches the window being processed.
    - If there is a match, increment the count of active clients.
    - After iterating through all clients, format the count as a string and return it.
- **Output**:
    - Returns a dynamically allocated string representing the number of active clients for the current window, or NULL if no clients are active.


---
### format_cb_window_active_clients_list<!-- {{#callable:format_cb_window_active_clients_list}} -->
The `format_cb_window_active_clients_list` function generates a comma-separated list of active client names associated with the currently active window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the active window.
- **Control Flow**:
    - Checks if the `wl` (window link) in the `format_tree` is NULL; if so, it returns NULL.
    - Retrieves the `window` associated with the `wl` from the `format_tree`.
    - Creates a new `evbuffer` to hold the list of active client names.
    - Iterates over all clients in the `clients` list using `TAILQ_FOREACH`.
    - For each client, checks if the client has an associated session and if the client's current window matches the active window.
    - If a match is found, appends the client's name to the `evbuffer`, separating names with commas as needed.
    - After processing all clients, checks the length of the `evbuffer` and, if not empty, allocates a string to hold the result and copies the data from the `evbuffer` into it.
    - Frees the `evbuffer` and returns the resulting string of active client names.
- **Output**:
    - Returns a dynamically allocated string containing a comma-separated list of active client names, or NULL if there are no active clients or if the window link is NULL.


---
### format_cb_window_layout<!-- {{#callable:format_cb_window_layout}} -->
The `format_cb_window_layout` function retrieves the layout of a window, returning either the saved layout or the current layout.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the window whose layout is to be retrieved.
- **Control Flow**:
    - The function first checks if the `window` pointer in the `format_tree` structure is NULL; if it is, the function returns NULL.
    - If the `window` has a saved layout (`saved_layout_root`), it calls `layout_dump` with the saved layout and returns the result.
    - If there is no saved layout, it calls `layout_dump` with the current layout (`layout_root`) and returns that result.
- **Output**:
    - The function returns a pointer to the layout representation of the window, which can be either the saved layout or the current layout, or NULL if the window is not defined.


---
### format_cb_window_visible_layout<!-- {{#callable:format_cb_window_visible_layout}} -->
The `format_cb_window_visible_layout` function retrieves the layout of a window if it is visible.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the window and its layout.
- **Control Flow**:
    - The function first retrieves the `window` pointer from the `format_tree` structure.
    - If the `window` pointer is `NULL`, the function returns `NULL` immediately.
    - If the `window` pointer is valid, it calls the `layout_dump` function with the `layout_root` of the window and returns the result.
- **Output**:
    - The function returns a pointer to the layout representation of the window if it is visible, or `NULL` if the window is not present.


---
### format_cb_start_command<!-- {{#callable:format_cb_start_command}} -->
The `format_cb_start_command` function retrieves the start command of a window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the format context, including the window pane.
- **Control Flow**:
    - The function first retrieves the `window_pane` associated with the `format_tree` structure.
    - If the `window_pane` pointer is `NULL`, the function returns `NULL` immediately.
    - If the `window_pane` is valid, it calls the `cmd_stringify_argv` function with the argument count and argument vector of the pane to convert them into a command string.
    - Finally, it returns the result of the `cmd_stringify_argv` function.
- **Output**:
    - The function returns a pointer to a string representing the start command of the window pane, or `NULL` if the pane is not set.


---
### format_cb_start_path<!-- {{#callable:format_cb_start_path}} -->
The `format_cb_start_path` function retrieves the current working directory of a window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first retrieves the `window_pane` associated with the `format_tree` structure.
    - If the `window_pane` is NULL, the function returns NULL.
    - If the `cwd` (current working directory) of the `window_pane` is NULL, it returns an empty string.
    - If the `cwd` is not NULL, it duplicates the string and returns it.
- **Output**:
    - The function returns a pointer to a string containing the current working directory of the window pane, or NULL if the pane is not set, or an empty string if the current working directory is not set.


---
### format_cb_current_command<!-- {{#callable:format_cb_current_command}} -->
The `format_cb_current_command` function retrieves the current command being executed in a specified window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` is NULL or if the `shell` attribute of `wp` is NULL, returning NULL if either condition is true.
    - It attempts to retrieve the command name associated with the file descriptor (`fd`) and terminal (`tty`) of the `window_pane` using the `osdep_get_name` function.
    - If the command name is NULL or an empty string, it attempts to stringify the command line arguments (`argv`) of the `window_pane` using `cmd_stringify_argv`.
    - If that also fails, it defaults to using the `shell` attribute of the `window_pane`.
    - The command string is then parsed for window name formatting using the `parse_window_name` function, and the resulting value is returned.
- **Output**:
    - Returns a dynamically allocated string containing the parsed window name based on the current command, or NULL if no valid command could be determined.


---
### format_cb_current_path<!-- {{#callable:format_cb_current_path}} -->
The `format_cb_current_path` function retrieves the current working directory of a specified window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` is NULL; if it is, the function returns NULL.
    - If `wp` is valid, it calls `osdep_get_cwd` with the file descriptor of the window pane to retrieve the current working directory.
    - If `cwd` is NULL (indicating an error in retrieving the current directory), the function returns NULL.
    - If the current directory is successfully retrieved, it duplicates the string using `xstrdup` and returns the pointer to the duplicated string.
- **Output**:
    - Returns a pointer to a string containing the current working directory of the window pane, or NULL if an error occurs.


---
### format_cb_history_bytes<!-- {{#callable:format_cb_history_bytes}} -->
Calculates the total memory size used by the history bytes of a given `window_pane`.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains the context for formatting, including the `window_pane`.
- **Control Flow**:
    - Checks if the `window_pane` pointer in the `format_tree` is NULL; if so, returns NULL.
    - Retrieves the `grid` associated with the `window_pane`.
    - Iterates over each line in the grid, calculating the total size of cell data and extended data for each line.
    - Calculates the total size by summing the sizes of all lines and their respective data.
    - Formats the total size into a string and returns it.
- **Output**:
    - Returns a dynamically allocated string representing the total size in bytes of the history data for the `window_pane`, or NULL if the `window_pane` is not set.


---
### format_cb_history_all_bytes<!-- {{#callable:format_cb_history_all_bytes}} -->
The `format_cb_history_all_bytes` function formats and returns a string containing the history statistics of a terminal pane.
- **Inputs**:
    - None
- **Control Flow**:
    - Check if the `window_pane` pointer `wp` is NULL; if it is, return NULL.
    - Retrieve the `grid` associated with the `window_pane`.
    - Calculate the total number of lines in the grid by summing `hsize` and `sy`.
    - Iterate over each line in the grid to accumulate the total number of cells and extended cells.
    - Format the accumulated values into a string using `xasprintf`.
- **Output**:
    - Returns a dynamically allocated string containing the number of lines, size of each line, total cell count, and extended cell count, formatted as 'lines, size_of_each_line, total_cells, size_of_cell_data, total_extended_cells, size_of_extended_data'. If the `window_pane` is NULL, it returns NULL.


---
### format_cb_pane_tabs<!-- {{#callable:format_cb_pane_tabs}} -->
The `format_cb_pane_tabs` function generates a comma-separated string of tab indices for a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` is NULL; if it is, the function returns NULL.
    - It creates a new `evbuffer` to hold the tab indices.
    - A loop iterates over the range of the pane's grid width (`sx`), checking each index to see if it is set in the `tabs` bitmap.
    - If a tab index is set, it appends the index to the `evbuffer`, preceded by a comma if it's not the first index.
    - After the loop, if the `evbuffer` contains data, it allocates a string to hold the formatted output and copies the data from the `evbuffer` into this string.
    - Finally, it frees the `evbuffer` and returns the formatted string.
- **Output**:
    - Returns a dynamically allocated string containing the comma-separated tab indices, or NULL if no tabs are set or if memory allocation fails.


---
### format_cb_pane_fg<!-- {{#callable:format_cb_pane_fg}} -->
The `format_cb_pane_fg` function retrieves the foreground color of a specified window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the format context, including the window pane.
- **Control Flow**:
    - The function first retrieves the `window_pane` associated with the `format_tree` structure.
    - If the `window_pane` is NULL, the function returns NULL immediately.
    - The function then calls `tty_default_colours` to populate a `grid_cell` structure with the default colors for the pane.
    - Finally, it converts the foreground color from the `grid_cell` to a string using `colour_tostring` and returns a duplicate of that string.
- **Output**:
    - The function returns a dynamically allocated string representing the foreground color of the pane, or NULL if the pane does not exist.


---
### format_cb_pane_bg<!-- {{#callable:format_cb_pane_bg}} -->
The `format_cb_pane_bg` function retrieves the background color of a specified window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first retrieves the `window_pane` associated with the `format_tree` structure.
    - If the `window_pane` is NULL, the function returns NULL immediately.
    - The function then calls `tty_default_colours` to populate a `grid_cell` structure with the default colors for the pane.
    - Finally, it converts the background color of the `grid_cell` to a string using `colour_tostring` and returns a duplicate of that string.
- **Output**:
    - The function returns a pointer to a string representing the background color of the pane, or NULL if the pane does not exist.


---
### format_cb_session_group_list<!-- {{#callable:format_cb_session_group_list}} -->
The `format_cb_session_group_list` function generates a comma-separated list of session names within a session group.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains the session and other context needed for formatting.
- **Control Flow**:
    - Check if the session pointer in the `format_tree` is NULL; if so, return NULL.
    - Retrieve the session group associated with the session; if none exists, return NULL.
    - Create a new `evbuffer` to hold the formatted session names; if memory allocation fails, call `fatalx`.
    - Iterate over each session in the session group using `TAILQ_FOREACH`.
    - If the buffer is not empty, prepend a comma before adding the session name.
    - Add the session name to the buffer using `evbuffer_add_printf`.
    - After the loop, check the length of the buffer; if it's not zero, allocate memory for the output string and copy the buffer's data into it.
    - Free the `evbuffer` and return the formatted string.
- **Output**:
    - Returns a dynamically allocated string containing a comma-separated list of session names, or NULL if there are no sessions or an error occurs.


---
### format_cb_session_group_attached_list<!-- {{#callable:format_cb_session_group_attached_list}} -->
The `format_cb_session_group_attached_list` function generates a comma-separated list of client names that are attached to the same session group as the specified session.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains the session information and context for formatting.
- **Control Flow**:
    - Check if the session pointer in the `format_tree` is NULL; if so, return NULL.
    - Retrieve the session group associated with the session; if it is NULL, return NULL.
    - Create a new `evbuffer` to hold the output string; if memory allocation fails, call `fatalx` to terminate the program.
    - Iterate over all clients in the `clients` list using `TAILQ_FOREACH`.
    - For each client, check if it has an associated session; if not, continue to the next client.
    - For each session in the session group, check if it matches the current client's session; if it does, add the client's name to the buffer.
    - If the buffer already contains data, prepend a comma before adding the client's name.
    - After processing all clients, check the length of the buffer; if it is greater than zero, allocate memory for the output string and copy the buffer's contents into it.
    - Free the `evbuffer` and return the generated string.
- **Output**:
    - Returns a dynamically allocated string containing a comma-separated list of client names attached to the same session group, or NULL if no clients are found or if an error occurs.


---
### format_cb_pane_in_mode<!-- {{#callable:format_cb_pane_in_mode}} -->
The `format_cb_pane_in_mode` function counts the number of modes currently active in a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the window pane.
- **Control Flow**:
    - The function first retrieves the `window_pane` associated with the `format_tree` structure.
    - If the `window_pane` is NULL, the function returns NULL immediately.
    - It then iterates over the list of `window_mode_entry` structures associated with the `window_pane`, counting each entry.
    - Finally, it formats the count as a string and returns it.
- **Output**:
    - The function returns a dynamically allocated string representing the number of modes active in the pane, or NULL if the pane is not set.


---
### format_cb_pane_at_top<!-- {{#callable:format_cb_pane_at_top}} -->
The `format_cb_pane_at_top` function checks if a pane is positioned at the top of its window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context.
- **Control Flow**:
    - The function first retrieves the `window_pane` from the `format_tree` structure.
    - If the `window_pane` is NULL, the function returns NULL immediately.
    - It then retrieves the associated `window` from the `window_pane`.
    - The function checks the `pane-border-status` option of the window to determine if the pane's border is at the top.
    - Based on the `pane-border-status`, it sets a flag indicating whether the pane is at the top or not.
    - Finally, it formats the flag as a string and returns it.
- **Output**:
    - The function returns a string representation of a flag (0 or 1) indicating whether the pane is at the top of the window.


---
### format_cb_pane_at_bottom<!-- {{#callable:format_cb_pane_at_bottom}} -->
The `format_cb_pane_at_bottom` function checks if a pane is positioned at the bottom of its window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the associated window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` is NULL; if it is, the function returns NULL.
    - If `wp` is not NULL, it retrieves the associated `window` object.
    - It then retrieves the pane border status option from the window's options.
    - Based on the pane border status, it calculates a flag indicating whether the pane is at the bottom of the window.
    - Finally, it formats the flag as a string and returns it.
- **Output**:
    - The function returns a string representation of an integer (0 or 1) indicating whether the pane is at the bottom of the window.


---
### format_cb_cursor_character<!-- {{#callable:format_cb_cursor_character}} -->
The `format_cb_cursor_character` function retrieves the character at the current cursor position in a specified window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - Check if the `window_pane` pointer in the `format_tree` is NULL; if it is, return NULL.
    - Retrieve the `grid_cell` at the current cursor position (cx, cy) of the `window_pane`.
    - If the `grid_cell` does not have the `GRID_FLAG_PADDING` flag set, format the cell's data into a string and assign it to `value`.
- **Output**:
    - Returns a dynamically allocated string containing the character data from the `grid_cell` at the cursor position, or NULL if the cursor is in a padding cell or if the `window_pane` is NULL.


---
### format_cb_cursor_colour<!-- {{#callable:format_cb_cursor_colour}} -->
The `format_cb_cursor_colour` function retrieves the current cursor color for a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first retrieves the `window_pane` associated with the `format_tree` structure.
    - It checks if the `window_pane` or its `screen` is NULL; if so, it returns NULL.
    - If the `screen`'s cursor color (`ccolour`) is not -1, it returns the string representation of that color.
    - If the `ccolour` is -1, it returns the string representation of the default cursor color (`default_ccolour`).
- **Output**:
    - The function returns a dynamically allocated string representing the cursor color, or NULL if the `window_pane` or its `screen` is not available.


---
### format_cb_mouse_word<!-- {{#callable:format_cb_mouse_word}} -->
The `format_cb_mouse_word` function retrieves the word at the mouse cursor's position in a terminal pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including mouse event data.
- **Control Flow**:
    - Checks if the mouse event in the `format_tree` is valid; if not, returns NULL.
    - Retrieves the `window_pane` associated with the mouse event; if it cannot be found, returns NULL.
    - Determines the mouse cursor's position within the pane; if the position is invalid, returns NULL.
    - Checks if the pane has any active modes; if it does and is not in 'no mode', retrieves the word at the cursor's position using `window_copy_get_word`.
    - If no active modes are present, retrieves the grid associated with the pane and calls [`format_grid_word`](#format_grid_word) to get the word at the cursor's coordinates.
- **Output**:
    - Returns a pointer to a string containing the word at the mouse cursor's position, or NULL if any checks fail.
- **Functions called**:
    - [`format_grid_word`](#format_grid_word)


---
### format_cb_mouse_hyperlink<!-- {{#callable:format_cb_mouse_hyperlink}} -->
The `format_cb_mouse_hyperlink` function retrieves a hyperlink from a mouse event in a terminal window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including mouse event details.
- **Control Flow**:
    - Check if the mouse event in `ft` is valid; if not, return NULL.
    - Retrieve the `window_pane` associated with the mouse event using `cmd_mouse_pane`.
    - If the `window_pane` is NULL, return NULL.
    - Determine the mouse coordinates (x, y) using `cmd_mouse_at`; if it fails, return NULL.
    - Check if the `window_pane` has any active modes; if it does and is not in 'no mode', retrieve the hyperlink using `window_copy_get_hyperlink`.
    - If no modes are active, get the grid from the `window_pane` and call [`format_grid_hyperlink`](#format_grid_hyperlink) to retrieve the hyperlink based on the coordinates.
- **Output**:
    - Returns a pointer to a string containing the hyperlink if found, or NULL if no hyperlink is available.
- **Functions called**:
    - [`format_grid_hyperlink`](#format_grid_hyperlink)


---
### format_cb_mouse_line<!-- {{#callable:format_cb_mouse_line}} -->
The `format_cb_mouse_line` function retrieves the text line at the mouse cursor's position in a specified pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including mouse event data.
- **Control Flow**:
    - Checks if the mouse event in the `format_tree` is valid; if not, returns NULL.
    - Retrieves the `window_pane` associated with the mouse event; if it cannot be found, returns NULL.
    - Determines the mouse cursor's position within the pane; if the position is invalid, returns NULL.
    - Checks if the pane has any active modes; if it does and is not in 'no mode', retrieves the line of text from the pane's copy buffer.
    - If no modes are active, retrieves the line from the pane's grid based on the cursor's Y position.
- **Output**:
    - Returns a pointer to a string containing the text line at the mouse cursor's position, or NULL if any checks fail.
- **Functions called**:
    - [`format_grid_line`](#format_grid_line)


---
### format_cb_mouse_status_line<!-- {{#callable:format_cb_mouse_status_line}} -->
The `format_cb_mouse_status_line` function formats and returns the mouse status line position based on the current mouse event.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including mouse event details.
- **Control Flow**:
    - The function first checks if the mouse event is valid by verifying `ft->m.valid`.
    - It then checks if the client associated with the format tree is not NULL and that the terminal has started.
    - If the `statusat` field is 0 and the mouse's y-coordinate is less than the number of status lines, it assigns the y-coordinate to `y`.
    - If `statusat` is greater than 0 and the mouse's y-coordinate is greater than or equal to `statusat`, it calculates the adjusted y-coordinate by subtracting `statusat` from the mouse's y-coordinate.
    - If neither condition is met, the function returns NULL.
    - Finally, it formats the y-coordinate as a string and returns it.
- **Output**:
    - The function returns a dynamically allocated string representing the y-coordinate of the mouse status line, or NULL if the conditions for a valid status line are not met.


---
### format_cb_mouse_status_range<!-- {{#callable:format_cb_mouse_status_range}} -->
The `format_cb_mouse_status_range` function retrieves the status range of the mouse cursor in a terminal interface.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including mouse event data.
- **Control Flow**:
    - Checks if the mouse event data in `ft` is valid; if not, returns NULL.
    - Checks if the associated client is valid and has started; if not, returns NULL.
    - Determines the coordinates (x, y) based on the mouse status and position.
    - Calls `status_get_range` to retrieve the style range based on the coordinates.
    - Returns NULL if the style range is not found or if its type is `STYLE_RANGE_NONE`.
    - Returns a string representation of the style range type (e.g., 'left', 'right', etc.) based on the retrieved style range.
- **Output**:
    - Returns a dynamically allocated string describing the type of the status range (e.g., 'left', 'right', 'pane', 'window', 'session', or a user-defined string), or NULL if no valid range is found.


---
### format_cb_alternate_on<!-- {{#callable:format_cb_alternate_on}} -->
The `format_cb_alternate_on` function checks if the alternate screen buffer is active for a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `saved_grid` member of the `base` structure within `wp` is not NULL.
    - If `saved_grid` is not NULL, it returns a string '1' indicating that the alternate screen buffer is active.
    - If `saved_grid` is NULL, it returns a string '0' indicating that the alternate screen buffer is not active.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the alternate screen buffer is active, '0' if it is not, or NULL if the window pane is not set.


---
### format_cb_alternate_saved_x<!-- {{#callable:format_cb_alternate_saved_x}} -->
The `format_cb_alternate_saved_x` function retrieves the saved x-coordinate of the cursor from a window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it retrieves the `saved_cx` value from the `base` member of `wp` and formats it as an unsigned integer string using `format_printf`.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string containing the formatted saved x-coordinate if the window pane exists, or NULL if it does not.


---
### format_cb_alternate_saved_y<!-- {{#callable:format_cb_alternate_saved_y}} -->
The `format_cb_alternate_saved_y` function retrieves the saved Y coordinate of a window pane if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it retrieves the `saved_cy` value from the `base` member of `wp` and formats it as an unsigned integer using `format_printf`.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a formatted string containing the saved Y coordinate if the window pane exists, otherwise it returns NULL.


---
### format_cb_buffer_name<!-- {{#callable:format_cb_buffer_name}} -->
The `format_cb_buffer_name` function retrieves the name of a paste buffer if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context, including the paste buffer.
- **Control Flow**:
    - The function checks if the `pb` (paste buffer) member of the `format_tree` structure is not NULL.
    - If `pb` is not NULL, it calls `paste_buffer_name` to get the name of the paste buffer and duplicates the string using `xstrdup`.
    - If `pb` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a duplicated string containing the name of the paste buffer, or NULL if the paste buffer does not exist.


---
### format_cb_buffer_sample<!-- {{#callable:format_cb_buffer_sample}} -->
The `format_cb_buffer_sample` function generates a sample representation of a paste buffer if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context, including a reference to a paste buffer.
- **Control Flow**:
    - The function first checks if the `pb` (paste buffer) member of the `format_tree` structure is not NULL.
    - If `pb` is not NULL, it calls the `paste_make_sample` function with `ft->pb` as an argument to generate a sample.
    - If `pb` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to the generated sample if the paste buffer exists, or NULL if it does not.


---
### format_cb_buffer_size<!-- {{#callable:format_cb_buffer_size}} -->
The `format_cb_buffer_size` function retrieves the size of the paste buffer and formats it as a string.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains the context for formatting, including a reference to the paste buffer.
- **Control Flow**:
    - The function first checks if the paste buffer (`pb`) in the `format_tree` structure is not NULL.
    - If the paste buffer is valid, it calls `paste_buffer_data` to retrieve the size of the data in the paste buffer.
    - The size is then formatted into a string using `format_printf` and returned.
    - If the paste buffer is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string representing the size of the paste buffer, or NULL if the paste buffer is not available.


---
### format_cb_client_cell_height<!-- {{#callable:format_cb_client_cell_height}} -->
The `format_cb_client_cell_height` function retrieves the height of a client's terminal cell in pixels.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client and its terminal.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL and if the `TTY_STARTED` flag is set in the client's terminal flags.
    - If both conditions are met, it calls `format_printf` to format the height of the terminal cell in pixels and returns the result.
    - If the conditions are not met, it returns NULL.
- **Output**:
    - The function returns a pointer to a string containing the height of the client's terminal cell in pixels, or NULL if the conditions are not satisfied.


---
### format_cb_client_cell_width<!-- {{#callable:format_cb_client_cell_width}} -->
The `format_cb_client_cell_width` function retrieves the width of a client's terminal cell in pixels.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client and its terminal.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL and if the `TTY_STARTED` flag is set in the client's terminal flags.
    - If both conditions are met, it calls `format_printf` to format the width of the terminal cell in pixels and returns the result.
    - If the conditions are not met, it returns NULL.
- **Output**:
    - Returns a formatted string representing the width of the client's terminal cell in pixels, or NULL if the conditions are not satisfied.


---
### format_cb_client_control_mode<!-- {{#callable:format_cb_client_control_mode}} -->
The `format_cb_client_control_mode` function checks if a client is in control mode and returns a string representation of that state.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client and its state.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the client exists, it checks if the `flags` field of the client contains the `CLIENT_CONTROL` flag.
    - If the `CLIENT_CONTROL` flag is set, the function returns a duplicated string '1'.
    - If the `CLIENT_CONTROL` flag is not set, it returns a duplicated string '0'.
    - If the client pointer is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the client is in control mode, '0' if not, or NULL if the client does not exist.


---
### format_cb_client_discarded<!-- {{#callable:format_cb_client_discarded}} -->
The `format_cb_client_discarded` function retrieves the number of discarded items for a client and formats it as a string.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client whose discarded count is to be retrieved.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` is valid, it retrieves the `discarded` count from the `client` structure and formats it as a string using `format_printf`.
    - If the `client` is NULL, the function returns NULL.
- **Output**:
    - The function returns a formatted string representing the number of discarded items for the client, or NULL if the client is not set.


---
### format_cb_client_flags<!-- {{#callable:format_cb_client_flags}} -->
The `format_cb_client_flags` function retrieves the flags associated with a client if the client is not NULL.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client whose flags are to be retrieved.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the client is not NULL, it calls the `server_client_get_flags` function to retrieve the client's flags.
    - The flags are then duplicated using `xstrdup` to create a new string that is returned.
    - If the client is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string containing the client's flags if the client exists, or NULL if the client is not present.


---
### format_cb_client_height<!-- {{#callable:format_cb_client_height}} -->
The `format_cb_client_height` function retrieves the height of a client terminal if the client is active.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client and its terminal.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL.
    - It then checks if the `tty` flags of the client indicate that the terminal has started (i.e., `TTY_STARTED` is set).
    - If both conditions are met, it retrieves the `sy` (screen height) from the client's `tty` structure and formats it as an unsigned integer string.
    - If either condition fails, the function returns NULL.
- **Output**:
    - The function returns a formatted string representing the height of the client terminal, or NULL if the client is not active or the terminal has not started.


---
### format_cb_client_key_table<!-- {{#callable:format_cb_client_key_table}} -->
The `format_cb_client_key_table` function retrieves the name of the key table associated with a client.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client and its associated key table.
- **Control Flow**:
    - The function checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` is not NULL, it retrieves the name of the key table from the `keytable` structure associated with the client.
    - If the `client` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a duplicated string containing the name of the key table if the client exists, or NULL if the client does not exist.


---
### format_cb_client_last_session<!-- {{#callable:format_cb_client_last_session}} -->
Retrieves the name of the last session associated with a client if it is alive.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client and session.
- **Control Flow**:
    - Checks if the `client` pointer in the `format_tree` is not NULL.
    - Checks if the `last_session` pointer in the `client` is not NULL.
    - Calls the `session_alive` function to determine if the last session is active.
    - If all checks pass, duplicates the name of the last session and returns it.
    - If any check fails, returns NULL.
- **Output**:
    - Returns a pointer to a string containing the name of the last session if it exists and is alive; otherwise, returns NULL.


---
### format_cb_client_name<!-- {{#callable:format_cb_client_name}} -->
The `format_cb_client_name` function retrieves the name of a client from a `format_tree` structure.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client.
- **Control Flow**:
    - The function checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` pointer is valid, it duplicates the `name` of the client using `xstrdup` and returns it.
    - If the `client` pointer is NULL, the function returns NULL.
- **Output**:
    - Returns a pointer to a duplicated string containing the client's name if the client exists; otherwise, it returns NULL.


---
### format_cb_client_pid<!-- {{#callable:format_cb_client_pid}} -->
The `format_cb_client_pid` function retrieves the process ID of a client if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client.
- **Control Flow**:
    - The function checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` exists, it retrieves the `pid` from the `client` structure and formats it as a long integer string using `format_printf`.
    - If the `client` pointer is NULL, the function returns NULL.
- **Output**:
    - The function returns a formatted string containing the process ID of the client, or NULL if the client does not exist.


---
### format_cb_client_prefix<!-- {{#callable:format_cb_client_prefix}} -->
The `format_cb_client_prefix` function returns a string indicating whether the client's key table matches the server's key table.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client and its associated data.
- **Control Flow**:
    - Check if the `client` pointer in the `format_tree` structure is not NULL.
    - Retrieve the key table name associated with the client using `server_client_get_key_table`.
    - Compare the client's key table name with the retrieved name.
    - If they match, return a duplicated string '0'; otherwise, return '1'.
    - If the `client` pointer is NULL, return NULL.
- **Output**:
    - Returns a duplicated string '0' if the client's key table matches the server's key table name, '1' if it does not, or NULL if the client is not set.


---
### format_cb_client_readonly<!-- {{#callable:format_cb_client_readonly}} -->
The `format_cb_client_readonly` function checks if a client is in read-only mode and returns a string representation of that state.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client and its state.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the client exists, it checks if the `flags` field of the client contains the `CLIENT_READONLY` flag.
    - If the `CLIENT_READONLY` flag is set, it returns a duplicated string '1'.
    - If the flag is not set, it returns a duplicated string '0'.
    - If the client pointer is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the client is read-only, '0' if it is not, or NULL if the client does not exist.


---
### format_cb_client_session<!-- {{#callable:format_cb_client_session}} -->
The `format_cb_client_session` function retrieves the name of the session associated with a client in a format tree.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client and session.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` is not NULL.
    - If the `client` is valid, it then checks if the `session` pointer within the `client` is also not NULL.
    - If both checks pass, it duplicates the session's name using `xstrdup` and returns it.
    - If either check fails, the function returns NULL.
- **Output**:
    - Returns a pointer to a duplicated string containing the session name if the client and session are valid; otherwise, it returns NULL.


---
### format_cb_client_termfeatures<!-- {{#callable:format_cb_client_termfeatures}} -->
The `format_cb_client_termfeatures` function retrieves the terminal features of a client if the client is not null.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client whose terminal features are to be retrieved.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not null.
    - If the `client` is not null, it calls the `tty_get_features` function with the terminal features of the client and duplicates the returned string using `xstrdup`.
    - If the `client` is null, the function returns null.
- **Output**:
    - The function returns a duplicated string of the terminal features if the client is valid, otherwise it returns null.


---
### format_cb_client_termname<!-- {{#callable:format_cb_client_termname}} -->
The `format_cb_client_termname` function retrieves the terminal name of a client from a `format_tree` structure.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client whose terminal name is to be retrieved.
- **Control Flow**:
    - The function checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` pointer is valid, it returns a duplicate string of the `term_name` associated with the client.
    - If the `client` pointer is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string containing the terminal name of the client, or NULL if the client is not set.


---
### format_cb_client_termtype<!-- {{#callable:format_cb_client_termtype}} -->
The `format_cb_client_termtype` function retrieves the terminal type of a client if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client whose terminal type is being retrieved.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` is not NULL, it checks if the `term_type` of the client is NULL.
    - If `term_type` is NULL, it returns an empty string.
    - If `term_type` is not NULL, it duplicates the string and returns it.
    - If the `client` pointer is NULL, the function returns NULL.
- **Output**:
    - The function returns a duplicated string of the terminal type if it exists, an empty string if the terminal type is NULL, or NULL if the client is not set.


---
### format_cb_client_tty<!-- {{#callable:format_cb_client_tty}} -->
The `format_cb_client_tty` function retrieves the terminal name associated with a client.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client.
- **Control Flow**:
    - The function checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` pointer is valid, it returns a duplicate string of the `ttyname` associated with the client.
    - If the `client` pointer is NULL, it returns NULL.
- **Output**:
    - Returns a pointer to a string containing the terminal name if the client is valid; otherwise, it returns NULL.


---
### format_cb_client_uid<!-- {{#callable:format_cb_client_uid}} -->
The `format_cb_client_uid` function retrieves the user ID of the client associated with the given `format_tree` and formats it as a string.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client whose user ID is to be retrieved.
- **Control Flow**:
    - Check if the `client` pointer in the `format_tree` structure is not NULL.
    - If the client is valid, retrieve the user ID using `proc_get_peer_uid` with the client's peer.
    - If the retrieved user ID is valid (not -1), format it as a string using `format_printf` and return the result.
    - If the client is NULL or the user ID is invalid, return NULL.
- **Output**:
    - Returns a formatted string representation of the user ID if successful, or NULL if the client is not valid or the user ID retrieval fails.


---
### format_cb_client_user<!-- {{#callable:format_cb_client_user}} -->
The `format_cb_client_user` function retrieves the username associated with the UID of the peer connected to a client.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client and session.
- **Control Flow**:
    - Checks if the `client` pointer in the `format_tree` structure is not NULL.
    - Retrieves the UID of the peer associated with the client using `proc_get_peer_uid`.
    - If the UID is valid (not -1), it attempts to get the corresponding `passwd` structure using `getpwuid`.
    - If the `passwd` structure is found, it duplicates the username string using `xstrdup` and returns it.
    - If any checks fail, it returns NULL.
- **Output**:
    - Returns a pointer to a string containing the username of the client, or NULL if the username cannot be determined.


---
### format_cb_client_utf8<!-- {{#callable:format_cb_client_utf8}} -->
The `format_cb_client_utf8` function checks if the client associated with a given `format_tree` structure supports UTF-8 encoding.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` is valid, it checks if the `CLIENT_UTF8` flag is set in the `flags` of the client.
    - If the `CLIENT_UTF8` flag is set, it returns a string '1' indicating UTF-8 support.
    - If the flag is not set, it returns a string '0' indicating no UTF-8 support.
    - If the `client` pointer is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string: '1' if UTF-8 is supported, '0' if not, or NULL if the client is not present.


---
### format_cb_client_width<!-- {{#callable:format_cb_client_width}} -->
The `format_cb_client_width` function retrieves the width of the terminal associated with a client.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client whose width is to be retrieved.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` pointer is valid, it retrieves the width from the `tty` structure associated with the client.
    - The width is then formatted as an unsigned integer and returned as a string using the `format_printf` function.
    - If the `client` pointer is NULL, the function returns NULL.
- **Output**:
    - Returns a string representation of the client's terminal width if the client is valid; otherwise, returns NULL.


---
### format_cb_client_written<!-- {{#callable:format_cb_client_written}} -->
The `format_cb_client_written` function retrieves the number of bytes written by a client.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client.
- **Control Flow**:
    - The function checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` is not NULL, it calls `format_printf` to format the number of bytes written by the client.
    - If the `client` is NULL, the function returns NULL.
- **Output**:
    - Returns a formatted string representing the number of bytes written by the client, or NULL if the client is not present.


---
### format_cb_client_theme<!-- {{#callable:format_cb_client_theme}} -->
The `format_cb_client_theme` function returns the theme of a client as a string.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client whose theme is being queried.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the client is valid, it checks the `theme` attribute of the client.
    - Based on the value of the `theme`, it returns a string representation: 'dark' for `THEME_DARK`, 'light' for `THEME_LIGHT`, or NULL for `THEME_UNKNOWN`.
    - If the client pointer is NULL, the function returns NULL.
- **Output**:
    - The function returns a dynamically allocated string representing the client's theme ('dark' or 'light'), or NULL if the theme is unknown or if the client is not set.


---
### format_cb_config_files<!-- {{#callable:format_cb_config_files}} -->
The `format_cb_config_files` function concatenates the names of configuration files into a single comma-separated string.
- **Inputs**:
    - None
- **Control Flow**:
    - Iterates over a global array `cfg_files` containing configuration file names.
    - For each file, calculates its length and reallocates a buffer to hold the concatenated string.
    - Uses `xsnprintf` to append each file name followed by a comma to the buffer.
    - If no files are found, returns an empty string.
    - Removes the trailing comma and returns the final concatenated string.
- **Output**:
    - Returns a dynamically allocated string containing all configuration file names separated by commas, or an empty string if no files are present.


---
### format_cb_cursor_flag<!-- {{#callable:format_cb_cursor_flag}} -->
The `format_cb_cursor_flag` function checks if the cursor mode is enabled for a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `mode` of the `base` structure of `wp` has the `MODE_CURSOR` flag set.
    - If the `MODE_CURSOR` flag is set, it returns a duplicated string '1'.
    - If the `MODE_CURSOR` flag is not set, it returns a duplicated string '0'.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a duplicated string '1' if the cursor mode is enabled, '0' if it is not, or NULL if the `window_pane` is not set.


---
### format_cb_cursor_shape<!-- {{#callable:format_cb_cursor_shape}} -->
The `format_cb_cursor_shape` function returns a string representation of the cursor shape based on the current screen settings.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - Check if the `window_pane` pointer in the `format_tree` is not NULL and if the `screen` pointer within the `window_pane` is also not NULL.
    - If both checks pass, evaluate the `cstyle` of the `screen` to determine the cursor style.
    - Return a string corresponding to the cursor style: 'block', 'underline', 'bar', or 'default' if none match.
    - If the initial checks fail, return NULL.
- **Output**:
    - Returns a dynamically allocated string representing the cursor shape or NULL if the `window_pane` or `screen` is not set.


---
### format_cb_cursor_very_visible<!-- {{#callable:format_cb_cursor_very_visible}} -->
The `format_cb_cursor_very_visible` function checks if the cursor is set to be very visible and returns a string representation of that state.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` is not NULL and if the `screen` pointer in `wp` is also not NULL.
    - If both pointers are valid, it checks if the `mode` of the `screen` includes the `MODE_CURSOR_VERY_VISIBLE` flag.
    - If the cursor is very visible, it returns a duplicated string '1'; otherwise, it returns a duplicated string '0'.
    - If either pointer is NULL, the function returns NULL.
- **Output**:
    - The function returns a duplicated string '1' if the cursor is very visible, '0' if it is not, or NULL if the `format_tree` or its `window_pane` or `screen` is not set.


---
### format_cb_cursor_x<!-- {{#callable:format_cb_cursor_x}} -->
The `format_cb_cursor_x` function retrieves the current x-coordinate of the cursor in a specified window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it retrieves the x-coordinate from `ft->wp->base.cx` and formats it as an unsigned integer string using `format_printf`.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string representing the x-coordinate of the cursor if the window pane is valid; otherwise, it returns NULL.


---
### format_cb_cursor_y<!-- {{#callable:format_cb_cursor_y}} -->
The `format_cb_cursor_y` function retrieves the current y-coordinate of the cursor in a window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it retrieves the y-coordinate from `wp->base.cy` and formats it as an unsigned integer using `format_printf`.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a formatted string containing the y-coordinate of the cursor, or NULL if the window pane is not available.


---
### format_cb_cursor_blinking<!-- {{#callable:format_cb_cursor_blinking}} -->
The `format_cb_cursor_blinking` function checks if the cursor is set to blink and returns a string representation of that state.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` is not NULL and if the `screen` pointer in `wp` is also not NULL.
    - If both pointers are valid, it checks if the `mode` of the `screen` includes the `MODE_CURSOR_BLINKING` flag.
    - If the cursor is blinking, it returns a duplicated string '1'; otherwise, it returns '0'.
    - If either pointer is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the cursor is blinking, '0' if it is not, or NULL if the input structure is invalid.


---
### format_cb_history_limit<!-- {{#callable:format_cb_history_limit}} -->
The `format_cb_history_limit` function retrieves the history limit of a window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it retrieves the `hlimit` (history limit) from the `grid` associated with the window pane and formats it as an unsigned integer string using `format_printf`.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string representing the history limit of the window pane if it exists, or NULL if the window pane is not set.


---
### format_cb_history_size<!-- {{#callable:format_cb_history_size}} -->
The `format_cb_history_size` function retrieves the history size of a specified window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it retrieves the history size from the `grid` structure associated with the window pane and formats it as an unsigned integer string using `format_printf`.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string representing the history size of the window pane if it exists, or NULL if the pane is not available.


---
### format_cb_insert_flag<!-- {{#callable:format_cb_insert_flag}} -->
The `format_cb_insert_flag` function checks if the current window pane is in insert mode and returns a string representation of the flag.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `mode` of the `base` structure of `wp` includes the `MODE_INSERT` flag.
    - If the `MODE_INSERT` flag is set, it returns a duplicated string '1'.
    - If the `MODE_INSERT` flag is not set, it returns a duplicated string '0'.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string that is '1' if the pane is in insert mode, '0' if it is not, or NULL if the pane is not available.


---
### format_cb_keypad_cursor_flag<!-- {{#callable:format_cb_keypad_cursor_flag}} -->
The `format_cb_keypad_cursor_flag` function checks if the keypad cursor mode is enabled for a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `mode` of the `base` structure of `wp` has the `MODE_KCURSOR` flag set.
    - If the `MODE_KCURSOR` flag is set, it returns a duplicated string '1'.
    - If the `MODE_KCURSOR` flag is not set, it returns a duplicated string '0'.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a duplicated string '1' if the keypad cursor mode is enabled, '0' if it is not, or NULL if the `window_pane` is not set.


---
### format_cb_keypad_flag<!-- {{#callable:format_cb_keypad_flag}} -->
The `format_cb_keypad_flag` function checks if the keypad mode is enabled for a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `base.mode` of the window pane has the `MODE_KKEYPAD` flag set.
    - If the `MODE_KKEYPAD` flag is set, it returns a duplicated string '1'.
    - If the flag is not set, it returns a duplicated string '0'.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string that is '1' if the keypad mode is enabled, '0' if it is not, or NULL if the window pane is not set.


---
### format_cb_mouse_all_flag<!-- {{#callable:format_cb_mouse_all_flag}} -->
The `format_cb_mouse_all_flag` function checks if mouse all mode is enabled for a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `base.mode` of the window pane has the `MODE_MOUSE_ALL` flag set.
    - If the `MODE_MOUSE_ALL` flag is set, it returns a string '1' indicating that mouse all mode is enabled.
    - If the flag is not set, it returns a string '0' indicating that mouse all mode is not enabled.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if mouse all mode is enabled, '0' if it is not enabled, or NULL if the window pane is not available.


---
### format_cb_mouse_any_flag<!-- {{#callable:format_cb_mouse_any_flag}} -->
The `format_cb_mouse_any_flag` function checks if any mouse modes are enabled for a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `base.mode` of the window pane includes the `ALL_MOUSE_MODES` flag.
    - If the `ALL_MOUSE_MODES` flag is set, it returns a string '1' indicating that mouse modes are enabled.
    - If the flag is not set, it returns a string '0' indicating that no mouse modes are enabled.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if any mouse modes are enabled, '0' if none are enabled, or NULL if the window pane is not set.


---
### format_cb_mouse_button_flag<!-- {{#callable:format_cb_mouse_button_flag}} -->
The `format_cb_mouse_button_flag` function checks if mouse button mode is enabled for a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `base.mode` of the window pane includes the `MODE_MOUSE_BUTTON` flag.
    - If the `MODE_MOUSE_BUTTON` flag is set, it returns a string '1' indicating that mouse button mode is enabled.
    - If the flag is not set, it returns a string '0' indicating that mouse button mode is not enabled.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if mouse button mode is enabled, '0' if it is not enabled, or NULL if the window pane is not available.


---
### format_cb_mouse_pane<!-- {{#callable:format_cb_mouse_pane}} -->
The `format_cb_mouse_pane` function retrieves the mouse pane associated with a valid mouse event and formats its ID.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including mouse event data.
- **Control Flow**:
    - The function first checks if the mouse event in the `format_tree` is valid.
    - If valid, it calls `cmd_mouse_pane` to retrieve the corresponding `window_pane` based on the mouse event.
    - If a valid `window_pane` is found, it formats the pane's ID using `format_printf` and returns it.
    - If the mouse event is invalid or no pane is found, the function returns NULL.
- **Output**:
    - Returns a formatted string representing the ID of the mouse pane if valid; otherwise, returns NULL.


---
### format_cb_mouse_sgr_flag<!-- {{#callable:format_cb_mouse_sgr_flag}} -->
The `format_cb_mouse_sgr_flag` function checks if mouse SGR mode is enabled for a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `base.mode` of the window pane includes the `MODE_MOUSE_SGR` flag.
    - If the `MODE_MOUSE_SGR` flag is set, the function returns a string '1' indicating that mouse SGR mode is enabled.
    - If the flag is not set, it returns a string '0' indicating that mouse SGR mode is not enabled.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a dynamically allocated string '1' if mouse SGR mode is enabled, '0' if it is not, or NULL if the window pane is not set.


---
### format_cb_mouse_standard_flag<!-- {{#callable:format_cb_mouse_standard_flag}} -->
The `format_cb_mouse_standard_flag` function checks if the mouse standard mode is enabled for a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `base.mode` of the window pane includes the `MODE_MOUSE_STANDARD` flag.
    - If the flag is set, it returns a duplicated string '1', indicating that the mouse standard mode is enabled.
    - If the flag is not set, it returns a duplicated string '0', indicating that the mouse standard mode is not enabled.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string that is '1' if the mouse standard mode is enabled, '0' if it is not, or NULL if the window pane is not set.


---
### format_cb_mouse_utf8_flag<!-- {{#callable:format_cb_mouse_utf8_flag}} -->
The `format_cb_mouse_utf8_flag` function checks if mouse UTF-8 mode is enabled for a given format tree.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `base.mode` of the `wp` includes the `MODE_MOUSE_UTF8` flag.
    - If the `MODE_MOUSE_UTF8` flag is set, it returns a duplicated string '1'.
    - If the flag is not set, it returns a duplicated string '0'.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string indicating whether mouse UTF-8 mode is enabled ('1' for enabled, '0' for disabled), or NULL if the window pane is not set.


---
### format_cb_mouse_x<!-- {{#callable:format_cb_mouse_x}} -->
The `format_cb_mouse_x` function retrieves the x-coordinate of the mouse cursor in a terminal window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including mouse event data.
- **Control Flow**:
    - Checks if the mouse event in the `format_tree` is valid.
    - Retrieves the `window_pane` associated with the mouse event.
    - If the `window_pane` is valid, it checks if the mouse is within the pane and retrieves the x-coordinate.
    - If the mouse is not in a pane, it checks if the terminal is started and retrieves the x-coordinate based on the status line.
    - Returns the x-coordinate as a formatted string or NULL if no valid coordinate is found.
- **Output**:
    - Returns a string representation of the x-coordinate of the mouse cursor or NULL if the coordinate cannot be determined.


---
### format_cb_mouse_y<!-- {{#callable:format_cb_mouse_y}} -->
The `format_cb_mouse_y` function retrieves the Y-coordinate of the mouse cursor in a terminal window.
- **Inputs**:
    - None
- **Control Flow**:
    - The function first checks if the mouse event in the `format_tree` is valid; if not, it returns NULL.
    - It attempts to get the `window_pane` associated with the mouse event using `cmd_mouse_pane`.
    - If a valid `window_pane` is found and the mouse position is valid, it returns the Y-coordinate using `format_printf`.
    - If the mouse event is not in a valid pane, it checks if the terminal is started and calculates the Y-coordinate based on the status line.
    - Finally, if none of the conditions are met, it returns NULL.
- **Output**:
    - The function returns the Y-coordinate of the mouse cursor as a string, or NULL if the conditions for retrieving the Y-coordinate are not met.


---
### format_cb_next_session_id<!-- {{#callable:format_cb_next_session_id}} -->
The `format_cb_next_session_id` function formats and returns the next session ID as a string.
- **Inputs**:
    - None
- **Control Flow**:
    - The function does not contain any control flow statements such as loops or conditionals.
    - It directly calls the `format_printf` function to format the output.
- **Output**:
    - The output is a dynamically allocated string containing the next session ID formatted as '$<next_session_id>'.


---
### format_cb_origin_flag<!-- {{#callable:format_cb_origin_flag}} -->
The `format_cb_origin_flag` function checks if the `window_pane` associated with a `format_tree` has the `MODE_ORIGIN` flag set.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` is not NULL.
    - If `wp` is not NULL, it checks if the `base.mode` of `wp` has the `MODE_ORIGIN` flag set.
    - If the `MODE_ORIGIN` flag is set, it returns a duplicated string '1'.
    - If the `MODE_ORIGIN` flag is not set, it returns a duplicated string '0'.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a duplicated string '1' if the `MODE_ORIGIN` flag is set, '0' if it is not set, or NULL if the `window_pane` is NULL.


---
### format_cb_pane_active<!-- {{#callable:format_cb_pane_active}} -->
The `format_cb_pane_active` function checks if a specific pane is the active pane in its window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane to check.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it then checks if `wp` is equal to the active pane of its associated window.
    - If they are equal, it returns a string '1' indicating that the pane is active.
    - If they are not equal, it returns a string '0' indicating that the pane is not active.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the pane is active, '0' if it is not, or NULL if the pane pointer is NULL.


---
### format_cb_pane_at_left<!-- {{#callable:format_cb_pane_at_left}} -->
The `format_cb_pane_at_left` function checks if a pane is positioned at the leftmost edge of its window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `xoff` property of the `window_pane` is equal to 0.
    - If `xoff` is 0, it returns a duplicated string '1', indicating the pane is at the left edge.
    - If `xoff` is not 0, it returns a duplicated string '0', indicating the pane is not at the left edge.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the pane is at the left edge, '0' if it is not, or NULL if the pane does not exist.


---
### format_cb_pane_at_right<!-- {{#callable:format_cb_pane_at_right}} -->
The `format_cb_pane_at_right` function checks if a pane is positioned at the right edge of its window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` is not NULL.
    - If `wp` is valid, it checks if the sum of `wp->xoff` (the x-offset of the pane) and `wp->sx` (the width of the pane) equals the width of the window (`wp->window->sx`).
    - If the condition is true, it returns a string '1' indicating the pane is at the right edge.
    - If the condition is false, it returns a string '0' indicating the pane is not at the right edge.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the pane is at the right edge of the window, '0' if it is not, or NULL if the pane is not defined.


---
### format_cb_pane_bottom<!-- {{#callable:format_cb_pane_bottom}} -->
The `format_cb_pane_bottom` function calculates and returns the bottom coordinate of a pane in a format tree.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it calculates the bottom coordinate by adding `yoff` (the vertical offset) and `sy` (the height of the pane) and subtracting 1.
    - The calculated value is then formatted as an unsigned integer and returned.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string representing the bottom coordinate of the pane if it exists, or NULL if the pane does not exist.


---
### format_cb_pane_dead<!-- {{#callable:format_cb_pane_dead}} -->
The `format_cb_pane_dead` function checks if a pane is dead based on its file descriptor.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks the file descriptor (`fd`) of the `window_pane`.
    - If the file descriptor is -1, it indicates that the pane is dead, and the function returns a string '1'.
    - If the file descriptor is not -1, it returns a string '0', indicating the pane is alive.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the pane is dead (file descriptor is -1), '0' if it is alive, or NULL if the pane does not exist.


---
### format_cb_pane_dead_signal<!-- {{#callable:format_cb_pane_dead_signal}} -->
The `format_cb_pane_dead_signal` function retrieves the signal name that caused a pane to terminate.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the pane.
- **Control Flow**:
    - The function first retrieves the `window_pane` associated with the `format_tree`.
    - If the `window_pane` is not NULL, it checks if the pane's status is ready and if it was terminated by a signal.
    - If both conditions are met, it retrieves the signal name using `sig2name` and formats it into a string.
    - If the conditions are not met, or if the `window_pane` is NULL, the function returns NULL.
- **Output**:
    - Returns a formatted string containing the name of the signal that caused the pane to terminate, or NULL if the pane is not ready or was not terminated by a signal.


---
### format_cb_pane_dead_status<!-- {{#callable:format_cb_pane_dead_status}} -->
The `format_cb_pane_dead_status` function retrieves the exit status of a dead pane if it is ready.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the pane.
- **Control Flow**:
    - The function first retrieves the `window_pane` associated with the `format_tree` structure.
    - If the `window_pane` is not NULL, it checks if the pane's status is ready and if it has exited.
    - If both conditions are met, it returns the exit status of the pane formatted as a string.
    - If the conditions are not met, or if the `window_pane` is NULL, it returns NULL.
- **Output**:
    - The function returns a string representation of the exit status of the pane if it is ready and has exited; otherwise, it returns NULL.


---
### format_cb_pane_dead_time<!-- {{#callable:format_cb_pane_dead_time}} -->
The `format_cb_pane_dead_time` function retrieves the dead time of a window pane if it is drawn.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the format context, including the window pane.
- **Control Flow**:
    - The function first retrieves the `window_pane` pointer from the `format_tree` structure.
    - If the `window_pane` pointer is not NULL, it checks if the `PANE_STATUSDRAWN` flag is set in the pane's flags.
    - If the flag is set, it returns a pointer to the `dead_time` of the pane.
    - If the flag is not set or the `window_pane` pointer is NULL, it returns NULL.
- **Output**:
    - Returns a pointer to the `dead_time` of the pane if it is drawn; otherwise, returns NULL.


---
### format_cb_pane_format<!-- {{#callable:format_cb_pane_format}} -->
The `format_cb_pane_format` function checks if the format type is a pane and returns '1' if true, otherwise returns '0'.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the format type.
- **Control Flow**:
    - The function first checks if the `type` field of the `format_tree` structure pointed to by `ft` is equal to `FORMAT_TYPE_PANE`.
    - If the condition is true, it returns a duplicated string '1'.
    - If the condition is false, it returns a duplicated string '0'.
- **Output**:
    - The function returns a pointer to a string that is either '1' or '0', indicating whether the format type is a pane.


---
### format_cb_pane_height<!-- {{#callable:format_cb_pane_height}} -->
The `format_cb_pane_height` function retrieves the height of a specified window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it retrieves the height of the pane (`sy`) and formats it as an unsigned integer string using `format_printf`.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string representing the height of the window pane if it exists, otherwise it returns NULL.


---
### format_cb_pane_id<!-- {{#callable:format_cb_pane_id}} -->
The `format_cb_pane_id` function formats and returns the ID of a window pane if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it calls the `format_printf` function to format the pane ID as a string in the format '%%%u'.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a formatted string representing the pane ID if the pane exists, otherwise it returns NULL.


---
### format_cb_pane_index<!-- {{#callable:format_cb_pane_index}} -->
The `format_cb_pane_index` function retrieves the index of a specified window pane and formats it as a string.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is valid, it calls the `window_pane_index` function to retrieve the index of the pane, storing it in the variable `idx`.
    - If `window_pane_index` returns 0 (indicating success), it formats the index as a string using `format_printf` and returns the result.
    - If `wp` is NULL or `window_pane_index` fails, the function returns NULL.
- **Output**:
    - The function returns a formatted string representation of the pane index if successful, or NULL if the pane is not valid or an error occurs.


---
### format_cb_pane_input_off<!-- {{#callable:format_cb_pane_input_off}} -->
The `format_cb_pane_input_off` function checks if the input for a pane is turned off and returns a string representation of the state.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `flags` of the `window_pane` contain the `PANE_INPUTOFF` flag.
    - If the `PANE_INPUTOFF` flag is set, it returns a duplicated string '1'.
    - If the `PANE_INPUTOFF` flag is not set, it returns a duplicated string '0'.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a duplicated string '1' if the pane input is off, '0' if it is on, or NULL if the pane does not exist.


---
### format_cb_pane_unseen_changes<!-- {{#callable:format_cb_pane_unseen_changes}} -->
The `format_cb_pane_unseen_changes` function checks if a pane has unseen changes and returns a string representation of the result.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `flags` of the `window_pane` contain the `PANE_UNSEENCHANGES` flag.
    - If the `PANE_UNSEENCHANGES` flag is set, it returns a duplicated string '1'.
    - If the flag is not set, it returns a duplicated string '0'.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a duplicated string '1' if there are unseen changes in the pane, '0' if there are none, or NULL if the pane does not exist.


---
### format_cb_pane_key_mode<!-- {{#callable:format_cb_pane_key_mode}} -->
The `format_cb_pane_key_mode` function returns a string representation of the key mode for a given pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` (`wp`) in the `format_tree` is not NULL and if its `screen` is also not NULL.
    - If both checks pass, it evaluates the `mode` of the `screen` using a bitwise AND operation with `EXTENDED_KEY_MODES`.
    - Based on the result of the bitwise operation, it uses a switch statement to determine the appropriate string to return: 'Ext 1' for `MODE_KEYS_EXTENDED`, 'Ext 2' for `MODE_KEYS_EXTENDED_2`, or 'VT10x' for any other mode.
    - If the initial checks fail, the function returns NULL.
- **Output**:
    - The function returns a dynamically allocated string representing the key mode, or NULL if the pane or its screen is not available.


---
### format_cb_pane_last<!-- {{#callable:format_cb_pane_last}} -->
The `format_cb_pane_last` function checks if the current pane is the last pane in its window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if `wp` is the first pane in the list of last panes of its associated window.
    - If `wp` is the first pane, it returns a duplicated string '1'.
    - If `wp` is not the first pane, it returns a duplicated string '0'.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a duplicated string '1' if the current pane is the last pane in its window, '0' if it is not, or NULL if the pane is not defined.


---
### format_cb_pane_left<!-- {{#callable:format_cb_pane_left}} -->
The `format_cb_pane_left` function retrieves the x-offset of a window pane if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it calls `format_printf` to format the x-offset of the window pane and returns the result.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a formatted string representing the x-offset of the window pane if it exists, or NULL if the pane does not exist.


---
### format_cb_pane_marked<!-- {{#callable:format_cb_pane_marked}} -->
The `format_cb_pane_marked` function checks if a specific pane is marked and returns a string indicating its status.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane being checked.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the server indicates that a pane is marked using `server_check_marked()` and if the marked pane is the same as the current pane (`marked_pane.wp == ft->wp`).
    - If both conditions are true, it returns a duplicated string '1' indicating the pane is marked.
    - If the conditions are not met, it returns a duplicated string '0' indicating the pane is not marked.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the pane is marked, '0' if it is not marked, or NULL if the pane is not specified.


---
### format_cb_pane_marked_set<!-- {{#callable:format_cb_pane_marked_set}} -->
The `format_cb_pane_marked_set` function checks if a pane is marked and returns a string indicating its status.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it calls the `server_check_marked` function to determine if the pane is marked.
    - If the pane is marked, it returns a duplicated string '1'.
    - If the pane is not marked, it returns a duplicated string '0'.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the pane is marked, '0' if it is not marked, or NULL if the pane does not exist.


---
### format_cb_pane_mode<!-- {{#callable:format_cb_pane_mode}} -->
The `format_cb_pane_mode` function retrieves the name of the first mode associated with a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer in the `format_tree` structure is not NULL.
    - If the `window_pane` is valid, it retrieves the first mode entry from the list of modes associated with the pane.
    - If a mode entry is found, it duplicates the mode's name and returns it.
    - If no mode entry is found or if the `window_pane` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string containing the name of the first mode associated with the pane, or NULL if no mode is found or if the pane is not valid.


---
### format_cb_pane_path<!-- {{#callable:format_cb_pane_path}} -->
The `format_cb_pane_path` function retrieves the file path of a window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `base.path` of the `window_pane` is NULL.
    - If `base.path` is NULL, it returns an empty string by duplicating it.
    - If `base.path` is not NULL, it returns a duplicate of the `base.path` string.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string containing the path of the window pane, an empty string if the path is NULL, or NULL if the window pane itself is NULL.


---
### format_cb_pane_pid<!-- {{#callable:format_cb_pane_pid}} -->
The `format_cb_pane_pid` function retrieves the process ID of a window pane if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it retrieves the `pid` (process ID) from the `wp` structure and formats it as a long integer string using `format_printf`.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string containing the process ID of the window pane, or NULL if the pane does not exist.


---
### format_cb_pane_pipe<!-- {{#callable:format_cb_pane_pipe}} -->
The `format_cb_pane_pipe` function checks if a window pane has a valid pipe file descriptor and returns a string representation of its status.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `pipe_fd` (pipe file descriptor) of the `window_pane` is not equal to -1.
    - If the `pipe_fd` is valid (not -1), it returns a duplicated string '1'.
    - If the `pipe_fd` is -1, it returns a duplicated string '0'.
    - If `wp` is NULL, it returns NULL.
- **Output**:
    - The function returns a duplicated string '1' if the pane has a valid pipe file descriptor, '0' if it does not, or NULL if the pane is not set.


---
### format_cb_pane_right<!-- {{#callable:format_cb_pane_right}} -->
The `format_cb_pane_right` function calculates and returns the rightmost x-coordinate of a pane in a format tree.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it calculates the rightmost x-coordinate by adding the `xoff` (x-offset) and `sx` (width) of the pane, then subtracting 1.
    - The calculated value is formatted as an unsigned integer and returned.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - Returns a pointer to a string representing the rightmost x-coordinate of the pane if it exists, otherwise returns NULL.


---
### format_cb_pane_search_string<!-- {{#callable:format_cb_pane_search_string}} -->
The `format_cb_pane_search_string` function retrieves the search string associated with a given pane in a format tree.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the pane whose search string is being accessed.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` is not NULL.
    - If `wp` is not NULL, it checks if the `searchstr` field of `wp` is NULL.
    - If `searchstr` is NULL, it returns a duplicate of an empty string.
    - If `searchstr` is not NULL, it returns a duplicate of the `searchstr` value.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a dynamically allocated string containing the search string of the pane, an empty string if no search string is set, or NULL if the pane does not exist.


---
### format_cb_pane_synchronized<!-- {{#callable:format_cb_pane_synchronized}} -->
The `format_cb_pane_synchronized` function checks if the current pane is set to synchronize with other panes.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it retrieves the value of the 'synchronize-panes' option from the pane's options.
    - If the option is set to a non-zero value, it returns a string '1', indicating synchronization is enabled.
    - If the option is not set, it returns a string '0', indicating synchronization is disabled.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if pane synchronization is enabled, '0' if it is disabled, or NULL if the pane does not exist.


---
### format_cb_pane_title<!-- {{#callable:format_cb_pane_title}} -->
The `format_cb_pane_title` function retrieves the title of a window pane if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it returns a duplicate of the title string from the `base` structure of the `window_pane`.
    - If `wp` is NULL, it returns NULL.
- **Output**:
    - The function returns a pointer to a duplicated string containing the title of the window pane, or NULL if the pane does not exist.


---
### format_cb_pane_top<!-- {{#callable:format_cb_pane_top}} -->
The `format_cb_pane_top` function retrieves the vertical offset of a window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context.
- **Control Flow**:
    - The function checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it retrieves the `yoff` (vertical offset) of the window pane and formats it as an unsigned integer string using `format_printf`.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string containing the vertical offset of the window pane if it exists, otherwise it returns NULL.


---
### format_cb_pane_tty<!-- {{#callable:format_cb_pane_tty}} -->
The `format_cb_pane_tty` function retrieves the terminal type associated with a specific pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the pane.
- **Control Flow**:
    - The function checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it returns a duplicate string of the `tty` field from the `window_pane` structure.
    - If `wp` is NULL, it returns NULL.
- **Output**:
    - Returns a pointer to a duplicated string containing the terminal type if the pane exists, otherwise returns NULL.


---
### format_cb_pane_width<!-- {{#callable:format_cb_pane_width}} -->
The `format_cb_pane_width` function retrieves the width of a specified pane in a format tree.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the pane whose width is to be retrieved.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it retrieves the width (`sx`) of the pane and formats it as an unsigned integer string using `format_printf`.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string containing the width of the pane as an unsigned integer, or NULL if the pane does not exist.


---
### format_cb_scroll_region_lower<!-- {{#callable:format_cb_scroll_region_lower}} -->
The `format_cb_scroll_region_lower` function retrieves the lower boundary of the scroll region for a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it retrieves the `rlower` value from the `base` structure of the `window_pane` and formats it as an unsigned integer string using `format_printf`.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a formatted string representing the lower boundary of the scroll region if the `window_pane` is valid; otherwise, it returns NULL.


---
### format_cb_scroll_region_upper<!-- {{#callable:format_cb_scroll_region_upper}} -->
The `format_cb_scroll_region_upper` function retrieves the upper scroll region value of a window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it retrieves the `rupper` value from the `base` member of the `wp` structure and formats it as an unsigned integer using `format_printf`.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a formatted string representing the upper scroll region value if the window pane exists, otherwise it returns NULL.


---
### format_cb_server_sessions<!-- {{#callable:format_cb_server_sessions}} -->
The `format_cb_server_sessions` function counts the number of active server sessions.
- **Inputs**:
    - None
- **Control Flow**:
    - The function initializes a session pointer `s` and a counter `n` to zero.
    - It iterates over all sessions using `RB_FOREACH`, incrementing the counter `n` for each session found.
    - Finally, it returns the count of sessions formatted as a string using `format_printf`.
- **Output**:
    - The output is a string representation of the total number of active server sessions.


---
### format_cb_session_attached<!-- {{#callable:format_cb_session_attached}} -->
The `format_cb_session_attached` function retrieves the number of clients attached to a session.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the session.
- **Control Flow**:
    - The function first checks if the session pointer in the `format_tree` structure is not NULL.
    - If the session is valid, it retrieves the `attached` count from the session structure and formats it as an unsigned integer string.
    - If the session pointer is NULL, the function returns NULL.
- **Output**:
    - The function returns a formatted string representing the number of clients attached to the session, or NULL if the session is not valid.


---
### format_cb_session_format<!-- {{#callable:format_cb_session_format}} -->
The `format_cb_session_format` function checks if the format type is a session and returns '1' if true, otherwise returns '0'.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the format type being checked.
- **Control Flow**:
    - The function first checks if the `type` field of the `format_tree` structure pointed to by `ft` is equal to `FORMAT_TYPE_SESSION`.
    - If the condition is true, it returns a duplicated string '1'.
    - If the condition is false, it returns a duplicated string '0'.
- **Output**:
    - The function returns a pointer to a string that is either '1' or '0', indicating whether the format type is a session.


---
### format_cb_session_group<!-- {{#callable:format_cb_session_group}} -->
The `format_cb_session_group` function retrieves the name of the session group associated with a given session.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the session and its associated data.
- **Control Flow**:
    - The function first checks if the session pointer in the `format_tree` structure is not NULL.
    - If the session is valid, it calls `session_group_contains` to check if the session belongs to a session group.
    - If a session group is found, it returns a duplicate of the group's name using `xstrdup`.
    - If no session group is found or the session pointer is NULL, it returns NULL.
- **Output**:
    - The function returns a pointer to a string containing the name of the session group if found, or NULL if the session is not part of a group or if the session is invalid.


---
### format_cb_session_group_attached<!-- {{#callable:format_cb_session_group_attached}} -->
The `format_cb_session_group_attached` function returns the count of sessions attached to a session group.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the session and its associated data.
- **Control Flow**:
    - The function first checks if the `session` pointer in the `format_tree` structure is not NULL.
    - If the session is valid, it attempts to retrieve the corresponding `session_group` using the `session_group_contains` function.
    - If a valid `session_group` is found, it calls `session_group_attached_count` to get the number of attached sessions and formats this count as a string using `format_printf`.
    - If no valid session or session group is found, the function returns NULL.
- **Output**:
    - The function returns a formatted string representing the number of sessions attached to the session group, or NULL if no session group is found.


---
### format_cb_session_group_many_attached<!-- {{#callable:format_cb_session_group_many_attached}} -->
The `format_cb_session_group_many_attached` function checks if a session group contains more than one attached session.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the session and its associated data.
- **Control Flow**:
    - The function first checks if the session pointer in the `format_tree` structure is not NULL.
    - If the session is valid, it checks if the session group contains the session.
    - If the session group is found, it checks the count of attached sessions in that group.
    - If the count of attached sessions is greater than 1, it returns a string '1'.
    - If the count is not greater than 1, it returns a string '0'.
    - If the session is NULL or the session group is not found, it returns NULL.
- **Output**:
    - Returns a string '1' if more than one session is attached to the group, '0' if one or none is attached, or NULL if the session is invalid.


---
### format_cb_session_group_size<!-- {{#callable:format_cb_session_group_size}} -->
The `format_cb_session_group_size` function formats and returns the size of a session group.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the session and its associated data.
- **Control Flow**:
    - The function first checks if the session pointer in the `format_tree` structure is not NULL.
    - If the session is valid, it attempts to retrieve the session group associated with the session using `session_group_contains`.
    - If a valid session group is found, it calls `session_group_count` to get the number of sessions in that group and formats it as an unsigned integer string using `format_printf`.
    - If the session is NULL or no session group is found, the function returns NULL.
- **Output**:
    - The function returns a formatted string representing the size of the session group if successful, or NULL if the session is invalid or no group is found.


---
### format_cb_session_grouped<!-- {{#callable:format_cb_session_grouped}} -->
The `format_cb_session_grouped` function checks if a session is part of a session group and returns a string indicating the result.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the session being checked.
- **Control Flow**:
    - The function first checks if the session pointer `ft->s` is not NULL.
    - If the session is valid, it calls `session_group_contains` to determine if the session is part of a session group.
    - If the session is part of a group, it returns a duplicated string '1'.
    - If the session is not part of a group, it returns a duplicated string '0'.
    - If the session pointer is NULL, it returns NULL.
- **Output**:
    - The function returns a pointer to a string '1' if the session is grouped, '0' if not, or NULL if the session is invalid.


---
### format_cb_session_id<!-- {{#callable:format_cb_session_id}} -->
Formats and returns the session ID from a `format_tree` structure.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains session information.
- **Control Flow**:
    - Checks if the session pointer `s` in the `format_tree` is not NULL.
    - If `s` is not NULL, it calls `format_printf` to format the session ID as a string prefixed with a dollar sign.
    - If `s` is NULL, it returns NULL.
- **Output**:
    - Returns a formatted string representing the session ID if the session exists, otherwise returns NULL.


---
### format_cb_session_many_attached<!-- {{#callable:format_cb_session_many_attached}} -->
The `format_cb_session_many_attached` function checks if a session has more than one attached client and returns a string representation of the result.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains session information.
- **Control Flow**:
    - The function first checks if the `session` pointer in the `format_tree` structure is not NULL.
    - If the session is attached to more than one client, it returns a duplicated string '1'.
    - If the session is attached to one or no clients, it returns a duplicated string '0'.
    - If the session pointer is NULL, it returns NULL.
- **Output**:
    - Returns a string '1' if more than one client is attached to the session, '0' if one or none is attached, or NULL if the session is not defined.


---
### format_cb_session_marked<!-- {{#callable:format_cb_session_marked}} -->
The `format_cb_session_marked` function checks if a session is marked and returns a string indicating its status.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the session being checked.
- **Control Flow**:
    - The function first checks if the session pointer `ft->s` is not NULL.
    - If the session is not NULL, it checks if the server has marked sessions and if the marked session matches the current session.
    - If both conditions are true, it returns a string '1' indicating the session is marked.
    - If the conditions are not met, it returns a string '0' indicating the session is not marked.
    - If the session pointer is NULL, it returns NULL.
- **Output**:
    - The function returns a string '1' if the session is marked, '0' if it is not marked, or NULL if the session pointer is NULL.


---
### format_cb_session_name<!-- {{#callable:format_cb_session_name}} -->
The `format_cb_session_name` function retrieves the name of a session from a `format_tree` structure.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the session.
- **Control Flow**:
    - The function checks if the session pointer `s` within the `format_tree` structure is not NULL.
    - If `s` is not NULL, it duplicates the session name using `xstrdup` and returns it.
    - If `s` is NULL, the function returns NULL.
- **Output**:
    - Returns a pointer to a duplicated string containing the session name if it exists, otherwise returns NULL.


---
### format_cb_session_path<!-- {{#callable:format_cb_session_path}} -->
The `format_cb_session_path` function retrieves the current working directory of a session if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains session information.
- **Control Flow**:
    - The function checks if the session pointer `s` in the `format_tree` structure is not NULL.
    - If `s` is not NULL, it returns a duplicate of the current working directory string `cwd` from the session.
    - If `s` is NULL, it returns NULL.
- **Output**:
    - Returns a pointer to a string containing the current working directory of the session, or NULL if the session does not exist.


---
### format_cb_session_windows<!-- {{#callable:format_cb_session_windows}} -->
The `format_cb_session_windows` function returns the count of windows in a session if the session is not null.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the session.
- **Control Flow**:
    - The function first checks if the session pointer (`ft->s`) is not null.
    - If the session is valid, it calls `winlink_count` to get the number of windows associated with the session and formats this count as a string using `format_printf`.
    - If the session is null, the function returns null.
- **Output**:
    - The function outputs a formatted string representing the number of windows in the session, or null if the session is not present.


---
### format_cb_socket_path<!-- {{#callable:format_cb_socket_path}} -->
The `format_cb_socket_path` function returns a duplicate of the global `socket_path` string.
- **Inputs**:
    - None
- **Control Flow**:
    - The function does not perform any conditional checks or loops.
    - It directly calls `xstrdup` to duplicate the `socket_path` string.
- **Output**:
    - The output is a pointer to a newly allocated string that is a duplicate of the `socket_path`.


---
### format_cb_version<!-- {{#callable:format_cb_version}} -->
The `format_cb_version` function retrieves and duplicates the current version string.
- **Inputs**:
    - None
- **Control Flow**:
    - The function calls `getversion()` to obtain the current version string.
    - It then uses `xstrdup()` to create a duplicate of the version string.
    - Finally, it returns the duplicated string.
- **Output**:
    - The function returns a pointer to a newly allocated string containing the current version.


---
### format_cb_sixel_support<!-- {{#callable:format_cb_sixel_support}} -->
The `format_cb_sixel_support` function returns a string indicating whether sixel support is enabled.
- **Inputs**:
    - None
- **Control Flow**:
    - The function checks if the `ENABLE_SIXEL` macro is defined.
    - If `ENABLE_SIXEL` is defined, it returns a duplicated string '1'.
    - If `ENABLE_SIXEL` is not defined, it returns a duplicated string '0'.
- **Output**:
    - The function outputs a string '1' if sixel support is enabled, otherwise it outputs '0'.


---
### format_cb_active_window_index<!-- {{#callable:format_cb_active_window_index}} -->
The `format_cb_active_window_index` function retrieves the index of the currently active window in a session.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current session and window.
- **Control Flow**:
    - The function first checks if the session pointer `ft->s` is not NULL.
    - If the session is valid, it retrieves the index of the current window using `ft->s->curw->idx` and formats it as an unsigned integer string.
    - If the session is NULL, the function returns NULL.
- **Output**:
    - Returns a pointer to a string containing the index of the active window if the session is valid; otherwise, it returns NULL.


---
### format_cb_last_window_index<!-- {{#callable:format_cb_last_window_index}} -->
Retrieves the index of the last window in a session.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains session information.
- **Control Flow**:
    - Checks if the session pointer in the `format_tree` is not NULL.
    - If the session is valid, retrieves the maximum `winlink` from the session's windows using `RB_MAX`.
    - Formats the index of the retrieved `winlink` using `format_printf` and returns it.
    - If the session is NULL, returns NULL.
- **Output**:
    - Returns a formatted string representing the index of the last window, or NULL if the session is not set.


---
### format_cb_window_active<!-- {{#callable:format_cb_window_active}} -->
The `format_cb_window_active` function checks if the current window link is active and returns a string representation of its state.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window link.
- **Control Flow**:
    - The function first checks if the `wl` (window link) member of the `format_tree` structure is not NULL.
    - If `wl` is not NULL, it checks if `wl` is equal to the current window (`curw`) of its session.
    - If they are equal, it returns a string '1' indicating the window is active.
    - If they are not equal, it returns a string '0' indicating the window is not active.
    - If `wl` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the window is active, '0' if it is not active, or NULL if the window link is not set.


---
### format_cb_window_activity_flag<!-- {{#callable:format_cb_window_activity_flag}} -->
The `format_cb_window_activity_flag` function checks if a window link has the activity flag set and returns a string representation of the result.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window link to check.
- **Control Flow**:
    - The function first checks if the `wl` (window link) member of the `format_tree` structure is not NULL.
    - If `wl` is not NULL, it checks if the `flags` member of `wl` has the `WINLINK_ACTIVITY` flag set.
    - If the `WINLINK_ACTIVITY` flag is set, it returns a string '1' indicating activity.
    - If the flag is not set, it returns a string '0' indicating no activity.
    - If `wl` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the window link has the activity flag set, '0' if it does not, or NULL if the window link is not present.


---
### format_cb_window_bell_flag<!-- {{#callable:format_cb_window_bell_flag}} -->
The `format_cb_window_bell_flag` function checks if the window link has the bell flag set and returns a string representation of the result.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window link.
- **Control Flow**:
    - The function first checks if the `wl` (window link) member of the `format_tree` structure is not NULL.
    - If `wl` is not NULL, it checks if the `flags` member of `wl` contains the `WINLINK_BELL` flag.
    - If the `WINLINK_BELL` flag is set, the function returns a duplicated string '1'.
    - If the `WINLINK_BELL` flag is not set, it returns a duplicated string '0'.
    - If `wl` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string '1' if the bell flag is set, '0' if it is not set, or NULL if the window link is not available.


---
### format_cb_window_bigger<!-- {{#callable:format_cb_window_bigger}} -->
The `format_cb_window_bigger` function checks if the terminal window associated with a client is larger than its offset.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the associated client.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the client is valid, it calls the `tty_window_offset` function to retrieve the window's offset and size.
    - If `tty_window_offset` returns a non-zero value, indicating an error, the function returns a string '1'.
    - If `tty_window_offset` succeeds, the function returns a string '0'.
    - If the client pointer is NULL, the function returns NULL.
- **Output**:
    - The function returns a string indicating whether the window is larger than its offset ('1' for true, '0' for false), or NULL if the client is not set.


---
### format_cb_window_cell_height<!-- {{#callable:format_cb_window_cell_height}} -->
The `format_cb_window_cell_height` function retrieves the height of a window cell in pixels.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the window whose cell height is to be retrieved.
- **Control Flow**:
    - The function checks if the `w` member of the `format_tree` structure is not NULL.
    - If `w` is not NULL, it retrieves the `ypixel` value from the `w` structure, which represents the height of the window cell in pixels.
    - If `w` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string representation of the window cell height in pixels, or NULL if the window is not defined.


---
### format_cb_window_cell_width<!-- {{#callable:format_cb_window_cell_width}} -->
The `format_cb_window_cell_width` function retrieves the width of a window cell in pixels.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the format context, including the window.
- **Control Flow**:
    - The function checks if the `w` member of the `format_tree` structure is not NULL.
    - If `w` is not NULL, it retrieves the `xpixel` value from the `w` structure and formats it as an unsigned integer string using `format_printf`.
    - If `w` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string representing the width of the window cell in pixels, or NULL if the window is not set.


---
### format_cb_window_end_flag<!-- {{#callable:format_cb_window_end_flag}} -->
The `format_cb_window_end_flag` function checks if the given window link is the last one in its session and returns a string indicating this.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window link to be checked.
- **Control Flow**:
    - The function first checks if the `wl` (window link) member of the `format_tree` structure is not NULL.
    - If `wl` is not NULL, it checks if `wl` is the maximum window link in the session's windows using `RB_MAX`.
    - If `wl` is the maximum, it returns a string '1', indicating it is the last window link.
    - If `wl` is not the maximum, it returns a string '0', indicating it is not the last window link.
    - If `wl` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the window link is the last in its session, '0' if it is not, or NULL if the window link is NULL.


---
### format_cb_window_flags<!-- {{#callable:format_cb_window_flags}} -->
The `format_cb_window_flags` function returns a duplicated string of printable window flags if the winlink is not NULL.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the winlink.
- **Control Flow**:
    - The function checks if the `wl` (winlink) member of the `format_tree` structure is not NULL.
    - If `wl` is not NULL, it calls the `window_printable_flags` function with `ft->wl` and a flag value of 1, and duplicates the resulting string using `xstrdup`.
    - If `wl` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a duplicated string of printable window flags if the winlink is present; otherwise, it returns NULL.


---
### format_cb_window_format<!-- {{#callable:format_cb_window_format}} -->
The `format_cb_window_format` function checks if the format type is a window and returns '1' if true, otherwise returns '0'.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the format type.
- **Control Flow**:
    - The function first checks if the `type` field of the `format_tree` structure pointed to by `ft` is equal to `FORMAT_TYPE_WINDOW`.
    - If the condition is true, it returns a duplicated string '1'.
    - If the condition is false, it returns a duplicated string '0'.
- **Output**:
    - The function returns a pointer to a string that is either '1' or '0', indicating whether the format type is a window.


---
### format_cb_window_height<!-- {{#callable:format_cb_window_height}} -->
The `format_cb_window_height` function retrieves the height of a specified window in a format tree.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the window whose height is to be retrieved.
- **Control Flow**:
    - The function first checks if the `w` member of the `format_tree` structure is not NULL.
    - If `w` is not NULL, it retrieves the `sy` member (which represents the height) and formats it as an unsigned integer string using `format_printf`.
    - If `w` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string containing the height of the window if it exists, or NULL if the window is not defined.


---
### format_cb_window_id<!-- {{#callable:format_cb_window_id}} -->
The `format_cb_window_id` function formats and returns the ID of a window in a specific string format.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context, including the window whose ID is to be formatted.
- **Control Flow**:
    - The function first checks if the `window` pointer in the `format_tree` structure is not NULL.
    - If the `window` pointer is valid, it calls the `format_printf` function to format the window ID as a string prefixed with '@'.
    - If the `window` pointer is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a formatted string containing the window ID prefixed with '@', or NULL if the window is not set.


---
### format_cb_window_index<!-- {{#callable:format_cb_window_index}} -->
The `format_cb_window_index` function retrieves the index of a window link from a `format_tree` structure.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window link.
- **Control Flow**:
    - The function first checks if the `wl` (window link) member of the `format_tree` structure is not NULL.
    - If `wl` is not NULL, it calls `format_printf` to format the index of the window link (`wl->idx`) as a string and returns it.
    - If `wl` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string representing the index of the window link if it exists, or NULL if it does not.


---
### format_cb_window_last_flag<!-- {{#callable:format_cb_window_last_flag}} -->
The `format_cb_window_last_flag` function checks if a given window link is the last window in its session.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window link to be checked.
- **Control Flow**:
    - The function first checks if the `wl` (window link) member of the `format_tree` structure is not NULL.
    - If `wl` is not NULL, it checks if `wl` is the first element in the list of last windows of its session.
    - If it is the first element, the function returns a string '1'.
    - If it is not the first element, the function returns a string '0'.
    - If `wl` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the window link is the last in its session, '0' if it is not, or NULL if the window link is NULL.


---
### format_cb_window_linked<!-- {{#callable:format_cb_window_linked}} -->
The `format_cb_window_linked` function checks if a specific window is linked to any session and returns a string indicating the result.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the window link to be checked.
- **Control Flow**:
    - The function first checks if the `wl` (window link) in the `format_tree` is not NULL.
    - If `wl` is not NULL, it iterates over all sessions using `RB_FOREACH`.
    - For each session, it iterates over the window links associated with that session.
    - If a matching window link is found, it checks if it has already been found before.
    - If it has been found before, it returns a string '1'.
    - If it is the first match, it sets the `found` flag to 1 and continues searching.
    - If no matching window link is found after checking all sessions, it returns '0'.
    - If `wl` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the window is linked to any session, '0' if it is not linked, or NULL if the input `wl` is NULL.


---
### format_cb_window_linked_sessions<!-- {{#callable:format_cb_window_linked_sessions}} -->
The `format_cb_window_linked_sessions` function counts the number of sessions linked to a specific window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context, including the window link.
- **Control Flow**:
    - The function first checks if the `wl` (window link) in the `format_tree` is NULL; if it is, the function returns NULL.
    - It retrieves the `window` associated with the `format_tree`.
    - The function iterates over all session groups and counts how many sessions in each group are linked to the specified window.
    - It then iterates over all sessions and counts how many of them are linked to the specified window, excluding those that belong to a session group.
    - Finally, it returns the count of linked sessions formatted as a string.
- **Output**:
    - The function returns a string representation of the number of sessions linked to the specified window.


---
### format_cb_window_marked_flag<!-- {{#callable:format_cb_window_marked_flag}} -->
The `format_cb_window_marked_flag` function checks if a specific window link is marked and returns a string representation of the flag.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context.
- **Control Flow**:
    - The function first checks if the `wl` (window link) member of the `format_tree` structure is not NULL.
    - If `wl` is not NULL, it checks if the server has a marked pane and if the marked pane's window link matches the current window link.
    - If both conditions are true, it returns a string '1' indicating the window is marked.
    - If the conditions are not met, it returns a string '0' indicating the window is not marked.
    - If `wl` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the window is marked, '0' if it is not marked, or NULL if the window link is not set.


---
### format_cb_window_name<!-- {{#callable:format_cb_window_name}} -->
The `format_cb_window_name` function retrieves the name of a window from a `format_tree` structure.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the window.
- **Control Flow**:
    - The function checks if the `window` pointer in the `format_tree` structure is not NULL.
    - If the `window` pointer is valid, it calls `format_printf` to format the window's name as a string.
    - If the `window` pointer is NULL, the function returns NULL.
- **Output**:
    - Returns a formatted string containing the name of the window if it exists, otherwise returns NULL.


---
### format_cb_window_offset_x<!-- {{#callable:format_cb_window_offset_x}} -->
The `format_cb_window_offset_x` function retrieves the x-offset of the terminal window associated with a given format tree.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the associated client.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` is valid, it calls the `tty_window_offset` function to retrieve the window offsets (ox, oy) and sizes (sx, sy).
    - If `tty_window_offset` returns a non-zero value, indicating success, it formats the x-offset (ox) as an unsigned integer and returns it.
    - If any checks fail, the function returns NULL.
- **Output**:
    - The function returns a formatted string representing the x-offset of the terminal window if successful, or NULL if the client is not set or if the offset retrieval fails.


---
### format_cb_window_offset_y<!-- {{#callable:format_cb_window_offset_y}} -->
The `format_cb_window_offset_y` function retrieves the vertical offset of a terminal window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` is valid, it calls the `tty_window_offset` function to retrieve the offsets (ox, oy) and sizes (sx, sy) of the terminal window.
    - If `tty_window_offset` returns a non-zero value, indicating success, it formats the vertical offset (oy) as a string and returns it.
    - If any of the checks fail, the function returns NULL.
- **Output**:
    - The function returns a formatted string representing the vertical offset of the terminal window if successful, or NULL if the `client` is NULL or if the offset retrieval fails.


---
### format_cb_window_panes<!-- {{#callable:format_cb_window_panes}} -->
The `format_cb_window_panes` function returns the number of panes in a specified window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the window whose panes are to be counted.
- **Control Flow**:
    - The function first checks if the `window` pointer in the `format_tree` structure is not NULL.
    - If the `window` pointer is valid, it calls the `window_count_panes` function to get the number of panes and formats it as an unsigned integer string using `format_printf`.
    - If the `window` pointer is NULL, the function returns NULL.
- **Output**:
    - The function returns a formatted string representing the number of panes in the window, or NULL if the window is not set.


---
### format_cb_window_raw_flags<!-- {{#callable:format_cb_window_raw_flags}} -->
The `format_cb_window_raw_flags` function retrieves the raw flags of a window link in a format tree.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context, including the window link.
- **Control Flow**:
    - The function first checks if the `wl` (window link) member of the `format_tree` structure is not NULL.
    - If `wl` is not NULL, it calls the `window_printable_flags` function with `ft->wl` and a flag value of 1 to get the printable representation of the window flags.
    - The result of `window_printable_flags` is duplicated using `xstrdup` and returned.
    - If `wl` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string containing the printable flags of the window link if it exists, or NULL if it does not.


---
### format_cb_window_silence_flag<!-- {{#callable:format_cb_window_silence_flag}} -->
The `format_cb_window_silence_flag` function checks if a window link is in silence mode and returns a corresponding string.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window link to check.
- **Control Flow**:
    - The function first checks if the `wl` (window link) member of the `format_tree` structure is not NULL.
    - If `wl` is not NULL, it checks if the `flags` member of `wl` contains the `WINLINK_SILENCE` flag.
    - If the `WINLINK_SILENCE` flag is set, the function returns a duplicated string '1'.
    - If the `WINLINK_SILENCE` flag is not set, it returns a duplicated string '0'.
    - If `wl` is NULL, the function returns NULL.
- **Output**:
    - The function returns a duplicated string '1' if the window link is in silence mode, '0' if it is not, or NULL if the window link is not present.


---
### format_cb_window_start_flag<!-- {{#callable:format_cb_window_start_flag}} -->
The `format_cb_window_start_flag` function checks if a given window link is the first in its session and returns a string indicating this.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window link to be checked.
- **Control Flow**:
    - The function first checks if the `wl` (window link) member of the `format_tree` structure is not NULL.
    - If `wl` is not NULL, it checks if `wl` is the minimum window link in the session's windows using `RB_MIN`.
    - If `wl` is the first window link, it returns a duplicated string '1'.
    - If `wl` is not the first window link, it returns a duplicated string '0'.
    - If `wl` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string: '1' if the window link is the first in its session, '0' if it is not, or NULL if the window link is NULL.


---
### format_cb_window_width<!-- {{#callable:format_cb_window_width}} -->
The `format_cb_window_width` function retrieves the width of a specified window in a format tree.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the window whose width is to be retrieved.
- **Control Flow**:
    - The function checks if the `w` member of the `format_tree` structure is not NULL.
    - If `w` is not NULL, it retrieves the `sx` member (which represents the width) and formats it as an unsigned integer string using `format_printf`.
    - If `w` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string containing the width of the window if it exists, or NULL if the window is not defined.


---
### format_cb_window_zoomed_flag<!-- {{#callable:format_cb_window_zoomed_flag}} -->
The `format_cb_window_zoomed_flag` function checks if a window is zoomed and returns a string representation of the zoomed state.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window to check.
- **Control Flow**:
    - The function first checks if the `window` pointer in the `format_tree` structure is not NULL.
    - If the `window` is not NULL, it checks if the `flags` field of the `window` contains the `WINDOW_ZOOMED` flag.
    - If the `WINDOW_ZOOMED` flag is set, it returns a duplicated string '1'.
    - If the `WINDOW_ZOOMED` flag is not set, it returns a duplicated string '0'.
    - If the `window` pointer is NULL, it returns NULL.
- **Output**:
    - The function returns a pointer to a string that is '1' if the window is zoomed, '0' if it is not zoomed, or NULL if the window pointer is NULL.


---
### format_cb_wrap_flag<!-- {{#callable:format_cb_wrap_flag}} -->
The `format_cb_wrap_flag` function checks if the wrap mode is enabled for a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `mode` member of the `base` structure of `wp` has the `MODE_WRAP` flag set.
    - If the `MODE_WRAP` flag is set, it returns a string '1' indicating that wrapping is enabled.
    - If the `MODE_WRAP` flag is not set, it returns a string '0' indicating that wrapping is disabled.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if wrap mode is enabled, '0' if it is disabled, or NULL if the window pane is not set.


---
### format_cb_buffer_created<!-- {{#callable:format_cb_buffer_created}} -->
The `format_cb_buffer_created` function retrieves the creation time of a paste buffer.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the paste buffer.
- **Control Flow**:
    - The function checks if the `pb` (paste buffer) member of the `format_tree` structure is not NULL.
    - If the paste buffer is valid, it clears the `tv` (time value) structure and sets its `tv_sec` member to the result of the `paste_buffer_created` function, which retrieves the creation time of the paste buffer.
    - The function then returns a pointer to the `tv` structure.
    - If the paste buffer is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a `struct timeval` containing the creation time of the paste buffer if it exists, or NULL if the paste buffer is not set.


---
### format_cb_client_activity<!-- {{#callable:format_cb_client_activity}} -->
Retrieves the activity time of a client if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client.
- **Control Flow**:
    - The function checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` pointer is valid, it returns a pointer to the `activity_time` field of the client.
    - If the `client` pointer is NULL, it returns NULL.
- **Output**:
    - Returns a pointer to the `activity_time` of the client if it exists, otherwise returns NULL.


---
### format_cb_client_created<!-- {{#callable:format_cb_client_created}} -->
Retrieves the creation time of a client if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client.
- **Control Flow**:
    - The function checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` pointer is not NULL, it returns a pointer to the `creation_time` of the client.
    - If the `client` pointer is NULL, it returns NULL.
- **Output**:
    - Returns a pointer to the `creation_time` of the client if it exists, otherwise returns NULL.


---
### format_cb_session_activity<!-- {{#callable:format_cb_session_activity}} -->
The `format_cb_session_activity` function retrieves the activity time of a session if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains session information.
- **Control Flow**:
    - The function checks if the session pointer (`s`) within the `format_tree` structure is not NULL.
    - If the session exists, it returns a pointer to the `activity_time` field of the session.
    - If the session does not exist, it returns NULL.
- **Output**:
    - Returns a pointer to the `activity_time` of the session if it exists, otherwise returns NULL.


---
### format_cb_session_created<!-- {{#callable:format_cb_session_created}} -->
Retrieves the creation time of a session if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains session information.
- **Control Flow**:
    - Checks if the session pointer in the `format_tree` structure is not NULL.
    - If the session exists, it returns a pointer to the `creation_time` field of the session.
    - If the session does not exist, it returns NULL.
- **Output**:
    - Returns a pointer to the `creation_time` of the session if it exists, otherwise returns NULL.


---
### format_cb_session_last_attached<!-- {{#callable:format_cb_session_last_attached}} -->
The `format_cb_session_last_attached` function retrieves the last attached time of a session if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the session.
- **Control Flow**:
    - The function first checks if the session pointer `s` within the `format_tree` structure `ft` is not NULL.
    - If `ft->s` is not NULL, it returns a pointer to the `last_attached_time` field of the session.
    - If `ft->s` is NULL, it returns NULL.
- **Output**:
    - The function returns a pointer to the `last_attached_time` of the session if it exists, otherwise it returns NULL.


---
### format_cb_start_time<!-- {{#callable:format_cb_start_time}} -->
The `format_cb_start_time` function returns a pointer to the global variable `start_time`.
- **Inputs**:
    - None
- **Control Flow**:
    - The function does not contain any control flow statements such as loops or conditionals.
    - It directly returns the address of the `start_time` variable.
- **Output**:
    - The function outputs a pointer to the `start_time` variable, which is expected to be of type `time_t`.


---
### format_cb_window_activity<!-- {{#callable:format_cb_window_activity}} -->
The `format_cb_window_activity` function retrieves the activity time of a window if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window.
- **Control Flow**:
    - The function first checks if the `window` pointer in the `format_tree` structure is not NULL.
    - If the `window` pointer is valid, it returns a pointer to the `activity_time` field of the `window` structure.
    - If the `window` pointer is NULL, it returns NULL.
- **Output**:
    - Returns a pointer to the `activity_time` of the window if it exists, otherwise returns NULL.


---
### format_cb_buffer_mode_format<!-- {{#callable:format_cb_buffer_mode_format}} -->
The `format_cb_buffer_mode_format` function returns a duplicate of the default format string for the window buffer mode.
- **Inputs**:
    - None
- **Control Flow**:
    - The function does not contain any control flow statements such as loops or conditionals.
    - It directly returns the result of `xstrdup` applied to `window_buffer_mode.default_format`.
- **Output**:
    - The output is a dynamically allocated string that is a duplicate of the default format string for the window buffer mode.


---
### format_cb_client_mode_format<!-- {{#callable:format_cb_client_mode_format}} -->
The `format_cb_client_mode_format` function returns a duplicate of the default format string for the client mode.
- **Inputs**:
    - None
- **Control Flow**:
    - The function does not contain any control flow statements such as loops or conditionals.
    - It directly calls `xstrdup` to duplicate the default format string.
- **Output**:
    - The function outputs a pointer to a newly allocated string that contains the default format for the client mode.


---
### format_cb_tree_mode_format<!-- {{#callable:format_cb_tree_mode_format}} -->
The `format_cb_tree_mode_format` function returns a duplicate of the default format string for the tree mode.
- **Inputs**:
    - None
- **Control Flow**:
    - The function does not contain any control flow statements such as loops or conditionals.
    - It directly calls `xstrdup` to duplicate the default format string.
- **Output**:
    - The output is a pointer to a newly allocated string that contains the default format for the tree mode.


---
### format_cb_uid<!-- {{#callable:format_cb_uid}} -->
The `format_cb_uid` function retrieves the user ID of the calling process and formats it as a string.
- **Inputs**:
    - None
- **Control Flow**:
    - The function calls `getuid()` to obtain the user ID of the current process.
    - It then formats this user ID as a long integer string using `format_printf`.
- **Output**:
    - The output is a string representation of the user ID, formatted as a long integer.


---
### format_cb_user<!-- {{#callable:format_cb_user}} -->
The `format_cb_user` function retrieves the username associated with the current user ID.
- **Inputs**:
    - None
- **Control Flow**:
    - The function begins by declaring a pointer to a `struct passwd` named `pw`.
    - It calls `getpwuid(getuid())` to retrieve the password record for the current user ID.
    - If the call to `getpwuid` is successful (i.e., `pw` is not NULL`), it duplicates the username using `xstrdup` and returns it.
    - If the call fails, it returns NULL.
- **Output**:
    - The function returns a pointer to a string containing the username of the current user, or NULL if the username could not be retrieved.


---
### format_table_compare<!-- {{#callable:format_table_compare}} -->
Compares a key with a format table entry's key.
- **Inputs**:
    - `key0`: A pointer to the key string to compare.
    - `entry0`: A pointer to a `format_table_entry` structure containing the key to compare against.
- **Control Flow**:
    - The function casts the input pointers to appropriate types: `key0` to a `const char*` and `entry0` to a `const struct format_table_entry*`.
    - It then uses the `strcmp` function to compare the `key` with the `entry->key` from the `format_table_entry` structure.
    - The result of the comparison is returned, which indicates whether the keys are equal, less than, or greater than each other.
- **Output**:
    - An integer result from `strcmp`, where 0 indicates the keys are equal, a negative value indicates `key` is less than `entry->key`, and a positive value indicates `key` is greater than `entry->key`.


---
### format_table_get<!-- {{#callable:format_table_get}} -->
The `format_table_get` function retrieves a pointer to a `format_table_entry` structure based on a given key.
- **Inputs**:
    - `key`: A constant character pointer representing the key to search for in the format table.
- **Control Flow**:
    - The function uses `bsearch` to perform a binary search on the `format_table` array.
    - It searches for the `key` in the `format_table`, which is an array of `format_table_entry` structures.
    - The search is conducted using the `format_table_compare` function to compare the keys.
- **Output**:
    - The function returns a pointer to the `format_table_entry` that matches the provided key, or NULL if no match is found.


---
### format_merge<!-- {{#callable:format_merge}} -->
The `format_merge` function merges format entries from one `format_tree` into another.
- **Inputs**:
    - `ft`: A pointer to the destination `format_tree` where entries will be merged.
    - `from`: A pointer to the source `format_tree` from which entries will be taken.
- **Control Flow**:
    - Iterates over each `format_entry` in the `from` tree using `RB_FOREACH`.
    - For each entry, checks if the `value` is not NULL.
    - If the value is valid, calls [`format_add`](#format_add) to add the key-value pair to the destination `format_tree`.
- **Output**:
    - The function does not return a value; it modifies the `ft` tree by adding entries from the `from` tree.
- **Functions called**:
    - [`format_add`](#format_add)


---
### format_get_pane<!-- {{#callable:format_get_pane}} -->
The `format_get_pane` function retrieves the `window_pane` associated with a given `format_tree`.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains the context from which the `window_pane` is to be retrieved.
- **Control Flow**:
    - The function directly accesses the `wp` member of the `format_tree` structure.
    - It returns the value of `wp`, which is a pointer to a `window_pane` structure.
- **Output**:
    - Returns a pointer to the `window_pane` associated with the provided `format_tree`.


---
### format_create_add_item<!-- {{#callable:format_create_add_item}} -->
The `format_create_add_item` function merges format data from a command queue item into a format tree.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that holds the format data.
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item from which to extract the event.
- **Control Flow**:
    - The function retrieves the `key_event` associated with the provided `cmdq_item` using `cmdq_get_event`.
    - It then accesses the `mouse_event` structure from the retrieved `key_event`.
    - The function calls `cmdq_merge_formats` to merge the formats from the `cmdq_item` into the `format_tree`.
    - Finally, it copies the `mouse_event` data into the `format_tree`'s mouse event structure using `memcpy`.
- **Output**:
    - The function does not return a value; it modifies the `format_tree` in place by merging formats and copying mouse event data.


---
### format_create<!-- {{#callable:format_create}} -->
Creates a new `format_tree` structure for formatting commands.
- **Inputs**:
    - `c`: A pointer to a `client` structure, which may be NULL.
    - `item`: A pointer to a `cmdq_item` structure, which may be NULL.
    - `tag`: An integer representing a tag for the format tree.
    - `flags`: An integer representing flags for the format tree.
- **Control Flow**:
    - Allocates memory for a new `format_tree` structure using `xcalloc`.
    - Initializes the red-black tree in the `format_tree` structure.
    - If the `client` pointer is not NULL, increments the reference count for the client.
    - Sets the `item`, `tag`, and `flags` fields of the `format_tree` structure.
    - If the `item` is not NULL, calls [`format_create_add_item`](#format_create_add_item) to add the item to the format tree.
    - Returns the newly created `format_tree` structure.
- **Output**:
    - Returns a pointer to the newly created `format_tree` structure.
- **Functions called**:
    - [`format_create_add_item`](#format_create_add_item)


---
### format_free<!-- {{#callable:format_free}} -->
The `format_free` function deallocates memory associated with a `format_tree` structure.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains format entries and associated client information.
- **Control Flow**:
    - Iterates over each `format_entry` in the `format_tree` using `RB_FOREACH_SAFE` to safely remove entries while iterating.
    - For each `format_entry`, it removes the entry from the tree, frees its `key` and `value`, and then frees the entry itself.
    - After processing all entries, it checks if the `client` associated with the `format_tree` is not NULL and calls `server_client_unref` to decrement its reference count.
    - Finally, it frees the `format_tree` itself.
- **Output**:
    - The function does not return a value; it performs memory cleanup and deallocates resources associated with the `format_tree`.


---
### format_log_debug_cb<!-- {{#callable:format_log_debug_cb}} -->
Logs a debug message with a specified key-value pair and a prefix.
- **Inputs**:
    - `key`: A string representing the key to be logged.
    - `value`: A string representing the value associated with the key.
    - `arg`: A pointer to a string that serves as a prefix for the log message.
- **Control Flow**:
    - The function retrieves the prefix from the `arg` parameter.
    - It then calls the `log_debug` function to log the message, formatting it with the prefix, key, and value.
- **Output**:
    - The function does not return a value; it performs a logging action.


---
### format_log_debug<!-- {{#callable:format_log_debug}} -->
The `format_log_debug` function formats and logs debug information for a given format tree with a specified prefix.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains the data to be logged.
    - `prefix`: A string that serves as a prefix for the log messages.
- **Control Flow**:
    - The function calls [`format_each`](#format_each), passing the `format_tree`, a callback function `format_log_debug_cb`, and the prefix as arguments.
    - The [`format_each`](#format_each) function iterates over each entry in the `format_tree`, invoking the callback for each entry.
- **Output**:
    - The function does not return a value; it logs formatted debug messages to the logging system.
- **Functions called**:
    - [`format_each`](#format_each)


---
### format_each<!-- {{#callable:format_each}} -->
The `format_each` function iterates over a format tree and applies a callback to each key-value pair.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains the format entries to be processed.
    - `cb`: A function pointer to a callback that takes a key, value, and an additional argument.
    - `arg`: A pointer to additional data that will be passed to the callback function.
- **Control Flow**:
    - The function begins by declaring local variables for format table entries, format entries, and other necessary data.
    - It iterates over the `format_table`, calling the associated callback for each entry to retrieve its value.
    - If the value is of type `FORMAT_TABLE_TIME`, it formats the time value and calls the callback with the formatted string.
    - For other types, it directly calls the callback with the retrieved value and frees the value afterwards.
    - Next, it iterates over the entries in the `format_entry_tree` of the `format_tree`, processing each entry similarly.
    - If an entry has a non-zero time, it formats it and calls the callback; otherwise, it checks if the entry's value is NULL and retrieves it if necessary.
- **Output**:
    - The function does not return a value; instead, it invokes the provided callback for each key-value pair in the format tree.


---
### format_add<!-- {{#callable:format_add}} -->
The `format_add` function adds a key-value pair to a format tree.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure where the key-value pair will be added.
    - `key`: A string representing the key to be added to the format tree.
    - `fmt`: A format string that will be used to create the value associated with the key.
    - `...`: Additional arguments for the format string, allowing for variable argument input.
- **Control Flow**:
    - Allocate memory for a new `format_entry` structure and duplicate the provided key string.
    - Attempt to insert the new entry into the `format_entry_tree` of the format tree.
    - If an entry with the same key already exists, free the memory of the new entry and replace it with the existing entry.
    - Initialize the callback and time fields of the entry.
    - Use `va_start` to initialize the variable argument list and format the value using `xvasprintf`.
    - End the variable argument list with `va_end`.
- **Output**:
    - The function does not return a value; it modifies the format tree by adding or updating a key-value pair.


---
### format_add_tv<!-- {{#callable:format_add_tv}} -->
The `format_add_tv` function adds a key-value pair to a format tree with an associated timestamp.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure where the key-value pair will be added.
    - `key`: A string representing the key to be added to the format tree.
    - `tv`: A pointer to a `struct timeval` that contains the timestamp to be associated with the key.
- **Control Flow**:
    - Allocate memory for a new `format_entry` structure and duplicate the provided key into it.
    - Attempt to insert the new entry into the `format_entry_tree` of the format tree.
    - If an entry with the same key already exists, free the memory of the new entry and its key, and also free the value of the existing entry.
    - Set the `time` field of the entry to the seconds from the provided `struct timeval` and set the `value` field to NULL.
- **Output**:
    - The function does not return a value; it modifies the format tree in place by adding or updating an entry.


---
### format_add_cb<!-- {{#callable:format_add_cb}} -->
The `format_add_cb` function adds a callback function to a format entry in a format tree.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that represents the format tree to which the entry will be added.
    - `key`: A string representing the key for the format entry.
    - `cb`: A function pointer of type `format_cb` that serves as the callback for the format entry.
- **Control Flow**:
    - Allocate memory for a new `format_entry` structure and duplicate the provided key string.
    - Attempt to insert the new entry into the `format_entry_tree` of the format tree.
    - If an entry with the same key already exists, free the memory of the new entry and its key, and also free the value of the existing entry.
    - Set the callback function pointer, initialize the time to zero, and set the value to NULL for the new entry.
- **Output**:
    - The function does not return a value; it modifies the format tree by adding or updating a format entry with the specified key and callback.


---
### format_quote_shell<!-- {{#callable:format_quote_shell}} -->
The `format_quote_shell` function escapes special shell characters in a given string.
- **Inputs**:
    - `s`: A pointer to a constant character string that needs to be formatted for shell quoting.
- **Control Flow**:
    - The function allocates memory for the output string, which is twice the length of the input string plus one for the null terminator.
    - It iterates through each character of the input string.
    - For each character, it checks if it is a special character that needs to be escaped using `strchr`.
    - If a special character is found, a backslash is added to the output string before the character.
    - The character is then added to the output string.
    - Finally, the output string is null-terminated and returned.
- **Output**:
    - Returns a pointer to a newly allocated string with special characters escaped for safe use in a shell command.


---
### format_quote_style<!-- {{#callable:format_quote_style}} -->
The `format_quote_style` function duplicates a string while escaping the '#' character.
- **Inputs**:
    - `s`: A pointer to a constant character string that needs to be formatted.
- **Control Flow**:
    - The function allocates memory for the output string, which is twice the length of the input string plus one for the null terminator.
    - It iterates through each character of the input string.
    - If the current character is a '#', it is added to the output string without modification.
    - All characters from the input string are added to the output string.
    - Finally, the output string is null-terminated and returned.
- **Output**:
    - Returns a pointer to a newly allocated string with '#' characters preserved and all other characters copied from the input string.


---
### format_pretty_time<!-- {{#callable:format_pretty_time}} -->
Formats a given time into a human-readable string based on its age.
- **Inputs**:
    - `t`: A `time_t` value representing the time to format.
    - `seconds`: An integer flag indicating whether to include seconds in the output.
- **Control Flow**:
    - The function retrieves the current time and calculates the age of the input time `t`.
    - If the age is less than 24 hours, it formats the time as hours and minutes or hours, minutes, and seconds based on the `seconds` flag.
    - If the age is less than 28 days and within the same month, it formats the time as day and month.
    - If the time is within the last 12 months but not in the same month, it formats the time as day and abbreviated month.
    - For times older than 12 months, it formats the time as abbreviated month and year.
- **Output**:
    - Returns a dynamically allocated string containing the formatted time.


---
### format_find<!-- {{#callable:format_find}} -->
The `format_find` function retrieves a formatted string based on a specified key and various modifiers.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains the context for the format.
    - `key`: A string representing the key for which the format is being searched.
    - `modifiers`: An integer representing various modifiers that affect the formatting behavior.
    - `time_format`: A string specifying the format for time representation, if applicable.
- **Control Flow**:
    - The function first attempts to retrieve an option associated with the key from various option sources.
    - If an option is found, it converts it to a string and returns it.
    - If no option is found, it checks a format table for a corresponding entry and retrieves its value.
    - If a format entry is found, it checks if it has a time value or a callback function to generate its value.
    - If the key is not found in the format table, it checks the environment variables for a match.
    - If the modifiers indicate a time string, it processes the found value accordingly, formatting it as specified.
    - Finally, it applies any additional modifiers such as basename, dirname, or quoting before returning the result.
- **Output**:
    - The function returns a dynamically allocated string containing the formatted value associated with the key, or NULL if no value is found.
- **Functions called**:
    - [`format_table_get`](#format_table_get)
    - [`format_pretty_time`](#format_pretty_time)
    - [`format_quote_shell`](#format_quote_shell)
    - [`format_quote_style`](#format_quote_style)


---
### format_unescape<!-- {{#callable:format_unescape}} -->
The `format_unescape` function processes a string to remove escape sequences and unescapes specific characters.
- **Inputs**:
    - `s`: A pointer to a constant character string that contains the input to be processed.
- **Control Flow**:
    - The function initializes an output pointer and a counter for brackets.
    - It iterates through each character of the input string.
    - If it encounters a '#{' sequence, it increments the bracket counter.
    - If it finds a '#' followed by a character in the set {',', '#', '{', '}', ':'} and the bracket counter is zero, it skips the '#' and copies the next character to the output.
    - If it encounters a '}', it decrements the bracket counter.
    - All other characters are copied directly to the output.
    - Finally, it null-terminates the output string and returns it.
- **Output**:
    - Returns a newly allocated string with the unescaped content, which must be freed by the caller.


---
### format_strip<!-- {{#callable:format_strip}} -->
The `format_strip` function removes specific escaped characters from a string while preserving certain characters based on context.
- **Inputs**:
    - `s`: A pointer to a constant character string that needs to be processed for stripping.
- **Control Flow**:
    - Initialize an output pointer `out` and a copy pointer `cp` to allocate memory for the output string based on the length of the input string.
    - Iterate through each character of the input string until the null terminator is reached.
    - Count opening brackets when encountering the sequence '#{' and decrement the count when encountering '}'.
    - If a '#' is followed by certain characters (',', '#', '{', ':') and there are open brackets, copy the '#' to the output.
    - For all other characters, copy them directly to the output.
    - Terminate the output string with a null character and return the processed string.
- **Output**:
    - Returns a newly allocated string with specific characters stripped based on the defined rules, which must be freed by the caller.


---
### format_skip<!-- {{#callable:format_skip}} -->
The `format_skip` function iterates through a string, counting nested brackets and skipping over certain characters until it finds a specified end character.
- **Inputs**:
    - `s`: A pointer to the input string to be processed.
    - `end`: A pointer to a string containing characters that, when encountered at the top level of nesting, will cause the function to stop iterating.
- **Control Flow**:
    - The function initializes a counter `brackets` to track the level of nested brackets.
    - It enters a loop that continues until the end of the string is reached.
    - For each character in the string, it checks if it is the start of a nested bracket sequence ('#{'), incrementing the `brackets` counter.
    - If it encounters a '#' followed by a character that is not a null terminator and is one of the specified characters (',', '#', '{', '}', ':'), it skips the next character.
    - If it finds a closing bracket ('}'), it decrements the `brackets` counter.
    - If it encounters a character from the `end` string and the `brackets` counter is zero, it breaks out of the loop.
    - If the end of the string is reached without finding an end character, it returns NULL.
- **Output**:
    - The function returns a pointer to the character in the input string where it stopped processing, or NULL if the end of the string was reached without finding an end character.


---
### format_choose<!-- {{#callable:format_choose}} -->
The `format_choose` function separates a string into left and right parts based on a comma and optionally expands them.
- **Inputs**:
    - `es`: A pointer to a `format_expand_state` structure that holds the state for format expansion.
    - `s`: A string containing the input to be split into left and right parts.
    - `left`: A pointer to a character pointer where the left part of the split will be stored.
    - `right`: A pointer to a character pointer where the right part of the split will be stored.
    - `expand`: An integer flag indicating whether to expand the left and right parts.
- **Control Flow**:
    - The function calls [`format_skip`](#format_skip) to find the position of the first comma in the string `s`.
    - If no comma is found, the function returns -1 to indicate an error.
    - The left part is extracted from the start of the string up to the comma, and the right part is extracted from the character after the comma to the end of the string.
    - If the `expand` flag is set, both the left and right parts are expanded using [`format_expand1`](#format_expand1), and the temporary strings are freed.
    - If the `expand` flag is not set, the left and right parts are assigned directly to the output pointers.
- **Output**:
    - The function returns 0 on success, indicating that the left and right parts have been successfully assigned.
- **Functions called**:
    - [`format_skip`](#format_skip)
    - [`format_expand1`](#format_expand1)


---
### format_true<!-- {{#callable:format_true}} -->
The `format_true` function checks if a given string represents a true value.
- **Inputs**:
    - `s`: A pointer to a constant character string that is to be evaluated for truthiness.
- **Control Flow**:
    - The function first checks if the input string `s` is not NULL and not empty.
    - It then checks if the first character of the string is not '0' or if the string is not just '0'.
    - If both conditions are satisfied, it returns 1 (true); otherwise, it returns 0 (false).
- **Output**:
    - Returns 1 if the string is considered true, otherwise returns 0.


---
### format_is_end<!-- {{#callable:format_is_end}} -->
The `format_is_end` function checks if a given character is a semicolon or a colon.
- **Inputs**:
    - `c`: A character to be checked.
- **Control Flow**:
    - The function evaluates whether the input character `c` is equal to either ';' or ':' using a logical OR operation.
    - It returns 1 (true) if the condition is met, otherwise it returns 0 (false).
- **Output**:
    - Returns 1 if the character is a semicolon or colon, otherwise returns 0.


---
### format_add_modifier<!-- {{#callable:format_add_modifier}} -->
The `format_add_modifier` function adds a new `format_modifier` to a dynamically allocated list.
- **Inputs**:
    - `list`: A pointer to a pointer of `format_modifier` structures, representing the list to which the new modifier will be added.
    - `count`: A pointer to an unsigned integer that keeps track of the current number of modifiers in the list.
    - `c`: A pointer to a character string representing the modifier to be added.
    - `n`: The size of the modifier string `c`.
    - `argv`: An array of character pointers representing additional arguments associated with the modifier.
    - `argc`: An integer representing the number of additional arguments in `argv`.
- **Control Flow**:
    - The function first reallocates the `list` to accommodate one more `format_modifier` using `xreallocarray`.
    - It then assigns the newly allocated `format_modifier` to the next position in the list.
    - The function copies the modifier string `c` into the `modifier` field of the new `format_modifier`.
    - The size of the modifier is stored in the `size` field of the new `format_modifier`.
    - Finally, it assigns the `argv` and `argc` values to the corresponding fields in the new `format_modifier`.
- **Output**:
    - The function does not return a value; it modifies the `list` and `count` in place to include the new modifier.


---
### format_free_modifiers<!-- {{#callable:format_free_modifiers}} -->
The `format_free_modifiers` function frees memory associated with an array of `format_modifier` structures.
- **Inputs**:
    - `list`: A pointer to an array of `format_modifier` structures that need to be freed.
    - `count`: An unsigned integer representing the number of `format_modifier` structures in the array.
- **Control Flow**:
    - The function iterates over the array of `format_modifier` structures using a for loop, indexed by `i`.
    - For each `format_modifier`, it calls `cmd_free_argv` to free the associated command-line arguments stored in `argv`.
    - After freeing all command-line arguments, it calls `free` to release the memory allocated for the array of `format_modifier` structures.
- **Output**:
    - The function does not return a value; it performs memory deallocation to prevent memory leaks.


---
### format_build_modifiers<!-- {{#callable:format_build_modifiers}} -->
The `format_build_modifiers` function constructs a list of format modifiers from a given string.
- **Inputs**:
    - `es`: A pointer to a `struct format_expand_state` that holds the state for format expansion.
    - `s`: A pointer to a constant character pointer that points to the string containing the modifiers.
    - `count`: A pointer to an unsigned integer that will hold the count of modifiers found.
- **Control Flow**:
    - Initializes the count of modifiers to zero.
    - Enters a loop that continues until the end of the string or a colon is encountered.
    - Skips any separator characters encountered in the string.
    - Checks for single character modifiers without arguments and adds them to the list.
    - Checks for double character modifiers without arguments and adds them to the list.
    - Handles single character modifiers that require arguments, processing them based on the presence of punctuation.
    - Handles multiple arguments wrapped in a specified character.
    - If the end of the string is not a colon, it frees the list and sets the count to zero before returning NULL.
- **Output**:
    - Returns a pointer to a dynamically allocated array of `struct format_modifier` containing the parsed modifiers, or NULL if the parsing fails.
- **Functions called**:
    - [`format_is_end`](#format_is_end)
    - [`format_add_modifier`](#format_add_modifier)
    - [`format_skip`](#format_skip)
    - [`format_expand1`](#format_expand1)
    - [`format_free_modifiers`](#format_free_modifiers)


---
### format_match<!-- {{#callable:format_match}} -->
The `format_match` function checks if a given text matches a specified pattern using either fnmatch or regular expressions based on the provided format modifiers.
- **Inputs**:
    - `fm`: A pointer to a `format_modifier` structure that contains modifiers for the matching process.
    - `pattern`: A string representing the pattern to match against.
    - `text`: A string representing the text to be checked for a match.
- **Control Flow**:
    - The function initializes a string `s` to an empty string and a regex_t variable `r` for regular expression compilation.
    - It checks if the `fm` structure has at least one argument; if so, it assigns the first argument to `s`.
    - If the character 'r' is not found in `s`, it uses `fnmatch` to perform a case-insensitive match if 'i' is present in `s`, returning '0' if no match is found.
    - If 'r' is found in `s`, it compiles the regex pattern with appropriate flags and checks for a match using `regexec`, returning '0' if no match is found.
    - Finally, if a match is found, it returns '1'.
- **Output**:
    - The function returns a dynamically allocated string '1' if a match is found, or '0' if no match is found.


---
### format_sub<!-- {{#callable:format_sub}} -->
The `format_sub` function performs a regex-based substitution on a given text using a specified pattern and replacement string.
- **Inputs**:
    - `fm`: A pointer to a `format_modifier` structure that contains flags and arguments for the substitution.
    - `text`: The input string on which the substitution will be performed.
    - `pattern`: The regex pattern used to find matches in the input string.
    - `with`: The string that will replace the matched patterns in the input string.
- **Control Flow**:
    - The function initializes a variable `flags` with the value `REG_EXTENDED` to enable extended regex syntax.
    - If the `fm` structure indicates case-insensitive matching (by checking the third argument), the `REG_ICASE` flag is added to `flags`.
    - The `regsub` function is called with the `pattern`, `with`, `text`, and `flags` to perform the substitution.
    - If `regsub` returns NULL (indicating no matches or an error), the function duplicates the original `text` and returns it.
    - If a substitution is successful, the resulting string is returned.
- **Output**:
    - The function returns a pointer to a new string containing the modified text after substitution, or a duplicate of the original text if no substitution was made.


---
### format_search<!-- {{#callable:format_search}} -->
The `format_search` function searches for a specified string within a given window pane, optionally using case-insensitive and regular expression matching.
- **Inputs**:
    - `fm`: A pointer to a `format_modifier` structure that contains search options, including flags for case insensitivity and regular expression usage.
    - `wp`: A pointer to a `window_pane` structure representing the pane in which the search is to be performed.
    - `s`: A constant character pointer to the string that is to be searched within the specified pane.
- **Control Flow**:
    - The function initializes two integer flags, `ignore` and `regex`, to zero.
    - If the `fm` structure has at least one argument, it checks the first argument for the presence of 'i' to set the `ignore` flag for case insensitivity, and 'r' to set the `regex` flag for regular expression matching.
    - The function then calls `window_pane_search` with the pane, search string, and the flags to perform the search.
    - Finally, it formats the result of the search into a string and returns it.
- **Output**:
    - The function returns a dynamically allocated string containing the result of the search, specifically the number of occurrences found.


---
### format_bool_op_1<!-- {{#callable:format_bool_op_1}} -->
The `format_bool_op_1` function evaluates a boolean expression and returns its result as a string.
- **Inputs**:
    - `es`: A pointer to a `format_expand_state` structure that holds the state for format expansion.
    - `fmt`: A string representing the boolean expression to evaluate.
    - `not`: An integer flag indicating whether to negate the result of the evaluation.
- **Control Flow**:
    - The function calls [`format_expand1`](#format_expand1) to expand the `fmt` string using the provided `format_expand_state`.
    - It then evaluates the expanded string using the [`format_true`](#format_true) function to determine if it represents a true value.
    - If the `not` flag is set, the result is negated.
    - Finally, the function returns a string representation of the result ('1' for true, '0' for false).
- **Output**:
    - The function returns a dynamically allocated string containing '1' if the evaluated expression is true, or '0' if it is false.
- **Functions called**:
    - [`format_expand1`](#format_expand1)
    - [`format_true`](#format_true)


---
### format_bool_op_n<!-- {{#callable:format_bool_op_n}} -->
The `format_bool_op_n` function evaluates a series of boolean expressions separated by commas, returning '1' for true and '0' for false.
- **Inputs**:
    - `es`: A pointer to a `format_expand_state` structure that holds the state for format expansion.
    - `fmt`: A string containing the boolean expressions to evaluate, separated by commas.
    - `and`: An integer flag indicating whether to evaluate using logical AND (1) or logical OR (0).
- **Control Flow**:
    - The function initializes a result variable based on the value of `and` (1 for true, 0 for false).
    - It enters a loop that continues until the result matches the desired condition based on the `and` flag.
    - Within the loop, it uses [`format_skip`](#format_skip) to find the next comma in the `fmt` string, separating the expressions.
    - For each expression, it duplicates the substring and expands it using [`format_expand1`](#format_expand1).
    - The expanded expression is logged, and its truthiness is evaluated using [`format_true`](#format_true), updating the result accordingly.
    - The loop continues until all expressions are evaluated or a terminating condition is met.
- **Output**:
    - The function returns a string '1' if the overall result is true, or '0' if false, indicating the evaluation of the boolean expressions.
- **Functions called**:
    - [`format_skip`](#format_skip)
    - [`format_expand1`](#format_expand1)
    - [`format_true`](#format_true)


---
### format_session_name<!-- {{#callable:format_session_name}} -->
The `format_session_name` function checks if a given session name exists among the active sessions.
- **Inputs**:
    - `es`: A pointer to a `format_expand_state` structure that holds the state for format expansion.
    - `fmt`: A string containing the format to be expanded, which represents the session name to check.
- **Control Flow**:
    - The function calls [`format_expand1`](#format_expand1) to expand the format string `fmt` into a session name.
    - It iterates over all sessions using `RB_FOREACH` to check if any session's name matches the expanded name.
    - If a match is found, it frees the expanded name and returns '1' (indicating the session exists).
    - If no match is found, it frees the expanded name and returns '0' (indicating the session does not exist).
- **Output**:
    - Returns a string '1' if the session name exists, otherwise returns '0'.
- **Functions called**:
    - [`format_expand1`](#format_expand1)


---
### format_loop_sessions<!-- {{#callable:format_loop_sessions}} -->
The `format_loop_sessions` function expands a given format string for each session in a loop.
- **Inputs**:
    - `es`: A pointer to a `format_expand_state` structure that holds the current state of the format expansion.
    - `fmt`: A pointer to a string containing the format to be expanded for each session.
- **Control Flow**:
    - The function initializes a dynamic string `value` to hold the concatenated results of the expanded format strings.
    - It iterates over all sessions using `RB_FOREACH`, logging the current session ID.
    - For each session, it creates a new format tree and sets its defaults based on the current session.
    - The function then copies the current format expansion state and expands the format string using [`format_expand1`](#format_expand1).
    - The expanded string is concatenated to `value`, and memory is managed appropriately.
- **Output**:
    - The function returns a dynamically allocated string containing the concatenated results of the expanded format strings for all sessions.
- **Functions called**:
    - [`format_create`](#format_create)
    - [`format_defaults`](#format_defaults)
    - [`format_expand1`](#format_expand1)
    - [`format_free`](#format_free)


---
### format_window_name<!-- {{#callable:format_window_name}} -->
The `format_window_name` function checks if a given window name exists in the current session.
- **Inputs**:
    - `es`: A pointer to a `format_expand_state` structure that holds the state for format expansion.
    - `fmt`: A string containing the format to be expanded, which represents the window name to check.
- **Control Flow**:
    - The function first checks if the session associated with the `format_tree` in `es` is NULL; if so, it logs an error and returns NULL.
    - It then calls [`format_expand1`](#format_expand1) to expand the format string `fmt` into a window name.
    - The function iterates over all windows in the current session using `RB_FOREACH` to check if any window's name matches the expanded name.
    - If a match is found, it frees the expanded name and returns a string '1'.
    - If no match is found, it frees the expanded name and returns a string '0'.
- **Output**:
    - The function returns a string '1' if the window name exists in the current session, '0' if it does not, or NULL if there is no session.
- **Functions called**:
    - [`format_expand1`](#format_expand1)


---
### format_loop_windows<!-- {{#callable:format_loop_windows}} -->
The `format_loop_windows` function expands a format string for each window in the current session.
- **Inputs**:
    - `es`: A pointer to a `format_expand_state` structure that holds the current state of the format expansion.
    - `fmt`: A string containing the format to be expanded for each window.
- **Control Flow**:
    - Checks if the session associated with the format tree is NULL, logging an error if so and returning NULL.
    - Calls [`format_choose`](#format_choose) to determine the format strings for all windows and the active window.
    - Initializes a buffer to hold the expanded format string.
    - Iterates over each window in the session's window list using `RB_FOREACH`.
    - For each window, logs the current window index and ID, and determines which format string to use based on whether the window is active.
    - Creates a new format tree for the current window and sets its defaults.
    - Expands the format string using [`format_expand1`](#format_expand1) and appends the result to the output buffer.
    - Frees the resources allocated for the format strings and returns the final expanded string.
- **Output**:
    - Returns a dynamically allocated string containing the expanded format for all windows, or NULL if an error occurs.
- **Functions called**:
    - [`format_choose`](#format_choose)
    - [`format_create`](#format_create)
    - [`format_defaults`](#format_defaults)
    - [`format_expand1`](#format_expand1)
    - [`format_free`](#format_free)


---
### format_loop_panes<!-- {{#callable:format_loop_panes}} -->
The `format_loop_panes` function expands a format string for each pane in a window.
- **Inputs**:
    - `es`: A pointer to a `format_expand_state` structure that holds the current state of format expansion.
    - `fmt`: A string containing the format to be expanded for each pane.
- **Control Flow**:
    - Checks if the window associated with the format tree is NULL; if so, logs an error and returns NULL.
    - Calls [`format_choose`](#format_choose) to determine the format strings for all panes and the active pane.
    - Initializes a value buffer to accumulate the expanded format results.
    - Iterates over each pane in the window using `TAILQ_FOREACH`.
    - For each pane, determines which format string to use (active or all) and creates a new format tree for that pane.
    - Expands the format string using [`format_expand1`](#format_expand1) and appends the result to the value buffer.
    - Frees allocated memory for the active and all format strings before returning the final expanded value.
- **Output**:
    - Returns a dynamically allocated string containing the concatenated results of expanding the format string for each pane, or NULL if an error occurs.
- **Functions called**:
    - [`format_choose`](#format_choose)
    - [`format_create`](#format_create)
    - [`format_defaults`](#format_defaults)
    - [`format_expand1`](#format_expand1)
    - [`format_free`](#format_free)


---
### format_loop_clients<!-- {{#callable:format_loop_clients}} -->
The `format_loop_clients` function expands a format string for each client in a loop.
- **Inputs**:
    - `es`: A pointer to a `format_expand_state` structure that holds the current state of the format expansion.
    - `fmt`: A pointer to a string containing the format to be expanded for each client.
- **Control Flow**:
    - Initializes a dynamic string `value` to hold the concatenated results of the expanded format strings.
    - Iterates over each `client` in the global `clients` list using `TAILQ_FOREACH`.
    - For each client, creates a new `format_tree` specific to that client and initializes it with the current format state.
    - Calls [`format_expand1`](#format_expand1) to expand the format string for the current client and appends the result to `value`.
    - Frees the allocated memory for the expanded string after appending.
- **Output**:
    - Returns a dynamically allocated string containing the concatenated results of the expanded format strings for all clients.
- **Functions called**:
    - [`format_create`](#format_create)
    - [`format_defaults`](#format_defaults)
    - [`format_expand1`](#format_expand1)
    - [`format_free`](#format_free)


---
### format_replace_expression<!-- {{#callable:format_replace_expression}} -->
The `format_replace_expression` function evaluates a mathematical expression based on provided operands and an operator.
- **Inputs**:
    - `mexp`: A pointer to a `format_modifier` structure containing the operator and operands for the expression.
    - `es`: A pointer to a `format_expand_state` structure that holds the state of the format expansion.
    - `copy`: A string representing the expression to be evaluated.
- **Control Flow**:
    - The function begins by determining the operator from the first argument in `mexp->argv`.
    - It checks for valid operators such as addition, subtraction, multiplication, etc., and assigns the corresponding enum value.
    - If the operator is invalid, it logs an error and jumps to the fail label.
    - It then checks for optional flags and precision from the subsequent arguments in `mexp->argv`.
    - The function attempts to parse the left and right operands from the expression using [`format_choose`](#format_choose).
    - It converts the left and right operands from strings to doubles, logging errors if the conversion fails.
    - Based on the operator, it performs the corresponding mathematical operation on the operands.
    - Finally, it formats the result according to the specified precision and logs the result before returning it.
- **Output**:
    - Returns a dynamically allocated string containing the result of the evaluated expression, or NULL if an error occurs.
- **Functions called**:
    - [`format_choose`](#format_choose)


---
### format_replace<!-- {{#callable:format_replace}} -->
The `format_replace` function processes a format string, replacing placeholders with corresponding values based on modifiers.
- **Inputs**:
    - `es`: A pointer to a `format_expand_state` structure that holds the current state of format expansion.
    - `key`: A string representing the key to be replaced in the format string.
    - `keylen`: The length of the key string.
    - `buf`: A pointer to a character pointer where the resulting formatted string will be stored.
    - `len`: A pointer to a size_t variable that holds the current length of the buffer.
    - `off`: A pointer to a size_t variable that indicates the current offset in the buffer.
- **Control Flow**:
    - The function begins by making a copy of the key to be processed.
    - It then builds a list of modifiers from the key, which dictate how the replacement should be handled.
    - The function checks for various modifiers such as literal strings, characters, colors, and loops over sessions, windows, or panes.
    - If a modifier indicates a literal string, it processes it directly.
    - For character and color modifiers, it expands the key and converts it accordingly.
    - If the key represents a loop or a conditional, it processes those cases by iterating over the relevant structures.
    - The function handles substitutions based on the modifiers and applies any length or width limits specified.
    - Finally, it expands the buffer to accommodate the new value and copies the result into the provided buffer.
- **Output**:
    - The function returns 0 on success, indicating that the replacement was successful, or -1 on failure, indicating an error occurred during processing.
- **Functions called**:
    - [`format_build_modifiers`](#format_build_modifiers)
    - [`format_logging`](#format_logging)
    - [`format_strip`](#format_strip)
    - [`format_unescape`](#format_unescape)
    - [`format_expand1`](#format_expand1)
    - [`format_loop_sessions`](#format_loop_sessions)
    - [`format_loop_windows`](#format_loop_windows)
    - [`format_loop_panes`](#format_loop_panes)
    - [`format_loop_clients`](#format_loop_clients)
    - [`format_window_name`](#format_window_name)
    - [`format_session_name`](#format_session_name)
    - [`format_search`](#format_search)
    - [`format_choose`](#format_choose)
    - [`format_bool_op_1`](#format_bool_op_1)
    - [`format_bool_op_n`](#format_bool_op_n)
    - [`format_match`](#format_match)
    - [`format_skip`](#format_skip)
    - [`format_find`](#format_find)
    - [`format_true`](#format_true)
    - [`format_replace_expression`](#format_replace_expression)
    - [`format_sub`](#format_sub)
    - [`format_free_modifiers`](#format_free_modifiers)


---
### format_expand1<!-- {{#callable:format_expand1}} -->
The `format_expand1` function expands format strings by replacing placeholders with their corresponding values.
- **Inputs**:
    - `es`: A pointer to a `format_expand_state` structure that holds the state for the format expansion.
    - `fmt`: A pointer to a constant character string representing the format string to be expanded.
- **Control Flow**:
    - Checks if the input format string is NULL or empty, returning a default string if true.
    - Increments the loop counter and checks for loop limit to prevent infinite recursion.
    - Logs the format string being expanded.
    - If the `FORMAT_EXPAND_TIME` flag is set and the format string contains a time format, it expands the time using `strftime`.
    - Allocates a buffer for the output and processes each character in the format string.
    - Handles different types of placeholders, including `#()`, `#{}`, and others, expanding them as necessary.
    - Logs the results of each expansion and appends them to the output buffer.
    - Returns the final expanded string after processing all placeholders.
- **Output**:
    - Returns a dynamically allocated string containing the expanded format, or an empty string if the format could not be expanded.
- **Functions called**:
    - [`format_logging`](#format_logging)
    - [`format_job_get`](#format_job_get)
    - [`format_skip`](#format_skip)
    - [`format_replace`](#format_replace)


---
### format_expand_time<!-- {{#callable:format_expand_time}} -->
The `format_expand_time` function expands a time format string using a specified format tree.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains the context for the format expansion.
    - `fmt`: A pointer to a constant character string representing the format to be expanded.
- **Control Flow**:
    - A `format_expand_state` structure is initialized and its fields are set, including the format tree and flags for time expansion.
    - The [`format_expand1`](#format_expand1) function is called with the initialized state and the format string to perform the actual expansion.
- **Output**:
    - Returns a pointer to a character string containing the expanded format based on the provided time and format tree.
- **Functions called**:
    - [`format_expand1`](#format_expand1)


---
### format_expand<!-- {{#callable:format_expand}} -->
The `format_expand` function expands format strings using a specified format tree.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains the context for the format expansion.
    - `fmt`: A constant character pointer to the format string that needs to be expanded.
- **Control Flow**:
    - The function initializes a `format_expand_state` structure and sets its `ft` member to the provided format tree.
    - It calls [`format_expand1`](#format_expand1) with the initialized state and the format string to perform the actual expansion.
    - The result of the expansion is returned to the caller.
- **Output**:
    - Returns a pointer to a character string that contains the expanded format string.
- **Functions called**:
    - [`format_expand1`](#format_expand1)


---
### format_single<!-- {{#callable:format_single}} -->
The `format_single` function expands a format string using a specified context.
- **Inputs**:
    - `item`: A pointer to a `struct cmdq_item` that represents the command queue item associated with the format.
    - `fmt`: A pointer to a constant character string that contains the format to be expanded.
    - `c`: A pointer to a `struct client` representing the client context for the format.
    - `s`: A pointer to a `struct session` representing the session context for the format.
    - `wl`: A pointer to a `struct winlink` representing the window link context for the format.
    - `wp`: A pointer to a `struct window_pane` representing the window pane context for the format.
- **Control Flow**:
    - The function begins by creating a new `format_tree` structure using the [`format_create_defaults`](#format_create_defaults) function, which initializes the format context with the provided arguments.
    - Next, it calls the [`format_expand`](#format_expand) function, passing the created format tree and the format string to expand it based on the context.
    - After the expansion, the function frees the allocated format tree using [`format_free`](#format_free) to prevent memory leaks.
    - Finally, it returns the expanded format string.
- **Output**:
    - The function returns a pointer to a character string that contains the expanded format based on the provided context.
- **Functions called**:
    - [`format_create_defaults`](#format_create_defaults)
    - [`format_expand`](#format_expand)
    - [`format_free`](#format_free)


---
### format_single_from_state<!-- {{#callable:format_single_from_state}} -->
The `format_single_from_state` function expands a format string using the state of a command find operation.
- **Inputs**:
    - `item`: A pointer to a `struct cmdq_item` representing the command queue item.
    - `fmt`: A pointer to a string containing the format to be expanded.
    - `c`: A pointer to a `struct client` representing the client context.
    - `fs`: A pointer to a `struct cmd_find_state` containing the state information for the command find operation.
- **Control Flow**:
    - The function calls [`format_single`](#format_single) with the provided arguments, extracting the session, winlink, and window pane from the `cmd_find_state` structure.
    - The [`format_single`](#format_single) function handles the actual expansion of the format string using the provided context.
- **Output**:
    - Returns a pointer to a string containing the expanded format, or NULL if the expansion fails.
- **Functions called**:
    - [`format_single`](#format_single)


---
### format_single_from_target<!-- {{#callable:format_single_from_target}} -->
The `format_single_from_target` function formats a string based on a target client and a specified format.
- **Inputs**:
    - `item`: A pointer to a `struct cmdq_item` that contains the command queue item.
    - `fmt`: A pointer to a constant character string that specifies the format to be applied.
- **Control Flow**:
    - The function retrieves the target client associated with the provided `cmdq_item` using `cmdq_get_target_client`.
    - It then calls [`format_single_from_state`](#format_single_from_state), passing the `item`, `fmt`, the retrieved client, and the target state obtained from `cmdq_get_target`.
- **Output**:
    - Returns a pointer to a formatted string based on the specified format and the target client.
- **Functions called**:
    - [`format_single_from_state`](#format_single_from_state)


---
### format_create_defaults<!-- {{#callable:format_create_defaults}} -->
Creates a `format_tree` with default values based on provided parameters.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure that may contain a command queue item.
    - `c`: A pointer to a `client` structure representing the client.
    - `s`: A pointer to a `session` structure representing the session.
    - `wl`: A pointer to a `winlink` structure representing the window link.
    - `wp`: A pointer to a `window_pane` structure representing the window pane.
- **Control Flow**:
    - Checks if `item` is not NULL and creates a `format_tree` using [`format_create`](#format_create) with the client from the item.
    - If `item` is NULL, it creates a `format_tree` with NULL client.
    - Calls [`format_defaults`](#format_defaults) to set default values in the `format_tree` using the provided client, session, winlink, and window pane.
    - Returns the created `format_tree`.
- **Output**:
    - Returns a pointer to a `format_tree` structure initialized with default values based on the provided parameters.
- **Functions called**:
    - [`format_create`](#format_create)
    - [`format_defaults`](#format_defaults)


---
### format_create_from_state<!-- {{#callable:format_create_from_state}} -->
Creates a `format_tree` structure from the given state.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item.
    - `c`: A pointer to a `client` structure representing the client.
    - `fs`: A pointer to a `cmd_find_state` structure containing the state information.
- **Control Flow**:
    - The function calls [`format_create_defaults`](#format_create_defaults) with the provided `item`, `client`, and the session, winlink, and window pane from the `cmd_find_state` structure.
    - The [`format_create_defaults`](#format_create_defaults) function initializes a new `format_tree` with default values based on the provided state.
- **Output**:
    - Returns a pointer to a `format_tree` structure initialized with the default values based on the provided state.
- **Functions called**:
    - [`format_create_defaults`](#format_create_defaults)


---
### format_create_from_target<!-- {{#callable:format_create_from_target}} -->
Creates a `format_tree` structure from a target client and command queue item.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure that contains the command queue item.
- **Control Flow**:
    - The function retrieves the target client associated with the provided `cmdq_item` using `cmdq_get_target_client`.
    - It then calls [`format_create_from_state`](#format_create_from_state), passing the `cmdq_item`, the retrieved client, and the target state obtained from `cmdq_get_target`.
- **Output**:
    - Returns a pointer to a `format_tree` structure that is created based on the state of the command queue item and the target client.
- **Functions called**:
    - [`format_create_from_state`](#format_create_from_state)


---
### format_defaults<!-- {{#callable:format_defaults}} -->
The `format_defaults` function initializes a `format_tree` structure with default values based on the provided client, session, winlink, and window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that will be populated with default values.
    - `c`: A pointer to a `client` structure, which may provide context for the format.
    - `s`: A pointer to a `session` structure, which may provide context for the format.
    - `wl`: A pointer to a `winlink` structure, which may provide context for the format.
    - `wp`: A pointer to a `window_pane` structure, which may provide context for the format.
- **Control Flow**:
    - The function starts by logging the state of the input parameters, checking if they are NULL and logging their respective identifiers.
    - It checks if the `client` and `session` are not NULL and if they do not match, it logs a warning.
    - The function determines the type of format based on the presence of the `window pane`, `winlink`, or `session` and sets the `format_tree` type accordingly.
    - If any of the parameters are NULL, it assigns default values from the `client` or `session` to ensure that the `format_tree` is fully populated.
    - It calls specific functions to set defaults for the `client`, `session`, `winlink`, and `window pane` if they are not NULL.
    - Finally, it checks for a top paste buffer and sets its defaults if available.
- **Output**:
    - The function does not return a value but modifies the `format_tree` structure pointed to by `ft` to include default values based on the provided parameters.
- **Functions called**:
    - [`format_defaults_client`](#format_defaults_client)
    - [`format_defaults_session`](#format_defaults_session)
    - [`format_defaults_winlink`](#format_defaults_winlink)
    - [`format_defaults_pane`](#format_defaults_pane)
    - [`format_defaults_paste_buffer`](#format_defaults_paste_buffer)


---
### format_defaults_session<!-- {{#callable:format_defaults_session}} -->
The `format_defaults_session` function sets the session pointer in a format tree.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that holds formatting information.
    - `s`: A pointer to a `session` structure representing the session to be associated with the format tree.
- **Control Flow**:
    - The function directly assigns the session pointer `s` to the `s` field of the `format_tree` structure `ft`.
- **Output**:
    - The function does not return a value; it modifies the `format_tree` structure in place by setting its session pointer.


---
### format_defaults_client<!-- {{#callable:format_defaults_client}} -->
The `format_defaults_client` function initializes the session field of a `format_tree` structure with the session of a given client if it is not already set.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that holds formatting information.
    - `c`: A pointer to a `client` structure representing the client whose session is to be used.
- **Control Flow**:
    - The function first checks if the session field (`s`) of the `format_tree` structure (`ft`) is NULL.
    - If the session field is NULL, it assigns the session of the provided client (`c->session`) to it.
    - Regardless of whether the session was NULL or not, it sets the client field (`c`) of the `format_tree` to the provided client.
- **Output**:
    - The function does not return a value; it modifies the `format_tree` structure in place.


---
### format_defaults_window<!-- {{#callable:format_defaults_window}} -->
The `format_defaults_window` function sets the window pointer in a format tree structure.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that holds various format-related information.
    - `w`: A pointer to a `window` structure that represents the window to be associated with the format tree.
- **Control Flow**:
    - The function directly assigns the `window` pointer `w` to the `w` member of the `format_tree` structure `ft`.
    - There are no conditional statements or loops; the function performs a single operation.
- **Output**:
    - The function does not return a value; it modifies the `format_tree` structure in place by setting its `w` member.


---
### format_defaults_winlink<!-- {{#callable:format_defaults_winlink}} -->
The `format_defaults_winlink` function sets default format keys for a given winlink in a format tree.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that holds the format context.
    - `wl`: A pointer to a `winlink` structure representing the winlink whose defaults are to be set.
- **Control Flow**:
    - The function first checks if the `window` field of the `format_tree` structure is NULL.
    - If it is NULL, it calls [`format_defaults_window`](#format_defaults_window) to set the default window format using the `window` associated with the winlink.
    - It then assigns the provided winlink to the `wl` field of the format tree.
- **Output**:
    - The function does not return a value; it modifies the `format_tree` structure in place by setting its `wl` field and potentially its `w` field.
- **Functions called**:
    - [`format_defaults_window`](#format_defaults_window)


---
### format_defaults_pane<!-- {{#callable:format_defaults_pane}} -->
The `format_defaults_pane` function sets default format values for a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that holds the format state.
    - `wp`: A pointer to a `window_pane` structure representing the pane for which defaults are being set.
- **Control Flow**:
    - Checks if the `window` field of the `format_tree` is NULL; if so, it calls [`format_defaults_window`](#format_defaults_window) to set the window defaults.
    - Sets the `wp` field of the `format_tree` to the provided `window_pane`.
    - Retrieves the first mode entry from the `window_pane`'s modes list.
    - If a mode entry exists and it has a `formats` callback, it calls this callback with the mode entry and the `format_tree`.
- **Output**:
    - The function does not return a value; it modifies the `format_tree` structure to include default values for the specified pane.
- **Functions called**:
    - [`format_defaults_window`](#format_defaults_window)


---
### format_defaults_paste_buffer<!-- {{#callable:format_defaults_paste_buffer}} -->
Sets the `paste_buffer` for a given `format_tree`.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that holds various formatting information.
    - `pb`: A pointer to a `paste_buffer` structure that contains the data to be associated with the format tree.
- **Control Flow**:
    - The function directly assigns the `paste_buffer` pointer to the `format_tree` structure.
    - There are no conditional statements or loops in this function.
- **Output**:
    - This function does not return a value; it modifies the `format_tree` structure in place by setting its `pb` member to the provided `paste_buffer`.


---
### format_is_word_separator<!-- {{#callable:format_is_word_separator}} -->
The `format_is_word_separator` function checks if a given grid cell is a word separator based on specified criteria.
- **Inputs**:
    - `ws`: A string containing characters that are considered as word separators.
    - `gc`: A pointer to a `struct grid_cell` that represents the grid cell to be checked.
- **Control Flow**:
    - The function first checks if the string `ws` contains the character data of the grid cell using `utf8_cstrhas`.
    - If the above check is true, it returns 1, indicating that the cell is a word separator.
    - Next, it checks if the `gc` flags indicate that the cell is a tab (using `GRID_FLAG_TAB`), returning 1 if true.
    - Finally, it checks if the size of the cell's data is 1 and if the data is a space character, returning 1 if both conditions are met.
    - If none of the conditions are satisfied, it returns 0, indicating that the cell is not a word separator.
- **Output**:
    - The function returns 1 if the grid cell is a word separator based on the specified conditions, and 0 otherwise.


---
### format_grid_word<!-- {{#callable:format_grid_word}} -->
The `format_grid_word` function retrieves a word from a grid at specified coordinates.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` representing the grid from which to extract the word.
    - `x`: An unsigned integer representing the x-coordinate in the grid.
    - `y`: An unsigned integer representing the y-coordinate in the grid.
- **Control Flow**:
    - The function initializes variables to track the grid cell and word separators.
    - It enters a loop to move left in the grid until it finds a word separator or reaches the beginning of the grid.
    - If a word separator is found, it marks the position and continues to find the end of the word.
    - It then enters another loop to move right from the found position until it finds another word separator or the end of the line.
    - The characters between the two separators are collected into a UTF-8 data structure.
    - Finally, it converts the collected data into a string and returns it.
- **Output**:
    - Returns a dynamically allocated string containing the word found at the specified coordinates, or NULL if no word is found.
- **Functions called**:
    - [`format_is_word_separator`](#format_is_word_separator)


---
### format_grid_line<!-- {{#callable:format_grid_line}} -->
The `format_grid_line` function constructs a string representation of a specific line in a grid by concatenating the non-padding cells.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` that represents the grid from which the line will be formatted.
    - `y`: An unsigned integer representing the specific line number in the grid to format.
- **Control Flow**:
    - Iterates over each cell in the specified line of the grid using the `grid_line_length` function to determine the number of cells.
    - For each cell, it retrieves the cell data using `grid_get_cell` and checks if the cell is a padding cell; if so, it skips to the next cell.
    - If the cell is a tab, it adds a tab character to the output; otherwise, it copies the cell's data into a dynamically allocated array.
    - After processing all cells, if any data was collected, it terminates the UTF-8 data array and converts it to a string using `utf8_tocstr`.
- **Output**:
    - Returns a dynamically allocated string containing the formatted line of text, or NULL if the line is empty or no valid cells were found.


---
### format_grid_hyperlink<!-- {{#callable:format_grid_hyperlink}} -->
The `format_grid_hyperlink` function retrieves the hyperlink associated with a specific cell in a grid.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` representing the grid from which to retrieve the hyperlink.
    - `x`: An unsigned integer representing the x-coordinate of the cell in the grid.
    - `y`: An unsigned integer representing the y-coordinate of the cell in the grid.
    - `s`: A pointer to a `struct screen` that contains hyperlink information.
- **Control Flow**:
    - The function enters an infinite loop to find the first non-padding cell starting from the given coordinates (x, y).
    - If the x-coordinate reaches 0 and no non-padding cell is found, the function returns NULL.
    - If the screen's hyperlink list is NULL or the cell does not have a link, the function returns NULL.
    - The function attempts to retrieve the hyperlink URI using the cell's link index from the screen's hyperlink list.
    - If successful, it duplicates the URI string and returns it; otherwise, it returns NULL.
- **Output**:
    - Returns a dynamically allocated string containing the hyperlink URI if found, or NULL if no hyperlink is associated with the specified cell.


# Function Declarations (Public API)

---
### format_job_get<!-- {{#callable_declaration:format_job_get}} -->
Retrieves or executes a job based on a command string.
- **Description**: This function is used to retrieve the output of a job associated with a given command string, or to execute the command if it hasn't been run yet. It manages job caching and execution, ensuring that the command is only run when necessary. This function should be called when you need to ensure that a command's output is available, and it handles job creation and management internally. It is important to note that this function requires a valid format_expand_state structure, and the command string must be non-null.
- **Inputs**:
    - `es`: A pointer to a format_expand_state structure, which must be valid and properly initialized. The caller retains ownership and must ensure it is not null.
    - `cmd`: A constant character pointer representing the command to be executed or retrieved. It must not be null, and it should be a valid command string.
- **Output**: Returns a dynamically allocated string containing the job's output or an empty string if no output is available. The caller is responsible for freeing the returned string.
- **See also**: [`format_job_get`](#format_job_get)  (Implementation)


---
### format_expand1<!-- {{#callable_declaration:format_expand1}} -->
Expands format strings with key-value pairs and special syntax.
- **Description**: This function processes a format string, expanding any embedded keys or special syntax into their corresponding values based on the provided format_expand_state. It is typically used to dynamically generate strings with embedded data, such as session or window information, in a structured format. The function handles various special cases, such as time formatting and job execution, and ensures that the expansion does not exceed a predefined recursion limit. It should be used when a formatted string needs to be generated from a set of key-value pairs, with support for advanced formatting features.
- **Inputs**:
    - `es`: A pointer to a format_expand_state structure that contains the current state of the format expansion, including the format tree and any flags. The caller retains ownership and it must not be null.
    - `fmt`: A constant character pointer to the format string to be expanded. It can be null or an empty string, in which case the function returns an empty string.
- **Output**: Returns a newly allocated string with the expanded format, which the caller is responsible for freeing.
- **See also**: [`format_expand1`](#format_expand1)  (Implementation)


---
### format_replace<!-- {{#callable_declaration:format_replace}} -->
Replaces format keys in a buffer with their corresponding values.
- **Description**: This function is used to replace format keys within a given buffer with their corresponding values based on the provided format expand state. It processes a key with optional modifiers and updates the buffer with the resolved value. This function is typically used in scenarios where dynamic content needs to be inserted into a template or format string. It requires a valid format expand state and a non-null buffer to operate. The function handles various modifiers and conditions, ensuring that the buffer is expanded correctly. It returns an integer indicating success or failure of the replacement operation.
- **Inputs**:
    - `es`: A pointer to a format_expand_state structure that holds the current state of format expansion. Must not be null.
    - `key`: A constant character pointer representing the format key to be replaced. It should point to a valid string.
    - `keylen`: The length of the key string. Must be a valid size_t value.
    - `buf`: A pointer to a character pointer representing the buffer where the expanded value will be written. Must not be null, and the buffer should be pre-allocated.
    - `len`: A pointer to a size_t representing the current length of the buffer. Must not be null.
    - `off`: A pointer to a size_t representing the current offset in the buffer where the new data should be written. Must not be null.
- **Output**: Returns 0 on success, or -1 on failure, indicating whether the replacement was successful.
- **See also**: [`format_replace`](#format_replace)  (Implementation)


---
### format_defaults_session<!-- {{#callable_declaration:format_defaults_session}} -->
Associates a session with a format tree.
- **Description**: This function is used to set a session within a format tree structure, which is typically used for formatting and expanding key-value pairs in strings. It should be called when you need to associate a specific session with a format tree, allowing the format tree to access session-specific information. This function does not perform any validation on the input parameters, so it is the caller's responsibility to ensure that the provided pointers are valid and that the format tree is properly initialized before calling this function.
- **Inputs**:
    - `ft`: A pointer to a format_tree structure. This must be a valid, initialized format_tree object. The caller retains ownership and must ensure it is not null.
    - `s`: A pointer to a session structure. This must be a valid session object that the caller wishes to associate with the format tree. The caller retains ownership and must ensure it is not null.
- **Output**: None
- **See also**: [`format_defaults_session`](#format_defaults_session)  (Implementation)


---
### format_defaults_client<!-- {{#callable_declaration:format_defaults_client}} -->
Sets default format values for a client in a format tree.
- **Description**: This function is used to set default values in a format tree based on the provided client. It should be called when you need to initialize or update a format tree with client-specific information. The function assumes that the format tree and client are valid and properly initialized. If the session in the format tree is not already set, it will be set to the session associated with the client.
- **Inputs**:
    - `ft`: A pointer to a format_tree structure where the default values will be set. Must not be null.
    - `c`: A pointer to a client structure from which default values will be derived. Must not be null.
- **Output**: None
- **See also**: [`format_defaults_client`](#format_defaults_client)  (Implementation)


---
### format_defaults_winlink<!-- {{#callable_declaration:format_defaults_winlink}} -->
Sets default format keys for a winlink.
- **Description**: This function is used to set default format keys for a given winlink within a format tree. It should be called when you need to associate a winlink with a format tree, typically as part of preparing the format tree for use in formatting operations. The function expects that the format tree and winlink are valid and properly initialized. It does not return any value or modify the winlink itself.
- **Inputs**:
    - `ft`: A pointer to a format_tree structure. This must be a valid, initialized format tree where the default keys will be set. The caller retains ownership.
    - `wl`: A pointer to a winlink structure. This must be a valid winlink that you want to associate with the format tree. The caller retains ownership.
- **Output**: None
- **See also**: [`format_defaults_winlink`](#format_defaults_winlink)  (Implementation)


---
### format_job_cmp<!-- {{#callable_declaration:format_job_cmp}} -->
Compare two format jobs based on their tags and commands.
- **Description**: Use this function to compare two `format_job` structures. It first compares the `tag` fields of the two jobs. If the tags are different, it returns -1 if the first job's tag is less than the second's, or 1 if greater. If the tags are equal, it compares the `cmd` strings using `strcmp` and returns the result of that comparison. This function is useful for sorting or ordering `format_job` structures.
- **Inputs**:
    - `fj1`: A pointer to the first `format_job` structure to compare. Must not be null.
    - `fj2`: A pointer to the second `format_job` structure to compare. Must not be null.
- **Output**: An integer less than, equal to, or greater than zero if `fj1` is found, respectively, to be less than, to match, or be greater than `fj2`.
- **See also**: [`format_job_cmp`](#format_job_cmp)  (Implementation)


---
### format_entry_cmp<!-- {{#callable_declaration:format_entry_cmp}} -->
Compares two format_entry structures based on their keys.
- **Description**: This function is used to compare two format_entry structures by their key fields. It is typically used in data structures that require ordering, such as binary trees, to determine the relative ordering of entries. The function returns an integer less than, equal to, or greater than zero if the key of the first format_entry is found, respectively, to be less than, to match, or be greater than the key of the second format_entry. This function assumes that the key fields of the format_entry structures are valid, null-terminated strings.
- **Inputs**:
    - `fe1`: A pointer to the first format_entry structure to compare. The key field must be a valid, null-terminated string.
    - `fe2`: A pointer to the second format_entry structure to compare. The key field must be a valid, null-terminated string.
- **Output**: An integer less than, equal to, or greater than zero if the key of fe1 is found, respectively, to be less than, to match, or be greater than the key of fe2.
- **See also**: [`format_entry_cmp`](#format_entry_cmp)  (Implementation)


---
### xvasprintf<!-- {{#callable_declaration:xvasprintf}} -->
Formats a string and stores it in a dynamically allocated buffer.
- **Description**: This function formats a string according to the specified format string and variable argument list, storing the result in a dynamically allocated buffer. It is similar to `vasprintf`, but will terminate the program if memory allocation fails. This function is useful when you need a formatted string and want to handle memory allocation errors by terminating the program. The function must be called with a valid format string and a `va_list` that has been initialized with `va_start`. The caller is responsible for freeing the allocated buffer.
- **Inputs**:
    - `ret`: A pointer to a `char*` where the address of the allocated buffer will be stored. Must not be null. The caller is responsible for freeing the allocated memory.
    - `fmt`: A format string that specifies how to format the output. Must not be null.
    - `ap`: A `va_list` containing the arguments for the format string. Must be initialized with `va_start` before calling this function.
- **Output**: Returns the number of characters printed (excluding the null byte used to end output to strings) or -1 if an error occurs during formatting.
- **See also**: [`xvasprintf`](xmalloc.c.driver.md#xvasprintf)  (Implementation)


---
### cmdq_print<!-- {{#callable_declaration:cmdq_print}} -->
Formats and prints a message to a command queue item.
- **Description**: Use this function to format a message according to a specified format string and print it to a command queue item. This function is useful for logging or displaying formatted output in a command queue context. It requires a valid command queue item and a format string, similar to printf-style formatting. Ensure that the command queue item is properly initialized before calling this function. The function handles memory allocation internally and will terminate the program if memory allocation fails.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure where the formatted message will be printed. Must not be null and should be a valid, initialized command queue item.
    - `fmt`: A format string that specifies how to format the message. It follows printf-style formatting rules. Must not be null.
    - `...`: A variable number of arguments that correspond to the format specifiers in `fmt`. The types and number of arguments must match the format specifiers in `fmt`.
- **Output**: None
- **See also**: [`cmdq_print`](cmd-queue.c.driver.md#cmdq_print)  (Implementation)


