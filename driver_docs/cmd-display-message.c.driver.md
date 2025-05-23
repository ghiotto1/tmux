# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file defines functionality for the "display-message" command, which is used to display messages in the status line of a tmux session. The command can be invoked with various options to customize the message format, target client, delay, and verbosity. The code includes a command entry structure (`cmd_display_message_entry`) that specifies the command's name, alias, arguments, usage, target, flags, and execution function ([`cmd_display_message_exec`](#cmd_display_message_exec)).

The core functionality is implemented in the [`cmd_display_message_exec`](#cmd_display_message_exec) function, which processes command-line arguments, formats the message using a template, and determines where and how to display the message. The function supports options for specifying a delay, choosing a target client, and formatting the message with session, window, and pane information. The code also includes helper functions like [`cmd_display_message_each`](#cmd_display_message_each) for iterating over format keys and values. This file is a focused component of the tmux codebase, providing a specific feature for user interaction and feedback within the tmux environment.
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
- **Description**: The `cmd_display_message_entry` is a constant structure of type `cmd_entry` that defines the command for displaying messages in the status line of the tmux terminal multiplexer. It includes the command name, alias, argument specifications, usage instructions, target specifications, flags, and the execution function associated with the command.
- **Use**: This variable is used to register and define the behavior of the 'display-message' command within the tmux application, allowing users to display messages in the status line.


# Functions

---
### cmd_display_message_each<!-- {{#callable:cmd_display_message_each}} -->
The function `cmd_display_message_each` prints a formatted key-value pair to a command queue item.
- **Inputs**:
    - `key`: A constant character pointer representing the key in the key-value pair.
    - `value`: A constant character pointer representing the value in the key-value pair.
    - `arg`: A void pointer that is expected to be a pointer to a `cmdq_item` structure, used for printing the message.
- **Control Flow**:
    - The function casts the `arg` parameter to a `cmdq_item` pointer.
    - It then calls `cmdq_print` with the `cmdq_item` pointer and a formatted string containing the key and value.
- **Output**:
    - The function does not return a value; it performs an action by printing a formatted message to a command queue item.


---
### cmd_display_message_exec<!-- {{#callable:cmd_display_message_exec}} -->
The `cmd_display_message_exec` function displays a formatted message in the status line or to a client in a tmux session, with various options for customization.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve command arguments and target information from the `cmd` and `cmdq_item` structures.
    - Check if the '-I' flag is set; if so, start input on the target window pane and handle the result accordingly.
    - Ensure that only one of '-F' or a positional argument is provided, returning an error if both are given.
    - Parse the '-d' flag to set a delay for the message, returning an error if the delay is invalid.
    - Determine the message template to use, defaulting to `DISPLAY_MESSAGE_TEMPLATE` if none is provided.
    - Select the appropriate client for displaying the message, preferring the target client if it matches the session.
    - Create a format tree with the appropriate flags and defaults based on the command context.
    - If the '-a' flag is set, iterate over each format and print it, then return.
    - Expand the message template using the format tree, either directly or with time expansion, based on the '-l' flag.
    - Display the message to the appropriate client or status line, handling different output methods based on flags and client state.
    - Free allocated resources and return `CMD_RETURN_NORMAL` to indicate successful execution.
- **Output**:
    - The function returns an `enum cmd_retval` indicating the result of the command execution, which can be `CMD_RETURN_NORMAL`, `CMD_RETURN_ERROR`, or `CMD_RETURN_WAIT`.


# Function Declarations (Public API)

---
### cmd_display_message_exec<!-- {{#callable_declaration:cmd_display_message_exec}} -->
Displays a message in the status line of a tmux client.
- **Description**: This function is used to display a message in the status line of a specified tmux client or pane. It can be customized with various options such as specifying a delay, choosing a target client, or formatting the message. The function must be called with appropriate arguments, and it handles different scenarios like input redirection, verbose output, and client control mode. It is important to ensure that the correct combination of arguments is used, as some options are mutually exclusive.
- **Inputs**:
    - `self`: A pointer to the cmd structure representing the command to be executed. Must not be null.
    - `item`: A pointer to the cmdq_item structure representing the command queue item. Must not be null.
- **Output**: Returns an enum cmd_retval indicating the result of the command execution, which can be CMD_RETURN_NORMAL, CMD_RETURN_ERROR, or CMD_RETURN_WAIT.
- **See also**: [`cmd_display_message_exec`](#cmd_display_message_exec)  (Implementation)


