# Purpose
This C source code file is part of the tmux terminal multiplexer project and implements the functionality to rotate panes within a window. The primary function, [`cmd_rotate_window_exec`](#cmd_rotate_window_exec), is responsible for executing the rotation of panes either clockwise or counterclockwise, depending on the command-line arguments provided. The code defines a command entry structure, `cmd_rotate_window_entry`, which specifies the command name, alias, arguments, usage, and the execution function. This structure is used to integrate the rotate-window command into the tmux command system, allowing users to manipulate the layout of panes within a window.

The code provides a specific functionality within the broader context of tmux, focusing on the manipulation of window panes. It does not define a public API or external interface but rather contributes to the internal command execution framework of tmux. The implementation involves manipulating linked lists of window panes, adjusting their layout cells, and updating their positions and sizes. The code ensures that the active pane is updated and the window is redrawn after the rotation operation, maintaining the integrity and visual consistency of the tmux session.
# Imports and Dependencies

---
- `sys/types.h`
- `tmux.h`


# Global Variables

---
### cmd_rotate_window_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_rotate_window_entry` is a constant structure of type `cmd_entry` that defines the command for rotating panes within a window in the tmux application. It specifies the command's name, alias, arguments, usage, target, flags, and the function to execute the command. This structure is part of the command handling mechanism in tmux, allowing users to rotate window panes either clockwise or counterclockwise.
- **Use**: This variable is used to register and define the behavior of the 'rotate-window' command in tmux, enabling users to manipulate the layout of panes within a window.


# Functions

---
### cmd_rotate_window_exec<!-- {{#callable:cmd_rotate_window_exec}} -->
The `cmd_rotate_window_exec` function rotates the panes within a window in either direction based on the provided arguments.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve the command arguments using [`cmd_get_args`](cmd.c.driver.md#cmd_get_args) and the current and target states using [`cmdq_get_current`](cmd-queue.c.driver.md#cmdq_get_current) and [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target).
    - Identify the target window and its panes from the `winlink` structure.
    - Check if the 'Z' argument is present to handle zoom state with [`window_push_zoom`](window.c.driver.md#window_push_zoom).
    - Determine the rotation direction based on the presence of the 'D' argument.
    - If 'D' is present, rotate the panes by moving the last pane to the front and adjust layout and offsets accordingly.
    - If 'D' is not present, rotate the panes by moving the first pane to the end and adjust layout and offsets accordingly.
    - Set the active pane to the new position after rotation using [`window_set_active_pane`](window.c.driver.md#window_set_active_pane).
    - Update the command find state with the new active pane using [`cmd_find_from_winlink_pane`](cmd-find.c.driver.md#cmd_find_from_winlink_pane).
    - Restore the zoom state with [`window_pop_zoom`](window.c.driver.md#window_pop_zoom) and redraw the window using [`server_redraw_window`](server-fn.c.driver.md#server_redraw_window).
- **Output**:
    - The function returns `CMD_RETURN_NORMAL` to indicate successful execution.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_current`](cmd-queue.c.driver.md#cmdq_get_current)
    - [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target)
    - [`window_push_zoom`](window.c.driver.md#window_push_zoom)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`window_pane_resize`](tmux.h.driver.md#window_pane_resize)
    - [`window_set_active_pane`](window.c.driver.md#window_set_active_pane)
    - [`cmd_find_from_winlink_pane`](cmd-find.c.driver.md#cmd_find_from_winlink_pane)
    - [`window_pop_zoom`](window.c.driver.md#window_pop_zoom)
    - [`server_redraw_window`](server-fn.c.driver.md#server_redraw_window)


