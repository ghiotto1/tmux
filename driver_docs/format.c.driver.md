# Purpose
The provided C code is part of the `tmux` project, specifically dealing with the formatting and expansion of strings within the `tmux` environment. This code is responsible for handling the dynamic generation of strings by expanding format variables, which are placeholders that can be replaced with actual values at runtime. The code includes functions to manage format trees, which are data structures that store key-value pairs representing these format variables. It also includes mechanisms to handle various modifiers that can be applied to these variables, such as quoting, truncating, or applying mathematical operations.

Key components of the code include the `format_tree` structure, which holds the context for format expansion, and the [`format_expand1`](#format_expand1) function, which performs the actual expansion of format strings. The code supports a wide range of format variables, including those related to sessions, windows, panes, and clients, and provides callbacks for each variable to retrieve its value. Additionally, the code includes logic for handling conditional expressions, loops over collections (such as sessions or windows), and various string manipulations. This functionality is crucial for `tmux` as it allows users to customize the display and behavior of the terminal multiplexer through dynamic and context-aware strings.
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
- **Type**: `static char*`
- **Description**: The `format_job_get` function is a static function that returns a pointer to a character string. It is designed to find or create a job associated with a specific command and format state, and then return the output of that job as a string.
- **Use**: This function is used to retrieve the output of a job associated with a specific command, creating the job if it does not already exist.


---
### format_expand1
- **Type**: ``char *``
- **Description**: The `format_expand1` function is a static function that takes a pointer to a `struct format_expand_state` and a constant character pointer as arguments. It is used to expand format strings by replacing placeholders with their corresponding values.
- **Use**: This function is used internally to process and expand format strings within the context of a format expansion state.


---
### format_jobs
- **Type**: ``struct format_job_tree``
- **Description**: The `format_jobs` variable is a static instance of a red-black tree data structure, specifically a `struct format_job_tree`, which is used to manage `format_job` entries. It is initialized using the `RB_INITIALIZER()` macro, which sets up the tree for use. The `format_job` entries in this tree are used to manage and track format jobs, which are likely related to processing or expanding format strings in the application.
- **Use**: This variable is used to store and manage `format_job` entries in a red-black tree, allowing for efficient insertion, deletion, and lookup operations.


---
### format_upper
- **Type**: `const char *[]`
- **Description**: `format_upper` is a static constant array of strings, where each index corresponds to an uppercase letter of the alphabet. The array is used to map certain uppercase letters to specific string identifiers, such as "pane_id" for 'D' and "window_flags" for 'F'. The array is initialized with NULL for letters that do not have a corresponding string identifier.
- **Use**: This variable is used to provide single-character uppercase aliases for specific format keys in the application.


---
### format_lower
- **Type**: ``const char *` array`
- **Description**: The `format_lower` variable is a static constant array of pointers to strings, with each element corresponding to a lowercase letter of the alphabet. Most elements are initialized to `NULL`, except for the element at index 7 (corresponding to the letter 'h'), which is initialized to the string "host_short".
- **Use**: This array is used to map single-character lowercase format specifiers to their corresponding string representations in a formatting system.


---
### format_table
- **Type**: ``static const struct format_table_entry[]``
- **Description**: The `format_table` is a static constant array of `struct format_table_entry` that defines a list of key-value pairs used for expanding format strings. Each entry in the array consists of a key, a type (either `FORMAT_TABLE_STRING` or `FORMAT_TABLE_TIME`), and a callback function that provides the value for the key.
- **Use**: This variable is used to map format keys to their corresponding values and callback functions for string expansion in the application.


# Data Structures

---
### format_job
- **Type**: `struct`
- **Members**:
    - `client`: Pointer to a client structure associated with the format job.
    - `tag`: Unsigned integer representing a unique identifier for the format job.
    - `cmd`: Pointer to a constant character string representing the command to be executed.
    - `expanded`: Pointer to a constant character string representing the expanded command.
    - `last`: Time of the last update or execution of the format job.
    - `out`: Pointer to a character string containing the output of the format job.
    - `updated`: Integer flag indicating if the format job has been updated.
    - `job`: Pointer to a job structure associated with the format job.
    - `status`: Integer representing the status of the format job.
    - `entry`: Red-Black tree entry for managing format jobs in a tree structure.
- **Description**: The `format_job` structure is used to manage and execute format jobs within a system, typically involving the execution of commands and handling their output. It contains pointers to associated client and job structures, as well as fields for managing command execution, output, and status. The structure is designed to be part of a Red-Black tree, allowing efficient management and retrieval of format jobs.


---
### format_entry
- **Type**: `struct`
- **Members**:
    - `key`: A pointer to a character string representing the key in the key-value pair.
    - `value`: A pointer to a character string representing the value in the key-value pair.
    - `time`: A time_t value representing the time associated with the format entry.
    - `cb`: A callback function of type format_cb associated with the format entry.
    - `entry`: An RB_ENTRY macro for integrating the format_entry into a red-black tree.
- **Description**: The `format_entry` structure is used to represent an entry in a format tree, which is part of a system for managing key-value pairs and their associated metadata. Each entry consists of a key and a value, both stored as strings, a timestamp indicating when the entry was created or last modified, a callback function for additional processing, and a red-black tree entry for efficient storage and retrieval within a tree structure. This structure is integral to the management of dynamic formatting and expansion of strings within the application.


---
### format_type
- **Type**: `enum`
- **Members**:
    - `FORMAT_TYPE_UNKNOWN`: Represents an unknown format type.
    - `FORMAT_TYPE_SESSION`: Represents a session format type.
    - `FORMAT_TYPE_WINDOW`: Represents a window format type.
    - `FORMAT_TYPE_PANE`: Represents a pane format type.
- **Description**: The `format_type` enumeration defines different types of formats that can be used within the application. It includes four possible values: `FORMAT_TYPE_UNKNOWN` for unknown formats, `FORMAT_TYPE_SESSION` for session-related formats, `FORMAT_TYPE_WINDOW` for window-related formats, and `FORMAT_TYPE_PANE` for pane-related formats. This enumeration is used to categorize and handle different formatting contexts within the application.


---
### format_tree
- **Type**: `struct`
- **Members**:
    - `type`: Specifies the type of format, such as session, window, or pane.
    - `c`: Pointer to a client structure associated with the format.
    - `s`: Pointer to a session structure associated with the format.
    - `wl`: Pointer to a winlink structure associated with the format.
    - `w`: Pointer to a window structure associated with the format.
    - `wp`: Pointer to a window pane structure associated with the format.
    - `pb`: Pointer to a paste buffer structure associated with the format.
    - `item`: Pointer to a command queue item associated with the format.
    - `client`: Pointer to a client structure, possibly redundant with 'c'.
    - `flags`: Integer representing various flags for the format.
    - `tag`: Unsigned integer used as a tag for the format.
    - `m`: Mouse event structure associated with the format.
    - `tree`: Red-black tree head for storing format entries.
- **Description**: The 'format_tree' structure is used to manage and store information related to formatting in the context of a terminal multiplexer, such as tmux. It holds pointers to various related structures like client, session, window, and pane, allowing it to access and format data from these components. The structure also includes a red-black tree to manage format entries, which are key-value pairs used to dynamically generate formatted strings. This structure is central to the process of expanding format strings by replacing placeholders with actual data from the associated components.


---
### format_expand_state
- **Type**: `struct`
- **Members**:
    - `ft`: A pointer to a `format_tree` structure, representing the format tree associated with this state.
    - `loop`: An unsigned integer representing the current recursion depth or loop count.
    - `time`: A `time_t` value representing the current time for formatting purposes.
    - `tm`: A `struct tm` representing the broken-down time corresponding to `time`.
    - `flags`: An integer representing various flags that control the behavior of the format expansion.
- **Description**: The `format_expand_state` structure is used to maintain the state during the expansion of format strings in the tmux application. It holds a reference to a `format_tree`, which contains the context for the expansion, such as client, session, and window information. The `loop` member is used to track the recursion depth to prevent infinite loops. The `time` and `tm` members are used for time-based formatting, allowing the expansion to include time-related data. The `flags` member is used to control specific behaviors during the format expansion process, such as whether to expand time or handle jobs.


---
### format_modifier
- **Type**: `struct`
- **Members**:
    - `modifier`: A character array of size 3 to store the format modifier.
    - `size`: An unsigned integer to store the size of the modifier.
    - `argv`: A pointer to an array of strings representing arguments for the modifier.
    - `argc`: An integer to store the count of arguments in argv.
- **Description**: The `format_modifier` structure is used to represent a format modifier in a format string, typically used in the context of expanding or processing format strings in a program. It contains a small character array to hold the modifier itself, a size field to indicate the length of the modifier, and an array of strings (`argv`) with a count (`argc`) to hold any additional arguments associated with the modifier. This structure is likely used in conjunction with other components to parse and apply formatting rules to strings.


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
    - `cb`: A callback function associated with the format table entry.
- **Description**: The `format_table_entry` structure is used to define entries in a format table, which are key-value pairs used for expanding format strings. Each entry consists of a key, a type indicating whether the entry is a string or a time, and a callback function that is used to retrieve or compute the value associated with the key. This structure is part of a system that processes and expands format strings by replacing keys with their corresponding values.


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
    - If the `tag` values are equal, it compares the `cmd` strings of both `format_job` structures using `strcmp`.
- **Output**:
    - Returns -1 if `fj1` is less than `fj2`, 1 if greater, or the result of `strcmp` for their `cmd` values if the tags are equal.


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
    - An integer indicating the result of the comparison: negative if `fe1` is less than `fe2`, positive if greater, and zero if they are equal.


---
### format_logging<!-- {{#callable:format_logging}} -->
The `format_logging` function checks if logging is enabled based on the log level and verbosity flags.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains flags indicating the logging verbosity.
- **Control Flow**:
    - The function retrieves the current log level using `log_get_level()`.
    - It checks if the log level is not zero or if the `FORMAT_VERBOSE` flag is set in the `ft->flags`.
    - If either condition is true, it returns a non-zero value indicating that logging is enabled.
- **Output**:
    - Returns an integer value: non-zero if logging is enabled, zero otherwise.
- **Functions called**:
    - [`log_get_level`](log.c.driver.md#log_get_level)


---
### printflike<!-- {{#callable:printflike}} -->
The `printflike` function formats a string using a variable argument list and returns the resulting string.
- **Inputs**:
    - `fmt`: A format string that specifies how subsequent arguments are converted for output.
    - `...`: A variable number of arguments that are formatted according to the format string.
- **Control Flow**:
    - The function starts by initializing a variable argument list using `va_start` with the format string `fmt`.
    - It then calls [`xvasprintf`](xmalloc.c.driver.md#xvasprintf), which formats the string according to the provided format and arguments, storing the result in the pointer `s`.
    - Finally, it cleans up the variable argument list with `va_end` and returns the formatted string.
- **Output**:
    - Returns a pointer to a dynamically allocated string containing the formatted output.
- **Functions called**:
    - [`xvasprintf`](xmalloc.c.driver.md#xvasprintf)


---
### format_job_update<!-- {{#callable:format_job_update}} -->
The `format_job_update` function updates the output of a job based on the input received from an event buffer.
- **Inputs**:
    - `job`: A pointer to a `struct job` that represents the job to be updated.
- **Control Flow**:
    - The function retrieves the job's associated data and the input event buffer.
    - It reads lines from the event buffer until there are no more lines, freeing the previous line each time.
    - If no line is read, the function returns early.
    - If a line is read, it marks the job as updated and replaces the previous output with the new line.
    - It logs the updated output along with the job's command.
    - If the job's status is set and the last update time is different from the current time, it updates the last update time and notifies the client if it exists.
- **Output**:
    - The function does not return a value; it modifies the job's output and updates its status internally.
- **Functions called**:
    - [`job_get_data`](job.c.driver.md#job_get_data)
    - [`job_get_event`](job.c.driver.md#job_get_event)
    - [`server_status_client`](server-fn.c.driver.md#server_status_client)


---
### format_job_complete<!-- {{#callable:format_job_complete}} -->
The `format_job_complete` function processes the completion of a job by reading its output from an event buffer and updating the job's status.
- **Inputs**:
    - `job`: A pointer to a `struct job` that represents the job whose completion is being processed.
- **Control Flow**:
    - The function retrieves job data from the `job` structure using [`job_get_data`](job.c.driver.md#job_get_data).
    - It reads the input from the event buffer associated with the job using [`job_get_event`](job.c.driver.md#job_get_event).
    - If the event buffer is empty, it allocates a buffer and copies the data from the event buffer into it.
    - If the buffer is not empty, it uses the line read from the event buffer directly.
    - It logs the job's command and output using `log_debug`.
    - If the output buffer is not empty or the job has been updated, it updates the job's output; otherwise, it frees the buffer.
    - If the job's status is set, it updates the client status and resets the job's status.
- **Output**:
    - The function does not return a value; it modifies the state of the job and its associated data structures.
- **Functions called**:
    - [`job_get_data`](job.c.driver.md#job_get_data)
    - [`job_get_event`](job.c.driver.md#job_get_event)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`server_status_client`](server-fn.c.driver.md#server_status_client)


---
### format_job_get<!-- {{#callable:format_job_get}} -->
The `format_job_get` function retrieves or creates a job associated with a specific command in a format tree.
- **Inputs**:
    - `es`: A pointer to a `format_expand_state` structure that holds the current state of format expansion.
    - `cmd`: A string representing the command associated with the job to be retrieved or created.
- **Control Flow**:
    - Checks if the `client` in the `format_tree` is NULL; if so, it uses a global job tree.
    - If the `client` has no associated jobs, it allocates memory for a new job tree.
    - Creates a `format_job` structure and checks if a job with the same tag and command already exists in the job tree.
    - If the job does not exist, it allocates memory for a new job and initializes it with the command and client information.
    - Expands the command using [`format_expand1`](#format_expand1) and checks if the expanded command differs from the previously stored command.
    - If the command has changed or if forced, it runs the job and updates the job structure accordingly.
    - Handles job output and status updates based on the job's execution state.
    - Returns the output of the job or an empty string if no output is available.
- **Output**:
    - Returns a dynamically allocated string containing the output of the job, or an empty string if there is no output.
- **Functions called**:
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`format_expand1`](#format_expand1)
    - [`job_free`](job.c.driver.md#job_free)
    - [`job_run`](job.c.driver.md#job_run)
    - [`server_client_get_cwd`](server-client.c.driver.md#server_client_get_cwd)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### format_job_tidy<!-- {{#callable:format_job_tidy}} -->
The `format_job_tidy` function cleans up and removes old jobs from a job tree.
- **Inputs**:
    - `jobs`: A pointer to a `format_job_tree` structure containing the jobs to be tidied.
    - `force`: An integer flag indicating whether to force the removal of jobs regardless of their last execution time.
- **Control Flow**:
    - The current time is retrieved using `time(NULL)`.
    - A safe iteration over the `format_job_tree` is performed using `RB_FOREACH_SAFE` to avoid issues when removing elements.
    - For each job, if `force` is not set and the job's last execution time is within the last hour, it is skipped.
    - If the job is to be removed, it is logged for debugging purposes.
    - The job's resources are freed, including its command, expanded command, and output buffer.
    - Finally, the job itself is freed.
- **Output**:
    - The function does not return a value; it modifies the job tree in place by removing old jobs.
- **Functions called**:
    - [`job_free`](job.c.driver.md#job_free)


---
### format_tidy_jobs<!-- {{#callable:format_tidy_jobs}} -->
The `format_tidy_jobs` function cleans up old format jobs for all clients and the global job list.
- **Inputs**:
    - None
- **Control Flow**:
    - Calls [`format_job_tidy`](#format_job_tidy) to clean up the global job list `format_jobs`.
    - Iterates over each `client` in the `clients` list using `TAILQ_FOREACH`.
    - For each client, if the client has jobs (`c->jobs` is not NULL), it calls [`format_job_tidy`](#format_job_tidy) on the client's job list.
- **Output**:
    - The function does not return a value; it performs cleanup operations on the job lists.
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
    - If `gethostname` fails (returns a non-zero value), the function returns an empty string by calling [`xstrdup`](xmalloc.c.driver.md#xstrdup) with an empty string.
    - If `gethostname` succeeds, it returns a duplicate of the hostname string using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
- **Output**:
    - The function returns a dynamically allocated string containing the hostname of the machine, or an empty string if the hostname could not be retrieved.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_host_short<!-- {{#callable:format_cb_host_short}} -->
The `format_cb_host_short` function retrieves the hostname of the current machine and returns it without any domain suffix.
- **Inputs**:
    - None
- **Control Flow**:
    - The function declares a character array `host` to store the hostname and a pointer `cp` for string manipulation.
    - It calls `gethostname` to fill `host` with the current hostname; if this call fails, it returns an empty string.
    - It checks if there is a '.' in the hostname using `strchr`; if found, it replaces the first occurrence with a null terminator, effectively truncating the hostname to its short form.
    - Finally, it returns a duplicate of the short hostname using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
- **Output**:
    - The function outputs a dynamically allocated string containing the short hostname (without domain) or an empty string if the hostname retrieval fails.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_pid<!-- {{#callable:format_cb_pid}} -->
The `format_cb_pid` function retrieves the current process ID and formats it as a string.
- **Inputs**:
    - None
- **Control Flow**:
    - The function calls `getpid()` to obtain the current process ID.
    - It then uses [`xasprintf`](xmalloc.c.driver.md#xasprintf) to format the process ID as a string.
    - Finally, it returns the formatted string.
- **Output**:
    - The output is a pointer to a string containing the current process ID formatted as a decimal number.
- **Functions called**:
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### format_cb_session_attached_list<!-- {{#callable:format_cb_session_attached_list}} -->
The `format_cb_session_attached_list` function generates a comma-separated list of client names attached to a specified session.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains the session information.
- **Control Flow**:
    - Checks if the session pointer in the `format_tree` is NULL; if so, returns NULL.
    - Creates a new `evbuffer` to hold the client names.
    - Iterates over all clients in the `clients` list using `TAILQ_FOREACH`.
    - For each client, checks if its session matches the specified session; if it does, appends the client's name to the buffer.
    - If the buffer is not empty, adds a comma before appending the next client's name.
    - After processing all clients, checks the length of the buffer; if it's greater than zero, allocates memory for the output string and copies the buffer's content into it.
    - Frees the `evbuffer` and returns the generated string of client names.
- **Output**:
    - Returns a dynamically allocated string containing a comma-separated list of names of clients attached to the session, or NULL if no clients are attached or if the session is NULL.
- **Functions called**:
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### format_cb_session_alerts<!-- {{#callable:format_cb_session_alerts}} -->
The `format_cb_session_alerts` function generates a formatted string of alert indicators for each window in a session.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains session and window information.
- **Control Flow**:
    - Checks if the session pointer in the `format_tree` is NULL; if so, returns NULL.
    - Initializes an empty string `alerts` to accumulate alert indicators.
    - Iterates over each `winlink` in the session's windows using `RB_FOREACH`.
    - For each `winlink`, checks if it has alert flags set; if not, continues to the next.
    - Formats the index of the `winlink` and appends it to the `alerts` string, adding specific characters based on the alert flags (e.g., '#', '!', '~').
    - Finally, duplicates the `alerts` string and returns it.
- **Output**:
    - Returns a dynamically allocated string containing a comma-separated list of window indices and their associated alert indicators.
- **Functions called**:
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)
    - [`strlcat`](compat/strlcat.c.driver.md#strlcat)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_session_stack<!-- {{#callable:format_cb_session_stack}} -->
The `format_cb_session_stack` function generates a comma-separated string of the indices of the current and last windows in a session.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains session and window information.
- **Control Flow**:
    - Check if the session pointer in the `format_tree` is NULL; if so, return NULL.
    - Initialize a result string with the index of the current window.
    - Iterate over the last windows in the session using a `TAILQ_FOREACH` loop.
    - For each window, append its index to the result string, separated by commas.
    - Return a duplicated string of the result.
- **Output**:
    - Returns a dynamically allocated string containing the indices of the current window and the last windows in the session, or NULL if the session is not set.
- **Functions called**:
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)
    - [`strlcat`](compat/strlcat.c.driver.md#strlcat)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_window_stack_index<!-- {{#callable:format_cb_window_stack_index}} -->
The `format_cb_window_stack_index` function calculates and returns the index of a specified `winlink` in the stack of windows for a given session.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current window link and session.
- **Control Flow**:
    - The function first checks if the `winlink` pointer in the `format_tree` structure is NULL; if it is, the function returns NULL.
    - If the `winlink` is valid, it retrieves the associated `session` from the `winlink`.
    - It initializes an index counter to zero and iterates through the list of `winlink` structures in the session's last window using a loop.
    - For each `winlink`, it increments the index counter until it finds the `winlink` that matches the one in the `format_tree`.
    - If the loop completes without finding the `winlink`, it returns a string representation of '0'.
    - If the `winlink` is found, it formats the index as a string and returns it.
- **Output**:
    - The function returns a dynamically allocated string representing the index of the specified `winlink` in the session's window stack, or '0' if the `winlink` is not found.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### format_cb_window_linked_sessions_list<!-- {{#callable:format_cb_window_linked_sessions_list}} -->
The `format_cb_window_linked_sessions_list` function generates a comma-separated list of session names linked to a specific window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context, including the window link.
- **Control Flow**:
    - The function first checks if the `winlink` pointer in the `format_tree` structure is NULL; if it is, the function returns NULL.
    - If the `winlink` is valid, it retrieves the associated `window` structure.
    - An `evbuffer` is created to hold the formatted session names, and if memory allocation fails, the program exits with an error.
    - The function iterates over each `winlink` in the `window`'s list of linked windows, appending each session's name to the `evbuffer`, separated by commas.
    - After building the list, the function checks if the buffer has any content; if so, it allocates memory for the output string and copies the buffer's content into it.
    - Finally, the `evbuffer` is freed, and the function returns the formatted string of session names.
- **Output**:
    - Returns a dynamically allocated string containing a comma-separated list of session names linked to the specified window, or NULL if there are no linked sessions or if an error occurs.
- **Functions called**:
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### format_cb_window_active_sessions<!-- {{#callable:format_cb_window_active_sessions}} -->
The `format_cb_window_active_sessions` function counts the number of active sessions in a given window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current window and its associated sessions.
- **Control Flow**:
    - The function first checks if the `winlink` pointer in the `format_tree` structure is NULL; if it is, the function returns NULL.
    - If the `winlink` is not NULL, it retrieves the associated `window` from the `winlink`.
    - It then iterates over all `winlink` entries in the `window`'s list using a loop.
    - For each `winlink`, it checks if the `curw` (current window) of the session associated with the `winlink` is the same as the current `winlink` being iterated.
    - If they match, it increments a counter that tracks the number of active sessions.
    - Finally, it formats the count into a string and returns it.
- **Output**:
    - The function returns a dynamically allocated string representing the number of active sessions in the window, or NULL if there are no active sessions.
- **Functions called**:
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### format_cb_window_active_sessions_list<!-- {{#callable:format_cb_window_active_sessions_list}} -->
The `format_cb_window_active_sessions_list` function generates a comma-separated list of names of active sessions associated with the current window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context, including the current window and session.
- **Control Flow**:
    - Checks if the `winlink` pointer in the `format_tree` is NULL; if so, it returns NULL.
    - Retrieves the `window` associated with the `winlink` from the `format_tree`.
    - Creates a new `evbuffer` to hold the session names, checking for memory allocation failure.
    - Iterates over each `winlink` in the `window`'s list, checking if the `curw` (current window) of the session matches the current `winlink`.
    - If a match is found, it appends the session name to the `evbuffer`, separating names with commas as needed.
    - After processing all `winlinks`, it checks the length of the `evbuffer` and, if not empty, allocates a string to hold the result and copies the data from the `evbuffer`.
    - Frees the `evbuffer` and returns the resulting string of session names.
- **Output**:
    - Returns a dynamically allocated string containing a comma-separated list of active session names, or NULL if there are no active sessions or if memory allocation fails.
- **Functions called**:
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### format_cb_window_active_clients<!-- {{#callable:format_cb_window_active_clients}} -->
The `format_cb_window_active_clients` function counts the number of active clients associated with the currently focused window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context, including the window link.
- **Control Flow**:
    - The function first checks if the `wl` (window link) in the `format_tree` is NULL; if it is, the function returns NULL.
    - If `wl` is not NULL, it retrieves the associated `window` from `wl`.
    - The function then iterates over all `clients` using a `TAILQ_FOREACH` loop.
    - For each `client`, it checks if the `client` has an associated `session` and if the current window of that session matches the window being processed.
    - If both conditions are met, it increments a counter `n`.
    - After iterating through all clients, the function formats the count `n` into a string and returns it.
- **Output**:
    - The function returns a dynamically allocated string representing the number of active clients in the current window, or NULL if there are no active clients or if the window link is NULL.
- **Functions called**:
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### format_cb_window_active_clients_list<!-- {{#callable:format_cb_window_active_clients_list}} -->
The `format_cb_window_active_clients_list` function generates a comma-separated list of active client names associated with the currently active window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context, including the active window.
- **Control Flow**:
    - Check if the `wl` (winlink) member of the `format_tree` is NULL; if it is, return NULL.
    - Retrieve the `window` associated with the winlink.
    - Create a new `evbuffer` to hold the list of active client names.
    - Iterate over all clients using `TAILQ_FOREACH`.
    - For each client, check if it has an associated session; if not, continue to the next client.
    - If the client's session's current window matches the active window, append the client's name to the buffer, separating names with a comma.
    - After processing all clients, check the length of the buffer; if it's greater than zero, allocate memory for the output string and copy the buffer's contents into it.
    - Free the `evbuffer` and return the output string.
- **Output**:
    - Returns a dynamically allocated string containing a comma-separated list of active client names, or NULL if there are no active clients or if the winlink is NULL.
- **Functions called**:
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### format_cb_window_layout<!-- {{#callable:format_cb_window_layout}} -->
The `format_cb_window_layout` function retrieves the layout of a window, returning either the saved layout or the current layout.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the window whose layout is to be retrieved.
- **Control Flow**:
    - Check if the window pointer `w` in the `format_tree` is NULL; if it is, return NULL.
    - Check if the `saved_layout_root` of the window is not NULL; if it exists, call [`layout_dump`](layout-custom.c.driver.md#layout_dump) with `saved_layout_root` and return the result.
    - If `saved_layout_root` is NULL, call [`layout_dump`](layout-custom.c.driver.md#layout_dump) with `layout_root` and return the result.
- **Output**:
    - Returns a pointer to the layout representation of the window, which can be either the saved layout or the current layout, or NULL if the window is not set.
- **Functions called**:
    - [`layout_dump`](layout-custom.c.driver.md#layout_dump)


---
### format_cb_window_visible_layout<!-- {{#callable:format_cb_window_visible_layout}} -->
The `format_cb_window_visible_layout` function retrieves the layout of a window if it is visible.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the window and its layout.
- **Control Flow**:
    - The function first retrieves the `window` pointer from the `format_tree` structure.
    - If the `window` pointer is `NULL`, the function returns `NULL` immediately.
    - If the `window` pointer is valid, the function calls [`layout_dump`](layout-custom.c.driver.md#layout_dump) with the `layout_root` of the window and returns its result.
- **Output**:
    - The function returns a pointer to the layout representation of the window, or `NULL` if the window is not visible.
- **Functions called**:
    - [`layout_dump`](layout-custom.c.driver.md#layout_dump)


---
### format_cb_start_command<!-- {{#callable:format_cb_start_command}} -->
The `format_cb_start_command` function retrieves the start command of a window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context.
- **Control Flow**:
    - The function first retrieves the `window_pane` associated with the `format_tree` structure.
    - If the `window_pane` pointer is `NULL`, the function returns `NULL` immediately.
    - If the `window_pane` is valid, it calls the [`cmd_stringify_argv`](cmd.c.driver.md#cmd_stringify_argv) function with the argument count and argument vector of the `window_pane` to convert them into a command string.
    - Finally, it returns the result of the [`cmd_stringify_argv`](cmd.c.driver.md#cmd_stringify_argv) function.
- **Output**:
    - The function returns a pointer to a string representing the start command of the window pane, or `NULL` if the pane is not valid.
- **Functions called**:
    - [`cmd_stringify_argv`](cmd.c.driver.md#cmd_stringify_argv)


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
    - The function returns a pointer to a string representing the current working directory of the window pane, or NULL if the pane is not set, or an empty string if the current working directory is not set.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_current_command<!-- {{#callable:format_cb_current_command}} -->
The `format_cb_current_command` function retrieves the current command being executed in a specified window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - Check if the `window_pane` pointer in the `format_tree` is NULL or if the `shell` pointer in the `window_pane` is NULL; if so, return NULL.
    - Attempt to retrieve the command name associated with the file descriptor and terminal of the window pane using `osdep_get_name`.
    - If the command name is NULL or an empty string, free the command name and attempt to stringify the command arguments using [`cmd_stringify_argv`](cmd.c.driver.md#cmd_stringify_argv).
    - If the stringified command is also NULL or empty, free it and duplicate the shell name from the window pane.
    - Parse the command name using [`parse_window_name`](names.c.driver.md#parse_window_name) and free the command name before returning the parsed value.
- **Output**:
    - Returns a dynamically allocated string containing the parsed command name, or NULL if no command could be determined.
- **Functions called**:
    - [`cmd_stringify_argv`](cmd.c.driver.md#cmd_stringify_argv)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`parse_window_name`](names.c.driver.md#parse_window_name)


---
### format_cb_current_path<!-- {{#callable:format_cb_current_path}} -->
The `format_cb_current_path` function retrieves the current working directory of a specified window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is NULL; if it is, the function returns NULL.
    - If `wp` is not NULL, it calls `osdep_get_cwd` with the file descriptor of the window pane to retrieve the current working directory.
    - If `osdep_get_cwd` returns NULL, indicating an error in retrieving the current directory, the function returns NULL.
    - If a valid current working directory is obtained, it duplicates the string using [`xstrdup`](xmalloc.c.driver.md#xstrdup) and returns the duplicated string.
- **Output**:
    - The function returns a pointer to a string containing the current working directory of the window pane, or NULL if an error occurs.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_history_bytes<!-- {{#callable:format_cb_history_bytes}} -->
Calculates the total size in bytes of the history buffer for a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the format context, including the window pane.
- **Control Flow**:
    - Checks if the `window_pane` pointer in the `format_tree` is NULL; if so, returns NULL.
    - Retrieves the `grid` associated with the `window_pane`.
    - Iterates over each line in the grid, calculating the total size by summing the sizes of cell data and extended data for each line.
    - Adds the size of the grid lines themselves to the total size.
    - Formats the total size as a string and returns it.
- **Output**:
    - Returns a pointer to a string representing the total size in bytes of the history buffer, or NULL if the `window_pane` is not set.
- **Functions called**:
    - [`grid_get_line`](grid.c.driver.md#grid_get_line)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### format_cb_history_all_bytes<!-- {{#callable:format_cb_history_all_bytes}} -->
The `format_cb_history_all_bytes` function formats and returns a string containing the history statistics of a window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - Check if the `window_pane` pointer in the `format_tree` is NULL; if so, return NULL.
    - Retrieve the `grid` associated with the `window_pane`.
    - Calculate the total number of lines in the grid by summing the height and the scroll offset.
    - Iterate through each line of the grid to accumulate the total number of cells and extended cells.
    - Format the accumulated statistics into a string using [`xasprintf`](xmalloc.c.driver.md#xasprintf).
- **Output**:
    - Returns a dynamically allocated string containing the number of lines, size of each line, total number of cells, total size of cell data, total number of extended cells, and total size of extended cell data, separated by commas.
- **Functions called**:
    - [`grid_get_line`](grid.c.driver.md#grid_get_line)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### format_cb_pane_tabs<!-- {{#callable:format_cb_pane_tabs}} -->
The `format_cb_pane_tabs` function generates a comma-separated string of tab indices for a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the window pane.
- **Control Flow**:
    - Check if the `window_pane` pointer (`wp`) in the `format_tree` is NULL; if it is, return NULL.
    - Create a new `evbuffer` to hold the tab indices.
    - Iterate over the range of the pane's grid width (`sx`), checking each index to see if it is set in the `tabs` bitmap.
    - If a tab index is set, append it to the `evbuffer`, preceded by a comma if it's not the first index.
    - After the loop, if the `evbuffer` contains data, allocate a string to hold the formatted output and copy the data from the `evbuffer`.
    - Free the `evbuffer` and return the formatted string.
- **Output**:
    - Returns a dynamically allocated string containing the comma-separated tab indices, or NULL if no tabs are set or if memory allocation fails.
- **Functions called**:
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### format_cb_pane_fg<!-- {{#callable:format_cb_pane_fg}} -->
The `format_cb_pane_fg` function retrieves the foreground color of a specified window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first retrieves the `window_pane` associated with the `format_tree` structure.
    - If the `window_pane` is NULL, the function returns NULL immediately.
    - The function then calls [`tty_default_colours`](tty.c.driver.md#tty_default_colours) to populate a `grid_cell` structure with the default colors for the pane.
    - Finally, it converts the foreground color from the `grid_cell` to a string using [`colour_tostring`](colour.c.driver.md#colour_tostring) and returns a duplicate of that string.
- **Output**:
    - The function returns a dynamically allocated string representing the foreground color of the pane, or NULL if the pane is not available.
- **Functions called**:
    - [`tty_default_colours`](tty.c.driver.md#tty_default_colours)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`colour_tostring`](colour.c.driver.md#colour_tostring)


---
### format_cb_pane_bg<!-- {{#callable:format_cb_pane_bg}} -->
The `format_cb_pane_bg` function retrieves the background color of a specified pane in a terminal multiplexer.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is NULL; if it is, the function returns NULL.
    - If `wp` is not NULL, it calls [`tty_default_colours`](tty.c.driver.md#tty_default_colours) to populate a `grid_cell` structure (`gc`) with the default colors for the pane.
    - Finally, it converts the background color from the `grid_cell` structure to a string using [`colour_tostring`](colour.c.driver.md#colour_tostring) and returns a duplicate of that string.
- **Output**:
    - The function returns a pointer to a string representing the background color of the pane, or NULL if the pane does not exist.
- **Functions called**:
    - [`tty_default_colours`](tty.c.driver.md#tty_default_colours)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`colour_tostring`](colour.c.driver.md#colour_tostring)


---
### format_cb_session_group_list<!-- {{#callable:format_cb_session_group_list}} -->
The `format_cb_session_group_list` function generates a comma-separated list of session names within a specific session group.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains the session and other context needed for formatting.
- **Control Flow**:
    - Check if the session pointer in the `format_tree` is NULL; if so, return NULL.
    - Retrieve the session group associated with the session using [`session_group_contains`](session.c.driver.md#session_group_contains).
    - If no session group is found, return NULL.
    - Create a new `evbuffer` to hold the formatted session names.
    - Iterate over each session in the session group using `TAILQ_FOREACH`.
    - If the buffer is not empty, add a comma before appending the next session name.
    - Append the session name to the buffer using `evbuffer_add_printf`.
    - After the loop, check the length of the buffer; if it's not zero, allocate memory for the output string and copy the buffer's content into it.
    - Free the `evbuffer` and return the formatted string.
- **Output**:
    - Returns a dynamically allocated string containing a comma-separated list of session names, or NULL if there are no sessions or if an error occurs.
- **Functions called**:
    - [`session_group_contains`](session.c.driver.md#session_group_contains)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### format_cb_session_group_attached_list<!-- {{#callable:format_cb_session_group_attached_list}} -->
The `format_cb_session_group_attached_list` function generates a comma-separated list of client names that are attached to the same session group as the specified session.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains the session information.
- **Control Flow**:
    - Checks if the session pointer in the `format_tree` is NULL; if so, returns NULL.
    - Retrieves the session group associated with the session; if none exists, returns NULL.
    - Creates a new `evbuffer` to hold the output string; if memory allocation fails, it calls `fatalx` to terminate the program.
    - Iterates over all clients, checking if each client's session is part of the session group.
    - If a client's session matches, appends the client's name to the `evbuffer`, separating names with commas.
    - After processing all clients, checks the length of the `evbuffer` and, if not empty, allocates memory for the output string and copies the buffer's content into it.
    - Frees the `evbuffer` and returns the constructed string of client names.
- **Output**:
    - Returns a dynamically allocated string containing the names of clients attached to the same session group, or NULL if no clients are found or if an error occurs.
- **Functions called**:
    - [`session_group_contains`](session.c.driver.md#session_group_contains)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### format_cb_pane_in_mode<!-- {{#callable:format_cb_pane_in_mode}} -->
The `format_cb_pane_in_mode` function counts the number of modes currently active in a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the window pane.
- **Control Flow**:
    - Check if the `window_pane` pointer (`wp`) in the `format_tree` is NULL; if it is, return NULL.
    - Iterate through the list of `window_mode_entry` structures associated with the `window_pane` using `TAILQ_FOREACH` to count the number of active modes.
    - Use [`xasprintf`](xmalloc.c.driver.md#xasprintf) to format the count of active modes into a string.
- **Output**:
    - Returns a dynamically allocated string representing the number of active modes in the window pane, or NULL if the pane is not set.
- **Functions called**:
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### format_cb_pane_at_top<!-- {{#callable:format_cb_pane_at_top}} -->
The `format_cb_pane_at_top` function checks if a pane is positioned at the top of its window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context.
- **Control Flow**:
    - The function first retrieves the `window_pane` from the `format_tree` structure.
    - If the `window_pane` is NULL, the function returns NULL.
    - It then retrieves the associated `window` from the `window_pane`.
    - The function checks the `pane-border-status` option of the window to determine if the pane's border is at the top.
    - Based on the `pane-border-status`, it sets a flag indicating whether the pane is at the top or not.
    - Finally, it formats the flag as a string and returns it.
- **Output**:
    - The function returns a string representation of a flag (0 or 1) indicating whether the pane is at the top of the window.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### format_cb_pane_at_bottom<!-- {{#callable:format_cb_pane_at_bottom}} -->
The `format_cb_pane_at_bottom` function checks if a pane is positioned at the bottom of its window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` is NULL; if it is, the function returns NULL.
    - If `wp` is valid, it retrieves the associated `window` object.
    - It then checks the `pane-border-status` option of the window to determine if the pane's border status is set to 'bottom'.
    - Based on the border status, it calculates a flag indicating whether the pane is at the bottom of the window.
    - Finally, it formats the flag value as a string and returns it.
- **Output**:
    - The function returns a string representation of an integer (0 or 1) indicating whether the pane is at the bottom of the window.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### format_cb_cursor_character<!-- {{#callable:format_cb_cursor_character}} -->
The `format_cb_cursor_character` function retrieves the character at the current cursor position in a specified window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer in the `format_tree` is NULL; if it is, the function returns NULL.
    - If the `window_pane` is valid, it retrieves the cell at the current cursor position using [`grid_view_get_cell`](grid-view.c.driver.md#grid_view_get_cell).
    - It then checks if the cell does not have the `GRID_FLAG_PADDING` flag set.
    - If the cell is valid and not a padding cell, it formats the character data into a string using [`xasprintf`](xmalloc.c.driver.md#xasprintf) and returns it.
- **Output**:
    - The function returns a dynamically allocated string containing the character at the cursor position, or NULL if the cursor is in a padding cell or if the window pane is invalid.
- **Functions called**:
    - [`grid_view_get_cell`](grid-view.c.driver.md#grid_view_get_cell)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### format_cb_cursor_colour<!-- {{#callable:format_cb_cursor_colour}} -->
The `format_cb_cursor_colour` function retrieves the current cursor color of a window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context.
- **Control Flow**:
    - The function first retrieves the `window_pane` associated with the `format_tree`.
    - If the `window_pane` or its `screen` is NULL, the function returns NULL.
    - It checks if the `ccolour` (current cursor color) of the screen is not -1.
    - If `ccolour` is valid, it converts it to a string using [`colour_tostring`](colour.c.driver.md#colour_tostring) and returns a duplicate of that string.
    - If `ccolour` is -1, it retrieves the `default_ccolour` and returns its string representation.
- **Output**:
    - Returns a dynamically allocated string representing the cursor color, or NULL if the `window_pane` or its `screen` is not available.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`colour_tostring`](colour.c.driver.md#colour_tostring)


---
### format_cb_mouse_word<!-- {{#callable:format_cb_mouse_word}} -->
The `format_cb_mouse_word` function retrieves the word at the mouse cursor's position in a terminal pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including mouse event data.
- **Control Flow**:
    - Checks if the mouse event in the `format_tree` is valid; if not, returns NULL.
    - Retrieves the `window_pane` associated with the mouse event; if it cannot be found, returns NULL.
    - Determines the mouse cursor's position within the pane; if the position is invalid, returns NULL.
    - Checks if the pane has any active modes; if it does and is not in 'no mode', retrieves the word at the cursor's position using [`window_copy_get_word`](window-copy.c.driver.md#window_copy_get_word).
    - If no active modes are present, retrieves the grid associated with the pane and calls [`format_grid_word`](#format_grid_word) to get the word at the cursor's coordinates.
- **Output**:
    - Returns a pointer to a string containing the word at the mouse cursor's position, or NULL if any checks fail.
- **Functions called**:
    - [`cmd_mouse_pane`](cmd.c.driver.md#cmd_mouse_pane)
    - [`cmd_mouse_at`](cmd.c.driver.md#cmd_mouse_at)
    - [`window_pane_mode`](window.c.driver.md#window_pane_mode)
    - [`window_copy_get_word`](window-copy.c.driver.md#window_copy_get_word)
    - [`format_grid_word`](#format_grid_word)


---
### format_cb_mouse_hyperlink<!-- {{#callable:format_cb_mouse_hyperlink}} -->
The `format_cb_mouse_hyperlink` function retrieves a hyperlink from a mouse event in a terminal window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including mouse event data.
- **Control Flow**:
    - Checks if the mouse event in the `format_tree` is valid; if not, returns NULL.
    - Retrieves the `window_pane` associated with the mouse event; if it fails, returns NULL.
    - Determines the mouse coordinates (x, y) within the `window_pane`; if it fails, returns NULL.
    - Checks if the `window_pane` has any active modes; if it does and is not in 'no mode', retrieves the hyperlink from the window copy mode.
    - If no modes are active, retrieves the grid associated with the `window_pane` and calls [`format_grid_hyperlink`](#format_grid_hyperlink) to get the hyperlink based on the coordinates.
- **Output**:
    - Returns a pointer to a string containing the hyperlink if found, or NULL if no hyperlink is available.
- **Functions called**:
    - [`cmd_mouse_pane`](cmd.c.driver.md#cmd_mouse_pane)
    - [`cmd_mouse_at`](cmd.c.driver.md#cmd_mouse_at)
    - [`window_pane_mode`](window.c.driver.md#window_pane_mode)
    - [`window_copy_get_hyperlink`](window-copy.c.driver.md#window_copy_get_hyperlink)
    - [`format_grid_hyperlink`](#format_grid_hyperlink)


---
### format_cb_mouse_line<!-- {{#callable:format_cb_mouse_line}} -->
The `format_cb_mouse_line` function retrieves the text line at the mouse cursor's position in a terminal window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including mouse event data.
- **Control Flow**:
    - Checks if the mouse event in the `format_tree` is valid; if not, returns NULL.
    - Retrieves the `window_pane` associated with the mouse event; if it fails, returns NULL.
    - Determines the mouse cursor's position within the `window_pane`; if it fails, returns NULL.
    - Checks if the `window_pane` has any active modes; if it does and is not in 'no mode', retrieves the line from the window copy.
    - If no modes are active, retrieves the grid associated with the `window_pane` and formats the line from the grid based on the cursor's Y position.
- **Output**:
    - Returns a pointer to a string containing the text of the line at the mouse cursor's Y position, or NULL if any checks fail.
- **Functions called**:
    - [`cmd_mouse_pane`](cmd.c.driver.md#cmd_mouse_pane)
    - [`cmd_mouse_at`](cmd.c.driver.md#cmd_mouse_at)
    - [`window_pane_mode`](window.c.driver.md#window_pane_mode)
    - [`window_copy_get_line`](window-copy.c.driver.md#window_copy_get_line)
    - [`format_grid_line`](#format_grid_line)


---
### format_cb_mouse_status_line<!-- {{#callable:format_cb_mouse_status_line}} -->
The `format_cb_mouse_status_line` function formats and returns the mouse status line position based on the current mouse event.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains the current state of the format, including mouse event information.
- **Control Flow**:
    - The function first checks if the mouse event is valid by verifying `ft->m.valid`.
    - It then checks if the client associated with the format tree is not NULL and that the terminal has started.
    - If the `statusat` field is 0 and the mouse's y-coordinate is less than the number of status lines, it assigns the y-coordinate to `y`.
    - If `statusat` is greater than 0 and the mouse's y-coordinate is greater than or equal to `statusat`, it adjusts `y` by subtracting `statusat` from the mouse's y-coordinate.
    - If neither condition is met, the function returns NULL.
    - Finally, it formats the value of `y` as a string and returns it.
- **Output**:
    - The function returns a dynamically allocated string representing the y-coordinate of the mouse status line, or NULL if the conditions for valid input are not met.
- **Functions called**:
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### format_cb_mouse_status_range<!-- {{#callable:format_cb_mouse_status_range}} -->
The `format_cb_mouse_status_range` function retrieves the status range of the mouse cursor in a terminal interface.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including mouse event data.
- **Control Flow**:
    - Checks if the mouse event data in `ft` is valid; if not, returns NULL.
    - Checks if the associated client is valid and has started; if not, returns NULL.
    - Determines the x and y coordinates based on the status line position and mouse position.
    - Calls [`status_get_range`](status.c.driver.md#status_get_range) to get the style range based on the calculated coordinates.
    - Returns NULL if the style range is not found or if its type is `STYLE_RANGE_NONE`.
    - Returns a string representation of the style range type (e.g., 'left', 'right', etc.) based on the type of the style range.
- **Output**:
    - Returns a dynamically allocated string describing the style range of the mouse cursor, or NULL if no valid range is found.
- **Functions called**:
    - [`status_get_range`](status.c.driver.md#status_get_range)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_alternate_on<!-- {{#callable:format_cb_alternate_on}} -->
The `format_cb_alternate_on` function checks if the alternate screen buffer is active for a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `saved_grid` member of the `base` structure within `wp` is not NULL.
    - If `saved_grid` is not NULL, it returns a duplicated string '1', indicating that the alternate screen is active.
    - If `saved_grid` is NULL, it returns a duplicated string '0', indicating that the alternate screen is not active.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a duplicated string '1' if the alternate screen buffer is active, '0' if it is not, or NULL if the window pane is not set.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_alternate_saved_x<!-- {{#callable:format_cb_alternate_saved_x}} -->
The `format_cb_alternate_saved_x` function retrieves the saved x-coordinate of the cursor position in a window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it retrieves the `saved_cx` value from the `base` member of the `wp` structure and formats it as an unsigned integer string using `format_printf`.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string representing the saved x-coordinate if the window pane exists, or NULL if it does not.


---
### format_cb_alternate_saved_y<!-- {{#callable:format_cb_alternate_saved_y}} -->
The `format_cb_alternate_saved_y` function retrieves the saved Y coordinate of a window pane if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it retrieves the `saved_cy` value from the `base` member of the `wp` structure and formats it as an unsigned integer string using `format_printf`.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string containing the saved Y coordinate of the window pane if it exists, or NULL if the pane is not available.


---
### format_cb_buffer_name<!-- {{#callable:format_cb_buffer_name}} -->
The `format_cb_buffer_name` function retrieves the name of a paste buffer if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the paste buffer.
- **Control Flow**:
    - The function checks if the `pb` (paste buffer) member of the `format_tree` structure is not NULL.
    - If `pb` is not NULL, it calls [`paste_buffer_name`](paste.c.driver.md#paste_buffer_name) to get the name of the paste buffer and duplicates the string using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - If `pb` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a duplicated string containing the name of the paste buffer, or NULL if the paste buffer does not exist.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`paste_buffer_name`](paste.c.driver.md#paste_buffer_name)


---
### format_cb_buffer_sample<!-- {{#callable:format_cb_buffer_sample}} -->
The `format_cb_buffer_sample` function generates a sample representation of a paste buffer if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context, including a reference to a paste buffer.
- **Control Flow**:
    - The function first checks if the `pb` (paste buffer) member of the `format_tree` structure is not NULL.
    - If `pb` is not NULL, it calls the [`paste_make_sample`](paste.c.driver.md#paste_make_sample) function with `ft->pb` as an argument to generate a sample.
    - If `pb` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to the generated sample if the paste buffer exists, or NULL if it does not.
- **Functions called**:
    - [`paste_make_sample`](paste.c.driver.md#paste_make_sample)


---
### format_cb_buffer_size<!-- {{#callable:format_cb_buffer_size}} -->
The `format_cb_buffer_size` function retrieves the size of the paste buffer and formats it as a string.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains the context for formatting, including a reference to the paste buffer.
- **Control Flow**:
    - The function checks if the paste buffer (`ft->pb`) is not NULL.
    - If the paste buffer is valid, it calls [`paste_buffer_data`](paste.c.driver.md#paste_buffer_data) to retrieve the size of the buffer.
    - The size is then formatted into a string using `format_printf` and returned.
    - If the paste buffer is NULL, the function returns NULL.
- **Output**:
    - Returns a pointer to a string representing the size of the paste buffer, or NULL if the paste buffer is not available.
- **Functions called**:
    - [`paste_buffer_data`](paste.c.driver.md#paste_buffer_data)


---
### format_cb_client_cell_height<!-- {{#callable:format_cb_client_cell_height}} -->
The `format_cb_client_cell_height` function retrieves the height of a client's terminal cell in pixels.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client and its terminal.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL and if the `TTY_STARTED` flag is set in the client's terminal flags.
    - If both conditions are met, it calls `format_printf` to format and return the height of the terminal cell in pixels.
    - If the conditions are not met, it returns NULL.
- **Output**:
    - Returns a formatted string representing the height of the client's terminal cell in pixels, or NULL if the conditions are not satisfied.


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
    - The function returns a pointer to a string containing the width of the client's terminal cell in pixels, or NULL if the client is not initialized or the terminal has not started.


---
### format_cb_client_control_mode<!-- {{#callable:format_cb_client_control_mode}} -->
The `format_cb_client_control_mode` function checks if a client is in control mode and returns a string representation of that state.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client and its state.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the client exists, it checks if the `flags` field of the client contains the `CLIENT_CONTROL` flag.
    - If the `CLIENT_CONTROL` flag is set, it returns a duplicated string '1'.
    - If the `CLIENT_CONTROL` flag is not set, it returns a duplicated string '0'.
    - If the client pointer is NULL, it returns NULL.
- **Output**:
    - The function returns a string '1' if the client is in control mode, '0' if not, or NULL if the client does not exist.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_client_discarded<!-- {{#callable:format_cb_client_discarded}} -->
The `format_cb_client_discarded` function retrieves the number of discarded items for a client if the client is not null.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client and other formatting details.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not null.
    - If the `client` is not null, it retrieves the `discarded` count from the `client` structure and formats it as a string using `format_printf`.
    - If the `client` is null, the function returns null.
- **Output**:
    - The function returns a formatted string representing the number of discarded items for the client, or null if the client is not present.


---
### format_cb_client_flags<!-- {{#callable:format_cb_client_flags}} -->
The `format_cb_client_flags` function retrieves the flags associated with a client if the client is not null.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client whose flags are to be retrieved.
- **Control Flow**:
    - The function checks if the `client` pointer in the `format_tree` structure is not null.
    - If the client is not null, it calls [`server_client_get_flags`](server-client.c.driver.md#server_client_get_flags) to retrieve the client's flags and duplicates the string using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - If the client is null, the function returns null.
- **Output**:
    - The function returns a duplicated string of the client's flags if the client exists; otherwise, it returns null.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`server_client_get_flags`](server-client.c.driver.md#server_client_get_flags)


---
### format_cb_client_height<!-- {{#callable:format_cb_client_height}} -->
The `format_cb_client_height` function retrieves the height of the client terminal if it has been started.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client and its terminal.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL.
    - It then checks if the `tty` flags of the client indicate that the terminal has started (i.e., `TTY_STARTED` flag is set).
    - If both conditions are met, it retrieves the `sy` (screen height) from the client's terminal and formats it as an unsigned integer string using `format_printf`.
    - If either condition fails, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string representing the height of the client terminal in pixels, or NULL if the client is not initialized or the terminal has not started.


---
### format_cb_client_key_table<!-- {{#callable:format_cb_client_key_table}} -->
The `format_cb_client_key_table` function retrieves the name of the key table associated with a client.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client and its associated key table.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` is not NULL, it retrieves the name of the key table from the `keytable` structure associated with the client.
    - The name is duplicated using [`xstrdup`](xmalloc.c.driver.md#xstrdup) to allocate memory for the string.
    - If the `client` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a newly allocated string containing the name of the key table, or NULL if the client is not set.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_client_last_session<!-- {{#callable:format_cb_client_last_session}} -->
The `format_cb_client_last_session` function retrieves the name of the last session associated with a client if it is alive.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client and session.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` is not NULL.
    - It then checks if the `last_session` pointer of the client is not NULL.
    - Next, it verifies if the last session is alive by calling the [`session_alive`](session.c.driver.md#session_alive) function.
    - If all conditions are met, it duplicates the name of the last session and returns it.
    - If any condition fails, it returns NULL.
- **Output**:
    - The function returns a pointer to a duplicated string containing the name of the last session if it exists and is alive; otherwise, it returns NULL.
- **Functions called**:
    - [`session_alive`](session.c.driver.md#session_alive)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_client_name<!-- {{#callable:format_cb_client_name}} -->
The `format_cb_client_name` function retrieves the name of a client from a `format_tree` structure.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client.
- **Control Flow**:
    - The function checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` pointer is valid, it duplicates the `name` string of the client using [`xstrdup`](xmalloc.c.driver.md#xstrdup) and returns it.
    - If the `client` pointer is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a duplicated string containing the client's name if the client exists, or NULL if the client does not exist.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_client_pid<!-- {{#callable:format_cb_client_pid}} -->
The `format_cb_client_pid` function retrieves the process ID of a client if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client.
- **Control Flow**:
    - The function checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` exists, it retrieves the `pid` from the `client` structure and formats it as a long integer string.
    - If the `client` does not exist, the function returns NULL.
- **Output**:
    - Returns a string representation of the client's process ID if the client exists, otherwise returns NULL.


---
### format_cb_client_prefix<!-- {{#callable:format_cb_client_prefix}} -->
The `format_cb_client_prefix` function determines the prefix state of a client based on its key table.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client and its associated data.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the client is valid, it retrieves the key table name associated with the client.
    - It then compares the client's key table name with the retrieved name.
    - If they are equal, it returns a duplicated string '0'; otherwise, it returns '1'.
    - If the client pointer is NULL, the function returns NULL.
- **Output**:
    - The function returns a duplicated string '0' or '1' based on whether the client's key table name matches the retrieved name, or NULL if the client is not set.
- **Functions called**:
    - [`server_client_get_key_table`](server-client.c.driver.md#server_client_get_key_table)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


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
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_client_session<!-- {{#callable:format_cb_client_session}} -->
The `format_cb_client_session` function retrieves the name of the session associated with a given client.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client and session.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` is valid, it then checks if the `session` pointer within the `client` is also not NULL.
    - If both checks pass, it returns a duplicate of the session's name using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - If either check fails, it returns NULL.
- **Output**:
    - The function returns a pointer to a string containing the name of the session if it exists, or NULL if the client or session is not valid.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_client_termfeatures<!-- {{#callable:format_cb_client_termfeatures}} -->
The `format_cb_client_termfeatures` function retrieves the terminal features of a client if the client is not null.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client and its terminal features.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not null.
    - If the `client` is not null, it calls [`tty_get_features`](tty-features.c.driver.md#tty_get_features) with the terminal features of the client and duplicates the returned string using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - If the `client` is null, the function returns null.
- **Output**:
    - The function returns a duplicated string of the terminal features if the client is valid, otherwise it returns null.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`tty_get_features`](tty-features.c.driver.md#tty_get_features)


---
### format_cb_client_termname<!-- {{#callable:format_cb_client_termname}} -->
The `format_cb_client_termname` function retrieves the terminal name of a client if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client.
- **Control Flow**:
    - The function checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` is not NULL, it returns a duplicate string of the `term_name` from the `client` structure.
    - If the `client` is NULL, it returns NULL.
- **Output**:
    - Returns a pointer to a duplicated string containing the terminal name of the client, or NULL if the client does not exist.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_client_termtype<!-- {{#callable:format_cb_client_termtype}} -->
The `format_cb_client_termtype` function retrieves the terminal type of a client if available.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client and its terminal.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` is valid, it checks if the `term_type` field of the `client` is NULL.
    - If `term_type` is NULL, it returns an empty string.
    - If `term_type` is not NULL, it duplicates the string and returns it.
    - If the `client` pointer is NULL, it returns NULL.
- **Output**:
    - The function returns a duplicated string of the terminal type if available, an empty string if the terminal type is NULL, or NULL if the client is not set.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_client_tty<!-- {{#callable:format_cb_client_tty}} -->
The `format_cb_client_tty` function retrieves the terminal name associated with a client.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client and its associated terminal.
- **Control Flow**:
    - The function checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` pointer is valid, it returns a duplicate string of the `ttyname` associated with the client.
    - If the `client` pointer is NULL, it returns NULL.
- **Output**:
    - Returns a pointer to a string containing the terminal name if the client is valid; otherwise, returns NULL.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_client_uid<!-- {{#callable:format_cb_client_uid}} -->
The `format_cb_client_uid` function retrieves the user ID of the client associated with a given `format_tree` structure.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client whose user ID is to be retrieved.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` is valid, it calls [`proc_get_peer_uid`](proc.c.driver.md#proc_get_peer_uid) with the `peer` associated with the client to retrieve the user ID.
    - If the retrieved user ID is valid (not equal to (uid_t)-1), it formats the user ID as a string using `format_printf` and returns it.
    - If the `client` is NULL or the user ID retrieval fails, the function returns NULL.
- **Output**:
    - The function returns a formatted string representation of the user ID of the client, or NULL if the client is not valid or the user ID cannot be retrieved.
- **Functions called**:
    - [`proc_get_peer_uid`](proc.c.driver.md#proc_get_peer_uid)


---
### format_cb_client_user<!-- {{#callable:format_cb_client_user}} -->
The `format_cb_client_user` function retrieves the username associated with the UID of the peer connected to a client.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client and its associated session.
- **Control Flow**:
    - Check if the `client` pointer in the `format_tree` structure is not NULL.
    - Retrieve the UID of the peer associated with the client using [`proc_get_peer_uid`](proc.c.driver.md#proc_get_peer_uid).
    - If the UID is valid (not -1), retrieve the corresponding `passwd` structure using `getpwuid`.
    - If the `passwd` structure is valid, duplicate the username string and return it.
    - If any checks fail, return NULL.
- **Output**:
    - Returns a dynamically allocated string containing the username of the client, or NULL if the client is not valid or the UID cannot be resolved.
- **Functions called**:
    - [`proc_get_peer_uid`](proc.c.driver.md#proc_get_peer_uid)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_client_utf8<!-- {{#callable:format_cb_client_utf8}} -->
The `format_cb_client_utf8` function checks if the client supports UTF-8 encoding and returns a string representation of the result.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the client exists, it checks if the `CLIENT_UTF8` flag is set in the client's flags.
    - If the `CLIENT_UTF8` flag is set, it returns a duplicated string '1'.
    - If the flag is not set, it returns a duplicated string '0'.
    - If the client pointer is NULL, it returns NULL.
- **Output**:
    - The function returns a pointer to a string that is '1' if the client supports UTF-8, '0' if it does not, or NULL if the client is not present.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_client_width<!-- {{#callable:format_cb_client_width}} -->
The `format_cb_client_width` function retrieves the width of the client's terminal.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client.
- **Control Flow**:
    - The function checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` is not NULL, it retrieves the width of the terminal from the `tty` structure and formats it as an unsigned integer.
    - If the `client` is NULL, the function returns NULL.
- **Output**:
    - Returns a formatted string representing the width of the client's terminal in pixels, or NULL if the client is not set.


---
### format_cb_client_written<!-- {{#callable:format_cb_client_written}} -->
The `format_cb_client_written` function retrieves the number of bytes written by a client.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client.
- **Control Flow**:
    - The function checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` pointer is valid, it calls `format_printf` to format the number of bytes written by the client.
    - If the `client` pointer is NULL, the function returns NULL.
- **Output**:
    - Returns a formatted string representing the number of bytes written by the client, or NULL if the client is not set.


---
### format_cb_client_theme<!-- {{#callable:format_cb_client_theme}} -->
The `format_cb_client_theme` function returns a string representation of the client's theme.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client and its theme.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the client is valid, it checks the `theme` attribute of the client.
    - Based on the value of the `theme`, it returns a corresponding string: 'dark' for `THEME_DARK`, 'light' for `THEME_LIGHT`, or NULL for `THEME_UNKNOWN`.
    - If the client pointer is NULL or the theme is unknown, the function returns NULL.
- **Output**:
    - The function returns a dynamically allocated string representing the client's theme ('dark' or 'light'), or NULL if the theme is unknown or the client is not set.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_config_files<!-- {{#callable:format_cb_config_files}} -->
The `format_cb_config_files` function constructs a comma-separated string of configuration file names.
- **Inputs**:
    - None
- **Control Flow**:
    - The function initializes a string pointer `s` and a size variable `slen` to keep track of the length of the resulting string.
    - It iterates over a global array `cfg_files` containing configuration file names, calculating the length of each file name and reallocating memory for the string `s` to accommodate the new name.
    - Each file name is appended to `s` followed by a comma.
    - After the loop, if `s` is still NULL (indicating no files were processed), it returns an empty string.
    - Finally, it replaces the last comma with a null terminator and returns the constructed string.
- **Output**:
    - The function returns a dynamically allocated string containing all configuration file names separated by commas, or an empty string if no files are present.
- **Functions called**:
    - [`xrealloc`](xmalloc.c.driver.md#xrealloc)
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_cursor_flag<!-- {{#callable:format_cb_cursor_flag}} -->
The `format_cb_cursor_flag` function checks if the cursor mode is enabled for a given window pane and returns a string representation of the flag.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `mode` of the `base` member of `wp` has the `MODE_CURSOR` flag set.
    - If the `MODE_CURSOR` flag is set, it returns a duplicated string '1'.
    - If the `MODE_CURSOR` flag is not set, it returns a duplicated string '0'.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a duplicated string '1' if the cursor mode is enabled, '0' if it is not, or NULL if the window pane is not set.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_cursor_shape<!-- {{#callable:format_cb_cursor_shape}} -->
The `format_cb_cursor_shape` function returns a string representation of the cursor shape based on the current screen settings.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` is not NULL and if the `screen` pointer in `wp` is also not NULL.
    - If both pointers are valid, it evaluates the `cstyle` attribute of the `screen` to determine the cursor style.
    - Based on the value of `cstyle`, it returns a corresponding string: 'block', 'underline', 'bar', or 'default'.
    - If the `window_pane` or `screen` is NULL, the function returns NULL.
- **Output**:
    - The function returns a dynamically allocated string representing the cursor shape, or NULL if the `window_pane` or `screen` is not set.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_cursor_very_visible<!-- {{#callable:format_cb_cursor_very_visible}} -->
The `format_cb_cursor_very_visible` function checks if the cursor is set to be very visible in the current window pane's screen.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` (`wp`) in the `format_tree` is not NULL and if its `screen` is also not NULL.
    - If both checks pass, it then checks if the `screen`'s `mode` includes the `MODE_CURSOR_VERY_VISIBLE` flag.
    - If the cursor is very visible, it returns a string '1'; otherwise, it returns '0'.
    - If either the `window_pane` or `screen` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the cursor is very visible, '0' if it is not, or NULL if the `window_pane` or `screen` is not available.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_cursor_x<!-- {{#callable:format_cb_cursor_x}} -->
The `format_cb_cursor_x` function retrieves the x-coordinate of the cursor position in a window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context.
- **Control Flow**:
    - The function checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it retrieves the x-coordinate from `ft->wp->base.cx` and formats it as an unsigned integer using `format_printf`.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a formatted string representing the x-coordinate of the cursor, or NULL if the window pane is not set.


---
### format_cb_cursor_y<!-- {{#callable:format_cb_cursor_y}} -->
The `format_cb_cursor_y` function retrieves the current Y-coordinate of the cursor in a window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it retrieves the Y-coordinate from `wp->base.cy` and formats it as an unsigned integer using `format_printf`.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a formatted string representing the Y-coordinate of the cursor, or NULL if the window pane is not available.


---
### format_cb_cursor_blinking<!-- {{#callable:format_cb_cursor_blinking}} -->
The `format_cb_cursor_blinking` function checks if the cursor is set to blink and returns a string representation of that state.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context.
- **Control Flow**:
    - The function first checks if the `window_pane` (`wp`) in the `format_tree` is not NULL and if its `screen` is also not NULL.
    - If the `screen`'s `mode` includes `MODE_CURSOR_BLINKING`, it returns a duplicated string '1'.
    - If the cursor is not set to blink, it returns a duplicated string '0'.
    - If either the `window_pane` or `screen` is NULL, it returns NULL.
- **Output**:
    - Returns a duplicated string '1' if the cursor is blinking, '0' if it is not, or NULL if the `window_pane` or `screen` is not available.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_history_limit<!-- {{#callable:format_cb_history_limit}} -->
The `format_cb_history_limit` function retrieves the history limit of a window pane and formats it as a string.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it retrieves the `hlimit` (history limit) from the `grid` associated with the window pane and formats it as an unsigned integer string using `format_printf`.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string containing the history limit of the window pane if it exists, or NULL if the window pane is not set.


---
### format_cb_history_size<!-- {{#callable:format_cb_history_size}} -->
The `format_cb_history_size` function retrieves the history size of a specified window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
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
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `mode` of the `base` structure of `wp` includes the `MODE_INSERT` flag.
    - If the `MODE_INSERT` flag is set, it returns a duplicated string '1'.
    - If the `MODE_INSERT` flag is not set, it returns a duplicated string '0'.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string '1' if the pane is in insert mode, '0' if it is not, or NULL if the pane is not available.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


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
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_keypad_flag<!-- {{#callable:format_cb_keypad_flag}} -->
The `format_cb_keypad_flag` function checks if the keypad mode is enabled for a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `base.mode` of the window pane includes the `MODE_KKEYPAD` flag.
    - If the `MODE_KKEYPAD` flag is set, it returns a duplicated string '1'.
    - If the flag is not set, it returns a duplicated string '0'.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a duplicated string '1' if the keypad mode is enabled, '0' if it is not, or NULL if the window pane is not set.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_mouse_all_flag<!-- {{#callable:format_cb_mouse_all_flag}} -->
The `format_cb_mouse_all_flag` function checks if mouse all mode is enabled for a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `base.mode` of the window pane includes the `MODE_MOUSE_ALL` flag.
    - If the `MODE_MOUSE_ALL` flag is set, it returns a string '1' indicating that mouse all mode is enabled.
    - If the flag is not set, it returns a string '0' indicating that mouse all mode is not enabled.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string '1' or '0' based on the mouse all mode status, or NULL if the window pane is not set.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_mouse_any_flag<!-- {{#callable:format_cb_mouse_any_flag}} -->
The `format_cb_mouse_any_flag` function checks if any mouse modes are enabled for a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer in the `format_tree` structure is not NULL.
    - If the `window_pane` is valid, it checks if the `base.mode` of the pane includes any mouse modes by performing a bitwise AND operation with `ALL_MOUSE_MODES`.
    - If any mouse modes are enabled, it returns a string '1' indicating true; otherwise, it returns '0'.
    - If the `window_pane` pointer is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string '1' if any mouse modes are enabled, '0' if none are enabled, or NULL if the `window_pane` is not set.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


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
    - The function returns a string '1' if mouse button mode is enabled, '0' if it is not, or NULL if the window pane is not set.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_mouse_pane<!-- {{#callable:format_cb_mouse_pane}} -->
The `format_cb_mouse_pane` function retrieves the mouse pane associated with a valid mouse event and formats its ID.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including mouse event data.
- **Control Flow**:
    - Check if the mouse event in the `format_tree` is valid.
    - If valid, call [`cmd_mouse_pane`](cmd.c.driver.md#cmd_mouse_pane) to get the corresponding `window_pane`.
    - If a valid `window_pane` is returned, format its ID using `format_printf` and return it.
    - If no valid `window_pane` is found or the mouse event is invalid, return NULL.
- **Output**:
    - Returns a formatted string representing the ID of the mouse pane if valid, otherwise returns NULL.
- **Functions called**:
    - [`cmd_mouse_pane`](cmd.c.driver.md#cmd_mouse_pane)


---
### format_cb_mouse_sgr_flag<!-- {{#callable:format_cb_mouse_sgr_flag}} -->
The `format_cb_mouse_sgr_flag` function checks if mouse SGR mode is enabled for a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer in the `format_tree` structure is not NULL.
    - If the `window_pane` is valid, it checks if the `mode` field of the `base` structure of the `window_pane` has the `MODE_MOUSE_SGR` flag set.
    - If the `MODE_MOUSE_SGR` flag is set, it returns a string '1' indicating that mouse SGR mode is enabled.
    - If the flag is not set, it returns a string '0' indicating that mouse SGR mode is not enabled.
    - If the `window_pane` pointer is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if mouse SGR mode is enabled, '0' if it is not enabled, or NULL if the `window_pane` is not set.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_mouse_standard_flag<!-- {{#callable:format_cb_mouse_standard_flag}} -->
The `format_cb_mouse_standard_flag` function checks if the mouse standard mode is enabled for a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `mode` of the `base` structure of `wp` has the `MODE_MOUSE_STANDARD` flag set.
    - If the flag is set, it returns a duplicated string '1', indicating that the mouse standard mode is enabled.
    - If the flag is not set, it returns a duplicated string '0', indicating that the mouse standard mode is not enabled.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a duplicated string '1' if the mouse standard mode is enabled, '0' if it is not enabled, or NULL if the `window_pane` is not set.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


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
    - The function returns a duplicated string '1' if mouse UTF-8 mode is enabled, '0' if it is not, or NULL if the window pane is not set.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_mouse_x<!-- {{#callable:format_cb_mouse_x}} -->
The `format_cb_mouse_x` function retrieves the x-coordinate of the mouse cursor in a terminal window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including mouse event data.
- **Control Flow**:
    - The function first checks if the mouse event in the `format_tree` is valid.
    - If valid, it attempts to get the `window_pane` associated with the mouse event.
    - If the `window_pane` is found and the mouse coordinates can be determined, it returns the x-coordinate.
    - If the mouse is not in a valid pane, it checks if the terminal is started and whether the mouse is in the status line, returning the x-coordinate accordingly.
    - If none of the conditions are met, it returns NULL.
- **Output**:
    - Returns the x-coordinate of the mouse cursor as a string if valid, otherwise returns NULL.
- **Functions called**:
    - [`cmd_mouse_pane`](cmd.c.driver.md#cmd_mouse_pane)
    - [`cmd_mouse_at`](cmd.c.driver.md#cmd_mouse_at)


---
### format_cb_mouse_y<!-- {{#callable:format_cb_mouse_y}} -->
The `format_cb_mouse_y` function retrieves the Y-coordinate of the mouse cursor in a terminal window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including mouse event data.
- **Control Flow**:
    - The function first checks if the mouse event data in `ft` is valid; if not, it returns NULL.
    - It attempts to get the `window_pane` associated with the mouse event using the [`cmd_mouse_pane`](cmd.c.driver.md#cmd_mouse_pane) function.
    - If a valid `window_pane` is found and the mouse position is valid, it retrieves the Y-coordinate using [`cmd_mouse_at`](cmd.c.driver.md#cmd_mouse_at) and returns it formatted as a string.
    - If the mouse position is not valid, it checks if the terminal is started and whether the mouse is within the status line; if so, it returns the Y-coordinate adjusted for the status line.
    - If none of the conditions are met, it returns NULL.
- **Output**:
    - Returns a formatted string representing the Y-coordinate of the mouse cursor, or NULL if the conditions for retrieving the Y-coordinate are not met.
- **Functions called**:
    - [`cmd_mouse_pane`](cmd.c.driver.md#cmd_mouse_pane)
    - [`cmd_mouse_at`](cmd.c.driver.md#cmd_mouse_at)


---
### format_cb_next_session_id<!-- {{#callable:format_cb_next_session_id}} -->
The `format_cb_next_session_id` function formats and returns the next session ID as a string.
- **Inputs**:
    - None
- **Control Flow**:
    - The function does not contain any control flow statements such as loops or conditionals.
    - It directly calls `format_printf` to format the output.
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
    - If `wp` is NULL, it returns NULL.
- **Output**:
    - The function returns a duplicated string '1' if the `MODE_ORIGIN` flag is set, '0' if it is not set, or NULL if the `window_pane` is NULL.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_pane_active<!-- {{#callable:format_cb_pane_active}} -->
The `format_cb_pane_active` function checks if a specific pane is the active pane in its window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane to check.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it then checks if `wp` is equal to the active pane of its window.
    - If they are equal, it returns a string '1' indicating that the pane is active.
    - If they are not equal, it returns a string '0' indicating that the pane is not active.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the pane is active, '0' if it is not, or NULL if the pane pointer is NULL.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


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
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_pane_at_right<!-- {{#callable:format_cb_pane_at_right}} -->
The `format_cb_pane_at_right` function checks if a pane is positioned at the right edge of its window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the sum of `xoff` (the x-offset of the pane) and `sx` (the width of the pane) equals the width of the window (`window->sx`).
    - If the condition is true, it returns a duplicated string '1', indicating the pane is at the right edge.
    - If the condition is false, it returns a duplicated string '0', indicating the pane is not at the right edge.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a duplicated string '1' if the pane is at the right edge of the window, '0' if it is not, or NULL if the pane does not exist.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_pane_bottom<!-- {{#callable:format_cb_pane_bottom}} -->
The `format_cb_pane_bottom` function calculates and returns the bottom coordinate of a pane in a terminal interface.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it calculates the bottom coordinate by adding the `yoff` (y offset) and `sy` (height) of the pane, then subtracting 1.
    - The calculated value is formatted as an unsigned integer and returned.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - Returns a pointer to a string representing the bottom coordinate of the pane if it exists, otherwise returns NULL.


---
### format_cb_pane_dead<!-- {{#callable:format_cb_pane_dead}} -->
The `format_cb_pane_dead` function checks if a pane is dead based on its file descriptor.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks the file descriptor (`fd`) of the `window_pane`.
    - If the file descriptor is -1, it indicates that the pane is dead, and the function returns a duplicated string '1'.
    - If the file descriptor is not -1, it returns a duplicated string '0', indicating the pane is alive.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the pane is dead (file descriptor is -1), '0' if it is alive, or NULL if the pane does not exist.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_pane_dead_signal<!-- {{#callable:format_cb_pane_dead_signal}} -->
The `format_cb_pane_dead_signal` function retrieves the signal name of a dead pane if it is ready.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the pane.
- **Control Flow**:
    - Check if the `window_pane` pointer `wp` in the `format_tree` is not NULL.
    - If `wp` is not NULL, check if the `PANE_STATUSREADY` flag is set and if the pane's status indicates it was signaled.
    - If both conditions are met, retrieve the signal name using [`sig2name`](tmux.c.driver.md#sig2name) and return it formatted as a string.
    - If any condition fails, return NULL.
- **Output**:
    - Returns a formatted string containing the signal name if the pane is dead and ready; otherwise, returns NULL.
- **Functions called**:
    - [`sig2name`](tmux.c.driver.md#sig2name)


---
### format_cb_pane_dead_status<!-- {{#callable:format_cb_pane_dead_status}} -->
The `format_cb_pane_dead_status` function retrieves the exit status of a dead pane if it is ready.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the pane.
- **Control Flow**:
    - The function first retrieves the `window_pane` associated with the `format_tree` structure.
    - If the `window_pane` is not NULL, it checks if the pane's status is ready and if it has exited.
    - If both conditions are met, it formats the exit status using `format_printf` and returns it.
    - If the conditions are not met, or if the `window_pane` is NULL, it returns NULL.
- **Output**:
    - The function returns a formatted string representing the exit status of the pane if it is ready and has exited; otherwise, it returns NULL.


---
### format_cb_pane_dead_time<!-- {{#callable:format_cb_pane_dead_time}} -->
The `format_cb_pane_dead_time` function retrieves the dead time of a window pane if it is drawn.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first retrieves the `window_pane` associated with the `format_tree` structure.
    - If the `window_pane` is not NULL, it checks if the `PANE_STATUSDRAWN` flag is set.
    - If the flag is set, it returns a pointer to the `dead_time` of the pane.
    - If the flag is not set or the `window_pane` is NULL, it returns NULL.
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
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_pane_height<!-- {{#callable:format_cb_pane_height}} -->
The `format_cb_pane_height` function retrieves the height of a specified pane in a format tree.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the pane whose height is to be retrieved.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it retrieves the height of the pane (`sy`) and formats it as an unsigned integer string using `format_printf`.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string representing the height of the pane if it exists, or NULL if the pane does not exist.


---
### format_cb_pane_id<!-- {{#callable:format_cb_pane_id}} -->
The `format_cb_pane_id` function formats and returns the ID of a window pane if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it calls the `format_printf` function to format the pane ID as a string in the format '%%%u'.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a formatted string containing the pane ID, or NULL if the pane does not exist.


---
### format_cb_pane_index<!-- {{#callable:format_cb_pane_index}} -->
The `format_cb_pane_index` function retrieves the index of a window pane and formats it as a string.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is valid, it calls the [`window_pane_index`](window.c.driver.md#window_pane_index) function to retrieve the index of the pane, storing it in the variable `idx`.
    - If the index retrieval is successful (returns 0), it formats the index as a string using `format_printf` and returns the formatted string.
    - If `wp` is NULL or the index retrieval fails, the function returns NULL.
- **Output**:
    - The function returns a formatted string representing the index of the window pane if successful, or NULL if the pane is not valid or the index cannot be retrieved.
- **Functions called**:
    - [`window_pane_index`](window.c.driver.md#window_pane_index)


---
### format_cb_pane_input_off<!-- {{#callable:format_cb_pane_input_off}} -->
The `format_cb_pane_input_off` function checks if the input for a pane is turned off and returns a string representation of the state.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `flags` member of `wp` contains the `PANE_INPUTOFF` flag.
    - If the `PANE_INPUTOFF` flag is set, the function returns a duplicated string '1'.
    - If the `PANE_INPUTOFF` flag is not set, it returns a duplicated string '0'.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a duplicated string '1' if the pane input is off, '0' if it is on, or NULL if the pane does not exist.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


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
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


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
    - The function returns a dynamically allocated string representing the key mode of the pane, or NULL if the pane or its screen is not available.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_pane_last<!-- {{#callable:format_cb_pane_last}} -->
The `format_cb_pane_last` function checks if a given pane is the last pane in its window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane being checked.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if `wp` is the first pane in the list of last panes of its associated window.
    - If `wp` is the first pane, it returns a duplicated string '1'.
    - If `wp` is not the first pane, it returns a duplicated string '0'.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a duplicated string '1' if the pane is the last in its window, '0' if it is not, or NULL if the pane does not exist.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_pane_left<!-- {{#callable:format_cb_pane_left}} -->
The `format_cb_pane_left` function retrieves the x-offset of a window pane if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it retrieves the `xoff` (x-offset) value from the `wp` structure and formats it as an unsigned integer string using `format_printf`.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string representing the x-offset of the window pane if it exists, or NULL if the pane does not exist.


---
### format_cb_pane_marked<!-- {{#callable:format_cb_pane_marked}} -->
The `format_cb_pane_marked` function checks if a specific pane is marked and returns a string indicating its status.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane to check.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the server indicates that a pane is marked using `server_check_marked()` and compares the current pane with the marked pane.
    - If both conditions are true, it returns a duplicated string '1'.
    - If the conditions are not met, it returns a duplicated string '0'.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the pane is marked, '0' if it is not marked, or NULL if the pane is not set.
- **Functions called**:
    - [`server_check_marked`](server.c.driver.md#server_check_marked)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_pane_marked_set<!-- {{#callable:format_cb_pane_marked_set}} -->
The `format_cb_pane_marked_set` function checks if a pane is marked and returns a string indicating its status.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it calls the [`server_check_marked`](server.c.driver.md#server_check_marked) function to determine if the pane is marked.
    - If the pane is marked, it returns a duplicated string '1'.
    - If the pane is not marked, it returns a duplicated string '0'.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the pane is marked, '0' if it is not marked, or NULL if the pane does not exist.
- **Functions called**:
    - [`server_check_marked`](server.c.driver.md#server_check_marked)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_pane_mode<!-- {{#callable:format_cb_pane_mode}} -->
The `format_cb_pane_mode` function retrieves the name of the first mode associated with a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it retrieves the first mode entry from the list of modes associated with the window pane.
    - If a mode entry is found, it returns a duplicate of the mode's name using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - If no mode entry is found or if `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string containing the name of the first mode associated with the window pane, or NULL if no such mode exists.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_pane_path<!-- {{#callable:format_cb_pane_path}} -->
The `format_cb_pane_path` function retrieves the file path of a specified window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `base.path` of the `window_pane` is NULL.
    - If `base.path` is NULL, it returns an empty string by duplicating an empty string.
    - If `base.path` is not NULL, it returns a duplicate of the `base.path` string.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string containing the path of the window pane, an empty string if the path is NULL, or NULL if the window pane is not set.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


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
    - The function returns a pointer to a string containing the process ID of the window pane, or NULL if the window pane does not exist.


---
### format_cb_pane_pipe<!-- {{#callable:format_cb_pane_pipe}} -->
The `format_cb_pane_pipe` function checks if a window pane has a valid pipe file descriptor and returns a string indicating its status.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `pipe_fd` (pipe file descriptor) of the `window_pane` is not equal to -1.
    - If the `pipe_fd` is valid (not -1), it returns a duplicated string '1'.
    - If the `pipe_fd` is -1, it returns a duplicated string '0'.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a duplicated string '1' if the pipe file descriptor is valid, '0' if it is invalid, or NULL if the window pane is not set.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_pane_right<!-- {{#callable:format_cb_pane_right}} -->
The `format_cb_pane_right` function calculates and returns the rightmost x-coordinate of a window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it calculates the rightmost x-coordinate by adding the `xoff` (x-offset) and `sx` (width) of the pane, subtracting 1 to get the last valid index.
    - The calculated value is then formatted as an unsigned integer string using `format_printf` and returned.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - Returns a pointer to a string representing the rightmost x-coordinate of the pane if it exists, otherwise returns NULL.


---
### format_cb_pane_search_string<!-- {{#callable:format_cb_pane_search_string}} -->
The `format_cb_pane_search_string` function retrieves the search string associated with a given pane in a format tree.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `searchstr` field of the `window_pane` is NULL.
    - If `searchstr` is NULL, it returns a duplicate of an empty string.
    - If `searchstr` is not NULL, it returns a duplicate of the `searchstr` value.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a dynamically allocated string containing the search string of the pane, an empty string if no search string is set, or NULL if the pane does not exist.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_pane_synchronized<!-- {{#callable:format_cb_pane_synchronized}} -->
The `format_cb_pane_synchronized` function checks if the current pane is synchronized and returns a string representation of its state.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it retrieves the value of the 'synchronize-panes' option from the pane's options.
    - If the option is set to a non-zero value, it returns a string '1', indicating that the panes are synchronized.
    - If the option is not set, it returns a string '0', indicating that the panes are not synchronized.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the panes are synchronized, '0' if they are not, or NULL if the pane is not available.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_pane_title<!-- {{#callable:format_cb_pane_title}} -->
The `format_cb_pane_title` function retrieves the title of a window pane if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it returns a duplicate of the `title` string from the `base` structure of the `window_pane`.
    - If `wp` is NULL, it returns NULL.
- **Output**:
    - Returns a pointer to a newly allocated string containing the title of the window pane, or NULL if the pane does not exist.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_pane_top<!-- {{#callable:format_cb_pane_top}} -->
The `format_cb_pane_top` function retrieves the vertical offset of a window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the format context, including the window pane.
- **Control Flow**:
    - The function checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it calls `format_printf` to format the vertical offset (`yoff`) of the window pane as an unsigned integer.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - Returns a formatted string representing the vertical offset of the window pane if it exists, otherwise returns NULL.


---
### format_cb_pane_tty<!-- {{#callable:format_cb_pane_tty}} -->
The `format_cb_pane_tty` function retrieves the terminal type associated with a specific window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - The function checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it returns a duplicate string of the `tty` field from the `window_pane` structure.
    - If `wp` is NULL, it returns NULL.
- **Output**:
    - Returns a pointer to a string containing the terminal type if the `window_pane` exists; otherwise, it returns NULL.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_pane_width<!-- {{#callable:format_cb_pane_width}} -->
The `format_cb_pane_width` function returns the width of a specified pane in a formatted string.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the pane whose width is to be retrieved.
- **Control Flow**:
    - The function checks if the `window_pane` pointer (`wp`) in the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it retrieves the width (`sx`) of the pane and formats it as an unsigned integer string using `format_printf`.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string containing the width of the pane if it exists, or NULL if the pane does not exist.


---
### format_cb_scroll_region_lower<!-- {{#callable:format_cb_scroll_region_lower}} -->
Retrieves the lower boundary of the scroll region for a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window pane.
- **Control Flow**:
    - Checks if the `window_pane` pointer in the `format_tree` structure is not NULL.
    - If the `window_pane` is valid, it retrieves the `rlower` value from the `base` structure of the `window_pane` and formats it as an unsigned integer.
    - If the `window_pane` is NULL, the function returns NULL.
- **Output**:
    - Returns a formatted string representing the lower boundary of the scroll region if the `window_pane` is valid; otherwise, returns NULL.


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
    - `ft`: A pointer to a `format_tree` structure that contains session information.
- **Control Flow**:
    - Checks if the session pointer in the `format_tree` structure is not NULL.
    - If the session is attached, it formats and returns the number of attached clients as a string.
    - If the session pointer is NULL, it returns NULL.
- **Output**:
    - Returns a string representation of the number of clients attached to the session, or NULL if the session is not defined.


---
### format_cb_session_format<!-- {{#callable:format_cb_session_format}} -->
The `format_cb_session_format` function checks if the format type is a session and returns '1' if true, otherwise returns '0'.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the format type.
- **Control Flow**:
    - The function first checks if the `type` field of the `format_tree` structure pointed to by `ft` is equal to `FORMAT_TYPE_SESSION`.
    - If the condition is true, it returns a duplicated string '1'.
    - If the condition is false, it returns a duplicated string '0'.
- **Output**:
    - The function returns a pointer to a string that is either '1' or '0', indicating whether the format type is a session.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_session_group<!-- {{#callable:format_cb_session_group}} -->
The `format_cb_session_group` function retrieves the name of the session group associated with a given session.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the session and other context.
- **Control Flow**:
    - Check if the session pointer in the `format_tree` structure is not NULL.
    - If the session is valid, call [`session_group_contains`](session.c.driver.md#session_group_contains) to check if the session belongs to a session group.
    - If a session group is found, return a duplicate of the group's name using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - If no session group is found or the session pointer is NULL, return NULL.
- **Output**:
    - Returns a pointer to a string containing the name of the session group if found, otherwise returns NULL.
- **Functions called**:
    - [`session_group_contains`](session.c.driver.md#session_group_contains)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_session_group_attached<!-- {{#callable:format_cb_session_group_attached}} -->
The `format_cb_session_group_attached` function returns the count of sessions attached to a session group.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the session and its associated data.
- **Control Flow**:
    - The function first checks if the `session` pointer in the `format_tree` structure is not NULL.
    - If the session is valid, it attempts to retrieve the corresponding `session_group` using the [`session_group_contains`](session.c.driver.md#session_group_contains) function.
    - If a valid `session_group` is found, it calls [`session_group_attached_count`](session.c.driver.md#session_group_attached_count) to get the number of attached sessions and formats this count as a string.
    - If no valid session or session group is found, the function returns NULL.
- **Output**:
    - The function returns a formatted string representing the number of sessions attached to the session group, or NULL if the session is invalid or not part of a session group.
- **Functions called**:
    - [`session_group_contains`](session.c.driver.md#session_group_contains)
    - [`session_group_attached_count`](session.c.driver.md#session_group_attached_count)


---
### format_cb_session_group_many_attached<!-- {{#callable:format_cb_session_group_many_attached}} -->
The `format_cb_session_group_many_attached` function checks if a session group contains more than one attached session.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the session and its associated data.
- **Control Flow**:
    - The function first checks if the `session` pointer in the `format_tree` structure is not NULL.
    - If the session is valid, it checks if the session group contains the session using [`session_group_contains`](session.c.driver.md#session_group_contains).
    - If the session group is found, it checks the count of attached sessions using [`session_group_attached_count`](session.c.driver.md#session_group_attached_count).
    - If the count of attached sessions is greater than 1, it returns a string '1'.
    - If the count is not greater than 1, it returns a string '0'.
    - If the session is NULL or the session group is not found, it returns NULL.
- **Output**:
    - The function returns a string '1' if more than one session is attached to the session group, '0' if only one is attached, or NULL if the session is not valid or the session group does not exist.
- **Functions called**:
    - [`session_group_contains`](session.c.driver.md#session_group_contains)
    - [`session_group_attached_count`](session.c.driver.md#session_group_attached_count)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_session_group_size<!-- {{#callable:format_cb_session_group_size}} -->
The `format_cb_session_group_size` function formats and returns the size of a session group.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the session and its associated data.
- **Control Flow**:
    - The function first checks if the session pointer in the `format_tree` structure is not NULL.
    - If the session is valid, it attempts to retrieve the session group associated with the session using [`session_group_contains`](session.c.driver.md#session_group_contains).
    - If a valid session group is found, it calls [`session_group_count`](session.c.driver.md#session_group_count) to get the number of sessions in that group and formats it as an unsigned integer string using `format_printf`.
    - If the session is NULL or no session group is found, the function returns NULL.
- **Output**:
    - The function returns a formatted string representing the size of the session group if valid, otherwise it returns NULL.
- **Functions called**:
    - [`session_group_contains`](session.c.driver.md#session_group_contains)
    - [`session_group_count`](session.c.driver.md#session_group_count)


---
### format_cb_session_grouped<!-- {{#callable:format_cb_session_grouped}} -->
The `format_cb_session_grouped` function checks if a session is part of a session group and returns a string indicating the result.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the session being checked.
- **Control Flow**:
    - The function first checks if the session pointer `ft->s` is not NULL.
    - If the session is valid, it calls [`session_group_contains`](session.c.driver.md#session_group_contains) to determine if the session is part of a session group.
    - If the session is part of a group, it returns a duplicated string '1'.
    - If the session is not part of a group, it returns a duplicated string '0'.
    - If the session pointer is NULL, it returns NULL.
- **Output**:
    - The function returns a duplicated string '1' if the session is part of a session group, '0' if it is not, or NULL if the session pointer is NULL.
- **Functions called**:
    - [`session_group_contains`](session.c.driver.md#session_group_contains)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_session_id<!-- {{#callable:format_cb_session_id}} -->
The `format_cb_session_id` function formats and returns the session ID of a given session in a specific string format.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the session.
- **Control Flow**:
    - The function first checks if the session pointer `s` within the `format_tree` structure is not NULL.
    - If the session is valid, it calls `format_printf` to format the session ID as a string prefixed with a dollar sign ('$').
    - If the session is NULL, the function returns NULL.
- **Output**:
    - The function returns a formatted string containing the session ID prefixed with a dollar sign ('$') if the session exists, or NULL if it does not.


---
### format_cb_session_many_attached<!-- {{#callable:format_cb_session_many_attached}} -->
The `format_cb_session_many_attached` function checks if a session has more than one attached client and returns a string representation of that state.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains session information.
- **Control Flow**:
    - The function first checks if the `session` pointer in the `format_tree` structure is not NULL.
    - If the session is attached to more than one client, it returns a duplicated string '1'.
    - If the session is attached to one or no clients, it returns a duplicated string '0'.
    - If the session pointer is NULL, it returns NULL.
- **Output**:
    - Returns a string '1' if more than one client is attached to the session, '0' if one or none is attached, or NULL if the session is not defined.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_session_marked<!-- {{#callable:format_cb_session_marked}} -->
The `format_cb_session_marked` function checks if a session is marked and returns a string indicating its status.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the session to be checked.
- **Control Flow**:
    - The function first checks if the session pointer `ft->s` is not NULL.
    - If the session is valid, it checks if the server indicates that a session is marked and if the marked session matches the current session.
    - If both conditions are true, it returns a string '1', indicating the session is marked.
    - If the session is not marked, it returns a string '0'.
    - If the session pointer is NULL, it returns NULL.
- **Output**:
    - The function returns a string '1' if the session is marked, '0' if it is not marked, or NULL if the session pointer is NULL.
- **Functions called**:
    - [`server_check_marked`](server.c.driver.md#server_check_marked)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_session_name<!-- {{#callable:format_cb_session_name}} -->
The `format_cb_session_name` function retrieves the name of a session from a `format_tree` structure.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains session information.
- **Control Flow**:
    - The function checks if the session pointer in the `format_tree` structure is not NULL.
    - If the session pointer is valid, it duplicates the session name using [`xstrdup`](xmalloc.c.driver.md#xstrdup) and returns it.
    - If the session pointer is NULL, it returns NULL.
- **Output**:
    - Returns a pointer to a duplicated string containing the session name if it exists, otherwise returns NULL.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_session_path<!-- {{#callable:format_cb_session_path}} -->
The `format_cb_session_path` function retrieves the current working directory of a session if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains session information.
- **Control Flow**:
    - The function checks if the session pointer in the `format_tree` structure is not NULL.
    - If the session exists, it duplicates the current working directory string using [`xstrdup`](xmalloc.c.driver.md#xstrdup) and returns it.
    - If the session does not exist, it returns NULL.
- **Output**:
    - Returns a pointer to a duplicated string of the current working directory of the session, or NULL if the session does not exist.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_session_windows<!-- {{#callable:format_cb_session_windows}} -->
The `format_cb_session_windows` function returns the count of windows in a session if the session is not null.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the session.
- **Control Flow**:
    - The function first checks if the session pointer (`ft->s`) is not null.
    - If the session is valid, it calls [`winlink_count`](window.c.driver.md#winlink_count) to get the number of windows associated with the session and formats this count as a string using `format_printf`.
    - If the session is null, the function returns null.
- **Output**:
    - The function outputs a formatted string representing the number of windows in the session, or null if the session is not present.
- **Functions called**:
    - [`winlink_count`](window.c.driver.md#winlink_count)


---
### format_cb_socket_path<!-- {{#callable:format_cb_socket_path}} -->
The `format_cb_socket_path` function returns a duplicate of the global `socket_path` string.
- **Inputs**:
    - None
- **Control Flow**:
    - The function does not contain any control flow statements such as loops or conditionals.
    - It directly calls [`xstrdup`](xmalloc.c.driver.md#xstrdup) to duplicate the `socket_path` string.
- **Output**:
    - The output is a pointer to a newly allocated string that is a duplicate of the `socket_path`.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_version<!-- {{#callable:format_cb_version}} -->
The `format_cb_version` function retrieves and duplicates the current version string of the software.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure, which is unused in this function.
- **Control Flow**:
    - The function calls `getversion()` to obtain the current version string.
    - It then uses `xstrdup()` to create a duplicate of the version string.
    - Finally, it returns the duplicated string.
- **Output**:
    - The function returns a pointer to a newly allocated string containing the current version of the software.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`getversion`](tmux.c.driver.md#getversion)


---
### format_cb_sixel_support<!-- {{#callable:format_cb_sixel_support}} -->
The `format_cb_sixel_support` function checks if sixel support is enabled and returns a string indicating the support status.
- **Inputs**:
    - None
- **Control Flow**:
    - The function checks if the `ENABLE_SIXEL` macro is defined.
    - If `ENABLE_SIXEL` is defined, it returns a duplicated string '1'.
    - If `ENABLE_SIXEL` is not defined, it returns a duplicated string '0'.
- **Output**:
    - The function returns a pointer to a string that indicates whether sixel support is enabled ('1') or not ('0').
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_active_window_index<!-- {{#callable:format_cb_active_window_index}} -->
The `format_cb_active_window_index` function retrieves the index of the currently active window in a session.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current session and window.
- **Control Flow**:
    - The function first checks if the session pointer (`s`) in the `format_tree` structure is not NULL.
    - If the session is valid, it retrieves the index of the current window (`curw->idx`) and formats it as an unsigned integer string using `format_printf`.
    - If the session is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string representing the index of the currently active window, or NULL if the session is not valid.


---
### format_cb_last_window_index<!-- {{#callable:format_cb_last_window_index}} -->
The `format_cb_last_window_index` function retrieves the index of the last window in a session.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the session and windows.
- **Control Flow**:
    - Check if the session pointer in the `format_tree` structure is not NULL.
    - If the session is valid, retrieve the maximum window link from the session's windows using `RB_MAX`.
    - Return the index of the retrieved window link formatted as a string using `format_printf`.
    - If the session is NULL, return NULL.
- **Output**:
    - Returns a string representation of the index of the last window in the session, or NULL if the session is not set.


---
### format_cb_window_active<!-- {{#callable:format_cb_window_active}} -->
The `format_cb_window_active` function checks if the current window link is active.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context.
- **Control Flow**:
    - The function first checks if the `wl` (window link) member of the `format_tree` structure is not NULL.
    - If `wl` is not NULL, it checks if `wl` is equal to the current window of its session (`wl->session->curw`).
    - If they are equal, it returns a string '1' indicating the window is active.
    - If they are not equal, it returns a string '0' indicating the window is not active.
    - If `wl` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the window is active, '0' if it is not, or NULL if the window link is not set.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_window_activity_flag<!-- {{#callable:format_cb_window_activity_flag}} -->
The `format_cb_window_activity_flag` function checks if a window link has the activity flag set.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context.
- **Control Flow**:
    - The function first checks if the `wl` (window link) member of the `format_tree` structure is not NULL.
    - If `wl` is not NULL, it checks if the `flags` member of `wl` has the `WINLINK_ACTIVITY` flag set.
    - If the `WINLINK_ACTIVITY` flag is set, it returns a string '1' indicating activity.
    - If the flag is not set, it returns a string '0' indicating no activity.
    - If `wl` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the window link has the activity flag set, '0' if not, or NULL if the window link is NULL.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_window_bell_flag<!-- {{#callable:format_cb_window_bell_flag}} -->
The `format_cb_window_bell_flag` function checks if the window link has the bell flag set and returns a string representation of the result.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window link.
- **Control Flow**:
    - The function first checks if the `wl` (window link) member of the `format_tree` structure is not NULL.
    - If `wl` is not NULL, it checks if the `flags` member of `wl` contains the `WINLINK_BELL` flag.
    - If the `WINLINK_BELL` flag is set, it returns a duplicated string '1'.
    - If the `WINLINK_BELL` flag is not set, it returns a duplicated string '0'.
    - If `wl` is NULL, the function returns NULL.
- **Output**:
    - The function returns a duplicated string '1' if the bell flag is set, '0' if it is not set, or NULL if the window link is not present.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_window_bigger<!-- {{#callable:format_cb_window_bigger}} -->
The `format_cb_window_bigger` function checks if the terminal window associated with a client is larger than its offset.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the associated client.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the client is valid, it calls the [`tty_window_offset`](tty.c.driver.md#tty_window_offset) function to retrieve the window's offset and size.
    - If [`tty_window_offset`](tty.c.driver.md#tty_window_offset) returns a non-zero value, indicating an error, the function returns a string '1'.
    - If the offset retrieval is successful, it returns '0'.
    - If the client pointer is NULL, the function returns NULL.
- **Output**:
    - The function returns a string indicating whether the window is larger than its offset ('1' for true, '0' for false, or NULL if the client is not set).
- **Functions called**:
    - [`tty_window_offset`](tty.c.driver.md#tty_window_offset)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_window_cell_height<!-- {{#callable:format_cb_window_cell_height}} -->
The `format_cb_window_cell_height` function retrieves the height of a window cell in pixels.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the window.
- **Control Flow**:
    - The function checks if the `w` member of the `format_tree` structure is not NULL.
    - If `w` is not NULL, it retrieves the `ypixel` value from the `w` structure and formats it as an unsigned integer string using `format_printf`.
    - If `w` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string representing the height of the window cell in pixels, or NULL if the window is not defined.


---
### format_cb_window_cell_width<!-- {{#callable:format_cb_window_cell_width}} -->
The `format_cb_window_cell_width` function retrieves the width of a window cell in pixels.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the window.
- **Control Flow**:
    - The function checks if the `w` member of the `format_tree` structure is not NULL.
    - If `w` is not NULL, it retrieves the `xpixel` value from the `w` structure and formats it as an unsigned integer string using `format_printf`.
    - If `w` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string representing the width of the window cell in pixels, or NULL if the window is not defined.


---
### format_cb_window_end_flag<!-- {{#callable:format_cb_window_end_flag}} -->
The `format_cb_window_end_flag` function checks if a given window link is the last one in its session and returns a string indicating this.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context, including the window link to be checked.
- **Control Flow**:
    - The function first checks if the `wl` (window link) member of the `format_tree` structure is not NULL.
    - If `wl` is not NULL, it checks if `wl` is the maximum window link in the session's windows using `RB_MAX`.
    - If `wl` is the maximum, it returns a string '1' indicating it is the last window link; otherwise, it returns '0'.
    - If `wl` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the window link is the last in its session, '0' if it is not, or NULL if the window link is NULL.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_window_flags<!-- {{#callable:format_cb_window_flags}} -->
The `format_cb_window_flags` function returns a duplicated string of window flags if the winlink is not NULL.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the winlink.
- **Control Flow**:
    - The function checks if the `wl` (winlink) member of the `format_tree` structure is not NULL.
    - If `wl` is not NULL, it calls the [`window_printable_flags`](window.c.driver.md#window_printable_flags) function with `ft->wl` and a flag value of 1, and duplicates the resulting string using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - If `wl` is NULL, the function returns NULL.
- **Output**:
    - The function outputs a duplicated string of the window flags if the winlink is present; otherwise, it returns NULL.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`window_printable_flags`](window.c.driver.md#window_printable_flags)


---
### format_cb_window_format<!-- {{#callable:format_cb_window_format}} -->
The `format_cb_window_format` function checks if the format type is a window and returns '1' if true, otherwise returns '0'.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the format type.
- **Control Flow**:
    - The function first checks if the `type` field of the `format_tree` structure pointed to by `ft` is equal to `FORMAT_TYPE_WINDOW`.
    - If the condition is true, it returns a duplicated string '1' using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - If the condition is false, it returns a duplicated string '0' using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
- **Output**:
    - The function returns a pointer to a string that is either '1' or '0', indicating whether the format type is a window.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_window_height<!-- {{#callable:format_cb_window_height}} -->
The `format_cb_window_height` function retrieves the height of a window in a format tree.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the window.
- **Control Flow**:
    - The function checks if the `w` member of the `format_tree` structure is not NULL.
    - If `w` is not NULL, it retrieves the `sy` (height) of the window and formats it as an unsigned integer string using `format_printf`.
    - If `w` is NULL, the function returns NULL.
- **Output**:
    - Returns a pointer to a string representing the height of the window if it exists, otherwise returns NULL.


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
    - The function returns a pointer to a formatted string containing the window ID prefixed with '@', or NULL if the window pointer is not valid.


---
### format_cb_window_index<!-- {{#callable:format_cb_window_index}} -->
The `format_cb_window_index` function retrieves the index of a window link in a format tree.
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
The `format_cb_window_last_flag` function checks if a given window link is the last one in its session.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context.
- **Control Flow**:
    - The function first checks if the `wl` (window link) member of the `format_tree` structure is not NULL.
    - If `wl` is not NULL, it checks if `wl` is the first element in the `lastw` (last window) queue of its session.
    - If it is the first element, the function returns a string '1' indicating it is the last window.
    - If it is not the first element, the function returns a string '0'.
    - If `wl` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the window link is the last in its session, '0' if it is not, or NULL if the window link is NULL.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


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
    - If it is the first match, it sets the `found` flag to 1.
    - If no matches are found after checking all sessions, it returns '0'.
    - If `wl` is NULL, it returns NULL.
- **Output**:
    - The function returns a dynamically allocated string '1' if the window is linked to any session, '0' if it is not linked, or NULL if the input `wl` is NULL.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_window_linked_sessions<!-- {{#callable:format_cb_window_linked_sessions}} -->
The `format_cb_window_linked_sessions` function counts the number of sessions linked to a specific window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context, including the window link.
- **Control Flow**:
    - Checks if the `wl` (window link) in the `format_tree` is NULL; if so, returns NULL.
    - Retrieves the `window` associated with the `format_tree`.
    - Iterates over all session groups and counts how many sessions are linked to the specified window.
    - Iterates over all sessions and counts how many are linked to the specified window, excluding those in a session group.
    - Returns the count of linked sessions formatted as a string.
- **Output**:
    - Returns a string representation of the number of sessions linked to the specified window.
- **Functions called**:
    - [`winlink_find_by_window`](window.c.driver.md#winlink_find_by_window)
    - [`session_group_contains`](session.c.driver.md#session_group_contains)


---
### format_cb_window_marked_flag<!-- {{#callable:format_cb_window_marked_flag}} -->
The `format_cb_window_marked_flag` function checks if a specific window link is marked and returns a string representation of the result.
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
- **Functions called**:
    - [`server_check_marked`](server.c.driver.md#server_check_marked)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_window_name<!-- {{#callable:format_cb_window_name}} -->
The `format_cb_window_name` function retrieves the name of a window from a `format_tree` structure.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the window.
- **Control Flow**:
    - The function checks if the `window` pointer in the `format_tree` structure is not NULL.
    - If the `window` pointer is valid, it retrieves the `name` of the window and formats it using `format_printf`.
    - If the `window` pointer is NULL, the function returns NULL.
- **Output**:
    - The function returns a formatted string containing the name of the window if it exists, or NULL if the window does not exist.


---
### format_cb_window_offset_x<!-- {{#callable:format_cb_window_offset_x}} -->
The `format_cb_window_offset_x` function retrieves the x-offset of the terminal window associated with a given format tree.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the associated client.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` is valid, it calls the [`tty_window_offset`](tty.c.driver.md#tty_window_offset) function to retrieve the x-offset (ox) and y-offset (oy) of the terminal window.
    - If [`tty_window_offset`](tty.c.driver.md#tty_window_offset) returns a non-zero value, indicating success, it formats the x-offset (ox) as an unsigned integer and returns it as a string.
    - If any of the checks fail, the function returns NULL.
- **Output**:
    - Returns a string representation of the x-offset of the terminal window if successful, or NULL if the client is not set or if the offset retrieval fails.
- **Functions called**:
    - [`tty_window_offset`](tty.c.driver.md#tty_window_offset)


---
### format_cb_window_offset_y<!-- {{#callable:format_cb_window_offset_y}} -->
The `format_cb_window_offset_y` function retrieves the vertical offset of a terminal window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context.
- **Control Flow**:
    - The function first checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` is valid, it calls the [`tty_window_offset`](tty.c.driver.md#tty_window_offset) function to retrieve the offsets (ox, oy) and sizes (sx, sy) of the terminal window.
    - If [`tty_window_offset`](tty.c.driver.md#tty_window_offset) returns a non-zero value, indicating success, it formats the vertical offset (oy) as an unsigned integer and returns it as a string.
    - If any of the checks fail, the function returns NULL.
- **Output**:
    - The function returns a formatted string representing the vertical offset of the terminal window if successful, or NULL if the client is not set or if the offset retrieval fails.
- **Functions called**:
    - [`tty_window_offset`](tty.c.driver.md#tty_window_offset)


---
### format_cb_window_panes<!-- {{#callable:format_cb_window_panes}} -->
The `format_cb_window_panes` function returns the number of panes in a specified window.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the window whose panes are to be counted.
- **Control Flow**:
    - The function first checks if the `window` pointer in the `format_tree` structure is not NULL.
    - If the `window` pointer is valid, it calls the [`window_count_panes`](window.c.driver.md#window_count_panes) function to get the number of panes and formats it as an unsigned integer string using `format_printf`.
    - If the `window` pointer is NULL, the function returns NULL.
- **Output**:
    - The function returns a formatted string representing the number of panes in the window, or NULL if the window is not set.
- **Functions called**:
    - [`window_count_panes`](window.c.driver.md#window_count_panes)


---
### format_cb_window_raw_flags<!-- {{#callable:format_cb_window_raw_flags}} -->
The `format_cb_window_raw_flags` function retrieves the raw flags of a window in a format tree.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the window and its associated flags.
- **Control Flow**:
    - The function checks if the `wl` (winlink) member of the `format_tree` structure is not NULL.
    - If `wl` is not NULL, it calls the [`window_printable_flags`](window.c.driver.md#window_printable_flags) function with `ft->wl` and a flag value of 1 to get the printable representation of the window flags.
    - The result of [`window_printable_flags`](window.c.driver.md#window_printable_flags) is duplicated using [`xstrdup`](xmalloc.c.driver.md#xstrdup) and returned.
    - If `wl` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string containing the printable representation of the window flags if `wl` is not NULL; otherwise, it returns NULL.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`window_printable_flags`](window.c.driver.md#window_printable_flags)


---
### format_cb_window_silence_flag<!-- {{#callable:format_cb_window_silence_flag}} -->
The `format_cb_window_silence_flag` function checks if a window link is in silence mode and returns a corresponding string.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context.
- **Control Flow**:
    - The function first checks if the `wl` (window link) member of the `format_tree` structure is not NULL.
    - If `wl` is not NULL, it checks if the `flags` member of `wl` contains the `WINLINK_SILENCE` flag.
    - If the `WINLINK_SILENCE` flag is set, the function returns a string '1'.
    - If the `WINLINK_SILENCE` flag is not set, it returns a string '0'.
    - If `wl` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the window link is in silence mode, '0' if it is not, or NULL if the window link is not present.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_window_start_flag<!-- {{#callable:format_cb_window_start_flag}} -->
The `format_cb_window_start_flag` function checks if the given window link is the first in its session and returns a string indicating its status.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window link to be checked.
- **Control Flow**:
    - The function first checks if the `wl` (window link) in the `format_tree` is not NULL.
    - If `wl` is not NULL, it checks if `wl` is the minimum window link in the session's windows.
    - If it is the minimum, the function returns a duplicated string '1'.
    - If it is not the minimum, the function returns a duplicated string '0'.
    - If `wl` is NULL, the function returns NULL.
- **Output**:
    - The function returns a string '1' if the window link is the first in its session, '0' if it is not, or NULL if the window link is NULL.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_window_width<!-- {{#callable:format_cb_window_width}} -->
The `format_cb_window_width` function returns the width of a specified window in a formatted string.
- **Inputs**:
    - `ft`: A pointer to a `struct format_tree` that contains information about the window whose width is to be formatted.
- **Control Flow**:
    - The function first checks if the `w` member of the `format_tree` structure is not NULL.
    - If `ft->w` is not NULL, it retrieves the width of the window (`ft->w->sx`) and formats it as an unsigned integer string using `format_printf`.
    - If `ft->w` is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string containing the width of the window formatted as an unsigned integer, or NULL if the window is not defined.


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
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_wrap_flag<!-- {{#callable:format_cb_wrap_flag}} -->
The `format_cb_wrap_flag` function checks if the wrap mode is enabled for a given window pane.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current formatting context, including the window pane.
- **Control Flow**:
    - The function first checks if the `wp` (window pane) member of the `format_tree` structure is not NULL.
    - If `wp` is not NULL, it checks if the `base.mode` of the window pane has the `MODE_WRAP` flag set.
    - If the `MODE_WRAP` flag is set, it returns a duplicated string '1'.
    - If the `MODE_WRAP` flag is not set, it returns a duplicated string '0'.
    - If `wp` is NULL, the function returns NULL.
- **Output**:
    - The function returns a duplicated string '1' if wrap mode is enabled, '0' if it is not, or NULL if the window pane is not set.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_buffer_created<!-- {{#callable:format_cb_buffer_created}} -->
The `format_cb_buffer_created` function retrieves the creation time of a paste buffer.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the paste buffer.
- **Control Flow**:
    - The function checks if the `pb` (paste buffer) member of the `format_tree` structure is not NULL.
    - If the paste buffer is valid, it clears the `tv` (time value) structure and sets its `tv_sec` member to the result of the [`paste_buffer_created`](paste.c.driver.md#paste_buffer_created) function, which retrieves the creation time of the paste buffer.
    - The function then returns a pointer to the `tv` structure.
    - If the paste buffer is NULL, the function returns NULL.
- **Output**:
    - The function returns a pointer to a `struct timeval` containing the creation time of the paste buffer if it exists, or NULL if the paste buffer is not set.
- **Functions called**:
    - [`paste_buffer_created`](paste.c.driver.md#paste_buffer_created)


---
### format_cb_client_activity<!-- {{#callable:format_cb_client_activity}} -->
The `format_cb_client_activity` function retrieves the activity time of a client if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client whose activity time is to be retrieved.
- **Control Flow**:
    - The function checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` pointer is not NULL, it returns a pointer to the `activity_time` field of the client.
    - If the `client` pointer is NULL, it returns NULL.
- **Output**:
    - The function returns a pointer to the `activity_time` of the client if it exists, otherwise it returns NULL.


---
### format_cb_client_created<!-- {{#callable:format_cb_client_created}} -->
The `format_cb_client_created` function retrieves the creation time of a client if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the client.
- **Control Flow**:
    - The function checks if the `client` pointer in the `format_tree` structure is not NULL.
    - If the `client` pointer is not NULL, it returns a pointer to the `creation_time` of the client.
    - If the `client` pointer is NULL, it returns NULL.
- **Output**:
    - The function returns a pointer to the `creation_time` of the client if it exists, otherwise it returns NULL.


---
### format_cb_session_activity<!-- {{#callable:format_cb_session_activity}} -->
The `format_cb_session_activity` function retrieves the activity time of a session if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains session information.
- **Control Flow**:
    - The function checks if the session pointer in the `format_tree` structure is not NULL.
    - If the session pointer is valid, it returns a pointer to the `activity_time` field of the session.
    - If the session pointer is NULL, it returns NULL.
- **Output**:
    - Returns a pointer to the `activity_time` of the session if it exists, otherwise returns NULL.


---
### format_cb_session_created<!-- {{#callable:format_cb_session_created}} -->
The `format_cb_session_created` function retrieves the creation time of a session if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains session information.
- **Control Flow**:
    - The function checks if the session pointer in the `format_tree` structure is not NULL.
    - If the session exists, it returns a pointer to the `creation_time` of the session.
    - If the session does not exist, it returns NULL.
- **Output**:
    - Returns a pointer to the session's creation time if the session exists, otherwise returns NULL.


---
### format_cb_session_last_attached<!-- {{#callable:format_cb_session_last_attached}} -->
The `format_cb_session_last_attached` function retrieves the last attached time of a session if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the session.
- **Control Flow**:
    - The function first checks if the session pointer in the `format_tree` structure is not NULL.
    - If the session pointer is valid, it returns a pointer to the `last_attached_time` field of the session.
    - If the session pointer is NULL, it returns NULL.
- **Output**:
    - Returns a pointer to the `last_attached_time` of the session if it exists, otherwise returns NULL.


---
### format_cb_start_time<!-- {{#callable:format_cb_start_time}} -->
The `format_cb_start_time` function returns a pointer to the global variable `start_time`.
- **Inputs**:
    - None
- **Control Flow**:
    - The function does not contain any control flow statements such as loops or conditionals.
    - It directly returns the address of the `start_time` variable.
- **Output**:
    - The output is a pointer to the `start_time` variable, which is expected to be of type `time_t`.


---
### format_cb_window_activity<!-- {{#callable:format_cb_window_activity}} -->
Retrieves the activity time of a window if it exists.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context, including the window.
- **Control Flow**:
    - The function checks if the `w` member of the `format_tree` structure is not NULL.
    - If `w` is not NULL, it returns a pointer to the `activity_time` member of the `w` structure.
    - If `w` is NULL, it returns NULL.
- **Output**:
    - Returns a pointer to the `activity_time` of the window if it exists, otherwise returns NULL.


---
### format_cb_buffer_mode_format<!-- {{#callable:format_cb_buffer_mode_format}} -->
The `format_cb_buffer_mode_format` function returns a duplicate of the default format string for the window buffer mode.
- **Inputs**:
    - None
- **Control Flow**:
    - The function does not contain any control flow statements such as loops or conditionals.
    - It directly returns the result of the [`xstrdup`](xmalloc.c.driver.md#xstrdup) function, which duplicates the string.
- **Output**:
    - The output is a pointer to a newly allocated string that contains the default format for the window buffer mode.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_client_mode_format<!-- {{#callable:format_cb_client_mode_format}} -->
The `format_cb_client_mode_format` function returns a duplicate of the default format string for the client mode.
- **Inputs**:
    - None
- **Control Flow**:
    - The function does not perform any conditional checks or loops.
    - It directly calls [`xstrdup`](xmalloc.c.driver.md#xstrdup) to duplicate the default format string.
- **Output**:
    - The function outputs a pointer to a newly allocated string that contains the default format for the client mode.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_tree_mode_format<!-- {{#callable:format_cb_tree_mode_format}} -->
The `format_cb_tree_mode_format` function returns a duplicated string of the default format for the tree mode.
- **Inputs**:
    - None
- **Control Flow**:
    - The function does not contain any control flow statements such as loops or conditionals.
    - It directly returns the result of [`xstrdup`](xmalloc.c.driver.md#xstrdup) applied to `window_tree_mode.default_format`.
- **Output**:
    - The output is a pointer to a newly allocated string that is a duplicate of the default format string for the tree mode.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_cb_uid<!-- {{#callable:format_cb_uid}} -->
The `format_cb_uid` function retrieves the user ID of the calling process and formats it as a string.
- **Inputs**:
    - None
- **Control Flow**:
    - The function calls `getuid()` to obtain the user ID of the current process.
    - The user ID is then formatted as a long integer string using `format_printf`.
    - Finally, the formatted string is returned.
- **Output**:
    - The output is a pointer to a string containing the user ID formatted as a decimal number.


---
### format_cb_user<!-- {{#callable:format_cb_user}} -->
The `format_cb_user` function retrieves the username associated with the current user ID.
- **Inputs**:
    - None
- **Control Flow**:
    - The function begins by declaring a pointer to a `struct passwd` named `pw`.
    - It calls `getpwuid` with the result of `getuid()` to retrieve the password entry for the current user.
    - If the call to `getpwuid` is successful (i.e., `pw` is not NULL`), it duplicates the username string using [`xstrdup`](xmalloc.c.driver.md#xstrdup) and returns it.
    - If the call fails, the function returns NULL.
- **Output**:
    - The function returns a pointer to a string containing the username of the current user, or NULL if the username could not be retrieved.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_table_compare<!-- {{#callable:format_table_compare}} -->
Compares a key with a format table entry's key.
- **Inputs**:
    - `key0`: A pointer to the key string to compare.
    - `entry0`: A pointer to a `format_table_entry` structure containing the key to compare against.
- **Control Flow**:
    - The function casts the input pointers to appropriate types: `key0` to a `const char*` and `entry0` to a `const struct format_table_entry*`.
    - It then uses the `strcmp` function to compare the `key` with the `entry->key` from the `format_table_entry` structure.
    - The result of the comparison is returned, which indicates the lexicographical order of the two strings.
- **Output**:
    - An integer value: 0 if the keys are equal, a negative value if `key` is less than `entry->key`, and a positive value if `key` is greater than `entry->key`.


---
### format_table_get<!-- {{#callable:format_table_get}} -->
The `format_table_get` function retrieves a format table entry based on a specified key.
- **Inputs**:
    - `key`: A constant character pointer representing the key to search for in the format table.
- **Control Flow**:
    - The function uses `bsearch` to perform a binary search on the `format_table` array.
    - It searches for the `key` in the `format_table` using the `format_table_compare` function to compare entries.
    - If a matching entry is found, it is returned; otherwise, NULL is returned.
- **Output**:
    - The function returns a pointer to the `format_table_entry` structure corresponding to the provided key, or NULL if the key is not found.


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
    - `ft`: A pointer to a `format_tree` structure that contains information about the current format context.
- **Control Flow**:
    - The function directly accesses the `wp` member of the `format_tree` structure.
    - It returns the value of `wp`, which is a pointer to a `window_pane` structure.
- **Output**:
    - Returns a pointer to the `window_pane` associated with the provided `format_tree`, or NULL if no pane is associated.


---
### format_create_add_item<!-- {{#callable:format_create_add_item}} -->
The `format_create_add_item` function merges format data from a command queue item into a format tree.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that holds the format data.
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item from which to extract the event.
- **Control Flow**:
    - The function retrieves the `key_event` associated with the provided `cmdq_item` using the [`cmdq_get_event`](cmd-queue.c.driver.md#cmdq_get_event) function.
    - It then accesses the `mouse_event` structure from the retrieved `key_event`.
    - The function calls [`cmdq_merge_formats`](cmd-queue.c.driver.md#cmdq_merge_formats) to merge the formats from the `cmdq_item` into the `format_tree`.
    - Finally, it copies the `mouse_event` data into the `format_tree`'s mouse event structure using `memcpy`.
- **Output**:
    - The function does not return a value; it modifies the `format_tree` in place by merging format data and copying mouse event information.
- **Functions called**:
    - [`cmdq_get_event`](cmd-queue.c.driver.md#cmdq_get_event)
    - [`cmdq_merge_formats`](cmd-queue.c.driver.md#cmdq_merge_formats)


---
### format_create<!-- {{#callable:format_create}} -->
Creates a new `format_tree` structure for formatting commands.
- **Inputs**:
    - `c`: A pointer to a `client` structure, which may be NULL.
    - `item`: A pointer to a `cmdq_item` structure, which may be NULL.
    - `tag`: An integer representing a tag associated with the format tree.
    - `flags`: An integer representing flags that modify the behavior of the format tree.
- **Control Flow**:
    - Allocates memory for a new `format_tree` structure using [`xcalloc`](xmalloc.c.driver.md#xcalloc).
    - Initializes the red-black tree within the `format_tree` structure.
    - If the `client` pointer is not NULL, increments the reference count for the client.
    - Sets the `item`, `tag`, and `flags` fields of the `format_tree` structure.
    - If the `item` is not NULL, calls [`format_create_add_item`](#format_create_add_item) to add the item to the format tree.
- **Output**:
    - Returns a pointer to the newly created `format_tree` structure.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`format_create_add_item`](#format_create_add_item)


---
### format_free<!-- {{#callable:format_free}} -->
The `format_free` function deallocates memory associated with a `format_tree` structure.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains format entries and associated client information.
- **Control Flow**:
    - Iterates over each `format_entry` in the `format_tree` using `RB_FOREACH_SAFE` to safely remove entries while iterating.
    - For each `format_entry`, it removes the entry from the tree, frees its `key` and `value` strings, and then frees the entry itself.
    - After processing all entries, it checks if the `client` pointer in the `format_tree` is not NULL, and if so, it calls [`server_client_unref`](server-client.c.driver.md#server_client_unref) to decrement the reference count.
    - Finally, it frees the `format_tree` structure itself.
- **Output**:
    - The function does not return a value; it performs cleanup by freeing allocated memory.
- **Functions called**:
    - [`server_client_unref`](server-client.c.driver.md#server_client_unref)


---
### format_log_debug_cb<!-- {{#callable:format_log_debug_cb}} -->
The `format_log_debug_cb` function logs a debug message with a specified key-value pair prefixed by a given string.
- **Inputs**:
    - `key`: A string representing the key to be logged.
    - `value`: A string representing the value associated with the key.
    - `arg`: A pointer to a string that serves as a prefix for the log message.
- **Control Flow**:
    - The function retrieves the prefix from the `arg` parameter.
    - It then calls the `log_debug` function to log the formatted message, which includes the prefix, key, and value.
- **Output**:
    - The function does not return a value; it performs a logging action instead.


---
### format_log_debug<!-- {{#callable:format_log_debug}} -->
The `format_log_debug` function formats and logs debug information for a given format tree with a specified prefix.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains the data to be logged.
    - `prefix`: A string that serves as a prefix for the log messages.
- **Control Flow**:
    - The function calls [`format_each`](#format_each), passing the `format_tree`, a callback function `format_log_debug_cb`, and the prefix.
    - The [`format_each`](#format_each) function iterates over each entry in the `format_tree`, invoking the callback for each entry.
- **Output**:
    - The function does not return a value; it logs formatted debug messages to the logging system.
- **Functions called**:
    - [`format_each`](#format_each)


---
### format_each<!-- {{#callable:format_each}} -->
The `format_each` function iterates over a format tree and applies a callback to each format entry.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains the format entries to be processed.
    - `cb`: A function pointer to a callback that takes a key, a value, and an additional argument.
    - `arg`: A pointer to additional data that will be passed to the callback function.
- **Control Flow**:
    - The function initializes a loop to iterate over a predefined format table.
    - For each entry in the format table, it calls the associated callback to retrieve a value.
    - If the value is NULL, it continues to the next entry.
    - If the entry type is `FORMAT_TABLE_TIME`, it formats the time value and calls the callback.
    - For other types, it directly calls the callback with the retrieved value and frees the value afterwards.
    - After processing the format table, it iterates over the entries in the format tree, handling time values and calling the callback with the appropriate values.
- **Output**:
    - The function does not return a value; instead, it invokes the provided callback for each format entry with the key and corresponding value.
- **Functions called**:
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_add<!-- {{#callable:format_add}} -->
The `format_add` function adds a key-value pair to a format tree.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure where the key-value pair will be added.
    - `key`: A string representing the key to be added to the format tree.
    - `fmt`: A format string that will be used to create the value associated with the key.
    - `...`: Additional arguments that will be used to format the value.
- **Control Flow**:
    - Allocate memory for a new `format_entry` structure and duplicate the `key` string into it.
    - Attempt to insert the new entry into the `format_entry_tree` of the `format_tree`.
    - If an entry with the same key already exists, free the memory of the new entry and its value, and reuse the existing entry.
    - Initialize the callback and time fields of the entry.
    - Use `va_start` to initialize the variable argument list and format the value using [`xvasprintf`](xmalloc.c.driver.md#xvasprintf).
    - Use `va_end` to clean up the variable argument list.
- **Output**:
    - The function does not return a value; it modifies the `format_tree` by adding or updating a key-value pair.
- **Functions called**:
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`xvasprintf`](xmalloc.c.driver.md#xvasprintf)


---
### format_add_tv<!-- {{#callable:format_add_tv}} -->
The `format_add_tv` function adds a key-value pair to a format tree with an associated timestamp.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure where the key-value pair will be added.
    - `key`: A string representing the key to be added to the format tree.
    - `tv`: A pointer to a `struct timeval` that contains the timestamp to be associated with the key.
- **Control Flow**:
    - Allocate memory for a new `format_entry` structure and duplicate the provided key string.
    - Attempt to insert the new entry into the `format_entry_tree` of the format tree.
    - If an entry with the same key already exists, free the memory of the new entry and its key, and also free the value of the existing entry.
    - Set the callback pointer to NULL and assign the timestamp from the `tv` structure to the new entry.
    - Set the value of the new entry to NULL.
- **Output**:
    - The function does not return a value; it modifies the `format_tree` by adding or updating an entry with the specified key and timestamp.
- **Functions called**:
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_add_cb<!-- {{#callable:format_add_cb}} -->
The `format_add_cb` function adds a callback function to a format tree entry identified by a key.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that represents the format tree to which the entry will be added.
    - `key`: A string representing the key for the format entry.
    - `cb`: A function pointer of type `format_cb` that serves as the callback for the format entry.
- **Control Flow**:
    - Allocate memory for a new `format_entry` structure and duplicate the provided key into it.
    - Attempt to insert the new entry into the red-black tree of the format tree.
    - If an entry with the same key already exists, free the memory of the new entry and its key, and also free the value of the existing entry.
    - Set the callback function pointer and initialize the time and value fields of the new entry.
- **Output**:
    - The function does not return a value; it modifies the format tree by adding or updating an entry with the specified key and callback.
- **Functions called**:
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


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
    - The character itself is then added to the output string.
    - Finally, the output string is null-terminated and returned.
- **Output**:
    - Returns a pointer to a newly allocated string with special characters escaped for safe use in a shell command.
- **Functions called**:
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)


---
### format_quote_style<!-- {{#callable:format_quote_style}} -->
The `format_quote_style` function duplicates a string while escaping certain characters.
- **Inputs**:
    - `s`: A pointer to a constant character string that needs to be formatted.
- **Control Flow**:
    - The function allocates memory for the output string, which is twice the length of the input string plus one for the null terminator.
    - It iterates through each character of the input string.
    - If the current character is a '#', it is added to the output string without modification.
    - Regardless of whether the character is a '#', the current character is added to the output string.
    - Finally, the output string is null-terminated and returned.
- **Output**:
    - Returns a pointer to a newly allocated string that contains the formatted version of the input string.
- **Functions called**:
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)


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
    - Returns a dynamically allocated string containing the formatted time representation.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_find<!-- {{#callable:format_find}} -->
The `format_find` function retrieves a formatted string based on a specified key and various modifiers.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that contains the context for the format.
    - `key`: A string representing the key for which the formatted value is to be found.
    - `modifiers`: An integer representing various formatting options that modify the output.
    - `time_format`: A string specifying the format for time representation, if applicable.
- **Control Flow**:
    - The function first attempts to retrieve an option associated with the key from various option sources, including global options and session-specific options.
    - If an option is found, it converts it to a string and returns it immediately.
    - If no option is found, it checks a format table for a corresponding entry and retrieves its value using a callback if available.
    - If a format entry is found, it checks if it has a time associated with it and retrieves that time if applicable.
    - If the key is not found in the format table, it checks the environment variables for a match.
    - If the modifiers indicate a time string, it processes the found value to convert it into a formatted time string based on the provided time format.
    - Finally, it applies any additional modifiers such as basename, dirname, or quoting to the found value before returning it.
- **Output**:
    - The function returns a dynamically allocated string containing the formatted value associated with the key, or NULL if no value could be found.
- **Functions called**:
    - [`options_parse_get`](options.c.driver.md#options_parse_get)
    - [`options_to_string`](options.c.driver.md#options_to_string)
    - [`format_table_get`](#format_table_get)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`environ_find`](environ.c.driver.md#environ_find)
    - [`strtonum`](compat/strtonum.c.driver.md#strtonum)
    - [`format_pretty_time`](#format_pretty_time)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
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
    - Returns a newly allocated string with the unescaped content.
- **Functions called**:
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)


---
### format_strip<!-- {{#callable:format_strip}} -->
The `format_strip` function removes specific formatting characters from a string.
- **Inputs**:
    - `s`: A pointer to a constant character string that needs to be processed.
- **Control Flow**:
    - Initialize an output pointer `out` and a character pointer `cp` to point to a newly allocated memory space based on the length of the input string.
    - Iterate through each character of the input string until the null terminator is reached.
    - Count opening brackets when encountering the sequence '#{' and decrement the count when encountering '}'.
    - If a '#' is followed by a character in the set {',', '#', '{', '}', ':'}, check if there are any open brackets; if so, copy the '#' to the output.
    - Copy all other characters directly to the output.
    - Terminate the output string with a null character and return the processed string.
- **Output**:
    - Returns a new string with specific formatting characters removed, dynamically allocated.
- **Functions called**:
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)


---
### format_skip<!-- {{#callable:format_skip}} -->
The `format_skip` function iterates through a string, skipping over certain characters and nested structures until it finds a specified end character.
- **Inputs**:
    - `s`: A pointer to the input string that will be traversed.
    - `end`: A pointer to a string containing characters that signify the end of the traversal.
- **Control Flow**:
    - The function initializes a counter `brackets` to track nested structures.
    - It enters a loop that continues until the end of the string is reached.
    - For each character in the string, it checks if it is the start of a nested structure (indicated by `#{`), incrementing the `brackets` counter.
    - If it encounters a `#` followed by a character in the set `,#{}:`, it skips the next character.
    - When it finds a `}`, it decrements the `brackets` counter.
    - If it encounters a character from the `end` string and the `brackets` counter is zero, it breaks the loop.
    - If the end of the string is reached without finding an end character, it returns NULL.
- **Output**:
    - Returns a pointer to the character in the string where the end character was found, or NULL if the end of the string is reached without finding it.


---
### format_choose<!-- {{#callable:format_choose}} -->
The `format_choose` function separates a string into left and right parts based on a comma delimiter and optionally expands them.
- **Inputs**:
    - `es`: A pointer to a `format_expand_state` structure that holds the state for format expansion.
    - `s`: A string containing the input to be split.
    - `left`: A pointer to a character pointer where the left part of the split string will be stored.
    - `right`: A pointer to a character pointer where the right part of the split string will be stored.
    - `expand`: An integer flag indicating whether to expand the left and right parts.
- **Control Flow**:
    - The function calls [`format_skip`](#format_skip) to find the position of the comma in the input string `s`.
    - If no comma is found, the function returns -1 to indicate an error.
    - The left part of the string is duplicated from the start to the position of the comma.
    - The right part of the string is duplicated from the position after the comma to the end.
    - If the `expand` flag is set, both the left and right parts are expanded using [`format_expand1`](#format_expand1), and the original duplicates are freed.
    - If the `expand` flag is not set, the original duplicates are assigned to the output pointers without expansion.
    - The function returns 0 to indicate success.
- **Output**:
    - The function returns 0 on success, and the left and right parts of the string are stored in the provided pointers. If an error occurs, it returns -1.
- **Functions called**:
    - [`format_skip`](#format_skip)
    - [`xstrndup`](xmalloc.c.driver.md#xstrndup)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`format_expand1`](#format_expand1)


---
### format_true<!-- {{#callable:format_true}} -->
The `format_true` function checks if a given string represents a true value.
- **Inputs**:
    - `s`: A pointer to a constant character string that is to be evaluated for its truthiness.
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
    - The function returns an integer: 1 if the character is a semicolon or colon, and 0 otherwise.


---
### format_add_modifier<!-- {{#callable:format_add_modifier}} -->
The `format_add_modifier` function adds a new `format_modifier` to a list, reallocating memory as necessary.
- **Inputs**:
    - `list`: A pointer to a pointer of `format_modifier` structures, representing the list to which the new modifier will be added.
    - `count`: A pointer to an unsigned integer that keeps track of the current number of modifiers in the list.
    - `c`: A pointer to a character array (string) that represents the modifier to be added.
    - `n`: The size of the modifier string `c`.
    - `argv`: A pointer to an array of character pointers, representing additional arguments associated with the modifier.
    - `argc`: An integer representing the number of additional arguments in `argv`.
- **Control Flow**:
    - The function begins by reallocating the `list` to accommodate one more `format_modifier` using [`xreallocarray`](xmalloc.c.driver.md#xreallocarray).
    - A pointer `fm` is assigned to the newly allocated space for the next `format_modifier`.
    - The function copies the modifier string `c` into the `modifier` field of the new `format_modifier` structure.
    - The size of the modifier is set to `n`, and the `argv` and `argc` fields are populated with the provided arguments.
- **Output**:
    - The function does not return a value; it modifies the `list` and `count` in place to include the new `format_modifier`.
- **Functions called**:
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)


---
### format_free_modifiers<!-- {{#callable:format_free_modifiers}} -->
The `format_free_modifiers` function frees memory associated with an array of `format_modifier` structures.
- **Inputs**:
    - `list`: A pointer to an array of `format_modifier` structures that need to be freed.
    - `count`: An unsigned integer representing the number of `format_modifier` structures in the array.
- **Control Flow**:
    - Iterates over each `format_modifier` in the provided list using a for loop.
    - Calls [`cmd_free_argv`](cmd.c.driver.md#cmd_free_argv) for each `format_modifier` to free its associated argument vector.
    - After freeing all argument vectors, it calls `free` on the list itself to release the memory.
- **Output**:
    - This function does not return a value; it performs memory deallocation.
- **Functions called**:
    - [`cmd_free_argv`](cmd.c.driver.md#cmd_free_argv)


---
### format_build_modifiers<!-- {{#callable:format_build_modifiers}} -->
The `format_build_modifiers` function constructs a list of format modifiers from a given string.
- **Inputs**:
    - `es`: A pointer to a `struct format_expand_state` that holds the state for format expansion.
    - `s`: A pointer to a constant character pointer that points to the string containing the modifiers.
    - `count`: A pointer to an unsigned integer that will hold the count of modifiers found.
- **Control Flow**:
    - The function initializes the count of modifiers to zero.
    - It enters a loop that continues until the end of the string or a colon is encountered.
    - Within the loop, it checks for various types of modifiers, including single-character modifiers, double-character modifiers, and modifiers with arguments.
    - For each recognized modifier, it calls [`format_add_modifier`](#format_add_modifier) to add it to the list.
    - If a colon is encountered before the end of the string, it breaks the loop and returns the list of modifiers.
- **Output**:
    - Returns a pointer to a dynamically allocated array of `struct format_modifier` containing the parsed modifiers, or NULL if the parsing fails.
- **Functions called**:
    - [`format_is_end`](#format_is_end)
    - [`format_add_modifier`](#format_add_modifier)
    - [`format_skip`](#format_skip)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xstrndup`](xmalloc.c.driver.md#xstrndup)
    - [`format_expand1`](#format_expand1)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
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
    - If 'r' is found in `s`, it compiles the regular expression with the appropriate flags and checks for a match using `regexec`, returning '0' if no match is found.
    - Finally, if a match is found, it returns '1'.
- **Output**:
    - The function returns a dynamically allocated string '1' if a match is found, or '0' if no match is found.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_sub<!-- {{#callable:format_sub}} -->
The `format_sub` function performs a regex-based substitution on a given text using a specified pattern and replacement string.
- **Inputs**:
    - `fm`: A pointer to a `format_modifier` structure that contains flags and arguments for the substitution.
    - `text`: The input string on which the substitution will be performed.
    - `pattern`: The regex pattern used to identify the parts of the text to be replaced.
    - `with`: The string that will replace the matched parts of the text.
- **Control Flow**:
    - The function initializes a variable `flags` with the value `REG_EXTENDED` to enable extended regex syntax.
    - If the `fm` structure indicates case-insensitive matching (by checking the third argument), the `REG_ICASE` flag is added to `flags`.
    - The [`regsub`](regsub.c.driver.md#regsub) function is called with the `pattern`, `with`, `text`, and `flags` to perform the substitution.
    - If [`regsub`](regsub.c.driver.md#regsub) returns NULL (indicating failure), the function duplicates the original `text` and returns it.
    - If the substitution is successful, the resulting string is returned.
- **Output**:
    - The function returns a pointer to a new string containing the modified text after substitution, or a duplicate of the original text if the substitution fails.
- **Functions called**:
    - [`regsub`](regsub.c.driver.md#regsub)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_search<!-- {{#callable:format_search}} -->
The `format_search` function searches for a specified string within a given window pane, optionally using case-insensitive and regular expression matching.
- **Inputs**:
    - `fm`: A pointer to a `format_modifier` structure that contains search options, such as case sensitivity and regex usage.
    - `wp`: A pointer to a `window_pane` structure representing the pane in which the search is to be performed.
    - `s`: A constant string that specifies the text to search for within the pane.
- **Control Flow**:
    - The function initializes two integer flags, `ignore` and `regex`, to determine search options.
    - If the `fm` structure has at least one argument, it checks for the presence of 'i' for case insensitivity and 'r' for regex matching, setting the respective flags.
    - The function then calls [`window_pane_search`](window.c.driver.md#window_pane_search) with the pane, search string, and the flags to perform the search.
    - Finally, it formats the result of the search into a string and returns it.
- **Output**:
    - The function returns a dynamically allocated string containing the result of the search, specifically the number of occurrences found.
- **Functions called**:
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`window_pane_search`](window.c.driver.md#window_pane_search)


---
### format_bool_op_1<!-- {{#callable:format_bool_op_1}} -->
The `format_bool_op_1` function evaluates a boolean expression and returns its result as a string.
- **Inputs**:
    - `es`: A pointer to a `format_expand_state` structure that holds the state for format expansion.
    - `fmt`: A string representing the boolean expression to evaluate.
    - `not`: An integer flag indicating whether to negate the result of the evaluation.
- **Control Flow**:
    - The function calls [`format_expand1`](#format_expand1) to expand the `fmt` string into a usable format.
    - It then evaluates the expanded string using [`format_true`](#format_true) to determine if it represents a true value.
    - If the `not` flag is set, the result is negated.
    - The expanded string is freed to avoid memory leaks.
    - Finally, the function returns '1' if the result is true, or '0' if false, as a duplicated string.
- **Output**:
    - The function returns a pointer to a string that is '1' for true or '0' for false, based on the evaluation of the boolean expression.
- **Functions called**:
    - [`format_expand1`](#format_expand1)
    - [`format_true`](#format_true)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_bool_op_n<!-- {{#callable:format_bool_op_n}} -->
The `format_bool_op_n` function evaluates a series of boolean expressions separated by commas, returning '1' for true and '0' for false.
- **Inputs**:
    - `es`: A pointer to a `format_expand_state` structure that holds the state for format expansion.
    - `fmt`: A string containing the boolean expressions to evaluate.
    - `and`: An integer flag indicating whether to evaluate using logical AND (1) or logical OR (0).
- **Control Flow**:
    - The function initializes a result variable based on the value of `and` (1 for true, 0 for false).
    - It enters a loop that continues until the result matches the desired condition (true for AND, false for OR).
    - Within the loop, it skips to the next comma in the `fmt` string to separate expressions.
    - For each expression, it duplicates the substring and expands it using [`format_expand1`](#format_expand1).
    - It logs the current operator and its operand.
    - The result is updated based on the evaluation of the expanded expression using [`format_true`](#format_true).
    - The loop continues until all expressions are evaluated or a terminating condition is met.
- **Output**:
    - The function returns a string '1' if the overall evaluation is true, or '0' if false.
- **Functions called**:
    - [`format_skip`](#format_skip)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`xstrndup`](xmalloc.c.driver.md#xstrndup)
    - [`format_expand1`](#format_expand1)
    - [`format_true`](#format_true)


---
### format_session_name<!-- {{#callable:format_session_name}} -->
The `format_session_name` function checks if a formatted session name exists among the current sessions.
- **Inputs**:
    - `es`: A pointer to a `format_expand_state` structure that holds the state for format expansion.
    - `fmt`: A string containing the format to be expanded and checked against existing session names.
- **Control Flow**:
    - The function calls [`format_expand1`](#format_expand1) to expand the format string `fmt` into a session name.
    - It iterates over all sessions using `RB_FOREACH` to check if any session's name matches the expanded name.
    - If a match is found, it frees the expanded name and returns '1' as a string.
    - If no match is found, it frees the expanded name and returns '0' as a string.
- **Output**:
    - Returns a string '1' if the session name exists, otherwise returns '0'.
- **Functions called**:
    - [`format_expand1`](#format_expand1)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


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
    - The expanded string is concatenated to `value`, and memory is reallocated as necessary.
    - Finally, the function returns the concatenated string containing all expanded format strings for the sessions.
- **Output**:
    - Returns a dynamically allocated string containing the concatenated results of the expanded format strings for all sessions.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`format_create`](#format_create)
    - [`format_defaults`](#format_defaults)
    - [`format_expand1`](#format_expand1)
    - [`format_free`](#format_free)
    - [`xrealloc`](xmalloc.c.driver.md#xrealloc)
    - [`strlcat`](compat/strlcat.c.driver.md#strlcat)


---
### format_window_name<!-- {{#callable:format_window_name}} -->
The `format_window_name` function checks if a given window name exists in the current session.
- **Inputs**:
    - `es`: A pointer to a `format_expand_state` structure that holds the state for format expansion.
    - `fmt`: A string containing the format to be expanded, which represents the window name to check.
- **Control Flow**:
    - The function first checks if the session associated with the `format_tree` in `es` is NULL; if so, it logs an error and returns NULL.
    - It then expands the format string `fmt` to get the actual window name to check.
    - The function iterates over all windows in the current session using a red-black tree traversal.
    - For each window, it compares the window's name with the expanded name.
    - If a match is found, it frees the expanded name and returns a string '1'.
    - If no match is found after checking all windows, it frees the expanded name and returns '0'.
- **Output**:
    - The function returns a string '1' if the window name exists in the current session, otherwise it returns '0'.
- **Functions called**:
    - [`format_expand1`](#format_expand1)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### format_loop_windows<!-- {{#callable:format_loop_windows}} -->
The `format_loop_windows` function expands a format string for each window in the current session.
- **Inputs**:
    - `es`: A pointer to a `format_expand_state` structure that holds the current state of the format expansion.
    - `fmt`: A string containing the format to be expanded for each window.
- **Control Flow**:
    - Checks if the session associated with the format tree is NULL, logging an error if so and returning NULL.
    - Calls [`format_choose`](#format_choose) to determine the format strings for all windows and the active window.
    - Initializes a buffer to accumulate the expanded format strings.
    - Iterates over each window in the session's window list using `RB_FOREACH`.
    - For each window, logs the current window index and ID, and determines which format string to use based on whether the window is active.
    - Creates a new format tree for the current window and sets its defaults.
    - Expands the format string using [`format_expand1`](#format_expand1) and appends the result to the accumulated buffer.
    - Frees the resources associated with the format strings and the format tree after processing all windows.
- **Output**:
    - Returns a dynamically allocated string containing the concatenated results of expanding the format string for each window, or NULL if an error occurs.
- **Functions called**:
    - [`format_choose`](#format_choose)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`format_create`](#format_create)
    - [`format_defaults`](#format_defaults)
    - [`format_expand1`](#format_expand1)
    - [`format_free`](#format_free)
    - [`xrealloc`](xmalloc.c.driver.md#xrealloc)
    - [`strlcat`](compat/strlcat.c.driver.md#strlcat)


---
### format_loop_panes<!-- {{#callable:format_loop_panes}} -->
The `format_loop_panes` function expands a format string for each pane in a window, allowing for dynamic content generation based on the state of each pane.
- **Inputs**:
    - `es`: A pointer to a `format_expand_state` structure that holds the current state of the format expansion.
    - `fmt`: A string containing the format to be expanded for each pane.
- **Control Flow**:
    - Checks if the window associated with the format tree is NULL, logging an error if so and returning NULL.
    - Calls [`format_choose`](#format_choose) to determine the format strings for all panes and the active pane, defaulting to the provided format if necessary.
    - Initializes a buffer to accumulate the expanded format strings for each pane.
    - Iterates over each pane in the window, logging the pane ID and determining which format string to use based on whether the pane is active.
    - Creates a new format tree for each pane and sets its defaults, then expands the format string using [`format_expand1`](#format_expand1).
    - Accumulates the expanded strings into a single output buffer, resizing as necessary.
    - Frees any allocated memory for the format strings and returns the final concatenated string.
- **Output**:
    - Returns a dynamically allocated string containing the concatenated results of expanding the format string for each pane, or NULL if an error occurs.
- **Functions called**:
    - [`format_choose`](#format_choose)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`format_create`](#format_create)
    - [`format_defaults`](#format_defaults)
    - [`format_expand1`](#format_expand1)
    - [`format_free`](#format_free)
    - [`xrealloc`](xmalloc.c.driver.md#xrealloc)
    - [`strlcat`](compat/strlcat.c.driver.md#strlcat)


---
### format_loop_clients<!-- {{#callable:format_loop_clients}} -->
The `format_loop_clients` function expands a format string for each client in a loop.
- **Inputs**:
    - `es`: A pointer to a `format_expand_state` structure that holds the current state of the format expansion.
    - `fmt`: A pointer to a string that contains the format to be expanded for each client.
- **Control Flow**:
    - Initializes a dynamic string `value` to hold the expanded results and sets its length to 1.
    - Iterates over each `client` in the global `clients` list using `TAILQ_FOREACH`.
    - For each client, logs the client's name and creates a new format tree `nft` specific to that client.
    - Defaults are set for the new format tree based on the current client and the existing format tree.
    - Copies the current format expansion state to a new state `next` and sets its format tree to `nft`.
    - Expands the format string `fmt` using the new state and appends the result to `value`.
    - Frees the memory allocated for the expanded string after each iteration.
- **Output**:
    - Returns a dynamically allocated string containing the concatenated results of expanding the format string for each client.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`format_create`](#format_create)
    - [`format_defaults`](#format_defaults)
    - [`format_expand1`](#format_expand1)
    - [`format_free`](#format_free)
    - [`xrealloc`](xmalloc.c.driver.md#xrealloc)
    - [`strlcat`](compat/strlcat.c.driver.md#strlcat)


---
### format_replace_expression<!-- {{#callable:format_replace_expression}} -->
The `format_replace_expression` function evaluates a mathematical expression based on provided operands and an operator.
- **Inputs**:
    - `mexp`: A pointer to a `format_modifier` structure containing the operator and operands for the expression.
    - `es`: A pointer to a `format_expand_state` structure that holds the state of the format expansion.
    - `copy`: A string representing the expression to be evaluated.
- **Control Flow**:
    - The function begins by determining the operator from the first argument in `mexp->argv` using string comparison.
    - It checks for additional flags and precision settings based on the second and third arguments in `mexp->argv`.
    - The function then attempts to parse the left and right operands from the expression using [`format_choose`](#format_choose).
    - It converts the left and right operands from strings to doubles, handling potential errors in conversion.
    - Based on the identified operator, it performs the corresponding mathematical operation on the operands.
    - Finally, it formats the result according to the specified precision and returns it as a string.
- **Output**:
    - Returns a dynamically allocated string containing the result of the evaluated expression, or NULL if an error occurs.
- **Functions called**:
    - [`strtonum`](compat/strtonum.c.driver.md#strtonum)
    - [`format_choose`](#format_choose)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### format_replace<!-- {{#callable:format_replace}} -->
The `format_replace` function processes a format string, applies various modifiers, and replaces placeholders with their corresponding values.
- **Inputs**:
    - `es`: A pointer to a `format_expand_state` structure that holds the current state of the format expansion.
    - `key`: A pointer to a string representing the key to be replaced in the format string.
    - `keylen`: The length of the key string.
    - `buf`: A pointer to a character pointer where the resulting formatted string will be stored.
    - `len`: A pointer to a size_t variable that holds the current length of the buffer.
    - `off`: A pointer to a size_t variable that indicates the current offset in the buffer.
- **Control Flow**:
    - The function begins by making a copy of the key to be processed.
    - It then builds a list of modifiers from the key and processes each modifier to determine its effect on the formatting.
    - Depending on the modifiers, it checks if the key represents a literal string, character, color, or a loop/condition.
    - If the key is a loop or condition, it processes it accordingly, potentially calling other functions to handle specific cases.
    - The function handles various modifiers such as length limits, padding, and substitutions based on the provided modifiers.
    - Finally, it expands the buffer to accommodate the resulting formatted string and copies the result into the buffer.
- **Output**:
    - The function returns 0 on success, indicating that the replacement was successful, or -1 on failure, indicating an error occurred during processing.
- **Functions called**:
    - [`xstrndup`](xmalloc.c.driver.md#xstrndup)
    - [`format_build_modifiers`](#format_build_modifiers)
    - [`format_logging`](#format_logging)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`strtonum`](compat/strtonum.c.driver.md#strtonum)
    - [`format_strip`](#format_strip)
    - [`format_unescape`](#format_unescape)
    - [`format_expand1`](#format_expand1)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`colour_fromstring`](colour.c.driver.md#colour_fromstring)
    - [`colour_force_rgb`](colour.c.driver.md#colour_force_rgb)
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
    - [`format_trim_left`](format-draw.c.driver.md#format_trim_left)
    - [`format_trim_right`](format-draw.c.driver.md#format_trim_right)
    - [`utf8_padcstr`](utf8.c.driver.md#utf8_padcstr)
    - [`utf8_rpadcstr`](utf8.c.driver.md#utf8_rpadcstr)
    - [`format_width`](format-draw.c.driver.md#format_width)
    - [`format_free_modifiers`](#format_free_modifiers)


---
### format_expand1<!-- {{#callable:format_expand1}} -->
The `format_expand1` function expands format strings based on a specified state.
- **Inputs**:
    - `es`: A pointer to a `format_expand_state` structure that holds the current state for format expansion.
    - `fmt`: A pointer to a constant character string representing the format to be expanded.
- **Control Flow**:
    - Checks if the input format string `fmt` is NULL or empty, returning a default string if true.
    - Increments the loop counter in the state to prevent infinite recursion.
    - Logs the format string being expanded.
    - If the `FORMAT_EXPAND_TIME` flag is set and the format string contains a time format, it expands the time using `strftime`.
    - Allocates a buffer for the output and processes each character in the format string.
    - Handles different format specifiers such as `#()`, `#{}`, and others, expanding them as necessary.
    - Logs the result of the expansion and decrements the loop counter before returning the final expanded string.
- **Output**:
    - Returns a dynamically allocated string containing the expanded format based on the input format string and the current state.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`format_logging`](#format_logging)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`xstrndup`](xmalloc.c.driver.md#xstrndup)
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
    - Returns a pointer to a character string containing the expanded format based on the provided time format.
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
    - `item`: A pointer to a `struct cmdq_item` that contains the command context.
    - `fmt`: A pointer to a string that contains the format to be expanded.
    - `c`: A pointer to a `struct client` representing the client context.
    - `s`: A pointer to a `struct session` representing the session context.
    - `wl`: A pointer to a `struct winlink` representing the window link context.
    - `wp`: A pointer to a `struct window_pane` representing the window pane context.
- **Control Flow**:
    - The function begins by creating a new `format_tree` structure using [`format_create_defaults`](#format_create_defaults), which initializes it with default values based on the provided context.
    - It then calls [`format_expand`](#format_expand) to process the format string `fmt` using the created `format_tree`.
    - After the expansion, the function frees the allocated `format_tree` using [`format_free`](#format_free).
    - Finally, it returns the expanded format string.
- **Output**:
    - The function returns a pointer to a dynamically allocated string containing the expanded format.
- **Functions called**:
    - [`format_create_defaults`](#format_create_defaults)
    - [`format_expand`](#format_expand)
    - [`format_free`](#format_free)


---
### format_single_from_state<!-- {{#callable:format_single_from_state}} -->
The `format_single_from_state` function expands a format string using the state of a command item, client, and command find state.
- **Inputs**:
    - `item`: A pointer to a `struct cmdq_item` representing the command item that contains the context for the format expansion.
    - `fmt`: A pointer to a constant character string that specifies the format to be expanded.
    - `c`: A pointer to a `struct client` representing the client context for the format expansion.
    - `fs`: A pointer to a `struct cmd_find_state` that contains the state information for the command find operation.
- **Control Flow**:
    - The function calls [`format_single`](#format_single) with the provided arguments, including the session, winlink, and window pane extracted from the `cmd_find_state` structure.
    - The [`format_single`](#format_single) function performs the actual format expansion based on the provided format string and the context from the command item and client.
- **Output**:
    - Returns a pointer to a dynamically allocated string containing the expanded format result.
- **Functions called**:
    - [`format_single`](#format_single)


---
### format_single_from_target<!-- {{#callable:format_single_from_target}} -->
The `format_single_from_target` function formats a string based on a target client and a specified format.
- **Inputs**:
    - `item`: A pointer to a `struct cmdq_item` that contains the command queue item.
    - `fmt`: A pointer to a constant character string that specifies the format to be applied.
- **Control Flow**:
    - The function retrieves the target client associated with the provided `item` using [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client).
    - It then calls [`format_single_from_state`](#format_single_from_state), passing the `item`, `fmt`, the retrieved client, and the target state obtained from [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target).
- **Output**:
    - Returns a pointer to a formatted string based on the specified format and the target client.
- **Functions called**:
    - [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client)
    - [`format_single_from_state`](#format_single_from_state)
    - [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target)


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
    - Returns a pointer to a `format_tree` structure initialized with default values.
- **Functions called**:
    - [`format_create`](#format_create)
    - [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client)
    - [`format_defaults`](#format_defaults)


---
### format_create_from_state<!-- {{#callable:format_create_from_state}} -->
Creates a `format_tree` structure from the given state of a command.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item.
    - `c`: A pointer to a `client` structure representing the client associated with the command.
    - `fs`: A pointer to a `cmd_find_state` structure containing the state of the command find operation.
- **Control Flow**:
    - The function calls [`format_create_defaults`](#format_create_defaults) with the provided `item`, `client`, and the session, window link, and window pane from the `cmd_find_state` structure.
    - The [`format_create_defaults`](#format_create_defaults) function initializes a new `format_tree` with default values based on the provided parameters.
- **Output**:
    - Returns a pointer to a `format_tree` structure that has been initialized with default values based on the provided command state.
- **Functions called**:
    - [`format_create_defaults`](#format_create_defaults)


---
### format_create_from_target<!-- {{#callable:format_create_from_target}} -->
Creates a `format_tree` structure from a target client.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure that contains the command queue item.
- **Control Flow**:
    - The function retrieves the target client associated with the provided `cmdq_item` using [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client).
    - It then calls [`format_create_from_state`](#format_create_from_state), passing the `cmdq_item`, the target client, and the target state obtained from [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target).
- **Output**:
    - Returns a pointer to a `format_tree` structure that has been created based on the target client and command queue item.
- **Functions called**:
    - [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client)
    - [`format_create_from_state`](#format_create_from_state)
    - [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target)


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
    - Logs the state of the client, session, winlink, and window pane if they are not NULL.
    - Checks if the provided client and session match; if not, logs a message.
    - Determines the type of format based on the presence of the window pane, winlink, or session.
    - If the session is NULL but a client is provided, it assigns the client's session to the session variable.
    - If the winlink is NULL but a session is provided, it assigns the current winlink of the session.
    - If the window pane is NULL but a winlink is provided, it assigns the active window of the winlink.
    - Calls helper functions to set default format values for the client, session, winlink, and window pane.
    - Retrieves the top paste buffer and sets its default format values if it exists.
- **Output**:
    - The function does not return a value but modifies the `format_tree` structure to include default values based on the provided parameters.
- **Functions called**:
    - [`format_defaults_client`](#format_defaults_client)
    - [`format_defaults_session`](#format_defaults_session)
    - [`format_defaults_winlink`](#format_defaults_winlink)
    - [`format_defaults_pane`](#format_defaults_pane)
    - [`paste_get_top`](paste.c.driver.md#paste_get_top)
    - [`format_defaults_paste_buffer`](#format_defaults_paste_buffer)


---
### format_defaults_session<!-- {{#callable:format_defaults_session}} -->
The `format_defaults_session` function sets the session pointer in a format tree.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that holds formatting information.
    - `s`: A pointer to a `session` structure representing the session to be associated with the format tree.
- **Control Flow**:
    - The function directly assigns the session pointer `s` to the `s` member of the `format_tree` structure `ft`.
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
The `format_defaults_window` function sets the `window` pointer in a `format_tree` structure.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that holds various formatting information.
    - `w`: A pointer to a `window` structure that represents the window to be associated with the format tree.
- **Control Flow**:
    - The function directly assigns the `window` pointer of the `format_tree` structure to the provided `window` pointer.
    - There are no conditional statements or loops in this function.
- **Output**:
    - The function does not return a value; it modifies the `format_tree` structure in place.


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
    - The function does not return a value; it modifies the `format_tree` structure in place to include the winlink's default settings.
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
    - Assigns the `window_pane` pointer to the `format_tree` structure.
    - Retrieves the first `window_mode_entry` from the list of modes associated with the pane.
    - If a mode entry exists and it has a `formats` callback, it calls this callback with the mode entry and the format tree.
- **Output**:
    - The function does not return a value; it modifies the `format_tree` structure to include default values for the specified pane.
- **Functions called**:
    - [`format_defaults_window`](#format_defaults_window)


---
### format_defaults_paste_buffer<!-- {{#callable:format_defaults_paste_buffer}} -->
The `format_defaults_paste_buffer` function assigns a `paste_buffer` to a `format_tree` structure.
- **Inputs**:
    - `ft`: A pointer to a `format_tree` structure that holds various formatting information.
    - `pb`: A pointer to a `paste_buffer` structure that contains the data to be associated with the format tree.
- **Control Flow**:
    - The function directly assigns the `paste_buffer` pointer `pb` to the `pb` member of the `format_tree` structure `ft`.
- **Output**:
    - The function does not return a value; it modifies the `format_tree` structure in place by setting its `pb` member.


---
### format_is_word_separator<!-- {{#callable:format_is_word_separator}} -->
The `format_is_word_separator` function checks if a given grid cell is a word separator based on specified criteria.
- **Inputs**:
    - `ws`: A string containing characters that are considered as word separators.
    - `gc`: A pointer to a `struct grid_cell` that represents the grid cell to be checked.
- **Control Flow**:
    - The function first checks if the string `ws` contains the data from the grid cell `gc` using the [`utf8_cstrhas`](utf8.c.driver.md#utf8_cstrhas) function.
    - If the first check fails, it checks if the `gc` has the `GRID_FLAG_TAB` flag set, indicating it is a tab character.
    - If both checks fail, it finally checks if the size of `gc->data` is 1 and if the character is a space.
- **Output**:
    - Returns 1 if the grid cell is a word separator, otherwise returns 0.
- **Functions called**:
    - [`utf8_cstrhas`](utf8.c.driver.md#utf8_cstrhas)


---
### format_grid_word<!-- {{#callable:format_grid_word}} -->
The `format_grid_word` function retrieves a word from a grid at specified coordinates.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` representing the grid from which to extract the word.
    - `x`: An unsigned integer representing the x-coordinate in the grid.
    - `y`: An unsigned integer representing the y-coordinate in the grid.
- **Control Flow**:
    - The function initializes variables to track the grid cell and word separators.
    - It enters a loop to move left in the grid until it finds a word separator or reaches the start of the grid.
    - If a word separator is found, it marks the position and continues to find the end of the word.
    - It then enters another loop to move right from the found position until it finds another word separator or the end of the line.
    - The characters between the two separators are collected into a UTF-8 data structure.
    - Finally, it converts the collected data into a string and returns it.
- **Output**:
    - Returns a dynamically allocated string containing the word found at the specified coordinates, or NULL if no word is found.
- **Functions called**:
    - [`options_get_string`](options.c.driver.md#options_get_string)
    - [`grid_get_cell`](grid.c.driver.md#grid_get_cell)
    - [`format_is_word_separator`](#format_is_word_separator)
    - [`grid_peek_line`](grid.c.driver.md#grid_peek_line)
    - [`grid_line_length`](grid.c.driver.md#grid_line_length)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`utf8_tocstr`](utf8.c.driver.md#utf8_tocstr)


---
### format_grid_line<!-- {{#callable:format_grid_line}} -->
The `format_grid_line` function constructs a string representation of a specified line in a grid, excluding padding cells.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` that represents the grid from which the line is to be formatted.
    - `y`: An unsigned integer representing the index of the line in the grid to be formatted.
- **Control Flow**:
    - Iterates over each cell in the specified line of the grid using the [`grid_line_length`](grid.c.driver.md#grid_line_length) function to determine the length of the line.
    - For each cell, it retrieves the cell's data and checks if it has the `GRID_FLAG_PADDING` flag; if so, it skips that cell.
    - If the cell is a tab (indicated by the `GRID_FLAG_TAB` flag), it adds a tab character to the output string; otherwise, it copies the cell's data directly.
    - After processing all relevant cells, it null-terminates the UTF-8 data and converts it to a C string using [`utf8_tocstr`](utf8.c.driver.md#utf8_tocstr).
- **Output**:
    - Returns a dynamically allocated string containing the formatted line from the grid, or NULL if the line is empty.
- **Functions called**:
    - [`grid_line_length`](grid.c.driver.md#grid_line_length)
    - [`grid_get_cell`](grid.c.driver.md#grid_get_cell)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`utf8_set`](utf8.c.driver.md#utf8_set)
    - [`utf8_tocstr`](utf8.c.driver.md#utf8_tocstr)


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
    - If the x-coordinate reaches 0 and no valid cell is found, the function returns NULL.
    - If the screen's hyperlink list is NULL or the cell does not have a link, the function returns NULL.
    - The function attempts to retrieve the URI associated with the cell's link using the [`hyperlinks_get`](hyperlinks.c.driver.md#hyperlinks_get) function.
    - If successful, it duplicates the URI string and returns it; otherwise, it returns NULL.
- **Output**:
    - Returns a pointer to a dynamically allocated string containing the hyperlink URI if found, or NULL if no hyperlink is associated with the specified cell.
- **Functions called**:
    - [`grid_get_cell`](grid.c.driver.md#grid_get_cell)
    - [`hyperlinks_get`](hyperlinks.c.driver.md#hyperlinks_get)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


