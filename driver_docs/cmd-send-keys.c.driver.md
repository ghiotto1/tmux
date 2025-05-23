# Purpose
This C source code file is part of the `tmux` terminal multiplexer project and is responsible for implementing the functionality to send key inputs to a client session. The file defines two command entries, `send-keys` and `send-prefix`, which are used to send specified keys or a prefix key to a target pane within a tmux session. The `send-keys` command allows for a variety of options, such as specifying a target client, repeating the key input a certain number of times, and handling different key input modes. The `send-prefix` command is a specialized version that sends a prefix key, which can be configured to use an alternate prefix if specified.

The core functionality is encapsulated in the [`cmd_send_keys_exec`](#cmd_send_keys_exec) function, which processes the command arguments and determines the appropriate action based on the options provided. It utilizes helper functions like [`cmd_send_keys_inject_key`](#cmd_send_keys_inject_key) and [`cmd_send_keys_inject_string`](#cmd_send_keys_inject_string) to handle the injection of key events into the target pane. The file is structured to integrate with the broader tmux command queue and client handling system, making use of structures and functions defined elsewhere in the tmux codebase. This file does not define a standalone executable but rather provides specific command implementations that are part of the tmux command interface, allowing users to interact with and control tmux sessions programmatically.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### cmd_send_keys_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_send_keys_entry` is a constant structure of type `cmd_entry` that defines the command 'send-keys' and its associated properties. It includes the command name, alias, argument specifications, usage instructions, target specifications, flags, and the execution function pointer. This structure is used to encapsulate all necessary information for the 'send-keys' command within the tmux application.
- **Use**: This variable is used to define and register the 'send-keys' command within the tmux command framework, allowing users to send key sequences to a target pane or client.


---
### cmd_send_prefix_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_send_prefix_entry` is a constant structure of type `cmd_entry` that defines the command entry for the 'send-prefix' command in the tmux application. It specifies the command's name, arguments, usage, target, flags, and execution function. This structure is used to handle the 'send-prefix' command, which is likely related to sending a prefix key to a target pane in tmux.
- **Use**: This variable is used to define and handle the 'send-prefix' command within the tmux application, specifying its behavior and execution.


# Functions

---
### cmd_send_keys_inject_key<!-- {{#callable:cmd_send_keys_inject_key}} -->
The `cmd_send_keys_inject_key` function processes a key event for a specified client and window pane, handling key bindings and mode-specific key tables if applicable.
- **Inputs**:
    - `item`: A pointer to the current command queue item, representing the context in which the key is being injected.
    - `after`: A pointer to the command queue item that should follow the current operation, used for chaining commands.
    - `args`: A pointer to an `args` structure containing command-line arguments that may modify the behavior of the function.
    - `key`: A `key_code` representing the key to be injected into the target client or window pane.
- **Control Flow**:
    - Retrieve the target client, session, winlink, and window pane from the command queue item.
    - Check if the 'K' argument is present; if so, handle the key event directly for the target client and return the current item.
    - Retrieve the first mode entry from the window pane's modes; if none exists or no key table is defined, send the key directly to the window pane and return the current item if successful.
    - If a key table is defined, retrieve the key binding for the given key from the table.
    - If a key binding is found, increment the table's reference count, dispatch the key binding, and then decrement the reference count.
    - Return the command queue item that should follow the current operation.
- **Output**:
    - Returns a pointer to the command queue item that should follow the current operation, which may be the same as the input `item` or a different item if a key binding was dispatched.
- **Functions called**:
    - [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target)
    - [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`server_client_handle_key`](server-client.c.driver.md#server_client_handle_key)
    - [`window_pane_key`](window.c.driver.md#window_pane_key)
    - [`key_bindings_get_table`](key-bindings.c.driver.md#key_bindings_get_table)
    - [`key_bindings_get`](key-bindings.c.driver.md#key_bindings_get)
    - [`key_bindings_dispatch`](key-bindings.c.driver.md#key_bindings_dispatch)
    - [`key_bindings_unref_table`](key-bindings.c.driver.md#key_bindings_unref_table)


---
### cmd_send_keys_inject_string<!-- {{#callable:cmd_send_keys_inject_string}} -->
The function `cmd_send_keys_inject_string` processes a string from command arguments and injects it as key events into a command queue, handling both literal and non-literal key representations.
- **Inputs**:
    - `item`: A pointer to the current command queue item, representing the context in which the command is executed.
    - `after`: A pointer to the command queue item after which the new key events should be injected.
    - `args`: A pointer to the structure containing command arguments, which may include flags and strings to be processed.
    - `i`: An integer index used to retrieve a specific string from the command arguments.
- **Control Flow**:
    - Retrieve the string `s` from the command arguments using the index `i`.
    - Check if the 'H' flag is present in the arguments; if so, attempt to parse `s` as a hexadecimal number and inject it as a literal key if valid.
    - Determine if the 'l' (literal) flag is present; if not, attempt to look up `s` as a key string and inject it if found.
    - If the key is not found or the literal flag is set, convert `s` to UTF-8 data and iterate over each character.
    - For each character, determine if it is a single-byte ASCII character or a multi-byte UTF-8 character, and inject it as a key event.
    - Free the allocated UTF-8 data after processing all characters.
    - Return the updated `after` pointer, which may point to the last injected key event.
- **Output**:
    - The function returns a pointer to the `cmdq_item` that follows the last injected key event, or the original `after` pointer if no keys were injected.
- **Functions called**:
    - [`args_string`](arguments.c.driver.md#args_string)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`cmd_send_keys_inject_key`](#cmd_send_keys_inject_key)
    - [`key_string_lookup_string`](key-string.c.driver.md#key_string_lookup_string)
    - [`utf8_fromcstr`](utf8.c.driver.md#utf8_fromcstr)
    - [`utf8_from_data`](utf8.c.driver.md#utf8_from_data)


---
### cmd_send_keys_exec<!-- {{#callable:cmd_send_keys_exec}} -->
The `cmd_send_keys_exec` function processes and executes the 'send-keys' command in tmux, handling various options and sending key events to the appropriate target.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item being processed.
- **Control Flow**:
    - Retrieve command arguments and target information from the command queue item.
    - Check if the 'N' option is present to determine the repeat count for key events, and handle errors if the count is invalid.
    - If the 'X' option is present, execute the command associated with the current window mode entry, if available, and return.
    - If the 'M' option is present, determine the target window pane using mouse events and send the key event to it, returning after execution.
    - If the command is 'send-prefix', determine the prefix key to send based on the '2' option and inject it into the command queue.
    - If the 'R' option is present, reset the input context and update the window pane's flags to indicate style and theme changes.
    - If no keys are specified, inject the event key into the command queue for the specified repeat count.
    - For each key specified in the arguments, inject the key or string into the command queue for the specified repeat count.
- **Output**:
    - Returns `CMD_RETURN_NORMAL` on successful execution or `CMD_RETURN_ERROR` if an error occurs during processing.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target)
    - [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client)
    - [`cmdq_get_event`](cmd-queue.c.driver.md#cmdq_get_event)
    - [`args_count`](arguments.c.driver.md#args_count)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`args_strtonum_and_expand`](arguments.c.driver.md#args_strtonum_and_expand)
    - [`cmd_mouse_pane`](cmd.c.driver.md#cmd_mouse_pane)
    - [`window_pane_key`](window.c.driver.md#window_pane_key)
    - [`cmd_get_entry`](cmd.c.driver.md#cmd_get_entry)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`cmd_send_keys_inject_key`](#cmd_send_keys_inject_key)
    - [`colour_palette_clear`](colour.c.driver.md#colour_palette_clear)
    - [`input_reset`](input.c.driver.md#input_reset)
    - [`cmd_send_keys_inject_string`](#cmd_send_keys_inject_string)


