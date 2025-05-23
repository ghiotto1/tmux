# Purpose
This C source code file is part of the tmux terminal multiplexer project and implements the functionality for listing panes within a specified window. The code defines a command, "list-panes" (with an alias "lsp"), which can be executed to display information about panes in a tmux session. The command can be targeted at a specific window, session, or all sessions, and it supports options for formatting and filtering the output. The primary function, [`cmd_list_panes_exec`](#cmd_list_panes_exec), determines the scope of the listing based on command-line arguments and delegates the task to helper functions that iterate over sessions, windows, and panes to gather and format the required information.

The file is structured around the `cmd_list_panes_entry` structure, which defines the command's name, alias, arguments, usage, and execution function. The execution function, [`cmd_list_panes_exec`](#cmd_list_panes_exec), uses helper functions like [`cmd_list_panes_server`](#cmd_list_panes_server), [`cmd_list_panes_session`](#cmd_list_panes_session), and [`cmd_list_panes_window`](#cmd_list_panes_window) to traverse the data structures representing tmux sessions and windows. These functions utilize a format tree to apply user-specified or default templates to format the output for each pane, allowing for detailed and customizable pane information to be printed. This code is a specific component of the tmux command suite, providing a focused utility for users to inspect the state and configuration of panes within their tmux environment.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `tmux.h`


# Global Variables

---
### cmd_list_panes_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_list_panes_entry` is a constant structure of type `cmd_entry` that defines the command for listing panes in a given window within the tmux application. It includes the command name, alias, arguments, usage, target, flags, and the execution function associated with the command.
- **Use**: This variable is used to register and define the behavior of the 'list-panes' command in tmux, allowing users to list all panes in a specified window.


# Functions

---
### cmd_list_panes_exec<!-- {{#callable:cmd_list_panes_exec}} -->
The `cmd_list_panes_exec` function executes the command to list panes based on the specified arguments, targeting either the server, a session, or a window.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve the arguments associated with the command using `cmd_get_args(self)`.
    - Get the target session and window link from the command queue item using `cmdq_get_target(item)`.
    - Check if the arguments include the 'a' flag; if so, call `cmd_list_panes_server(self, item)` to list panes for the entire server.
    - If the 's' flag is present, call `cmd_list_panes_session(self, s, item, 1)` to list panes for the specified session.
    - If neither 'a' nor 's' flags are present, call `cmd_list_panes_window(self, s, wl, item, 0)` to list panes for the specified window.
    - Return `CMD_RETURN_NORMAL` to indicate successful execution.
- **Output**:
    - The function returns an enum value `CMD_RETURN_NORMAL` indicating the command executed successfully.
- **Functions called**:
    - [`cmd_list_panes_server`](#cmd_list_panes_server)
    - [`cmd_list_panes_session`](#cmd_list_panes_session)
    - [`cmd_list_panes_window`](#cmd_list_panes_window)


---
### cmd_list_panes_server<!-- {{#callable:cmd_list_panes_server}} -->
The `cmd_list_panes_server` function iterates over all sessions and lists panes for each session using a specific format.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - The function uses the `RB_FOREACH` macro to iterate over all sessions in the `sessions` red-black tree.
    - For each session, it calls the [`cmd_list_panes_session`](#cmd_list_panes_session) function with the current session, the command, the command queue item, and a type value of 2.
- **Output**:
    - The function does not return a value; it performs its operations by calling another function to list panes for each session.
- **Functions called**:
    - [`cmd_list_panes_session`](#cmd_list_panes_session)


---
### cmd_list_panes_session<!-- {{#callable:cmd_list_panes_session}} -->
The `cmd_list_panes_session` function iterates over all windows in a given session and lists the panes for each window using a specified format.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `s`: A pointer to the `session` structure representing the session whose panes are to be listed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
    - `type`: An integer indicating the format type to be used when listing panes.
- **Control Flow**:
    - The function uses the `RB_FOREACH` macro to iterate over each `winlink` in the session's `windows` red-black tree.
    - For each `winlink`, it calls the [`cmd_list_panes_window`](#cmd_list_panes_window) function, passing the current window link and other parameters.
- **Output**:
    - The function does not return a value; it performs its operations by calling another function to list panes for each window in the session.
- **Functions called**:
    - [`cmd_list_panes_window`](#cmd_list_panes_window)


---
### cmd_list_panes_window<!-- {{#callable:cmd_list_panes_window}} -->
The `cmd_list_panes_window` function lists all panes in a specified window, formatting the output based on a given or default template and applying an optional filter.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `s`: A pointer to the `session` structure representing the current session.
    - `wl`: A pointer to the `winlink` structure representing the window link in the session.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
    - `type`: An integer indicating the type of template to use for formatting the output.
- **Control Flow**:
    - Retrieve the command arguments using `cmd_get_args`.
    - Determine the template for formatting output based on the `type` parameter or use a default template if none is provided.
    - Retrieve an optional filter from the arguments using `args_get`.
    - Initialize a counter `n` to zero for pane indexing.
    - Iterate over each pane in the window using `TAILQ_FOREACH`.
    - For each pane, create a format tree with `format_create` and add default formatting values.
    - If a filter is provided, expand it using `format_expand` and evaluate it with `format_true`; otherwise, set the flag to true.
    - If the flag is true, expand the template using `format_expand` and print the formatted line with `cmdq_print`.
    - Free the format tree and increment the pane counter `n`.
- **Output**:
    - The function outputs formatted information about each pane in the specified window to the command queue item, applying any specified filter.


# Function Declarations (Public API)

---
### cmd_list_panes_exec<!-- {{#callable_declaration:cmd_list_panes_exec}} -->
Lists panes in the specified window, session, or server.
- **Description**: This function is used to list all panes within a specified target, which can be a window, session, or the entire server, depending on the arguments provided. It should be called when you need to retrieve and display information about panes, such as their dimensions and status, in a structured format. The function requires a valid command and command queue item, and it assumes that the target window or session is correctly specified. It handles different levels of detail based on the arguments, allowing for server-wide, session-specific, or window-specific pane listings.
- **Inputs**:
    - `self`: A pointer to a 'struct cmd' representing the command to be executed. Must not be null.
    - `item`: A pointer to a 'struct cmdq_item' representing the command queue item. Must not be null.
- **Output**: Returns an enum cmd_retval indicating the success of the operation, typically CMD_RETURN_NORMAL.
- **See also**: [`cmd_list_panes_exec`](#cmd_list_panes_exec)  (Implementation)


---
### cmd_list_panes_server<!-- {{#callable_declaration:cmd_list_panes_server}} -->
Lists panes for all sessions on the server.
- **Description**: This function is used to list all panes across all sessions on the server. It should be called when a comprehensive overview of all panes in all sessions is required. The function iterates over each session and lists the panes within them. It is typically invoked when the 'list-panes' command is executed with the appropriate arguments to target all sessions. This function does not return any value and is expected to be used within the context of command execution in a session management environment.
- **Inputs**:
    - `self`: A pointer to a 'cmd' structure representing the command being executed. Must not be null.
    - `item`: A pointer to a 'cmdq_item' structure representing the command queue item. Must not be null.
- **Output**: None
- **See also**: [`cmd_list_panes_server`](#cmd_list_panes_server)  (Implementation)


---
### cmd_list_panes_session<!-- {{#callable_declaration:cmd_list_panes_session}} -->
Lists all panes in a given session.
- **Description**: This function is used to iterate over all windows in a specified session and list the panes within each window. It is typically called when there is a need to display or process information about all panes in a session. The function requires a valid session object and is part of a command execution flow, where it interacts with command and queue items. It should be used in contexts where the session is guaranteed to be valid and initialized.
- **Inputs**:
    - `self`: A pointer to a cmd structure representing the command context. Must not be null.
    - `s`: A pointer to a session structure representing the session whose panes are to be listed. Must not be null and should point to a valid session.
    - `item`: A pointer to a cmdq_item structure representing the command queue item. Must not be null and is used for command execution context.
    - `type`: An integer indicating the type of listing format to be used. Valid values are typically 0, 1, or 2, each corresponding to a different format style.
- **Output**: None
- **See also**: [`cmd_list_panes_session`](#cmd_list_panes_session)  (Implementation)


---
### cmd_list_panes_window<!-- {{#callable_declaration:cmd_list_panes_window}} -->
List panes in a specified window.
- **Description**: This function is used to list all panes within a specified window of a session. It formats and prints information about each pane according to a specified template or a default format if none is provided. The function can also apply a filter to determine which panes to include in the output. It is typically called when detailed information about the panes in a window is needed, such as their dimensions, history size, and status. The function should be called with valid session and window link objects, and it assumes that the command queue item is properly initialized.
- **Inputs**:
    - `self`: A pointer to the command structure. This parameter is used to retrieve command arguments and must not be null.
    - `s`: A pointer to the session structure representing the current session. This parameter must not be null.
    - `wl`: A pointer to the winlink structure representing the window link within the session. This parameter must not be null.
    - `item`: A pointer to the command queue item structure used for output. This parameter must not be null.
    - `type`: An integer indicating the type of format to use for the output. Valid values are 0, 1, or 2, each corresponding to a different default template format.
- **Output**: None
- **See also**: [`cmd_list_panes_window`](#cmd_list_panes_window)  (Implementation)


