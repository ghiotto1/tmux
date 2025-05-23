# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file defines functionality for moving and linking windows within tmux sessions. It includes two command entries, `cmd_move_window_entry` and `cmd_link_window_entry`, which correspond to the "move-window" and "link-window" commands, respectively. These commands allow users to move a window from one session to another or link a window to multiple sessions. The file provides a specific implementation of these commands, focusing on the manipulation of window positions and session management within tmux.

The core functionality is encapsulated in the [`cmd_move_window_exec`](#cmd_move_window_exec) function, which handles the execution logic for both moving and linking windows. This function processes command-line arguments, identifies source and target sessions, and performs operations such as renumbering windows, shuffling window positions, and linking or unlinking windows between sessions. The code is structured to handle various command options, such as renumbering windows (`-r`), keeping or discarding windows (`-k` and `-d`), and specifying window positions (`-a`, `-b`, `-s`). The file is designed to be part of the tmux command execution framework, providing specific command implementations that integrate with the broader tmux system.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `tmux.h`


# Global Variables

---
### cmd_move_window_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_move_window_entry` is a constant structure of type `cmd_entry` that defines the command for moving a window within the tmux application. It includes the command name, alias, argument specifications, usage information, source window finding criteria, flags, and the execution function pointer. This structure is used to register and handle the 'move-window' command in tmux.
- **Use**: This variable is used to define and execute the 'move-window' command in the tmux application, allowing users to move windows between sessions.


---
### cmd_link_window_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_link_window_entry` is a constant structure of type `cmd_entry` that defines the command 'link-window' and its associated properties. It includes the command name, alias, argument specifications, usage instructions, source window finding parameters, flags, and the execution function pointer. This structure is used to define the behavior and characteristics of the 'link-window' command within the application.
- **Use**: This variable is used to register and define the 'link-window' command, specifying its behavior and how it should be executed in the application.


# Functions

---
### cmd_move_window_exec<!-- {{#callable:cmd_move_window_exec}} -->
The `cmd_move_window_exec` function moves a window from one session to another in the tmux environment, handling various options for window placement and session management.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve command arguments using `cmd_get_args` and the source state using `cmdq_get_source`.
    - Check if the 'r' flag is present; if so, find the target session and renumber its windows, then recalculate sizes and update the session status.
    - If the 'r' flag is not present, find the target window and index using `cmd_find_target`.
    - Determine the destination session and index from the target state.
    - Check for the presence of 'k', 'd', 's', 'a', and 'b' flags to set corresponding variables.
    - If 'a' or 'b' flags are present, shuffle the window links up in the destination session to determine the new index.
    - Attempt to link the window from the source to the destination session using `server_link_window`; handle errors if linking fails.
    - If the command entry is `cmd_move_window_entry`, unlink the window from the source session using `server_unlink_window`.
    - Renumber windows in the source session if the 's' flag is not set and the session option `renumber-windows` is enabled.
    - Recalculate window sizes after moving the window.
    - Return `CMD_RETURN_NORMAL` if successful, or `CMD_RETURN_ERROR` if any operation fails.
- **Output**:
    - Returns an `enum cmd_retval` indicating the success (`CMD_RETURN_NORMAL`) or failure (`CMD_RETURN_ERROR`) of the window move operation.


# Function Declarations (Public API)

---
### cmd_move_window_exec<!-- {{#callable_declaration:cmd_move_window_exec}} -->
Move a window to a different session or position.
- **Description**: This function is used to move a window from one session to another or to a different position within the same session. It can also renumber windows if specified. The function must be called with valid source and target window identifiers. It handles various flags to control behavior, such as whether to keep the window in the source session, whether to renumber windows, and where to place the window in the target session. The function returns an error if the target cannot be found or if the move operation fails.
- **Inputs**:
    - `self`: A pointer to the command structure representing the move-window command. Must not be null.
    - `item`: A pointer to the command queue item, which provides context for the command execution. Must not be null.
- **Output**: Returns an enum cmd_retval indicating success (CMD_RETURN_NORMAL) or failure (CMD_RETURN_ERROR).
- **See also**: [`cmd_move_window_exec`](#cmd_move_window_exec)  (Implementation)


