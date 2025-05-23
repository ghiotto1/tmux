# Purpose
This C source code file is part of the tmux terminal multiplexer project and implements the functionality for the "refresh-client" command. The primary purpose of this file is to handle the refreshing of client states within tmux, which involves updating the display and managing client-specific settings. The code defines a command entry structure, `cmd_refresh_client_entry`, which specifies the command's name, alias, arguments, usage, flags, and the function to execute when the command is invoked. This structure is crucial for integrating the command into the tmux command system.

The file contains several static functions that perform specific tasks related to client management. These include updating client subscriptions, controlling client size, managing clipboard interactions, and adjusting client offsets. The [`cmd_refresh_client_exec`](#cmd_refresh_client_exec) function is the main execution function for the command, handling various command-line arguments to perform actions such as adjusting the client view, updating the clipboard, and setting client flags. The code is designed to be integrated into the broader tmux system, providing a focused set of functionalities for client refresh operations, and it does not define public APIs or external interfaces beyond its integration with the tmux command framework.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### cmd_refresh_client_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_refresh_client_entry` is a constant structure of type `cmd_entry` that defines the command 'refresh-client' and its alias 'refresh'. It specifies the arguments, usage, flags, and the execution function associated with this command.
- **Use**: This variable is used to register and define the behavior of the 'refresh-client' command within the application, allowing it to be executed with specific options and flags.


# Functions

---
### cmd_refresh_client_update_subscription<!-- {{#callable:cmd_refresh_client_update_subscription}} -->
The function `cmd_refresh_client_update_subscription` updates a client's subscription based on a given string value, parsing it to determine the subscription type and associated identifiers.
- **Inputs**:
    - `tc`: A pointer to a `client` structure representing the client whose subscription is to be updated.
    - `value`: A constant character pointer representing the subscription string, which includes the subscription name and type information separated by colons.
- **Control Flow**:
    - Duplicate the input string `value` into `copy` and `name` using `xstrdup`.
    - Search for the first colon in `copy` to separate the subscription name from the rest of the string; if not found, remove the subscription and exit.
    - Set `what` to the substring after the first colon and search for the next colon to separate the subscription type from additional data; if not found, exit.
    - Determine the subscription type (`subtype`) and optional identifier (`subid`) based on the `what` string using a series of conditional checks and string comparisons.
    - Call `control_add_sub` to add the subscription to the client with the parsed name, type, identifier, and additional data.
    - Free the duplicated string `copy` before exiting the function.
- **Output**:
    - The function does not return a value; it performs operations on the client structure to update its subscription state.


---
### cmd_refresh_client_control_client_size<!-- {{#callable:cmd_refresh_client_control_client_size}} -->
The function `cmd_refresh_client_control_client_size` adjusts the size of a client's window or terminal based on a specified size argument.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve the command arguments using `cmd_get_args` and the target client using `cmdq_get_target_client`.
    - Extract the size argument from the command arguments using `args_get`.
    - Attempt to parse the size argument in the format '@w:WxH' using `sscanf`.
    - If the size is valid and within the defined minimum and maximum window sizes, log the size, add a client window, set its size, update client flags, recalculate sizes, and return `CMD_RETURN_NORMAL`.
    - If the size is in the format '@w:', retrieve the client window, log the action, set the window size to zero, recalculate sizes, and return `CMD_RETURN_NORMAL`.
    - If the size is not in the expected formats, attempt to parse it as 'WxH' or 'W,H'.
    - If the parsed size is valid and within limits, set the terminal size, update client flags, recalculate sizes, and return `CMD_RETURN_NORMAL`.
    - If the size is invalid or out of bounds, log an error and return `CMD_RETURN_ERROR`.
- **Output**:
    - The function returns an `enum cmd_retval` which can be `CMD_RETURN_NORMAL` on success or `CMD_RETURN_ERROR` on failure.


---
### cmd_refresh_client_update_offset<!-- {{#callable:cmd_refresh_client_update_offset}} -->
The function `cmd_refresh_client_update_offset` updates the control state of a specific window pane for a client based on a given string value.
- **Inputs**:
    - `tc`: A pointer to a `client` structure representing the target client.
    - `value`: A string representing the pane ID and the desired control state, formatted as '%<pane_id>:<state>'.
- **Control Flow**:
    - Check if the first character of `value` is '%'; if not, return immediately.
    - Duplicate the `value` string into `copy` and attempt to split it at the first ':' character.
    - If no ':' is found, or if the pane ID cannot be parsed from the string, or if the pane cannot be found by ID, free `copy` and exit.
    - Determine the control state from the substring after ':' and call the appropriate control function (`control_set_pane_on`, `control_set_pane_off`, `control_continue_pane`, or `control_pause_pane`) on the pane.
    - Free the `copy` string before exiting the function.
- **Output**:
    - The function does not return a value; it performs actions on the client and window pane based on the input string.


---
### cmd_refresh_client_clipboard<!-- {{#callable:cmd_refresh_client_clipboard}} -->
The `cmd_refresh_client_clipboard` function updates the clipboard state of a client based on the provided arguments and target client information.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item, which contains context such as the target client.
- **Control Flow**:
    - Retrieve the command arguments using `cmd_get_args` and the target client using `cmdq_get_target_client`.
    - Check if the '-l' argument is provided; if not, check if the client already has the `CLIENT_CLIPBOARDBUFFER` flag set.
    - If the `CLIENT_CLIPBOARDBUFFER` flag is not set, set it and return `CMD_RETURN_NORMAL`.
    - If the '-l' argument is provided, attempt to find the target pane using `cmd_find_target`.
    - If the target pane is not found, return `CMD_RETURN_ERROR`.
    - Iterate over the client's clipboard panes to check if the target pane is already included.
    - If the target pane is not already included, reallocate the clipboard panes array to include the new pane ID.
    - Call `tty_clipboard_query` to update the clipboard state of the client's terminal.
    - Return `CMD_RETURN_NORMAL` to indicate successful execution.
- **Output**:
    - The function returns an `enum cmd_retval`, which is either `CMD_RETURN_NORMAL` for successful execution or `CMD_RETURN_ERROR` if an error occurs during target pane finding.


---
### cmd_refresh_report<!-- {{#callable:cmd_refresh_report}} -->
The `cmd_refresh_report` function processes a string value to update the color settings of a specific window pane in a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal interface.
    - `value`: A constant character pointer representing a string value that specifies the pane and color information.
- **Control Flow**:
    - Check if the first character of `value` is '%'; if not, return immediately.
    - Duplicate the `value` string into `copy` using `xstrdup`.
    - Search for the ':' character in `copy` to split the string; if not found, jump to cleanup and exit.
    - Replace ':' with a null terminator and increment `split` to point to the next character.
    - Parse the pane ID from `copy` using `sscanf`; if parsing fails, jump to cleanup and exit.
    - Find the window pane by ID using `window_pane_find_by_id`; if not found, jump to cleanup and exit.
    - Call `tty_keys_colours` to update the color settings of the pane using the remaining string in `split`.
    - Free the allocated memory for `copy` before exiting.
- **Output**:
    - The function does not return a value; it performs operations on the `tty` and `window_pane` structures.


---
### cmd_refresh_client_exec<!-- {{#callable:cmd_refresh_client_exec}} -->
The `cmd_refresh_client_exec` function processes various command-line arguments to refresh or adjust the state of a client in a terminal multiplexer environment.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve the command arguments and target client from the provided structures.
    - Check for the presence of specific flags ('c', 'L', 'R', 'U', 'D') to determine if a panning adjustment is needed.
    - If panning is required, calculate the adjustment value and update the client's panning offsets accordingly.
    - If the 'l' flag is present, call [`cmd_refresh_client_clipboard`](#cmd_refresh_client_clipboard) to handle clipboard-related operations.
    - Set client flags based on 'F' or 'f' arguments, and call [`cmd_refresh_report`](#cmd_refresh_report) if 'r' is present.
    - For 'A', 'B', and 'C' flags, ensure the client is a control client and perform specific updates or size adjustments.
    - If 'S' is present, force a status update; otherwise, redraw the client.
    - Return `CMD_RETURN_NORMAL` if successful, or `CMD_RETURN_ERROR` if an error occurs, such as when a non-control client attempts a control operation.
- **Output**:
    - The function returns an `enum cmd_retval`, which indicates the success or failure of the command execution, with possible values being `CMD_RETURN_NORMAL` for success or `CMD_RETURN_ERROR` for failure.
- **Functions called**:
    - [`cmd_refresh_client_clipboard`](#cmd_refresh_client_clipboard)
    - [`cmd_refresh_report`](#cmd_refresh_report)
    - [`cmd_refresh_client_update_offset`](#cmd_refresh_client_update_offset)
    - [`cmd_refresh_client_update_subscription`](#cmd_refresh_client_update_subscription)
    - [`cmd_refresh_client_control_client_size`](#cmd_refresh_client_control_client_size)


# Function Declarations (Public API)

---
### cmd_refresh_client_exec<!-- {{#callable_declaration:cmd_refresh_client_exec}} -->
Refreshes the client state based on specified arguments.
- **Description**: This function is used to refresh the state of a client in a terminal multiplexer environment, such as tmux, based on a variety of command-line arguments. It can adjust the client's view, update client flags, handle clipboard operations, and manage control client settings. The function must be called with a valid command and command queue item, and it expects the client to be properly initialized. It handles various flags for panning, resizing, and updating client status, and it returns an error if the client is not a control client when required.
- **Inputs**:
    - `self`: A pointer to the command structure. Must not be null. The caller retains ownership.
    - `item`: A pointer to the command queue item. Must not be null. The caller retains ownership.
- **Output**: Returns an enum cmd_retval indicating success or error, based on the execution of the command.
- **See also**: [`cmd_refresh_client_exec`](#cmd_refresh_client_exec)  (Implementation)


