# Purpose
This C source code file is part of the tmux terminal multiplexer project and provides functionality for resizing panes within a tmux session. The file defines a command, `resize-pane`, which allows users to adjust the size of panes either by specific dimensions or through mouse interactions. The command can be executed with various options to increase or decrease the pane size in different directions (left, right, up, down) or to set specific width and height values. The code includes mechanisms to handle mouse events for resizing, allowing users to drag pane borders to adjust their size dynamically.

The file is structured around the [`cmd_resize_pane_exec`](#cmd_resize_pane_exec) function, which is the core execution logic for the resize-pane command. It processes command-line arguments to determine the resizing action and applies the changes to the targeted pane. The file also includes a mouse event update function, [`cmd_resize_pane_mouse_update`](#cmd_resize_pane_mouse_update), which facilitates resizing through mouse interactions. This code is part of a larger system and is intended to be integrated with other components of tmux, as indicated by the inclusion of the "tmux.h" header. The command entry structure, `cmd_resize_pane_entry`, defines the command's name, alias, arguments, usage, and execution function, making it a part of the public API for tmux commands.
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
- **Use**: This variable is used to register and define the behavior of the 'resize-pane' command within the tmux application, allowing users to adjust the size of terminal panes.


# Functions

---
### cmd_resize_pane_exec<!-- {{#callable:cmd_resize_pane_exec}} -->
The `cmd_resize_pane_exec` function handles resizing of a window pane in a terminal multiplexer based on various command arguments.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve command arguments, target pane, event, client, session, and window information.
    - Check if the 'T' flag is set; if so, adjust the pane's history and redraw it if necessary.
    - Check if the 'M' flag is set; if so, handle mouse-based resizing if the event is valid and the client session matches.
    - Check if the 'Z' flag is set; if so, toggle zoom on the window and redraw it.
    - If no specific adjustment is provided, default to an adjustment of 1; otherwise, parse the adjustment value from arguments.
    - If the 'x' flag is set, calculate the new width percentage and resize the pane horizontally.
    - If the 'y' flag is set, calculate the new height percentage and resize the pane vertically, considering pane border status.
    - Handle directional resizing based on 'L', 'R', 'U', 'D' flags to adjust the pane size in the specified direction.
    - Redraw the window after resizing operations.
- **Output**:
    - Returns `CMD_RETURN_NORMAL` on successful execution or `CMD_RETURN_ERROR` if an error occurs during argument parsing.
- **Functions called**:
    - [`cmd_resize_pane_mouse_update`](#cmd_resize_pane_mouse_update)


---
### cmd_resize_pane_mouse_update<!-- {{#callable:cmd_resize_pane_mouse_update}} -->
The function `cmd_resize_pane_mouse_update` updates the size of a pane in a window based on mouse drag events.
- **Inputs**:
    - `c`: A pointer to the client structure, representing the client session where the mouse event occurred.
    - `m`: A pointer to the mouse_event structure, containing details about the mouse event, such as coordinates and status.
- **Control Flow**:
    - Retrieve the winlink associated with the mouse event using `cmd_mouse_window`; if not found, set `mouse_drag_update` to NULL and return.
    - Calculate the current and last mouse positions, adjusting for status lines if necessary.
    - Iterate over predefined offset positions to find layout cells near the mouse event's border using `layout_search_by_border`.
    - Check for duplicate cells and store unique cells in the `cells` array.
    - If no cells are found, return without making changes.
    - For each cell, determine if resizing is needed based on the layout type and mouse movement direction (vertical or horizontal).
    - If resizing is performed, increment the `resizes` counter.
    - If any resizing occurred, trigger a redraw of the window using `server_redraw_window`.
- **Output**:
    - The function does not return a value but updates the pane size and potentially redraws the window if resizing occurs.


# Function Declarations (Public API)

---
### cmd_resize_pane_exec<!-- {{#callable_declaration:cmd_resize_pane_exec}} -->
Adjusts the size of a specified pane in a tmux session.
- **Description**: This function is used to resize a pane within a tmux session, allowing for both manual and automatic adjustments based on specified arguments. It can handle resizing through direct size specifications or by using directional adjustments. The function supports various options, such as zooming, mouse-based resizing, and history trimming. It must be called with a valid command and target pane, and it assumes that the tmux environment is properly initialized. The function handles invalid input by returning an error status.
- **Inputs**:
    - `self`: A pointer to the command structure representing the resize-pane command. Must not be null.
    - `item`: A pointer to the command queue item, which provides context such as the target pane and client. Must not be null.
- **Output**: Returns an enum cmd_retval indicating the success or failure of the operation, with CMD_RETURN_NORMAL for success and CMD_RETURN_ERROR for failure.
- **See also**: [`cmd_resize_pane_exec`](#cmd_resize_pane_exec)  (Implementation)


---
### cmd_resize_pane_mouse_update<!-- {{#callable_declaration:cmd_resize_pane_mouse_update}} -->
Updates pane size based on mouse drag events.
- **Description**: This function is used to adjust the size of a pane in response to mouse drag events. It should be called when a mouse drag event is detected on a client, and it will update the pane size accordingly. The function requires a valid client and mouse event structure, and it will handle the necessary calculations to determine the new pane size. If the mouse event does not correspond to a valid window, the function will disable further mouse drag updates. This function is typically used in environments where mouse interactions are supported for resizing panes.
- **Inputs**:
    - `c`: A pointer to a 'client' structure representing the client where the mouse event occurred. Must not be null. The function will disable mouse drag updates if the client is not associated with a valid window.
    - `m`: A pointer to a 'mouse_event' structure containing details about the mouse event, including coordinates and status lines. Must not be null. The function uses this information to calculate the new pane size.
- **Output**: None
- **See also**: [`cmd_resize_pane_mouse_update`](#cmd_resize_pane_mouse_update)  (Implementation)


