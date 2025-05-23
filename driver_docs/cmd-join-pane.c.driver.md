# Purpose
This C source code file is part of the tmux terminal multiplexer project, providing functionality to manage and manipulate terminal panes. Specifically, it defines the implementation for two commands: `join-pane` and `move-pane`. These commands allow users to join or move a pane into another, similar to operations like split, swap, or kill. The file includes the necessary logic to handle these operations, such as determining the layout type (either top-bottom or left-right), calculating the size of the panes, and managing the pane's position within the window. The code also handles various flags that modify the behavior of the commands, such as whether to place the pane before or after the target pane, or whether to use the full size of the window.

The file defines two command entries, `cmd_join_pane_entry` and `cmd_move_pane_entry`, which specify the command names, aliases, arguments, usage, and execution function. The core functionality is implemented in the [`cmd_join_pane_exec`](#cmd_join_pane_exec) function, which performs the actual pane manipulation. This function checks for errors, such as attempting to join a pane with itself, and manages the layout and size adjustments required to integrate the source pane into the target window. The code also includes mechanisms to update the user interface and notify other components of layout changes. This file is a crucial part of the tmux command handling system, providing specific functionality for pane management within the broader context of terminal session management.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `unistd.h`
- `tmux.h`


# Global Variables

---
### cmd_join_pane_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_join_pane_entry` is a constant structure of type `cmd_entry` that defines the command entry for the 'join-pane' command in the tmux application. It includes the command's name, alias, argument specifications, usage information, source and target pane specifications, flags, and the function to execute the command.
- **Use**: This variable is used to register and define the behavior of the 'join-pane' command, allowing users to join or move a pane into another within the tmux environment.


---
### cmd_move_pane_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_move_pane_entry` is a constant structure of type `cmd_entry` that defines the command entry for the 'move-pane' command in the tmux application. It includes the command's name, alias, argument specifications, usage information, source and target pane specifications, flags, and the execution function associated with the command.
- **Use**: This variable is used to register and define the behavior of the 'move-pane' command within the tmux application, allowing users to move panes between windows.


# Functions

---
### cmd_join_pane_exec<!-- {{#callable:cmd_join_pane_exec}} -->
The `cmd_join_pane_exec` function moves a source pane into a target pane within a tmux session, adjusting layout and handling various options for pane placement and size.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve command arguments and current, target, and source states from the command queue item.
    - Extract destination session, window link, window, and pane from the target state.
    - Unzoom the destination and source windows if they are zoomed.
    - Check if the source and target panes are the same and return an error if they are.
    - Determine the layout type based on the presence of the 'h' flag (horizontal or vertical split).
    - Calculate the size for the new pane based on the 'l' or 'p' flags and handle any errors in size calculation.
    - Set flags for pane placement based on the 'b' and 'f' flags.
    - Attempt to split the target pane and handle errors if the pane is too small.
    - Close the source pane and remove it from its current window.
    - Reassign the source pane to the destination window, updating its options and flags.
    - Insert the source pane into the destination window's pane list before or after the target pane based on flags.
    - Assign the layout cell to the source pane and update its color palette.
    - Recalculate window sizes and redraw the source and destination windows.
    - If the 'd' flag is not set, set the source pane as active, select the session, and redraw the session; otherwise, update the session status.
    - Check if the source window has no more panes and kill it if empty, otherwise notify layout change.
    - Notify layout change for the destination window.
- **Output**:
    - Returns `CMD_RETURN_NORMAL` on successful execution or `CMD_RETURN_ERROR` if an error occurs during execution.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_current`](cmd-queue.c.driver.md#cmdq_get_current)
    - [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target)
    - [`cmdq_get_source`](cmd-queue.c.driver.md#cmdq_get_source)
    - [`server_unzoom_window`](server-fn.c.driver.md#server_unzoom_window)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`args_percentage_and_expand`](arguments.c.driver.md#args_percentage_and_expand)
    - [`args_strtonum_and_expand`](arguments.c.driver.md#args_strtonum_and_expand)
    - [`layout_split_pane`](layout.c.driver.md#layout_split_pane)
    - [`layout_close_pane`](layout.c.driver.md#layout_close_pane)
    - [`server_client_remove_pane`](server-client.c.driver.md#server_client_remove_pane)
    - [`window_lost_pane`](window.c.driver.md#window_lost_pane)
    - [`options_set_parent`](options.c.driver.md#options_set_parent)
    - [`layout_assign_pane`](layout.c.driver.md#layout_assign_pane)
    - [`colour_palette_from_option`](colour.c.driver.md#colour_palette_from_option)
    - [`recalculate_sizes`](resize.c.driver.md#recalculate_sizes)
    - [`server_redraw_window`](server-fn.c.driver.md#server_redraw_window)
    - [`window_set_active_pane`](window.c.driver.md#window_set_active_pane)
    - [`session_select`](session.c.driver.md#session_select)
    - [`cmd_find_from_session`](cmd-find.c.driver.md#cmd_find_from_session)
    - [`server_redraw_session`](server-fn.c.driver.md#server_redraw_session)
    - [`server_status_session`](server-fn.c.driver.md#server_status_session)
    - [`window_count_panes`](window.c.driver.md#window_count_panes)
    - [`server_kill_window`](server-fn.c.driver.md#server_kill_window)
    - [`notify_window`](notify.c.driver.md#notify_window)


