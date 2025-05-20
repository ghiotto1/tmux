# Purpose
This C source code file is part of the tmux terminal multiplexer project and provides functionality for resizing panes within a tmux session. The file defines a command, `resize-pane`, which allows users to adjust the size of panes either by specifying dimensions directly or by using directional adjustments. The command can be executed with various options, such as `-D`, `-L`, `-M`, `-R`, `-T`, `-U`, `-x`, `-y`, and `-Z`, which control the resizing behavior, including mouse-based resizing and zoom toggling.

The core functionality is implemented in the [`cmd_resize_pane_exec`](#cmd_resize_pane_exec) function, which processes command arguments and performs the necessary resizing operations on the target pane. The file also includes a mouse event handler, [`cmd_resize_pane_mouse_update`](#cmd_resize_pane_mouse_update), which updates pane sizes based on mouse drag events. This code is designed to be integrated into the larger tmux application, providing a specific and narrow functionality focused on pane resizing. It does not define a public API but rather contributes to the internal command execution framework of tmux.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### cmd_resize_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_resize_pane_entry` is a constant structure of type `cmd_entry` that defines the command for resizing a pane in the tmux terminal multiplexer. It includes the command name, alias, argument specifications, usage instructions, target specifications, flags, and the function to execute the command.
- **Use**: This variable is used to register and define the behavior of the 'resize-pane' command within the tmux application.


# Functions

---
### cmd_resize_pane_exec<!-- {{#callable:cmd_resize_pane_exec}} -->
The `cmd_resize_pane_exec` function handles resizing of a pane in a terminal multiplexer based on various command-line arguments and conditions.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve command arguments, target pane, event, and other related structures from the input parameters.
    - Check if the 'T' flag is present; if so, adjust the pane's history and redraw the pane if necessary.
    - Check if the 'M' flag is present; if so, handle mouse-based resizing if the event is valid and the client session matches.
    - Check if the 'Z' flag is present; if so, toggle the zoom state of the window and redraw it.
    - If no specific adjustment is provided, default to an adjustment of 1; otherwise, parse the adjustment value from the arguments.
    - If the 'x' flag is present, calculate the new width as a percentage and resize the pane horizontally.
    - If the 'y' flag is present, calculate the new height as a percentage and resize the pane vertically, considering pane border status.
    - Handle directional resizing based on 'L', 'R', 'U', and 'D' flags to adjust the pane size in the specified direction.
    - Redraw the window after resizing operations.
- **Output**:
    - The function returns an enum value `CMD_RETURN_NORMAL` on successful execution or `CMD_RETURN_ERROR` if an error occurs during argument parsing.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target)
    - [`cmdq_get_event`](cmd-queue.c.driver.md#cmdq_get_event)
    - [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`grid_remove_history`](grid.c.driver.md#grid_remove_history)
    - [`cmd_mouse_window`](cmd.c.driver.md#cmd_mouse_window)
    - [`cmd_resize_pane_mouse_update`](#cmd_resize_pane_mouse_update)
    - [`window_unzoom`](window.c.driver.md#window_unzoom)
    - [`window_zoom`](window.c.driver.md#window_zoom)
    - [`server_redraw_window`](server-fn.c.driver.md#server_redraw_window)
    - [`server_unzoom_window`](server-fn.c.driver.md#server_unzoom_window)
    - [`args_count`](arguments.c.driver.md#args_count)
    - [`strtonum`](compat/strtonum.c.driver.md#strtonum)
    - [`args_string`](arguments.c.driver.md#args_string)
    - [`args_percentage`](arguments.c.driver.md#args_percentage)
    - [`layout_resize_pane_to`](layout.c.driver.md#layout_resize_pane_to)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`layout_resize_pane`](layout.c.driver.md#layout_resize_pane)


---
### cmd_resize_pane_mouse_update<!-- {{#callable:cmd_resize_pane_mouse_update}} -->
The function `cmd_resize_pane_mouse_update` updates the size of a pane in a window based on mouse drag events.
- **Inputs**:
    - `c`: A pointer to the client structure, representing the client session where the mouse event occurred.
    - `m`: A pointer to the mouse_event structure, containing details about the mouse event, such as coordinates and status.
- **Control Flow**:
    - Retrieve the winlink associated with the mouse event using [`cmd_mouse_window`](cmd.c.driver.md#cmd_mouse_window); if not found, set `mouse_drag_update` to NULL and return.
    - Calculate the current and last mouse positions, adjusting for status lines if necessary.
    - Iterate over predefined offset positions to find layout cells near the mouse event's border using [`layout_search_by_border`](layout.c.driver.md#layout_search_by_border).
    - Check for duplicate cells and store unique cells in the `cells` array.
    - If no cells are found, return without making changes.
    - For each cell, determine if resizing is needed based on the layout type (top-bottom or left-right) and the difference in mouse positions.
    - If any resizing occurs, call [`server_redraw_window`](server-fn.c.driver.md#server_redraw_window) to update the window display.
- **Output**:
    - The function does not return a value but updates the pane size and potentially redraws the window if resizing occurs.
- **Functions called**:
    - [`cmd_mouse_window`](cmd.c.driver.md#cmd_mouse_window)
    - [`layout_search_by_border`](layout.c.driver.md#layout_search_by_border)
    - [`layout_resize_layout`](layout.c.driver.md#layout_resize_layout)
    - [`server_redraw_window`](server-fn.c.driver.md#server_redraw_window)


