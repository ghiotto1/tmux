# Purpose
This C source code file is part of the tmux terminal multiplexer project and provides functionality for sending key inputs to a client session. The file defines two command entries, `cmd_send_keys_entry` and `cmd_send_prefix_entry`, which are used to send keys or a prefix key to a specified target pane within a tmux session. The primary function, [`cmd_send_keys_exec`](#cmd_send_keys_exec), executes the logic for these commands, handling various options such as repeating key inputs, sending keys in literal form, and interacting with mouse events. The code is structured to handle different modes and states of a tmux session, allowing for flexible input handling.

The file is not a standalone executable but rather a component of the larger tmux application, intended to be integrated with other parts of the system. It defines internal functions and structures that are used to manipulate key events and interact with the tmux client and session objects. The code includes mechanisms for handling key tables and bindings, injecting key events, and managing command execution flow. This file is crucial for enabling user interaction with tmux sessions through keyboard inputs, enhancing the overall functionality of the tmux environment.
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
- **Description**: The `cmd_send_keys_entry` is a constant structure of type `cmd_entry` that defines the command 'send-keys' and its associated properties. It includes the command name, alias, argument specifications, usage instructions, target specifications, flags, and the execution function pointer. This structure is used to configure and execute the 'send-keys' command within the application.
- **Use**: This variable is used to define and execute the 'send-keys' command, which sends key inputs to a specified client or pane in the application.


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
    - Returns a pointer to the command queue item that should follow the current operation, which may be the same as the input 'item' or a different item if a key binding was dispatched.


---
### cmd_send_keys_inject_string<!-- {{#callable:cmd_send_keys_inject_string}} -->
The `cmd_send_keys_inject_string` function processes a string from the command arguments and injects it as key events into a command queue, handling both literal and special key codes.
- **Inputs**:
    - `item`: A pointer to the current command queue item.
    - `after`: A pointer to the command queue item after which the keys should be injected.
    - `args`: A pointer to the arguments structure containing command-line arguments.
    - `i`: An integer index specifying which argument string to process.
- **Control Flow**:
    - Retrieve the string from the arguments using the provided index.
    - Check if the 'H' flag is present, indicating a hexadecimal key code, and convert the string to a long integer.
    - If the conversion is valid and within range, inject the key as a literal key code and return.
    - Check if the 'l' flag is present to determine if the string should be treated literally.
    - If not literal, attempt to look up the string as a key code using `key_string_lookup_string`.
    - If a valid key code is found, inject it and return the updated 'after' item.
    - If the string is to be treated literally, convert it to UTF-8 data and iterate over each character.
    - For each character, determine if it is a single-byte ASCII or a multi-byte UTF-8 character.
    - Inject each character as a key event using [`cmd_send_keys_inject_key`](#cmd_send_keys_inject_key).
    - Free the UTF-8 data after processing all characters.
    - Return the updated 'after' item.
- **Output**:
    - Returns a pointer to the command queue item after which the keys were injected, or the original item if no injection occurred.
- **Functions called**:
    - [`cmd_send_keys_inject_key`](#cmd_send_keys_inject_key)


---
### cmd_send_keys_exec<!-- {{#callable:cmd_send_keys_exec}} -->
The `cmd_send_keys_exec` function processes and executes the 'send-keys' command in tmux, handling various options and injecting key events into the target pane or client.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item being processed.
- **Control Flow**:
    - Retrieve command arguments, target client, session, winlink, window pane, and key event from the command queue item.
    - Check if the '-N' option is present to set a repeat count, and handle errors if the count is invalid.
    - If the '-X' option is present, execute the command associated with the current window mode entry, if available, and return.
    - If the '-M' option is present, determine the target pane from the mouse event and inject the mouse key event into it, returning an error if no target is found.
    - If the command is 'send-prefix', inject the appropriate prefix key based on the '-2' option and return.
    - If the '-R' option is present, reset the pane's input context and update its flags to indicate style and theme changes.
    - If no keys are specified, inject the event key for the specified repeat count and return.
    - For each repeat count, inject each specified key string into the target, using [`cmd_send_keys_inject_string`](#cmd_send_keys_inject_string).
- **Output**:
    - Returns `CMD_RETURN_NORMAL` on successful execution or `CMD_RETURN_ERROR` if an error occurs during processing.
- **Functions called**:
    - [`cmd_send_keys_inject_key`](#cmd_send_keys_inject_key)
    - [`cmd_send_keys_inject_string`](#cmd_send_keys_inject_string)


# Function Declarations (Public API)

---
### cmd_send_keys_exec<!-- {{#callable_declaration:cmd_send_keys_exec}} -->
Send specified keys to a target client or pane.
- **Description**: This function is used to send a sequence of keys to a specified target client or pane within a tmux session. It can handle various options such as repeating the key sequence, sending mouse events, or executing commands in a specific mode. The function must be called with a valid command and target item, and it supports additional options for customization, such as specifying a repeat count or handling mouse events. It returns a status indicating success or error based on the validity of the input and the current mode of the target pane.
- **Inputs**:
    - `self`: A pointer to the command structure representing the 'send-keys' or 'send-prefix' command. Must not be null.
    - `item`: A pointer to the command queue item that contains the context for the command execution, including the target client and pane. Must not be null.
- **Output**: Returns an enum cmd_retval indicating the result of the command execution, which can be CMD_RETURN_NORMAL for success or CMD_RETURN_ERROR for failure.
- **See also**: [`cmd_send_keys_exec`](#cmd_send_keys_exec)  (Implementation)


