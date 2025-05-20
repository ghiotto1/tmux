# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file defines functionality for the "display-message" command, which is used to display messages in the status line of a tmux session. The command can be invoked with various options to customize the message format, target client, delay, and other parameters. The code includes a command entry structure (`cmd_display_message_entry`) that specifies the command's name, alias, arguments, usage, target, flags, and execution function. The execution function ([`cmd_display_message_exec`](#cmd_display_message_exec)) processes the command's arguments, formats the message according to the specified template, and displays it to the appropriate client or pane.

The file provides a narrow functionality focused on message display within tmux, and it is not intended to be a standalone executable but rather a component of the larger tmux application. The code defines a public API for the "display-message" command, allowing it to be used within the tmux environment. Key technical components include the use of format trees for message formatting, handling of command-line arguments, and interaction with tmux's client and session structures. The code also includes error handling for invalid arguments and memory management for dynamically allocated strings.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `time.h`
- `tmux.h`


# Global Variables

---
### cmd_display_message_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_display_message_entry` is a constant structure of type `cmd_entry` that defines the command 'display-message' and its alias 'display'. It specifies the arguments, usage, target, flags, and execution function associated with the command.
- **Use**: This variable is used to register and define the behavior of the 'display-message' command within the application, allowing it to display messages in the status line.


# Functions

---
### cmd_display_message_each<!-- {{#callable:cmd_display_message_each}} -->
The function `cmd_display_message_each` prints a formatted key-value pair to a command queue item.
- **Inputs**:
    - `key`: A constant character pointer representing the key in a key-value pair.
    - `value`: A constant character pointer representing the value in a key-value pair.
    - `arg`: A void pointer that is expected to be a pointer to a `cmdq_item` structure, used for printing the message.
- **Control Flow**:
    - The function casts the `arg` parameter to a `cmdq_item` pointer.
    - It then calls `cmdq_print` with the `cmdq_item` pointer and a formatted string containing the key and value.
- **Output**:
    - The function does not return a value; it performs an action by printing a formatted message to a command queue item.


---
### cmd_display_message_exec<!-- {{#callable:cmd_display_message_exec}} -->
The `cmd_display_message_exec` function displays a formatted message in the status line or to a client in a tmux session, with various options for customization and output control.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve command arguments and target information from the `cmd` and `cmdq_item` structures.
    - Check if the '-I' flag is set and handle input start for the window pane, returning appropriate command return values based on the result.
    - Validate that only one of '-F' or a message argument is provided, returning an error if both are given.
    - Parse the '-d' flag to set a delay for the message, returning an error if the delay is invalid.
    - Determine the message template to use, defaulting to `DISPLAY_MESSAGE_TEMPLATE` if none is provided.
    - Select the appropriate client for message display based on the '-c' flag and session information.
    - Create a format tree with default values and flags based on the '-v' flag.
    - If the '-a' flag is set, iterate over each format and print key-value pairs, then return.
    - Expand the message template using the format tree, or duplicate it if the '-l' flag is set.
    - Determine the output method based on the presence of a client, the '-p' flag, and client control flags, then display the message accordingly.
    - Free allocated resources such as the message string and format tree before returning.
- **Output**:
    - Returns an `enum cmd_retval` indicating the result of the command execution, which can be `CMD_RETURN_NORMAL`, `CMD_RETURN_ERROR`, or `CMD_RETURN_WAIT`.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target)
    - [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`args_count`](arguments.c.driver.md#args_count)
    - [`window_pane_start_input`](window.c.driver.md#window_pane_start_input)
    - [`args_strtonum`](arguments.c.driver.md#args_strtonum)
    - [`args_string`](arguments.c.driver.md#args_string)
    - [`args_get`](arguments.c.driver.md#args_get)
    - [`cmd_find_best_client`](cmd-find.c.driver.md#cmd_find_best_client)
    - [`format_create`](format.c.driver.md#format_create)
    - [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client)
    - [`format_defaults`](format.c.driver.md#format_defaults)
    - [`format_each`](format.c.driver.md#format_each)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`format_expand_time`](format.c.driver.md#format_expand_time)
    - [`server_client_print`](server-client.c.driver.md#server_client_print)
    - [`format_free`](format.c.driver.md#format_free)


