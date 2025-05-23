# Purpose
This C source code file is part of the tmux terminal multiplexer project and implements the functionality to swap one window with another within tmux sessions. The file defines a command, "swap-window," which is registered with the tmux command system through the `cmd_swap_window_entry` structure. This structure specifies the command's name, alias, arguments, usage, and the function to execute the command, [`cmd_swap_window_exec`](#cmd_swap_window_exec). The command allows users to swap the positions of two windows, either within the same session or across different sessions, provided the sessions are not grouped together.

The core functionality is encapsulated in the [`cmd_swap_window_exec`](#cmd_swap_window_exec) function, which handles the logic for swapping windows. It checks if the source and target windows belong to the same session group, in which case swapping is not allowed. If the windows are different, it proceeds to swap them by manipulating the window links within the session's data structures. The function also handles optional arguments, such as the `-d` flag, which determines whether the sessions should be automatically selected after the swap. The code ensures that the session states are synchronized and the user interface is updated to reflect the changes. This file is a specific component of the tmux command system, providing a narrow but essential functionality for window management within the application.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `tmux.h`


# Global Variables

---
### cmd_swap_window_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_swap_window_entry` is a constant structure of type `cmd_entry` that defines the command for swapping one window with another in the tmux application. It includes fields such as the command name, alias, arguments, usage, source and target specifications, flags, and the execution function pointer. This structure is used to register and handle the 'swap-window' command within the tmux command framework.
- **Use**: This variable is used to define and execute the 'swap-window' command in tmux, allowing users to swap the positions of two windows.


# Functions

---
### cmd_swap_window_exec<!-- {{#callable:cmd_swap_window_exec}} -->
The `cmd_swap_window_exec` function swaps two windows between source and target sessions, handling session group constraints and updating session states accordingly.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item, which contains context for the command execution.
- **Control Flow**:
    - Retrieve arguments and source/target session states from the command and command queue item.
    - Check if the source and target sessions are in the same session group; if so, return an error as swapping is not allowed.
    - If the source and target windows are the same, return normally without making changes.
    - Remove the target window link from its current window and the source window link from its current window.
    - Swap the windows by assigning the source window to the target window link and vice versa, then reinsert the links into their new windows.
    - If the '-d' argument is present, select the target window in the destination session and the source window in the source session if they are different.
    - Synchronize and redraw the session groups for both source and target sessions if they are different.
    - Recalculate window sizes to reflect the changes.
    - Return a normal command return value indicating successful execution.
- **Output**:
    - The function returns an `enum cmd_retval` indicating the result of the command execution, either `CMD_RETURN_ERROR` if an error occurs or `CMD_RETURN_NORMAL` if the swap is successful.


# Function Declarations (Public API)

---
### cmd_swap_window_exec<!-- {{#callable_declaration:cmd_swap_window_exec}} -->
Swap one window with another in tmux.
- **Description**: This function is used to swap two windows within tmux, identified by source and target parameters. It is typically called when a user wants to exchange the positions of two windows in a session. The function checks if the source and target windows belong to the same session group, in which case the swap is not allowed and an error is returned. If the windows are different, they are swapped, and the display is updated accordingly. The '-d' flag can be used to prevent the window from being selected after the swap. This function should be used when the user needs to rearrange windows within or across sessions, ensuring that the sessions are not grouped together.
- **Inputs**:
    - `self`: A pointer to the cmd structure representing the command being executed. Must not be null.
    - `item`: A pointer to the cmdq_item structure representing the command queue item. Must not be null.
- **Output**: Returns CMD_RETURN_NORMAL on success or CMD_RETURN_ERROR if the swap cannot be performed due to session grouping constraints.
- **See also**: [`cmd_swap_window_exec`](#cmd_swap_window_exec)  (Implementation)


