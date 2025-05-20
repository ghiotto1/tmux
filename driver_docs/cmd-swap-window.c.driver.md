# Purpose
This C source code file is part of the tmux terminal multiplexer project and implements the functionality to swap one window with another within tmux sessions. The file defines a command, "swap-window," which is registered with the tmux command system through the `cmd_swap_window_entry` structure. This structure specifies the command's name, alias, arguments, usage, and the function to execute when the command is invoked. The primary function, [`cmd_swap_window_exec`](#cmd_swap_window_exec), handles the logic for swapping windows between sessions, ensuring that the operation is valid and updating the session and window states accordingly.

The code provides a specific functionality within the broader tmux application, focusing on window management. It includes checks to prevent swapping windows between grouped sessions and updates the session display and window links to reflect the swap. The file does not define a public API or external interface but rather integrates into the tmux command execution framework. The use of structures like `cmd_entry` and functions like `cmd_get_args` and `cmdq_get_source` indicates that this code is part of a larger system designed to handle command parsing and execution within tmux.
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
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item, which contains the source and target states.
- **Control Flow**:
    - Retrieve the command arguments using [`cmd_get_args`](cmd.c.driver.md#cmd_get_args) and the source and target states using [`cmdq_get_source`](cmd-queue.c.driver.md#cmdq_get_source) and [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target).
    - Determine the source and target sessions and their respective session groups.
    - Check if the source and target sessions are different but belong to the same session group; if so, return an error as swapping is not allowed in this case.
    - Check if the source and target windows are the same; if so, return normally as no swap is needed.
    - Remove the target window link from its window's list and the source window link from its window's list.
    - Swap the windows by updating the window links to point to the opposite windows and reinsert them into the respective window lists.
    - If the '-d' argument is present, select the target window in the destination session and, if the sessions differ, select the source window in the source session.
    - Synchronize and redraw the session groups for both source and target sessions if they are different.
    - Recalculate the sizes of the windows after the swap.
- **Output**:
    - Returns `CMD_RETURN_NORMAL` on successful swap or `CMD_RETURN_ERROR` if the swap is not allowed due to session group constraints.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_source`](cmd-queue.c.driver.md#cmdq_get_source)
    - [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target)
    - [`session_group_contains`](session.c.driver.md#session_group_contains)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`session_select`](session.c.driver.md#session_select)
    - [`session_group_synchronize_from`](session.c.driver.md#session_group_synchronize_from)
    - [`server_redraw_session_group`](server-fn.c.driver.md#server_redraw_session_group)
    - [`recalculate_sizes`](resize.c.driver.md#recalculate_sizes)


