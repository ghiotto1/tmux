# Purpose
This C source code file is part of the tmux terminal multiplexer project and implements the functionality for the "refresh-client" command. The primary purpose of this file is to handle the refreshing of client interfaces within tmux, allowing for updates to client display settings, such as size adjustments, clipboard management, and control client subscriptions. The file defines a command entry structure, `cmd_refresh_client_entry`, which specifies the command's name, alias, arguments, usage, and execution function. The execution function, [`cmd_refresh_client_exec`](#cmd_refresh_client_exec), processes various command-line options to perform tasks like adjusting client window sizes, updating client offsets, managing clipboard contents, and handling control client subscriptions.

The file contains several static functions that support the main execution function by performing specific tasks. For instance, [`cmd_refresh_client_update_subscription`](#cmd_refresh_client_update_subscription) manages control client subscriptions, [`cmd_refresh_client_control_client_size`](#cmd_refresh_client_control_client_size) adjusts the size of client windows, and [`cmd_refresh_client_clipboard`](#cmd_refresh_client_clipboard) handles clipboard operations. These functions interact with the tmux client and server structures to modify client states and trigger necessary redraws or updates. The code is designed to be integrated into the broader tmux application, providing a specific set of functionalities related to client interface management, and does not define public APIs or external interfaces beyond the command entry point.
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
- **Description**: The `cmd_refresh_client_entry` is a constant structure of type `cmd_entry` that defines the command 'refresh-client' with an alias 'refresh'. It specifies the arguments, usage, flags, and the execution function associated with this command. This structure is part of the command handling mechanism in the tmux application, allowing users to refresh the client state with various options.
- **Use**: This variable is used to register and define the behavior of the 'refresh-client' command within the tmux application, enabling client refresh operations.


# Functions

---
### cmd_refresh_client_update_subscription<!-- {{#callable:cmd_refresh_client_update_subscription}} -->
The function `cmd_refresh_client_update_subscription` updates a client's subscription based on a given string value, parsing it to determine the subscription type and identifier.
- **Inputs**:
    - `tc`: A pointer to a `client` structure representing the client whose subscription is to be updated.
    - `value`: A constant character pointer representing the subscription string to be parsed and used for updating the client's subscription.
- **Control Flow**:
    - Duplicate the input string `value` into `copy` and `name` using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - Check for the first colon ':' in `copy` to split the string into `name` and `what`; if not found, remove the subscription and exit.
    - Split `what` further by the next colon ':' to separate the subscription type from the format string; if not found, exit.
    - Determine the subscription type (`subtype`) based on the value of `what` using string comparison and `sscanf` to parse pane or window identifiers.
    - Call [`control_add_sub`](control.c.driver.md#control_add_sub) with the parsed `name`, `subtype`, `subid`, and the remaining format string to add the subscription.
    - Free the duplicated string `copy` before exiting the function.
- **Output**:
    - The function does not return a value; it performs operations on the `client` structure and manages memory for the duplicated string.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`control_remove_sub`](control.c.driver.md#control_remove_sub)
    - [`control_add_sub`](control.c.driver.md#control_add_sub)


---
### cmd_refresh_client_control_client_size<!-- {{#callable:cmd_refresh_client_control_client_size}} -->
The function `cmd_refresh_client_control_client_size` adjusts the size of a client's window or terminal based on a specified size argument.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve the command arguments using [`cmd_get_args`](cmd.c.driver.md#cmd_get_args) and the target client using [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client).
    - Extract the size argument from the command arguments using [`args_get`](arguments.c.driver.md#args_get).
    - Attempt to parse the size argument in the format '@w:WxH' using `sscanf`.
    - If the size is successfully parsed and within valid bounds, log the size, update the client's window size, set the `CLIENT_WINDOWSIZECHANGED` flag, and recalculate sizes.
    - If the size is in the format '@w:', retrieve the client window, log the action, reset the window size, and recalculate sizes.
    - If the size is in the format 'WxH' or 'W,H', validate the size, set the terminal size using [`tty_set_size`](tty.c.driver.md#tty_set_size), set the `CLIENT_SIZECHANGED` flag, and recalculate sizes.
    - Return `CMD_RETURN_NORMAL` if successful, or `CMD_RETURN_ERROR` if there is an error in parsing or size validation.
- **Output**:
    - The function returns an `enum cmd_retval` indicating success (`CMD_RETURN_NORMAL`) or failure (`CMD_RETURN_ERROR`).
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client)
    - [`args_get`](arguments.c.driver.md#args_get)
    - [`server_client_add_client_window`](server-client.c.driver.md#server_client_add_client_window)
    - [`recalculate_sizes_now`](resize.c.driver.md#recalculate_sizes_now)
    - [`server_client_get_client_window`](server-client.c.driver.md#server_client_get_client_window)
    - [`tty_set_size`](tty.c.driver.md#tty_set_size)


---
### cmd_refresh_client_update_offset<!-- {{#callable:cmd_refresh_client_update_offset}} -->
The function `cmd_refresh_client_update_offset` updates the state of a specific window pane for a client based on a given command string.
- **Inputs**:
    - `tc`: A pointer to a `client` structure representing the client whose window pane state is to be updated.
    - `value`: A string containing the pane identifier and the desired state, formatted as '%<pane_id>:<state>'.
- **Control Flow**:
    - Check if the first character of `value` is '%'; if not, return immediately.
    - Duplicate the `value` string into `copy` and attempt to split it at the first ':' character.
    - If no ':' is found, or if the pane ID cannot be parsed from `copy`, or if the pane cannot be found by ID, free `copy` and exit.
    - Determine the desired state from the `split` string and call the appropriate function to update the pane state ([`control_set_pane_on`](control.c.driver.md#control_set_pane_on), [`control_set_pane_off`](control.c.driver.md#control_set_pane_off), [`control_continue_pane`](control.c.driver.md#control_continue_pane), or [`control_pause_pane`](control.c.driver.md#control_pause_pane)).
    - Free the `copy` string before exiting the function.
- **Output**:
    - The function does not return a value; it performs actions to update the state of a window pane for a client.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`window_pane_find_by_id`](window.c.driver.md#window_pane_find_by_id)
    - [`control_set_pane_on`](control.c.driver.md#control_set_pane_on)
    - [`control_set_pane_off`](control.c.driver.md#control_set_pane_off)
    - [`control_continue_pane`](control.c.driver.md#control_continue_pane)
    - [`control_pause_pane`](control.c.driver.md#control_pause_pane)


---
### cmd_refresh_client_clipboard<!-- {{#callable:cmd_refresh_client_clipboard}} -->
The function `cmd_refresh_client_clipboard` manages the clipboard state of a client, either by setting a clipboard buffer flag or by adding a pane to the clipboard list based on the provided arguments.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve the command arguments using [`cmd_get_args`](cmd.c.driver.md#cmd_get_args) and the target client using [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client).
    - Check if the '-l' argument is provided; if not, check if the client already has the `CLIENT_CLIPBOARDBUFFER` flag set.
    - If the `CLIENT_CLIPBOARDBUFFER` flag is not set, set it and return `CMD_RETURN_NORMAL`.
    - If the '-l' argument is provided, attempt to find the target pane using [`cmd_find_target`](cmd-find.c.driver.md#cmd_find_target).
    - If the target pane is not found, return `CMD_RETURN_ERROR`.
    - Iterate through the client's clipboard panes to check if the target pane is already in the list.
    - If the target pane is not in the list, reallocate the clipboard panes array to add the new pane ID.
    - Call [`tty_clipboard_query`](tty.c.driver.md#tty_clipboard_query) to update the clipboard state and return `CMD_RETURN_NORMAL`.
- **Output**:
    - The function returns an `enum cmd_retval`, which is either `CMD_RETURN_NORMAL` if the operation is successful or `CMD_RETURN_ERROR` if an error occurs.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client)
    - [`args_get`](arguments.c.driver.md#args_get)
    - [`cmd_find_target`](cmd-find.c.driver.md#cmd_find_target)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`tty_clipboard_query`](tty.c.driver.md#tty_clipboard_query)


---
### cmd_refresh_report<!-- {{#callable:cmd_refresh_report}} -->
The `cmd_refresh_report` function processes a string value to update the color settings of a specific window pane in a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal interface.
    - `value`: A constant character pointer representing a string value that specifies the pane and color information.
- **Control Flow**:
    - Check if the first character of `value` is '%'; if not, return immediately.
    - Duplicate the `value` string into `copy` using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - Search for the ':' character in `copy` to split the string; if not found, jump to cleanup and exit.
    - Replace ':' with a null terminator and increment `split` to point to the next character.
    - Parse the `copy` string to extract the pane ID using `sscanf`; if parsing fails, jump to cleanup and exit.
    - Find the window pane by ID using [`window_pane_find_by_id`](window.c.driver.md#window_pane_find_by_id); if not found, jump to cleanup and exit.
    - Call [`tty_keys_colours`](tty-keys.c.driver.md#tty_keys_colours) to update the color settings of the pane using the remaining string in `split`.
    - Free the allocated memory for `copy` before exiting.
- **Output**:
    - The function does not return a value; it performs operations on the terminal interface and window pane structures.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`window_pane_find_by_id`](window.c.driver.md#window_pane_find_by_id)
    - [`tty_keys_colours`](tty-keys.c.driver.md#tty_keys_colours)


---
### cmd_refresh_client_exec<!-- {{#callable:cmd_refresh_client_exec}} -->
The `cmd_refresh_client_exec` function processes various command-line arguments to refresh or adjust the state of a client in a terminal multiplexer environment.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item being processed.
- **Control Flow**:
    - Retrieve the command arguments and target client from the provided structures.
    - Check if any of the arguments 'c', 'L', 'R', 'U', or 'D' are present to adjust the client's view or reset the pan window.
    - If adjustment arguments are present, calculate the adjustment value or default to 1 if not specified.
    - Adjust the client's pan offset based on the direction specified by the arguments ('L', 'R', 'U', 'D').
    - Update the client's offset and redraw the client if necessary.
    - If the 'l' argument is present, call [`cmd_refresh_client_clipboard`](#cmd_refresh_client_clipboard) to handle clipboard operations.
    - Set client flags if 'F' or 'f' arguments are present.
    - Generate a report if the 'r' argument is present.
    - If 'A', 'B', or 'C' arguments are present, ensure the client is a control client and perform specific updates or size adjustments.
    - If 'S' is present, force a status update; otherwise, redraw the client.
    - Return `CMD_RETURN_NORMAL` if successful, or `CMD_RETURN_ERROR` if an error occurs, such as when a non-control client attempts to use control-specific arguments.
- **Output**:
    - The function returns an `enum cmd_retval`, which indicates the success or failure of the command execution, with possible values being `CMD_RETURN_NORMAL` for success or `CMD_RETURN_ERROR` for failure.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`args_count`](arguments.c.driver.md#args_count)
    - [`strtonum`](compat/strtonum.c.driver.md#strtonum)
    - [`args_string`](arguments.c.driver.md#args_string)
    - [`tty_update_client_offset`](tty.c.driver.md#tty_update_client_offset)
    - [`server_redraw_client`](server-fn.c.driver.md#server_redraw_client)
    - [`cmd_refresh_client_clipboard`](#cmd_refresh_client_clipboard)
    - [`server_client_set_flags`](server-client.c.driver.md#server_client_set_flags)
    - [`args_get`](arguments.c.driver.md#args_get)
    - [`cmd_refresh_report`](#cmd_refresh_report)
    - [`args_first_value`](arguments.c.driver.md#args_first_value)
    - [`cmd_refresh_client_update_offset`](#cmd_refresh_client_update_offset)
    - [`args_next_value`](arguments.c.driver.md#args_next_value)
    - [`cmd_refresh_client_update_subscription`](#cmd_refresh_client_update_subscription)
    - [`cmd_refresh_client_control_client_size`](#cmd_refresh_client_control_client_size)
    - [`server_status_client`](server-fn.c.driver.md#server_status_client)


