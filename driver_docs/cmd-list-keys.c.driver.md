# Purpose
This C source code file is part of the `tmux` project, a terminal multiplexer, and it provides functionality for listing key bindings and commands within the `tmux` environment. The file defines two primary command entries: `list-keys` and `list-commands`, which are used to display the current key bindings and available commands, respectively. The `list-keys` command can be executed with various options to filter or format the output, such as specifying a key table or prefix string, while the `list-commands` command allows for formatting the output using a specified template.

The code includes several static functions that handle the execution of these commands, such as [`cmd_list_keys_exec`](#cmd_list_keys_exec) and [`cmd_list_keys_commands`](#cmd_list_keys_commands), which are responsible for iterating over key tables and command entries to generate the appropriate output. The file also contains utility functions to assist with formatting and displaying the key bindings, such as calculating the width of key strings and printing notes associated with key bindings. This file is integral to the `tmux` command-line interface, providing users with the ability to query and understand the key bindings and commands available in their `tmux` sessions.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### cmd_list_keys_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_list_keys_entry` is a constant structure of type `cmd_entry` that defines the command for listing key bindings in the application. It includes the command name 'list-keys', an alias 'lsk', argument specifications, usage instructions, flags, and a pointer to the execution function `cmd_list_keys_exec`. This structure is part of the command table used to manage and execute commands within the application.
- **Use**: This variable is used to define and execute the 'list-keys' command, which lists key bindings in the application.


---
### cmd_list_commands_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_list_commands_entry` is a constant structure of type `cmd_entry` that defines the properties and behavior of the 'list-commands' command in the application. It includes the command's name, alias, argument specifications, usage instructions, flags, and the function to execute when the command is invoked.
- **Use**: This variable is used to register and define the 'list-commands' command, allowing it to be recognized and executed within the application.


# Functions

---
### cmd_list_keys_get_width<!-- {{#callable:cmd_list_keys_get_width}} -->
The function `cmd_list_keys_get_width` calculates the maximum display width of key bindings' string representations in a specified key table, optionally filtered by a specific key code.
- **Inputs**:
    - `tablename`: A string representing the name of the key table to search for key bindings.
    - `only`: A key_code value that, if not equal to KEYC_UNKNOWN, filters the key bindings to only consider those matching this specific key code.
- **Control Flow**:
    - Retrieve the key table using `key_bindings_get_table` with the provided `tablename`.
    - If the table is not found, return 0 as the width.
    - Initialize `keywidth` to 0 to track the maximum width found.
    - Iterate over each key binding in the table using `key_bindings_first` and `key_bindings_next`.
    - For each key binding, check if it should be skipped based on the `only` filter, if it is a mouse key, or if it has no note.
    - If the key binding is valid, calculate its display width using `utf8_cstrwidth` and update `keywidth` if this width is greater than the current `keywidth`.
    - Continue to the next key binding until all are processed.
    - Return the maximum width found (`keywidth`).
- **Output**:
    - The function returns an unsigned integer representing the maximum width of the string representation of the key bindings in the specified table, filtered by the given key code if applicable.


---
### cmd_list_keys_print_notes<!-- {{#callable:cmd_list_keys_print_notes}} -->
The function `cmd_list_keys_print_notes` iterates through key bindings in a specified table and prints their associated notes if they meet certain criteria.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, representing the command queue item.
    - `args`: A pointer to an `args` structure, containing command-line arguments.
    - `tablename`: A string representing the name of the key table to be processed.
    - `keywidth`: An unsigned integer specifying the width for formatting the key strings.
    - `only`: A `key_code` specifying a particular key to filter the bindings, or `KEYC_UNKNOWN` to include all keys.
    - `prefix`: A string prefix to prepend to each printed line.
- **Control Flow**:
    - Retrieve the target client from the command queue item.
    - Get the key table using the provided table name; return 0 if the table is not found.
    - Iterate over each key binding in the table.
    - For each binding, check if it matches the specified key (if any), is not a mouse key, and has a note or the 'a' argument is present.
    - If the binding meets the criteria, set `found` to 1 and retrieve the key string.
    - Determine the note to print, either from the binding's note or by printing the command list.
    - Format the key string with padding and print the formatted key and note, either to the status message or command queue, depending on the '1' argument.
    - Free allocated memory for temporary strings.
    - If the '1' argument is present, break the loop after the first match.
    - Return the `found` status indicating if any matching bindings were found.
- **Output**:
    - Returns an integer indicating whether any matching key bindings with notes were found (1 if found, 0 otherwise).


---
### cmd_list_keys_get_prefix<!-- {{#callable:cmd_list_keys_get_prefix}} -->
The function `cmd_list_keys_get_prefix` retrieves a prefix string for key bindings based on the provided arguments and global options.
- **Inputs**:
    - `args`: A pointer to a `struct args` which contains command-line arguments.
    - `prefix`: A pointer to a `key_code` where the function will store the retrieved prefix key code.
- **Control Flow**:
    - Retrieve the prefix key code from global options using `options_get_number` and store it in `*prefix`.
    - Check if the `args` structure has the 'P' option using `args_has`.
    - If the 'P' option is not present, check if the retrieved prefix is not `KEYC_NONE`.
    - If the prefix is not `KEYC_NONE`, convert the prefix key code to a string using `key_string_lookup_key` and format it with a space using `xasprintf`.
    - If the prefix is `KEYC_NONE`, allocate an empty string using `xstrdup`.
    - If the 'P' option is present, retrieve the prefix string from `args` using `args_get` and duplicate it using `xstrdup`.
    - Return the resulting string.
- **Output**:
    - Returns a dynamically allocated string representing the prefix, which should be freed by the caller.


---
### cmd_list_keys_exec<!-- {{#callable:cmd_list_keys_exec}} -->
The `cmd_list_keys_exec` function lists key bindings in a specified key table or all tables, optionally filtering by a specific key or prefix, and outputs the results to a command queue item.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item where output will be sent.
- **Control Flow**:
    - Retrieve command arguments and target client from the command queue item.
    - Check if the command is 'list-commands' and delegate to [`cmd_list_keys_commands`](#cmd_list_keys_commands) if true.
    - If a specific key is provided, validate and convert it to a key code, returning an error if invalid.
    - If a specific table name is provided, check its existence and return an error if it doesn't exist.
    - If the '-N' flag is set, print notes for the specified or default key table(s) and exit.
    - Initialize variables for tracking repeat flags and maximum widths for table and key names.
    - Iterate over all key tables, filtering by the specified table name and key code if provided, to determine maximum widths.
    - Allocate memory for a temporary buffer to format output strings.
    - Iterate over all key tables again, formatting and printing each key binding to the command queue item, respecting the '-1' flag for single output.
    - Free allocated memory and handle the case where a specified key was not found, returning an error if necessary.
    - Return `CMD_RETURN_NORMAL` if successful.
- **Output**:
    - Returns an `enum cmd_retval` indicating the success or failure of the command execution, specifically `CMD_RETURN_NORMAL` for success or `CMD_RETURN_ERROR` for errors.
- **Functions called**:
    - [`cmd_list_keys_commands`](#cmd_list_keys_commands)
    - [`cmd_list_keys_get_prefix`](#cmd_list_keys_get_prefix)
    - [`cmd_list_keys_get_width`](#cmd_list_keys_get_width)
    - [`cmd_list_keys_print_notes`](#cmd_list_keys_print_notes)


---
### cmd_list_single_command<!-- {{#callable:cmd_list_single_command}} -->
The `cmd_list_single_command` function formats and prints details of a single command entry using a specified template.
- **Inputs**:
    - `entry`: A pointer to a `cmd_entry` structure containing details about the command such as its name, alias, and usage.
    - `ft`: A pointer to a `format_tree` structure used for storing formatted data.
    - `template`: A string representing the template used to format the command details.
    - `item`: A pointer to a `cmdq_item` structure used for queueing command output.
- **Control Flow**:
    - Add the command name from `entry` to the format tree `ft` under the key `command_list_name`.
    - Check if the command has an alias; if so, add it to `ft` under the key `command_list_alias`, otherwise add an empty string.
    - Check if the command has usage information; if so, add it to `ft` under the key `command_list_usage`, otherwise add an empty string.
    - Expand the template using the format tree `ft` to produce a formatted line.
    - If the resulting line is not empty, print it using `cmdq_print` with the `item`.
    - Free the memory allocated for the expanded line.
- **Output**:
    - The function does not return a value; it outputs formatted command details to the command queue item.


---
### cmd_list_keys_commands<!-- {{#callable:cmd_list_keys_commands}} -->
The `cmd_list_keys_commands` function lists available commands or details of a specific command using a specified format template.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve the arguments for the command using `cmd_get_args` and store them in `args`.
    - Check if a format template is provided with the '-F' option; if not, use a default template.
    - Create a format tree `ft` using `format_create` with the client from the command queue item.
    - Set default values in the format tree using `format_defaults`.
    - Retrieve the command name from the arguments using `args_string`.
    - If no specific command is provided, iterate over all command entries in `cmd_table` and list each command using [`cmd_list_single_command`](#cmd_list_single_command).
    - If a specific command is provided, find the command entry using `cmd_find`.
    - If the command entry is found, list the command using [`cmd_list_single_command`](#cmd_list_single_command); otherwise, report an error using `cmdq_error`.
    - Free the format tree `ft` before returning.
    - Return `CMD_RETURN_NORMAL` if successful, or `CMD_RETURN_ERROR` if an error occurred.
- **Output**:
    - The function returns an `enum cmd_retval` indicating the success or failure of the command listing operation, specifically `CMD_RETURN_NORMAL` for success and `CMD_RETURN_ERROR` for failure.
- **Functions called**:
    - [`cmd_list_single_command`](#cmd_list_single_command)


# Function Declarations (Public API)

---
### cmd_list_keys_exec<!-- {{#callable_declaration:cmd_list_keys_exec}} -->
Lists key bindings in the specified key table.
- **Description**: This function is used to list all key bindings in a specified key table or across all tables if no specific table is provided. It can filter the list by a specific key if provided. The function must be called with a valid command and queue item, and it will output the list of key bindings to the command queue. If a specific key or table is specified and not found, an error is returned. This function is typically used in environments where key bindings need to be reviewed or debugged.
- **Inputs**:
    - `self`: A pointer to the command structure representing the current command. Must not be null.
    - `item`: A pointer to the command queue item structure representing the current command queue item. Must not be null.
- **Output**: Returns CMD_RETURN_NORMAL on success or CMD_RETURN_ERROR if an invalid key or table is specified.
- **See also**: [`cmd_list_keys_exec`](#cmd_list_keys_exec)  (Implementation)


---
### cmd_list_keys_commands<!-- {{#callable_declaration:cmd_list_keys_commands}} -->
Lists available commands and their details.
- **Description**: This function is used to list all available commands or details of a specific command in a formatted manner. It can be called to display command names, aliases, and usage information. The function supports custom formatting through a template string and can handle cases where a specific command is queried. If the command is not found, an error is reported. It is typically used in environments where command details need to be displayed to the user, such as in a command-line interface.
- **Inputs**:
    - `self`: A pointer to a `struct cmd` representing the command context. Must not be null.
    - `item`: A pointer to a `struct cmdq_item` representing the command queue item. Must not be null.
- **Output**: Returns `CMD_RETURN_NORMAL` on success or `CMD_RETURN_ERROR` if a specified command is not found.
- **See also**: [`cmd_list_keys_commands`](#cmd_list_keys_commands)  (Implementation)


