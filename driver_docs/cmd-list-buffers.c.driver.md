# Purpose
This C source code file is part of the tmux terminal multiplexer project and is responsible for implementing the functionality to list paste buffers. The code defines a command, `list-buffers`, which can be invoked within tmux to display information about the current paste buffers. The command is defined with a name and alias, and it accepts optional arguments for formatting (`-F`) and filtering (`-f`) the output. The core functionality is encapsulated in the [`cmd_list_buffers_exec`](#cmd_list_buffers_exec) function, which iterates over available paste buffers, applies any specified filters, and formats the output according to the provided or default template.

The file is structured to integrate with the broader tmux command framework, as evidenced by the use of structures like `cmd_entry` and functions such as `cmd_get_args` and `cmdq_print`. The code leverages a format tree to handle dynamic formatting and filtering of buffer information, allowing users to customize the output. This file is not a standalone executable but rather a component of the tmux application, providing a specific feature within the larger system. It does not define public APIs or external interfaces but instead contributes to the internal command processing capabilities of tmux.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### cmd_list_buffers_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_list_buffers_entry` is a constant structure of type `cmd_entry` that defines the command for listing paste buffers in the application. It includes the command name, alias, argument specifications, usage instructions, flags, and a pointer to the execution function.
- **Use**: This variable is used to register and define the behavior of the 'list-buffers' command within the application, allowing users to list and filter paste buffers.


# Functions

---
### cmd_list_buffers_exec<!-- {{#callable:cmd_list_buffers_exec}} -->
The `cmd_list_buffers_exec` function lists all paste buffers, optionally filtering and formatting the output based on user-specified templates and filters.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve the command arguments using `cmd_get_args`.
    - Check if a custom format template is provided with the '-F' option; if not, use the default `LIST_BUFFERS_TEMPLATE`.
    - Retrieve any filter expression provided with the '-f' option.
    - Initialize a loop to iterate over all paste buffers using `paste_walk`.
    - For each paste buffer, create a format tree with `format_create` and set default values using `format_defaults_paste_buffer`.
    - If a filter is specified, expand it using `format_expand` and evaluate it with `format_true`; otherwise, set the flag to true.
    - If the flag is true, expand the template using `format_expand` and print the formatted line using `cmdq_print`.
    - Free the format tree and any dynamically allocated memory after use.
    - Return `CMD_RETURN_NORMAL` to indicate successful execution.
- **Output**:
    - The function returns `CMD_RETURN_NORMAL`, indicating successful execution of the command.


# Function Declarations (Public API)

---
### cmd_list_buffers_exec<!-- {{#callable_declaration:cmd_list_buffers_exec}} -->
Lists all available paste buffers with optional formatting and filtering.
- **Description**: This function is used to list all paste buffers available in the current session, optionally applying a user-defined format and filter. It should be called when a user needs to view the contents of paste buffers, potentially in a customized format. The function iterates over each paste buffer, applying the specified format and filter, and outputs the result. It requires a valid command and command queue item to operate, and it assumes that the paste buffers are already initialized and available. The function does not modify any input parameters or global state.
- **Inputs**:
    - `self`: A pointer to the command structure representing the current command. Must not be null. The caller retains ownership.
    - `item`: A pointer to the command queue item associated with the current command execution. Must not be null. The caller retains ownership.
- **Output**: Returns CMD_RETURN_NORMAL to indicate successful execution.
- **See also**: [`cmd_list_buffers_exec`](#cmd_list_buffers_exec)  (Implementation)


