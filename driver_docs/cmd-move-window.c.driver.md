# Purpose
This C source code file is part of the tmux terminal multiplexer project, specifically implementing functionality to move or link windows between sessions. The file defines two command entries, `cmd_move_window_entry` and `cmd_link_window_entry`, which correspond to the "move-window" and "link-window" commands in tmux. These commands allow users to either move a window from one session to another or create a link to a window in another session, respectively. The commands are defined with specific options and usage patterns, indicating their integration into the broader tmux command framework.

The core functionality is implemented in the [`cmd_move_window_exec`](#cmd_move_window_exec) function, which handles the execution logic for both moving and linking windows. This function processes command-line arguments, identifies source and target sessions and windows, and performs the necessary operations to move or link the windows. It includes options for renumbering windows, handling window indices, and managing session states. The file is structured to be part of a larger codebase, likely imported and used by other components of tmux, and does not define a standalone executable. Instead, it provides specific command implementations that integrate with tmux's command execution system.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `tmux.h`


# Global Variables

---
### cmd_move_window_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_move_window_entry` is a constant structure of type `cmd_entry` that defines the command for moving a window in the tmux application. It includes fields such as the command name, alias, arguments, usage, source, flags, and the execution function. This structure is used to register and handle the 'move-window' command within the tmux command processing framework.
- **Use**: This variable is used to define and execute the 'move-window' command in tmux, allowing users to move windows between sessions or within a session.


---
### cmd_link_window_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_link_window_entry` is a constant structure of type `cmd_entry` that defines the command 'link-window' and its associated properties. It includes the command name, alias, argument specifications, usage instructions, source window finding parameters, flags, and the execution function pointer. This structure is used to encapsulate all necessary information for the 'link-window' command within the application.
- **Use**: This variable is used to define and register the 'link-window' command, allowing it to be executed with specified arguments and behavior in the application.


# Functions

---
### cmd_move_window_exec<!-- {{#callable:cmd_move_window_exec}} -->
The `cmd_move_window_exec` function moves a window from one session to another, with options to renumber windows, shuffle positions, and handle conflicts.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve command arguments using [`cmd_get_args`](cmd.c.driver.md#cmd_get_args) and the source state using [`cmdq_get_source`](cmd-queue.c.driver.md#cmdq_get_source).
    - Check if the 'r' flag is present; if so, find the target session and renumber its windows, then recalculate sizes and update the session status.
    - If the 'r' flag is not present, find the target window and session using [`cmd_find_target`](cmd-find.c.driver.md#cmd_find_target).
    - Determine the destination session and index from the target state.
    - Check for the presence of flags 'k', 'd', 's', 'b', and 'a' to set corresponding variables.
    - If 'a' or 'b' flags are present, shuffle the window links up in the destination session to determine the new index.
    - Attempt to link the window from the source to the destination session using [`server_link_window`](server-fn.c.driver.md#server_link_window), handling errors if they occur.
    - If the command entry is `cmd_move_window_entry`, unlink the window from the source session using [`server_unlink_window`](server-fn.c.driver.md#server_unlink_window).
    - Renumber windows in the source session if the 's' flag is not set and the session option `renumber-windows` is enabled.
    - Recalculate window sizes after moving the window.
- **Output**:
    - Returns `CMD_RETURN_NORMAL` on successful execution or `CMD_RETURN_ERROR` if an error occurs during the process.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_source`](cmd-queue.c.driver.md#cmdq_get_source)
    - [`args_get`](arguments.c.driver.md#args_get)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`cmd_find_target`](cmd-find.c.driver.md#cmd_find_target)
    - [`session_renumber_windows`](session.c.driver.md#session_renumber_windows)
    - [`recalculate_sizes`](resize.c.driver.md#recalculate_sizes)
    - [`server_status_session`](server-fn.c.driver.md#server_status_session)
    - [`winlink_shuffle_up`](window.c.driver.md#winlink_shuffle_up)
    - [`server_link_window`](server-fn.c.driver.md#server_link_window)
    - [`cmd_get_entry`](cmd.c.driver.md#cmd_get_entry)
    - [`server_unlink_window`](server-fn.c.driver.md#server_unlink_window)
    - [`options_get_number`](options.c.driver.md#options_get_number)


