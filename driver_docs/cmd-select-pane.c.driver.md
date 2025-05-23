# Purpose
This C source code file is part of the tmux terminal multiplexer project, specifically implementing functionality related to selecting and managing panes within a tmux session. The file defines two command entries, `select-pane` and `last-pane`, which allow users to switch between different panes in a tmux window. The `select-pane` command provides various options for navigating between panes, such as moving left, right, up, or down, and for setting or clearing pane input modes. The `last-pane` command allows users to switch back to the previously active pane. Both commands are executed through the [`cmd_select_pane_exec`](#cmd_select_pane_exec) function, which handles the logic for pane selection and manipulation based on the provided command-line arguments.

The file includes several key technical components, such as the [`cmd_select_pane_exec`](#cmd_select_pane_exec) function, which is responsible for executing the pane selection logic, and the [`cmd_select_pane_redraw`](#cmd_select_pane_redraw) function, which manages the redrawing of the window and pane borders to reflect changes in pane selection. The code interacts with various tmux structures, such as `window`, `session`, and `window_pane`, to perform operations like setting active panes, marking panes, and updating pane styles. This file is a crucial part of the tmux command execution framework, providing specific functionality for pane management within the broader context of the tmux application.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### cmd_select_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_select_pane_entry` is a constant structure of type `cmd_entry` that defines the command entry for the 'select-pane' command in the tmux application. It includes the command's name, alias, argument specifications, usage information, target specifications, flags, and the function to execute the command.
- **Use**: This variable is used to register and define the behavior of the 'select-pane' command, allowing users to select and manipulate panes within a tmux session.


---
### cmd_last_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_last_pane_entry` is a constant structure of type `cmd_entry` that defines the command entry for the 'last-pane' command in the tmux application. It includes the command name, alias, argument specifications, usage information, target specifications, flags, and the execution function associated with the command.
- **Use**: This variable is used to define and register the 'last-pane' command, which allows users to switch to the last active pane in a tmux session.


# Functions

---
### cmd_select_pane_redraw<!-- {{#callable:cmd_select_pane_redraw}} -->
The `cmd_select_pane_redraw` function updates the display of clients associated with a given window, redrawing the entire window if it is larger than the client's display area, or updating borders and status otherwise.
- **Inputs**:
    - `w`: A pointer to a `struct window` representing the window to be redrawn.
- **Control Flow**:
    - Iterate over each client in the global `clients` list using `TAILQ_FOREACH`.
    - For each client, check if the client is not associated with a session or if it is in control mode, and skip such clients.
    - If the client's current window matches the given window `w` and the client's terminal is smaller than the window, call `server_redraw_client` to redraw the entire client.
    - Otherwise, if the client's current window matches `w`, set the `CLIENT_REDRAWBORDERS` flag to redraw borders.
    - Additionally, if the session contains the window `w`, set the `CLIENT_REDRAWSTATUS` flag to update the status.
- **Output**:
    - The function does not return a value; it modifies the state of clients to trigger appropriate redraw actions.


---
### cmd_select_pane_exec<!-- {{#callable:cmd_select_pane_exec}} -->
The `cmd_select_pane_exec` function handles the logic for selecting and managing panes within a window in a terminal multiplexer, based on various command arguments.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve command arguments and entry details using `cmd_get_args` and `cmd_get_entry` respectively.
    - Determine the current and target states using `cmdq_get_current` and `cmdq_get_target`.
    - Identify the client, window link, window, session, and pane details from the target state.
    - Check if the command is 'last-pane' or has the 'l' argument to handle last pane selection logic.
    - If 'm' or 'M' arguments are present, manage pane marking and unmarking logic.
    - Handle style setting with the 'P' argument and print current style with 'g'.
    - Navigate panes using 'L', 'R', 'U', 'D' arguments to move left, right, up, or down respectively.
    - Enable or disable pane input with 'e' or 'd' arguments.
    - Set pane title with 'T' argument and update the display if the title changes.
    - If the pane is not already active, switch to the target pane and update the client or window state accordingly.
    - Insert a hook for 'after-select-pane' and redraw the window if necessary.
    - Return `CMD_RETURN_NORMAL` or `CMD_RETURN_ERROR` based on the execution outcome.
- **Output**:
    - The function returns an `enum cmd_retval` which indicates the success or failure of the command execution, either `CMD_RETURN_NORMAL` or `CMD_RETURN_ERROR`.
- **Functions called**:
    - [`cmd_select_pane_redraw`](#cmd_select_pane_redraw)


# Function Declarations (Public API)

---
### cmd_select_pane_exec<!-- {{#callable_declaration:cmd_select_pane_exec}} -->
Selects and manipulates a pane in a tmux session.
- **Description**: This function is used to select a specific pane within a tmux session and perform various operations on it, such as changing its input state, marking it, or adjusting its style. It can also navigate between panes in different directions or set a pane's title. The function must be called with a valid command and target pane specified. It handles various flags to determine the specific action to perform, such as enabling or disabling input, marking panes, or changing styles. The function returns an error if the specified operation cannot be completed, such as when a last pane is not found.
- **Inputs**:
    - `self`: A pointer to the command structure representing the current command. Must not be null.
    - `item`: A pointer to the command queue item, representing the current command execution context. Must not be null.
- **Output**: Returns an enum cmd_retval indicating success (CMD_RETURN_NORMAL) or failure (CMD_RETURN_ERROR) of the operation.
- **See also**: [`cmd_select_pane_exec`](#cmd_select_pane_exec)  (Implementation)


