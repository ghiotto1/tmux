# Purpose
This C source code file is part of the `tmux` project, a terminal multiplexer, and it provides functionality for listing key bindings and commands within the `tmux` environment. The file defines two primary command entries: `list-keys` and `list-commands`, which are used to display the current key bindings and available commands, respectively. The `list-keys` command can be executed with various options to filter and format the output, such as specifying a key table or prefix string, while the `list-commands` command allows for formatting the output using a specified template.

The code includes several static functions that handle the execution logic for these commands, such as [`cmd_list_keys_exec`](#cmd_list_keys_exec) and [`cmd_list_keys_commands`](#cmd_list_keys_commands). These functions interact with the `tmux` key binding and command infrastructure to retrieve and format the necessary information. The file also includes utility functions to calculate the display width of key bindings and to print notes associated with them. The code is structured to be integrated into the larger `tmux` application, utilizing its internal data structures and APIs to provide a cohesive user experience for managing key bindings and commands.
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
- **Description**: The `cmd_list_keys_entry` is a constant structure of type `cmd_entry` that defines the command for listing key bindings in the application. It includes the command name 'list-keys', an alias 'lsk', argument specifications, usage instructions, flags, and a pointer to the execution function `cmd_list_keys_exec`. This structure is part of a command table used to manage and execute commands within the application.
- **Use**: This variable is used to define and register the 'list-keys' command, allowing users to list key bindings with specified options and arguments.


---
### cmd_list_commands_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_list_commands_entry` is a constant structure of type `cmd_entry` that defines the properties and behavior of the 'list-commands' command in the application. It includes the command's name, alias, argument specifications, usage instructions, flags, and the function to execute when the command is invoked.
- **Use**: This variable is used to register and define the 'list-commands' command, allowing it to be recognized and executed within the application.


# Functions

---
### cmd_list_keys_get_width<!-- {{#callable:cmd_list_keys_get_width}} -->
The function `cmd_list_keys_get_width` calculates the maximum display width of key bindings' notes in a specified key table, optionally filtering by a specific key.
- **Inputs**:
    - `tablename`: A string representing the name of the key table to retrieve key bindings from.
    - `only`: A `key_code` value representing a specific key to filter the key bindings; if set to `KEYC_UNKNOWN`, no filtering is applied.
- **Control Flow**:
    - Retrieve the key table using [`key_bindings_get_table`](key-bindings.c.driver.md#key_bindings_get_table) with the provided `tablename` and check if it is NULL; return 0 if it is.
    - Initialize `keywidth` to 0 and get the first key binding from the table using [`key_bindings_first`](key-bindings.c.driver.md#key_bindings_first).
    - Iterate over each key binding in the table using a while loop.
    - For each key binding, check if it should be skipped based on the `only` filter, if it is a mouse key, or if its note is NULL or empty; if so, continue to the next binding.
    - Calculate the display width of the key binding's string representation using [`utf8_cstrwidth`](utf8.c.driver.md#utf8_cstrwidth) and update `keywidth` if this width is greater than the current `keywidth`.
    - Move to the next key binding using [`key_bindings_next`](key-bindings.c.driver.md#key_bindings_next) and repeat the process until all bindings are processed.
    - Return the maximum `keywidth` found.
- **Output**:
    - The function returns an unsigned integer representing the maximum width of the key bindings' notes in the specified table, considering the optional key filter.
- **Functions called**:
    - [`key_bindings_get_table`](key-bindings.c.driver.md#key_bindings_get_table)
    - [`key_bindings_first`](key-bindings.c.driver.md#key_bindings_first)
    - [`key_bindings_next`](key-bindings.c.driver.md#key_bindings_next)
    - [`utf8_cstrwidth`](utf8.c.driver.md#utf8_cstrwidth)
    - [`key_string_lookup_key`](key-string.c.driver.md#key_string_lookup_key)


---
### cmd_list_keys_print_notes<!-- {{#callable:cmd_list_keys_print_notes}} -->
The function `cmd_list_keys_print_notes` iterates through key bindings in a specified table and prints or displays their notes if they meet certain criteria.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, representing the command queue item for which the function is executed.
    - `args`: A pointer to an `args` structure, containing the arguments passed to the command.
    - `tablename`: A constant character pointer representing the name of the key table to be processed.
    - `keywidth`: An unsigned integer representing the width to be used for formatting the key strings.
    - `only`: A `key_code` value specifying a particular key to filter the bindings, or `KEYC_UNKNOWN` to include all keys.
    - `prefix`: A constant character pointer representing a prefix string to be prepended to each output line.
- **Control Flow**:
    - Retrieve the target client from the command queue item.
    - Get the key table using the provided table name; return 0 if the table is not found.
    - Iterate over each key binding in the table.
    - For each binding, check if it matches the specified key (if any), is not a mouse key, and has a note or the 'a' argument is present.
    - If the binding meets the criteria, set `found` to 1 and retrieve the key string.
    - Determine the note to display, either from the binding's note or by printing the command list.
    - Format the key string with padding and print or display the note with the prefix, depending on the presence of the '1' argument and the target client.
    - Free the allocated memory for the temporary strings.
    - If the '1' argument is present, break the loop after the first match.
    - Return the `found` status indicating if any matching bindings were processed.
- **Output**:
    - The function returns an integer indicating whether any matching key bindings with notes were found and processed (1 if found, 0 otherwise).
- **Functions called**:
    - [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client)
    - [`key_bindings_get_table`](key-bindings.c.driver.md#key_bindings_get_table)
    - [`key_bindings_first`](key-bindings.c.driver.md#key_bindings_first)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`key_bindings_next`](key-bindings.c.driver.md#key_bindings_next)
    - [`key_string_lookup_key`](key-string.c.driver.md#key_string_lookup_key)
    - [`cmd_list_print`](cmd.c.driver.md#cmd_list_print)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`utf8_padcstr`](utf8.c.driver.md#utf8_padcstr)


---
### cmd_list_keys_get_prefix<!-- {{#callable:cmd_list_keys_get_prefix}} -->
The function `cmd_list_keys_get_prefix` retrieves a prefix string for key bindings based on the provided arguments and global options.
- **Inputs**:
    - `args`: A pointer to a `struct args` which contains command-line arguments.
    - `prefix`: A pointer to a `key_code` where the function will store the retrieved prefix key code.
- **Control Flow**:
    - Retrieve the prefix key code from global options using [`options_get_number`](options.c.driver.md#options_get_number) and store it in `*prefix`.
    - Check if the argument 'P' is present in `args` using [`args_has`](arguments.c.driver.md#args_has).
    - If 'P' is not present and `*prefix` is not `KEYC_NONE`, format the prefix key code into a string with a trailing space using [`xasprintf`](xmalloc.c.driver.md#xasprintf).
    - If 'P' is not present and `*prefix` is `KEYC_NONE`, allocate an empty string using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - If 'P' is present, duplicate the string associated with 'P' in `args` using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - Return the resulting string.
- **Output**:
    - A dynamically allocated string representing the prefix, which must be freed by the caller.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`key_string_lookup_key`](key-string.c.driver.md#key_string_lookup_key)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`args_get`](arguments.c.driver.md#args_get)


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
    - If a specific table name is provided, check its existence, returning an error if it doesn't exist.
    - If the '-N' flag is set, print notes for the specified or default key table(s) and exit.
    - Initialize variables for tracking repeat flags and width calculations.
    - Iterate over all key tables or the specified table, calculating maximum widths for table names and keys.
    - Allocate memory for a temporary buffer to format output strings.
    - Iterate over key bindings, formatting and printing each binding to the command queue item, respecting repeat flags and width constraints.
    - Free allocated memory and handle the case where a specified key was not found, returning an error if necessary.
    - Return `CMD_RETURN_NORMAL` if successful, or `CMD_RETURN_ERROR` if an error occurred.
- **Output**:
    - The function returns an `enum cmd_retval` indicating success (`CMD_RETURN_NORMAL`) or error (`CMD_RETURN_ERROR`).
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client)
    - [`cmd_get_entry`](cmd.c.driver.md#cmd_get_entry)
    - [`cmd_list_keys_commands`](#cmd_list_keys_commands)
    - [`args_string`](arguments.c.driver.md#args_string)
    - [`key_string_lookup_string`](key-string.c.driver.md#key_string_lookup_string)
    - [`args_get`](arguments.c.driver.md#args_get)
    - [`key_bindings_get_table`](key-bindings.c.driver.md#key_bindings_get_table)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`cmd_list_keys_get_prefix`](#cmd_list_keys_get_prefix)
    - [`cmd_list_keys_get_width`](#cmd_list_keys_get_width)
    - [`utf8_padcstr`](utf8.c.driver.md#utf8_padcstr)
    - [`utf8_cstrwidth`](utf8.c.driver.md#utf8_cstrwidth)
    - [`cmd_list_keys_print_notes`](#cmd_list_keys_print_notes)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`key_bindings_first_table`](key-bindings.c.driver.md#key_bindings_first_table)
    - [`key_bindings_next_table`](key-bindings.c.driver.md#key_bindings_next_table)
    - [`key_bindings_first`](key-bindings.c.driver.md#key_bindings_first)
    - [`key_bindings_next`](key-bindings.c.driver.md#key_bindings_next)
    - [`args_escape`](arguments.c.driver.md#args_escape)
    - [`key_string_lookup_key`](key-string.c.driver.md#key_string_lookup_key)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)
    - [`xrealloc`](xmalloc.c.driver.md#xrealloc)
    - [`strlcat`](compat/strlcat.c.driver.md#strlcat)
    - [`cmd_list_print`](cmd.c.driver.md#cmd_list_print)


---
### cmd_list_single_command<!-- {{#callable:cmd_list_single_command}} -->
The `cmd_list_single_command` function formats and prints details of a single command entry using a specified template.
- **Inputs**:
    - `entry`: A pointer to a `cmd_entry` structure containing details about the command such as its name, alias, and usage.
    - `ft`: A pointer to a `format_tree` structure used for formatting the command details.
    - `template`: A string template used to format the command details.
    - `item`: A pointer to a `cmdq_item` structure used for printing the formatted command details.
- **Control Flow**:
    - Add the command name from `entry` to the format tree `ft` with the key 'command_list_name'.
    - Check if the command has an alias; if so, add it to `ft` with the key 'command_list_alias', otherwise add an empty string.
    - Check if the command has usage information; if so, add it to `ft` with the key 'command_list_usage', otherwise add an empty string.
    - Expand the format tree `ft` using the provided `template` to generate a formatted line.
    - If the generated line is not empty, print it using `cmdq_print` with the `item`.
    - Free the memory allocated for the formatted line.
- **Output**:
    - The function does not return a value; it outputs the formatted command details to the command queue item if the expanded line is not empty.
- **Functions called**:
    - [`format_expand`](format.c.driver.md#format_expand)


---
### cmd_list_keys_commands<!-- {{#callable:cmd_list_keys_commands}} -->
The `cmd_list_keys_commands` function lists available commands or details of a specific command using a specified format template.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve the arguments for the command using [`cmd_get_args`](cmd.c.driver.md#cmd_get_args) and store them in `args`.
    - Check if a format template is provided with the '-F' option; if not, use a default template.
    - Create a format tree `ft` using [`format_create`](format.c.driver.md#format_create) and set default values with [`format_defaults`](format.c.driver.md#format_defaults).
    - Retrieve the command name from the arguments using [`args_string`](arguments.c.driver.md#args_string).
    - If no specific command is provided, iterate over all command entries in `cmd_table` and list each using [`cmd_list_single_command`](#cmd_list_single_command).
    - If a specific command is provided, find the command entry using [`cmd_find`](cmd.c.driver.md#cmd_find).
    - If the command entry is found, list it using [`cmd_list_single_command`](#cmd_list_single_command); otherwise, report an error using `cmdq_error` and return `CMD_RETURN_ERROR`.
    - Free the format tree `ft` and return `CMD_RETURN_NORMAL` if successful.
- **Output**:
    - Returns `CMD_RETURN_NORMAL` on success or `CMD_RETURN_ERROR` if an error occurs, such as when a specified command is not found.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`args_get`](arguments.c.driver.md#args_get)
    - [`format_create`](format.c.driver.md#format_create)
    - [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client)
    - [`format_defaults`](format.c.driver.md#format_defaults)
    - [`args_string`](arguments.c.driver.md#args_string)
    - [`cmd_list_single_command`](#cmd_list_single_command)
    - [`cmd_find`](cmd.c.driver.md#cmd_find)
    - [`format_free`](format.c.driver.md#format_free)


