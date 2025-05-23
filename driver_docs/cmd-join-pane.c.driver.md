# Purpose
This C source code file is part of the tmux terminal multiplexer project, specifically implementing functionality for managing panes within tmux sessions. The file defines two command entries, `join-pane` and `move-pane`, which allow users to join or move a pane into another pane, similar to operations like split, swap, or kill. The primary function, [`cmd_join_pane_exec`](#cmd_join_pane_exec), executes the logic for these commands, handling the movement and resizing of panes within a tmux window. It manages the layout of panes, ensuring that the source and target panes are different, and adjusts the layout based on user-specified options such as size and orientation.

The code is structured to provide a specific set of functionalities related to pane management, with a focus on executing commands that modify the layout of panes in a tmux session. It includes detailed handling of command arguments, error checking, and layout adjustments, ensuring robust operation within the tmux environment. The file is not a standalone executable but rather a component of the larger tmux application, intended to be integrated with other parts of the tmux codebase. It defines internal command execution logic rather than public APIs, focusing on the internal mechanics of pane manipulation within tmux.
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
- **Use**: This variable is used to register and define the behavior of the 'move-pane' command within the tmux command framework.


# Functions

---
### cmd_join_pane_exec<!-- {{#callable:cmd_join_pane_exec}} -->
The `cmd_join_pane_exec` function moves a source pane into a target pane's window, adjusting layout and handling various options for pane placement and size.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve command arguments and current, target, and source states from the command queue item.
    - Extract destination session, window link, window pane, and window from the target state.
    - Unzoom the destination and source windows if they are zoomed.
    - Check if the source and target panes are the same and return an error if they are.
    - Determine the layout type based on the presence of the 'h' argument (horizontal or vertical split).
    - Calculate the size for the new pane based on 'l' or 'p' arguments, handling errors if size calculation fails.
    - Set flags for pane placement based on 'b' (before) and 'f' (full size) arguments.
    - Attempt to split the target pane using the calculated size and flags, returning an error if the split fails.
    - Close the source pane and remove it from its current window.
    - Reassign the source pane to the destination window, updating its options and flags.
    - Insert the source pane into the destination window's pane list before or after the target pane based on flags.
    - Recalculate window sizes and redraw the source and destination windows.
    - If the 'd' argument is not present, set the source pane as active and update the session and redraw it.
    - Check if the source window has no more panes and kill it if empty, otherwise notify layout change.
    - Notify layout change for the destination window.
    - Return normal command execution status.
- **Output**:
    - Returns `CMD_RETURN_NORMAL` on successful execution or `CMD_RETURN_ERROR` if an error occurs during execution.


# Function Declarations (Public API)

---
### cmd_join_pane_exec<!-- {{#callable_declaration:cmd_join_pane_exec}} -->
Joins or moves a source pane into a target pane.
- **Description**: This function is used to join or move a source pane into a target pane within a tmux session. It is typically called when a user wants to reorganize panes by moving one pane into another, either splitting the target pane or replacing it. The function requires that the source and target panes be different, and it supports various options to control the layout and behavior of the operation, such as specifying the layout direction, size, and whether to keep the source pane active. It must be called with valid source and target pane identifiers, and it handles errors such as invalid sizes or identical source and target panes by returning an error code.
- **Inputs**:
    - `self`: A pointer to the cmd structure representing the command being executed. Must not be null.
    - `item`: A pointer to the cmdq_item structure representing the command queue item. Must not be null.
- **Output**: Returns an enum cmd_retval indicating success (CMD_RETURN_NORMAL) or failure (CMD_RETURN_ERROR).
- **See also**: [`cmd_join_pane_exec`](#cmd_join_pane_exec)  (Implementation)


