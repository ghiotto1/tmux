# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file specifically implements the functionality to list windows within a given tmux session. It defines a command, "list-windows" (with an alias "lsw"), which can be executed to display information about the windows in a session. The command can be customized with options such as specifying a format for the output or applying a filter to the list of windows. The code includes templates for formatting the output, which display details like window index, name, number of panes, dimensions, layout, and whether the window is active.

The file contains several key components: the command entry structure (`cmd_list_windows_entry`), which defines the command's name, alias, arguments, usage, and execution function; and the execution logic ([`cmd_list_windows_exec`](#cmd_list_windows_exec)), which determines whether to list windows for a specific session or all sessions. The code uses a recursive binary tree (RB tree) to iterate over sessions and windows, applying the specified format and filter to generate the output. This file is a part of the broader tmux codebase and is intended to be compiled and linked with other components to form the complete tmux application. It does not define public APIs or external interfaces but rather contributes to the internal command processing mechanism of tmux.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `unistd.h`
- `tmux.h`


# Global Variables

---
### cmd_list_windows_entry
- **Type**: ``const struct cmd_entry``
- **Description**: The `cmd_list_windows_entry` is a constant structure of type `cmd_entry` that defines the command for listing windows in a session. It includes the command name, alias, arguments, usage, target, flags, and the execution function. This structure is used to register and handle the 'list-windows' command within the application.
- **Use**: This variable is used to define and execute the 'list-windows' command, which lists all windows in a specified session.


# Functions

---
### cmd_list_windows_exec<!-- {{#callable:cmd_list_windows_exec}} -->
The `cmd_list_windows_exec` function executes the command to list windows either for the entire server or a specific session based on the provided arguments.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve the arguments associated with the command using `cmd_get_args(self)` and store them in `args`.
    - Retrieve the target session from the command queue item using `cmdq_get_target(item)` and store it in `target`.
    - Check if the argument 'a' is present in `args` using `args_has(args, 'a')`.
    - If 'a' is present, call `cmd_list_windows_server(self, item)` to list windows for the entire server.
    - If 'a' is not present, call `cmd_list_windows_session(self, target->s, item, 0)` to list windows for the specific session identified by `target->s`.
    - Return `CMD_RETURN_NORMAL` to indicate successful execution.
- **Output**:
    - The function returns an `enum cmd_retval` value, specifically `CMD_RETURN_NORMAL`, indicating the command executed successfully.
- **Functions called**:
    - [`cmd_list_windows_server`](#cmd_list_windows_server)
    - [`cmd_list_windows_session`](#cmd_list_windows_session)


---
### cmd_list_windows_server<!-- {{#callable:cmd_list_windows_server}} -->
The `cmd_list_windows_server` function iterates over all sessions and lists windows for each session using a specified format.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - The function begins by declaring a pointer `s` to a `session` structure.
    - It uses the `RB_FOREACH` macro to iterate over all sessions in the global `sessions` tree.
    - For each session `s`, it calls the [`cmd_list_windows_session`](#cmd_list_windows_session) function with the current session, the command, the command queue item, and a flag set to 1.
- **Output**:
    - The function does not return a value; it performs its operations by calling another function to list windows for each session.
- **Functions called**:
    - [`cmd_list_windows_session`](#cmd_list_windows_session)


---
### cmd_list_windows_session<!-- {{#callable:cmd_list_windows_session}} -->
The `cmd_list_windows_session` function lists all windows in a given session, optionally filtering and formatting the output based on provided arguments.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `s`: A pointer to the `session` structure representing the session whose windows are to be listed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item for output.
    - `type`: An integer indicating the type of template to use for formatting the output (0 for default, 1 for including session name).
- **Control Flow**:
    - Retrieve the command arguments using `cmd_get_args`.
    - Determine the template for formatting the output based on the `type` argument or use a default template if not specified.
    - Retrieve the filter argument if provided.
    - Initialize a counter `n` to zero for window indexing.
    - Iterate over each window link (`winlink`) in the session's windows using `RB_FOREACH`.
    - For each window, create a format tree and add default formatting values.
    - If a filter is specified, expand it using the format tree and evaluate its truth value to decide if the window should be included.
    - If the window passes the filter (or no filter is specified), expand the template using the format tree and print the formatted line using `cmdq_print`.
    - Free the format tree and increment the window counter `n`.
- **Output**:
    - The function does not return a value but outputs formatted information about each window in the session to the command queue item.


# Function Declarations (Public API)

---
### cmd_list_windows_exec<!-- {{#callable_declaration:cmd_list_windows_exec}} -->
Lists windows for a specified session or all sessions.
- **Description**: This function is used to list windows associated with a specific session or across all sessions, depending on the provided arguments. It should be called when a user needs to retrieve and display information about windows in a session or all sessions. The function requires a valid command structure and command queue item to execute properly. It handles the presence of specific arguments to determine whether to list windows for a single session or all sessions.
- **Inputs**:
    - `self`: A pointer to a 'struct cmd' representing the command to be executed. Must not be null and should be properly initialized with the necessary arguments.
    - `item`: A pointer to a 'struct cmdq_item' representing the command queue item. Must not be null and should be associated with a valid command queue.
- **Output**: Returns an enum value 'CMD_RETURN_NORMAL' indicating successful execution.
- **See also**: [`cmd_list_windows_exec`](#cmd_list_windows_exec)  (Implementation)


---
### cmd_list_windows_server<!-- {{#callable_declaration:cmd_list_windows_server}} -->
Lists all windows across all sessions on the server.
- **Description**: This function is used to list all windows from every session currently active on the server. It should be called when a comprehensive overview of all windows across sessions is needed. The function iterates over each session and lists the windows within them, providing details according to a predefined format. It is typically invoked when the '-a' flag is used with the list-windows command, indicating that the operation should apply to all sessions. The function does not return any value and is expected to be used within the context of a command execution framework that handles command and session objects.
- **Inputs**:
    - `self`: A pointer to a 'cmd' structure representing the command being executed. The caller retains ownership and it must not be null.
    - `item`: A pointer to a 'cmdq_item' structure representing the command queue item. The caller retains ownership and it must not be null.
- **Output**: None
- **See also**: [`cmd_list_windows_server`](#cmd_list_windows_server)  (Implementation)


---
### cmd_list_windows_session<!-- {{#callable_declaration:cmd_list_windows_session}} -->
List windows in a specified session.
- **Description**: This function is used to list all windows within a given session, optionally applying a format and filter to the output. It is typically called when a user wants to view the windows associated with a particular session in a tmux environment. The function requires a valid session and command queue item, and it can use a specified format template or a default one based on the type parameter. Filters can be applied to control which windows are included in the output. The function must be called with a valid session and command queue item, and it assumes that the session contains a list of windows.
- **Inputs**:
    - `self`: A pointer to the command structure. This parameter is used to access command arguments and must not be null.
    - `s`: A pointer to the session structure representing the session whose windows are to be listed. This parameter must not be null and should point to a valid session object.
    - `item`: A pointer to the command queue item structure. This parameter is used for output and must not be null.
    - `type`: An integer indicating the type of template to use for formatting the output. Valid values are 0 or 1, which determine the default template if none is specified in the command arguments.
- **Output**: None
- **See also**: [`cmd_list_windows_session`](#cmd_list_windows_session)  (Implementation)


