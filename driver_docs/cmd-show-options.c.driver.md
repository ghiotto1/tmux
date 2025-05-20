# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file specifically implements functionality for displaying various options and settings within tmux. It defines several command entries, such as "show-options," "show-window-options," and "show-hooks," each associated with a specific command execution function, [`cmd_show_options_exec`](#cmd_show_options_exec). These commands are used to retrieve and display configuration options for tmux sessions, windows, and hooks, providing users with the ability to query and view current settings.

The file includes several static functions that handle the execution and output formatting of these commands. The [`cmd_show_options_exec`](#cmd_show_options_exec) function is the core execution function that processes command arguments, determines the scope of options to display, and calls helper functions to print the options. The [`cmd_show_options_print`](#cmd_show_options_print) function formats and prints individual options, while [`cmd_show_options_all`](#cmd_show_options_all) iterates over all options within a specified scope and prints them. The code is structured to handle different types of options, including arrays and hooks, and provides mechanisms to handle ambiguous or invalid option names. This file is integral to the tmux command-line interface, enabling users to interactively manage and inspect their tmux environment settings.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### cmd_show_options_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_show_options_entry` is a constant structure of type `cmd_entry` that defines the command 'show-options' and its alias 'show'. It specifies the arguments, usage, target, flags, and execution function associated with this command.
- **Use**: This variable is used to register and define the behavior of the 'show-options' command within the application, including how it is invoked and executed.


---
### cmd_show_window_options_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_show_window_options_entry` is a constant structure of type `cmd_entry` that defines the command for showing window options in the tmux application. It includes the command name, alias, argument specifications, usage instructions, target specifications, flags, and the execution function associated with the command.
- **Use**: This variable is used to register and define the behavior of the 'show-window-options' command within the tmux command processing system.


---
### cmd_show_hooks_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_show_hooks_entry` is a constant structure of type `cmd_entry` that defines the command entry for the 'show-hooks' command in the tmux application. It specifies the command's name, arguments, usage, target, flags, and execution function. This structure is used to register and handle the 'show-hooks' command within the tmux command framework.
- **Use**: This variable is used to define and register the 'show-hooks' command, allowing users to display hooks in tmux.


# Functions

---
### cmd_show_options_exec<!-- {{#callable:cmd_show_options_exec}} -->
The `cmd_show_options_exec` function processes and displays options for a given command, handling both specific and all options based on the provided arguments.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve the arguments and target from the command and command queue item respectively.
    - Determine if the command is for window options by comparing the command entry.
    - If no arguments are provided, determine the options scope from flags and display all options if valid, otherwise handle errors.
    - Format the first argument and attempt to match it to an option name, handling errors for invalid or ambiguous options.
    - Determine the options scope from the option name and display the option if valid, otherwise handle errors.
    - Retrieve the specific option entry and print it if found, considering parent options if the 'A' flag is set.
    - Free allocated memory for the option name and argument before returning.
    - Return `CMD_RETURN_NORMAL` if successful, otherwise return `CMD_RETURN_ERROR` on failure.
- **Output**:
    - Returns an `enum cmd_retval` indicating the success or failure of the command execution, specifically `CMD_RETURN_NORMAL` for success and `CMD_RETURN_ERROR` for failure.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target)
    - [`cmd_get_entry`](cmd.c.driver.md#cmd_get_entry)
    - [`args_count`](arguments.c.driver.md#args_count)
    - [`options_scope_from_flags`](options.c.driver.md#options_scope_from_flags)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`cmd_show_options_all`](#cmd_show_options_all)
    - [`format_single_from_target`](format.c.driver.md#format_single_from_target)
    - [`args_string`](arguments.c.driver.md#args_string)
    - [`options_match`](options.c.driver.md#options_match)
    - [`options_scope_from_name`](options.c.driver.md#options_scope_from_name)
    - [`options_get_only`](options.c.driver.md#options_get_only)
    - [`options_get`](options.c.driver.md#options_get)
    - [`cmd_show_options_print`](#cmd_show_options_print)


---
### cmd_show_options_print<!-- {{#callable:cmd_show_options_print}} -->
The [`cmd_show_options_print`](#cmd_show_options_print) function prints the value of a specified option, handling both array and non-array options, and formats the output based on the presence of a parent option and verbosity flag.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command context.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
    - `o`: A pointer to the `options_entry` structure representing the option to be printed.
    - `idx`: An integer representing the index of the option if it is part of an array, or -1 if not applicable.
    - `parent`: An integer flag indicating if the option is a parent option (1 if true, 0 if false).
- **Control Flow**:
    - Retrieve the command arguments using `cmd_get_args(self)`.
    - Get the option name using `options_name(o)`.
    - If `idx` is not -1, format the name with the index using [`xasprintf`](xmalloc.c.driver.md#xasprintf).
    - If `idx` is -1 and the option is an array, iterate over each array item and recursively call [`cmd_show_options_print`](#cmd_show_options_print) for each index.
    - Convert the option value to a string using [`options_to_string`](options.c.driver.md#options_to_string).
    - If the verbose flag ('v') is set in the arguments, print only the value.
    - If the option is a string, escape the value and print it with the name, prefixed by '*' if it is a parent option.
    - If the option is not a string, print the name and value, prefixed by '*' if it is a parent option.
    - Free any dynamically allocated memory for `value` and `tmp`.
- **Output**:
    - The function does not return a value; it outputs the formatted option name and value to the command queue item.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`options_name`](options.c.driver.md#options_name)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`options_is_array`](options.c.driver.md#options_is_array)
    - [`options_array_first`](options.c.driver.md#options_array_first)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`options_array_item_index`](options.c.driver.md#options_array_item_index)
    - [`cmd_show_options_print`](#cmd_show_options_print)
    - [`options_array_next`](options.c.driver.md#options_array_next)
    - [`options_to_string`](options.c.driver.md#options_to_string)
    - [`options_is_string`](options.c.driver.md#options_is_string)
    - [`args_escape`](arguments.c.driver.md#args_escape)


---
### cmd_show_options_all<!-- {{#callable:cmd_show_options_all}} -->
The `cmd_show_options_all` function iterates over all options in a given scope and prints their values, handling both regular and array options, and considering parent options if specified.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
    - `scope`: An integer representing the scope of options to be shown.
    - `oo`: A pointer to the `options` structure containing the options to be processed.
- **Control Flow**:
    - Retrieve the command arguments using `cmd_get_args(self)`.
    - Check if the command entry is not `cmd_show_hooks_entry` and iterate over all options in `oo`, printing those without a table entry.
    - Iterate over the global `options_table` entries.
    - For each entry, check if the entry's scope matches the provided scope; if not, continue to the next entry.
    - Check if the entry is a hook and if it should be skipped based on the command entry and arguments.
    - Retrieve the option from `oo` using [`options_get_only`](options.c.driver.md#options_get_only); if not found and the 'A' argument is not present, continue to the next entry.
    - Determine if the option is a parent option based on its presence in `oo`.
    - If the option is not an array, print it using [`cmd_show_options_print`](#cmd_show_options_print).
    - If the option is an array, iterate over its items and print each using [`cmd_show_options_print`](#cmd_show_options_print).
    - Return `CMD_RETURN_NORMAL` to indicate successful execution.
- **Output**:
    - The function returns `CMD_RETURN_NORMAL`, indicating successful execution.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmd_get_entry`](cmd.c.driver.md#cmd_get_entry)
    - [`options_first`](options.c.driver.md#options_first)
    - [`options_table_entry`](tmux.h.driver.md#options_table_entry)
    - [`cmd_show_options_print`](#cmd_show_options_print)
    - [`options_next`](options.c.driver.md#options_next)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`options_get_only`](options.c.driver.md#options_get_only)
    - [`options_get`](options.c.driver.md#options_get)
    - [`options_is_array`](options.c.driver.md#options_is_array)
    - [`options_array_first`](options.c.driver.md#options_array_first)
    - [`options_name`](options.c.driver.md#options_name)
    - [`options_array_item_index`](options.c.driver.md#options_array_item_index)
    - [`options_array_next`](options.c.driver.md#options_array_next)


