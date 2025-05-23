# Purpose
This C source code file is part of the tmux terminal multiplexer project and implements the functionality to rotate panes within a window. The primary function, [`cmd_rotate_window_exec`](#cmd_rotate_window_exec), is responsible for executing the rotation of panes either clockwise or counterclockwise, depending on the command-line arguments provided. The code defines a command entry structure, `cmd_rotate_window_entry`, which specifies the command name, alias, arguments, usage, and the execution function. This structure is used to integrate the rotate-window command into the tmux command system, allowing users to manipulate the layout of panes within a tmux session.

The code provides a specific functionality within the broader context of tmux, focusing on the manipulation of window panes. It does not define a public API or external interface but rather contributes to the internal command execution framework of tmux. The implementation involves manipulating linked lists of window panes, adjusting their layout cells, and updating their positions and sizes to achieve the rotation effect. The code also handles the active pane selection and ensures the window's display is updated after the rotation operation. This file is a clear example of a narrowly focused utility within a larger software system, providing a specific feature to enhance user interaction with tmux sessions.
# Imports and Dependencies

---
- `sys/types.h`
- `tmux.h`


# Global Variables

---
### cmd_rotate_window_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_rotate_window_entry` is a constant structure of type `cmd_entry` that defines the command for rotating panes within a window in the tmux application. It specifies the command's name, alias, arguments, usage, target, flags, and the function to execute the command. This structure is part of the command handling mechanism in tmux, allowing users to rotate the layout of panes within a window.
- **Use**: This variable is used to register and define the behavior of the 'rotate-window' command in tmux, enabling users to rotate panes within a window.


# Functions

---
### cmd_rotate_window_exec<!-- {{#callable:cmd_rotate_window_exec}} -->
The `cmd_rotate_window_exec` function rotates the panes within a window in either direction based on the provided arguments.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve the command arguments using `cmd_get_args` and the current and target states using `cmdq_get_current` and `cmdq_get_target`.
    - Get the target window link and window from the target state.
    - Push the current zoom state of the window using `window_push_zoom`.
    - Check if the 'D' argument is present to determine the rotation direction.
    - If 'D' is present, rotate the panes by moving the last pane to the front and adjust the layout and offsets accordingly.
    - If 'D' is not present, rotate the panes by moving the first pane to the end and adjust the layout and offsets accordingly.
    - Set the active pane to the previous or next pane based on the rotation direction.
    - Update the current command find state with the new active pane.
    - Pop the zoom state of the window using `window_pop_zoom`.
    - Redraw the window using `server_redraw_window`.
- **Output**:
    - Returns `CMD_RETURN_NORMAL` to indicate successful execution of the command.
- **Functions called**:
    - [`window_pane_resize`](tmux.h.driver.md#window_pane_resize)


# Function Declarations (Public API)

---
### cmd_rotate_window_exec<!-- {{#callable_declaration:cmd_rotate_window_exec}} -->
Rotate the panes within a specified window.
- **Description**: This function is used to rotate the panes within a specified window in a tmux session. It can be called to either rotate the panes clockwise or counterclockwise, depending on the provided arguments. The function must be called with a valid command and command queue item, and it operates on the target window specified in the command. The function handles the active pane and ensures the window layout is updated accordingly. It is important to ensure that the command and command queue item are properly initialized before calling this function.
- **Inputs**:
    - `self`: A pointer to the command structure representing the current command. Must not be null and should be properly initialized with the necessary arguments.
    - `item`: A pointer to the command queue item structure representing the current command queue item. Must not be null and should be properly initialized to reflect the current and target states.
- **Output**: Returns an enum value of type cmd_retval indicating the success of the operation, typically CMD_RETURN_NORMAL.
- **See also**: [`cmd_rotate_window_exec`](#cmd_rotate_window_exec)  (Implementation)


