# Purpose
This C source code file is part of the tmux terminal multiplexer project and implements the functionality for listing panes within a specified window. The code defines a command, "list-panes" (with an alias "lsp"), which can be executed to display information about the panes in a tmux session. The command can be targeted at a specific window, session, or all sessions, and it supports options for formatting and filtering the output. The primary function, [`cmd_list_panes_exec`](#cmd_list_panes_exec), determines the scope of the listing based on the provided arguments and calls the appropriate helper functions to gather and display the pane information.

The file is structured around the `cmd_list_panes_entry` structure, which defines the command's name, alias, arguments, usage, target, flags, and execution function. The execution function, [`cmd_list_panes_exec`](#cmd_list_panes_exec), delegates the task of listing panes to specific functions depending on whether the command is to list panes for the entire server, a specific session, or a specific window. The code utilizes a format tree to customize the output format, allowing users to specify how pane information is displayed. This file is a part of the broader tmux codebase and is designed to be integrated into the tmux command execution framework, providing a specific utility within the larger application.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `tmux.h`


# Global Variables

---
### cmd_list_panes_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_list_panes_entry` is a constant structure of type `cmd_entry` that defines the command for listing panes in a given window within the tmux application. It includes the command name, alias, arguments, usage information, target specifications, flags, and the execution function associated with the command.
- **Use**: This variable is used to register and define the behavior of the 'list-panes' command in tmux, allowing users to list all panes in a specified window.


# Functions

---
### cmd_list_panes_exec<!-- {{#callable:cmd_list_panes_exec}} -->
The `cmd_list_panes_exec` function executes the command to list panes based on the specified arguments, targeting either the entire server, a session, or a specific window.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item, which provides context for the command execution.
- **Control Flow**:
    - Retrieve the command arguments using `cmd_get_args(self)` and store them in `args`.
    - Get the target session and window link from the command queue item using `cmdq_get_target(item)` and store them in `target`, `s`, and `wl`.
    - Check if the 'a' flag is present in the arguments using `args_has(args, 'a')`. If true, call `cmd_list_panes_server(self, item)` to list panes for the entire server.
    - If the 'a' flag is not present, check if the 's' flag is present using `args_has(args, 's')`. If true, call `cmd_list_panes_session(self, s, item, 1)` to list panes for the session.
    - If neither 'a' nor 's' flags are present, call `cmd_list_panes_window(self, s, wl, item, 0)` to list panes for the specific window.
    - Return `CMD_RETURN_NORMAL` to indicate successful execution.
- **Output**:
    - The function returns an enum value `CMD_RETURN_NORMAL` indicating the command executed successfully.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target)
    - [`args_has`](arguments.c.driver.md#args_has)
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
    - The function begins by declaring a pointer `s` to a `session` structure.
    - It uses the `RB_FOREACH` macro to iterate over all sessions in the global `sessions` tree.
    - For each session `s`, it calls the [`cmd_list_panes_session`](#cmd_list_panes_session) function with `self`, `s`, `item`, and the integer `2` as arguments.
- **Output**:
    - The function does not return a value; it performs its operations by calling another function to list panes for each session.
- **Functions called**:
    - [`cmd_list_panes_session`](#cmd_list_panes_session)


---
### cmd_list_panes_session<!-- {{#callable:cmd_list_panes_session}} -->
The function `cmd_list_panes_session` iterates over all windows in a given session and lists the panes for each window using a specified format.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `s`: A pointer to the `session` structure representing the session whose panes are to be listed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
    - `type`: An integer indicating the format type to be used when listing panes.
- **Control Flow**:
    - The function uses the `RB_FOREACH` macro to iterate over each `winlink` in the session's `windows` red-black tree.
    - For each `winlink`, it calls the [`cmd_list_panes_window`](#cmd_list_panes_window) function, passing the current window link and other parameters to list the panes in that window.
- **Output**:
    - The function does not return a value; it performs its operations by calling another function to list panes for each window in the session.
- **Functions called**:
    - [`cmd_list_panes_window`](#cmd_list_panes_window)


---
### cmd_list_panes_window<!-- {{#callable:cmd_list_panes_window}} -->
The `cmd_list_panes_window` function lists all panes in a specified window, formatting the output based on a given template and optional filter.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `s`: A pointer to the `session` structure representing the current session.
    - `wl`: A pointer to the `winlink` structure representing the window link in the session.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
    - `type`: An integer indicating the type of template to use for formatting the output.
- **Control Flow**:
    - Retrieve the command arguments using [`cmd_get_args`](cmd.c.driver.md#cmd_get_args) and store them in `args`.
    - Check if a custom format template is provided with the '-F' argument; if not, select a default template based on the `type` parameter.
    - Retrieve the filter string from the arguments using the '-f' option.
    - Initialize a counter `n` to zero to track the pane index.
    - Iterate over each pane in the window using `TAILQ_FOREACH`.
    - For each pane, create a format tree `ft` using [`format_create`](format.c.driver.md#format_create) and add default formatting values with [`format_defaults`](format.c.driver.md#format_defaults).
    - If a filter is specified, expand it using [`format_expand`](format.c.driver.md#format_expand) and evaluate it with [`format_true`](format.c.driver.md#format_true); otherwise, set `flag` to 1.
    - If `flag` is true, expand the template using [`format_expand`](format.c.driver.md#format_expand) and print the formatted line using `cmdq_print`.
    - Free the memory allocated for the expanded line and the format tree.
- **Output**:
    - The function does not return a value but prints formatted information about each pane in the specified window to the command queue item.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`args_get`](arguments.c.driver.md#args_get)
    - [`format_create`](format.c.driver.md#format_create)
    - [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client)
    - [`format_defaults`](format.c.driver.md#format_defaults)
    - [`format_expand`](format.c.driver.md#format_expand)
    - [`format_true`](format.c.driver.md#format_true)
    - [`format_free`](format.c.driver.md#format_free)


