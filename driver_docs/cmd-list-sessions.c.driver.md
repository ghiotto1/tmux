# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file specifically implements the functionality to list all active sessions within tmux. It defines a command, "list-sessions" (with an alias "ls"), which can be executed to display information about each session, including the session name, the number of windows in the session, the creation time, and whether the session is part of a group or currently attached. The command supports optional formatting and filtering through the use of command-line arguments, allowing users to customize the output.

The file includes a key function, [`cmd_list_sessions_exec`](#cmd_list_sessions_exec), which is responsible for iterating over all sessions and applying the specified format and filter to generate the output. The function utilizes a format tree to manage the formatting of session details, and it employs a red-black tree to traverse the list of sessions. The code is structured to be part of a larger application, with the command entry defined in a `cmd_entry` structure, indicating that it is intended to be integrated into the tmux command execution framework. This file does not define a public API but rather contributes to the internal command handling mechanism of tmux.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `time.h`
- `tmux.h`


# Global Variables

---
### cmd_list_sessions_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_list_sessions_entry` is a constant structure of type `cmd_entry` that defines the command for listing all sessions in the application. It includes the command name 'list-sessions', an alias 'ls', argument specifications, usage instructions, command flags, and a pointer to the execution function `cmd_list_sessions_exec`. This structure is part of the command handling mechanism in the application, allowing users to list active sessions with optional formatting and filtering.
- **Use**: This variable is used to register and define the behavior of the 'list-sessions' command within the application.


# Functions

---
### cmd_list_sessions_exec<!-- {{#callable:cmd_list_sessions_exec}} -->
The `cmd_list_sessions_exec` function lists all active sessions, optionally filtering and formatting the output based on user-specified arguments.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve the command arguments using `cmd_get_args` function.
    - Check if a custom format template is provided with the '-F' argument; if not, use the default `LIST_SESSIONS_TEMPLATE`.
    - Retrieve the filter expression from the '-f' argument, if provided.
    - Initialize a counter `n` to zero to track the session index.
    - Iterate over each session in the global `sessions` tree using `RB_FOREACH`.
    - For each session, create a format tree using `format_create` and add default session information to it.
    - If a filter is specified, expand it using `format_expand` and evaluate it with `format_true`; otherwise, set the flag to true.
    - If the flag is true, expand the format template and print the formatted session information using `cmdq_print`.
    - Free the allocated format tree and increment the session counter `n`.
    - Return `CMD_RETURN_NORMAL` to indicate successful execution.
- **Output**:
    - The function returns `CMD_RETURN_NORMAL`, indicating successful execution of the command.


# Function Declarations (Public API)

---
### cmd_list_sessions_exec<!-- {{#callable_declaration:cmd_list_sessions_exec}} -->
Lists all active sessions with optional formatting and filtering.
- **Description**: This function is used to list all active sessions in the system, providing the ability to format the output using a specified template and to filter sessions based on a given condition. It should be called when a user needs to retrieve and display session information, potentially in a customized format. The function expects a command structure and a command queue item as inputs, and it processes each session, applying the specified format and filter if provided. The default template is used if no custom format is specified. The function assumes that the session data is available and valid, and it outputs the formatted session information to the command queue.
- **Inputs**:
    - `self`: A pointer to a 'struct cmd' representing the command context. Must not be null. The caller retains ownership.
    - `item`: A pointer to a 'struct cmdq_item' representing the command queue item where output will be printed. Must not be null. The caller retains ownership.
- **Output**: Returns CMD_RETURN_NORMAL to indicate successful execution.
- **See also**: [`cmd_list_sessions_exec`](#cmd_list_sessions_exec)  (Implementation)


