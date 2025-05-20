# Purpose
This C source code file is part of the tmux terminal multiplexer project, specifically implementing functionality for selecting and managing panes within a tmux session. The file defines two command entries, `select-pane` and `last-pane`, which allow users to switch between different panes in a tmux window. The `select-pane` command provides various options for navigating and modifying panes, such as moving focus to adjacent panes, enabling or disabling input, and setting pane styles. The `last-pane` command allows users to switch back to the most recently active pane. The code includes functions to handle the execution of these commands, manage pane states, and update the display to reflect changes.

The file is structured around the [`cmd_select_pane_exec`](#cmd_select_pane_exec) function, which is responsible for executing the logic associated with the pane selection commands. It interacts with various tmux components, such as sessions, windows, and panes, to perform actions like changing the active pane, marking panes, and updating pane styles. The code also includes mechanisms to redraw the window and update client displays when pane states change. This file is a critical part of the tmux command handling system, providing users with the ability to efficiently manage and navigate between multiple panes within their terminal sessions.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### cmd_select_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_select_pane_entry` is a constant structure of type `cmd_entry` that defines the command entry for the 'select-pane' command in the tmux application. It includes the command's name, alias, argument specifications, usage information, target specifications, flags, and the function to execute the command.
- **Use**: This variable is used to register and define the behavior of the 'select-pane' command within the tmux command processing system.


---
### cmd_last_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_last_pane_entry` is a constant structure of type `cmd_entry` that defines the command entry for the 'last-pane' command in the tmux application. It includes the command name, alias, argument specifications, usage information, target specifications, flags, and the execution function associated with the command.
- **Use**: This variable is used to define and register the 'last-pane' command, allowing users to switch to the last active pane in a tmux session.


# Functions

---
### cmd_select_pane_redraw<!-- {{#callable:cmd_select_pane_redraw}} -->
The function `cmd_select_pane_redraw` updates the display of clients associated with a given window, redrawing the entire window if it is larger than the client's display area, or updating borders and status otherwise.
- **Inputs**:
    - `w`: A pointer to a `struct window` representing the window to be redrawn.
- **Control Flow**:
    - Iterate over each client in the global `clients` list using `TAILQ_FOREACH`.
    - For each client, check if the client is not associated with a session or if the client is in control mode, and skip such clients.
    - If the client's current window matches the given window `w` and the client's terminal is smaller than the window, call [`server_redraw_client`](server-fn.c.driver.md#server_redraw_client) to redraw the entire client.
    - Otherwise, if the client's current window matches `w`, set the `CLIENT_REDRAWBORDERS` flag to redraw borders.
    - Additionally, if the session contains the window `w`, set the `CLIENT_REDRAWSTATUS` flag to update the status.
- **Output**:
    - The function does not return a value; it modifies the state of clients to trigger appropriate redraw actions.
- **Functions called**:
    - [`tty_window_bigger`](tty.c.driver.md#tty_window_bigger)
    - [`server_redraw_client`](server-fn.c.driver.md#server_redraw_client)
    - [`session_has`](session.c.driver.md#session_has)


---
### cmd_select_pane_exec<!-- {{#callable:cmd_select_pane_exec}} -->
The `cmd_select_pane_exec` function handles the logic for selecting and managing panes within a window in a terminal multiplexer, based on various command arguments.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item being processed.
- **Control Flow**:
    - Retrieve command arguments and entry details using [`cmd_get_args`](cmd.c.driver.md#cmd_get_args) and [`cmd_get_entry`](cmd.c.driver.md#cmd_get_entry) functions.
    - Determine the current and target states using [`cmdq_get_current`](cmd-queue.c.driver.md#cmdq_get_current) and [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target).
    - Identify the client, window link, window, session, and pane from the target state.
    - Check if the command is 'last-pane' or has the 'l' argument to handle last pane selection logic.
    - If 'm' or 'M' arguments are present, manage pane marking and unmarking logic.
    - Handle style setting with the 'P' argument and print current style with 'g'.
    - Navigate panes using 'L', 'R', 'U', 'D' arguments to find left, right, up, or down panes respectively.
    - Enable or disable pane input with 'e' or 'd' arguments.
    - Set pane title with 'T' argument and update the pane's title if changed.
    - Check if the pane is already active and switch to it if not, handling zoom and redraw operations.
    - Insert a hook for 'after-select-pane' and redraw the window if necessary.
- **Output**:
    - Returns `CMD_RETURN_NORMAL` on successful execution or `CMD_RETURN_ERROR` if an error occurs, such as when no last pane is found or a bad style is specified.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmd_get_entry`](cmd.c.driver.md#cmd_get_entry)
    - [`cmdq_get_current`](cmd-queue.c.driver.md#cmdq_get_current)
    - [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target)
    - [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`window_count_panes`](window.c.driver.md#window_count_panes)
    - [`server_redraw_window_borders`](server-fn.c.driver.md#server_redraw_window_borders)
    - [`server_status_window`](server-fn.c.driver.md#server_status_window)
    - [`window_push_zoom`](window.c.driver.md#window_push_zoom)
    - [`server_redraw_window`](server-fn.c.driver.md#server_redraw_window)
    - [`window_redraw_active_switch`](window.c.driver.md#window_redraw_active_switch)
    - [`window_set_active_pane`](window.c.driver.md#window_set_active_pane)
    - [`cmd_find_from_winlink`](cmd-find.c.driver.md#cmd_find_from_winlink)
    - [`cmd_select_pane_redraw`](#cmd_select_pane_redraw)
    - [`window_pop_zoom`](window.c.driver.md#window_pop_zoom)
    - [`window_pane_visible`](window.c.driver.md#window_pane_visible)
    - [`server_check_marked`](server.c.driver.md#server_check_marked)
    - [`server_is_marked`](server.c.driver.md#server_is_marked)
    - [`server_clear_marked`](server.c.driver.md#server_clear_marked)
    - [`server_set_marked`](server.c.driver.md#server_set_marked)
    - [`args_get`](arguments.c.driver.md#args_get)
    - [`options_get_string`](options.c.driver.md#options_get_string)
    - [`window_pane_find_left`](window.c.driver.md#window_pane_find_left)
    - [`window_pane_find_right`](window.c.driver.md#window_pane_find_right)
    - [`window_pane_find_up`](window.c.driver.md#window_pane_find_up)
    - [`window_pane_find_down`](window.c.driver.md#window_pane_find_down)
    - [`format_single_from_target`](format.c.driver.md#format_single_from_target)
    - [`screen_set_title`](screen.c.driver.md#screen_set_title)
    - [`notify_pane`](notify.c.driver.md#notify_pane)
    - [`server_client_get_pane`](server-client.c.driver.md#server_client_get_pane)
    - [`server_client_set_pane`](server-client.c.driver.md#server_client_set_pane)
    - [`cmd_find_from_winlink_pane`](cmd-find.c.driver.md#cmd_find_from_winlink_pane)


