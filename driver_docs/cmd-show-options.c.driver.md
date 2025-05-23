# Purpose
This C source code file is part of the `tmux` project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file specifically implements functionality for displaying various options and settings within `tmux`. It defines several command entries, such as `show-options`, `show-window-options`, and `show-hooks`, each associated with specific command-line arguments and usage patterns. These commands are used to query and display the current configuration options for different scopes, such as panes or windows, and can handle both individual options and entire sets of options.

The core functionality is encapsulated in the [`cmd_show_options_exec`](#cmd_show_options_exec) function, which processes the command-line arguments, determines the appropriate scope for the options, and retrieves and displays the requested options. The file also includes helper functions like [`cmd_show_options_print`](#cmd_show_options_print) and [`cmd_show_options_all`](#cmd_show_options_all) to format and print the options in a user-friendly manner. The code is structured to handle various scenarios, such as ambiguous or invalid options, and provides mechanisms to display options in different formats based on user preferences. This file is integral to the `tmux` command-line interface, enabling users to introspect and manage their session configurations effectively.
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
- **Description**: The `cmd_show_options_entry` is a constant structure of type `cmd_entry` that defines the command 'show-options' and its alias 'show'. It specifies the arguments, usage, target, flags, and execution function for the command.
- **Use**: This variable is used to register and define the behavior of the 'show-options' command within the application, allowing it to be executed with specified arguments and options.


---
### cmd_show_window_options_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_show_window_options_entry` is a constant structure of type `cmd_entry` that defines the command for showing window options in the application. It includes details such as the command name, alias, arguments, usage, target, flags, and the execution function associated with the command.
- **Use**: This variable is used to register and define the behavior of the 'show-window-options' command within the application, allowing users to view options related to windows.


---
### cmd_show_hooks_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_show_hooks_entry` is a constant structure of type `cmd_entry` that defines the command entry for the 'show-hooks' command in the application. It specifies the command's name, arguments, usage, target, flags, and the function to execute when the command is invoked.
- **Use**: This variable is used to register and define the behavior of the 'show-hooks' command within the application, allowing users to view hooks.


# Functions

---
### cmd_show_options_exec<!-- {{#callable:cmd_show_options_exec}} -->
The `cmd_show_options_exec` function processes and displays options for a given command, handling both specific and all options based on the provided arguments.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve the command arguments and target state using `cmd_get_args` and `cmdq_get_target` respectively.
    - Determine if the command is for window options by comparing the command entry with `cmd_show_window_options_entry`.
    - If no arguments are provided, determine the options scope using `options_scope_from_flags` and display all options using [`cmd_show_options_all`](#cmd_show_options_all) if the scope is valid.
    - If an argument is provided, format it and attempt to match it to an option name using `options_match`.
    - If the option name is ambiguous or invalid, handle errors based on the presence of the 'q' flag, otherwise proceed to determine the scope using `options_scope_from_name`.
    - Retrieve the specific option entry using `options_get_only` or `options_get` if the 'A' flag is present, and print the option using [`cmd_show_options_print`](#cmd_show_options_print).
    - Free allocated memory for `name` and `argument` before returning the appropriate command return value.
- **Output**:
    - The function returns an `enum cmd_retval` indicating the success or failure of the command execution, specifically `CMD_RETURN_NORMAL` or `CMD_RETURN_ERROR`.
- **Functions called**:
    - [`cmd_show_options_all`](#cmd_show_options_all)
    - [`cmd_show_options_print`](#cmd_show_options_print)


---
### cmd_show_options_print<!-- {{#callable:cmd_show_options_print}} -->
The [`cmd_show_options_print`](#cmd_show_options_print) function prints the value of a specified option, handling both array and non-array options, and formatting the output based on verbosity and parent option status.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command context.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
    - `o`: A pointer to the `options_entry` structure representing the option to be printed.
    - `idx`: An integer representing the index of the option if it is part of an array, or -1 if not applicable.
    - `parent`: An integer flag indicating whether the option is a parent option (1 if true, 0 if false).
- **Control Flow**:
    - Retrieve the command arguments using `cmd_get_args` function.
    - Get the option name using `options_name` function.
    - If `idx` is not -1, format the name with the index using `xasprintf`.
    - If `idx` is -1 and the option is an array, iterate over each array item and recursively call [`cmd_show_options_print`](#cmd_show_options_print) for each index.
    - Convert the option value to a string using `options_to_string`.
    - If the verbose flag ('v') is set in the arguments, print only the value.
    - If the option is a string, escape the value using `args_escape` and print it with the name, prefixed by '*' if it is a parent option.
    - If the option is not a string, print the name and value, prefixed by '*' if it is a parent option.
    - Free any dynamically allocated memory for the value and temporary name.
- **Output**:
    - The function does not return a value; it outputs the formatted option name and value to the command queue item.
- **Functions called**:
    - [`cmd_show_options_print`](#cmd_show_options_print)


---
### cmd_show_options_all<!-- {{#callable:cmd_show_options_all}} -->
The `cmd_show_options_all` function iterates over all options in a given scope and prints their values, handling both individual and array options, while considering specific command arguments and conditions.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
    - `scope`: An integer representing the scope of options to be shown.
    - `oo`: A pointer to the `options` structure containing the options to be processed.
- **Control Flow**:
    - Retrieve the command arguments using `cmd_get_args(self)` and store them in `args`.
    - Check if the command entry is not `cmd_show_hooks_entry`, then iterate over all options in `oo` using `options_first` and `options_next`, printing each option that does not have a table entry using [`cmd_show_options_print`](#cmd_show_options_print).
    - Iterate over the global `options_table` entries, checking if each entry's scope matches the provided `scope`.
    - For each entry, determine if it should be processed based on the command entry type and the presence of specific flags in `args`.
    - Retrieve the option from `oo` using `options_get_only`; if not found and the 'A' flag is present, retrieve it using `options_get` and mark it as a parent option.
    - If the option is not an array, print it using [`cmd_show_options_print`](#cmd_show_options_print); if it is an array, iterate over its items using `options_array_first` and `options_array_next`, printing each item with its index.
    - Return `CMD_RETURN_NORMAL` to indicate successful execution.
- **Output**:
    - The function returns an `enum cmd_retval`, specifically `CMD_RETURN_NORMAL`, indicating successful execution.
- **Functions called**:
    - [`options_table_entry`](tmux.h.driver.md#options_table_entry)
    - [`cmd_show_options_print`](#cmd_show_options_print)


# Function Declarations (Public API)

---
### cmd_show_options_exec<!-- {{#callable_declaration:cmd_show_options_exec}} -->
Displays the current options for a specified target.
- **Description**: This function is used to display the current options for a specified target, such as a window or pane, in the tmux environment. It can be called with or without specifying a particular option. If no option is specified, it will display all options for the target. The function handles ambiguous or invalid options by returning an error unless the quiet flag is set. It must be called with a valid command and command queue item, and it respects various flags that modify its behavior, such as quiet mode or showing parent options.
- **Inputs**:
    - `self`: A pointer to the command structure representing the current command. Must not be null.
    - `item`: A pointer to the command queue item associated with the command execution. Must not be null.
- **Output**: Returns CMD_RETURN_NORMAL on success or CMD_RETURN_ERROR on failure, such as when an invalid option is specified without the quiet flag.
- **See also**: [`cmd_show_options_exec`](#cmd_show_options_exec)  (Implementation)


---
### cmd_show_options_print<!-- {{#callable_declaration:cmd_show_options_print}} -->
Prints the value of a specified option.
- **Description**: This function is used to display the value of a given option, which can be part of an array or a standalone option. It should be called when you need to output the current setting of an option, potentially with additional formatting based on verbosity or parent settings. The function handles both array and non-array options, printing each element of an array individually if necessary. It is important to ensure that the option entry provided is valid and that the command and item structures are properly initialized before calling this function.
- **Inputs**:
    - `self`: A pointer to a 'cmd' structure representing the command context. Must not be null.
    - `item`: A pointer to a 'cmdq_item' structure representing the command queue item. Must not be null.
    - `o`: A pointer to an 'options_entry' structure representing the option to be printed. Must not be null.
    - `idx`: An integer representing the index of the option if it is part of an array, or -1 if it is not an array.
    - `parent`: An integer flag indicating whether the option is inherited from a parent scope (non-zero) or not (zero).
- **Output**: None
- **See also**: [`cmd_show_options_print`](#cmd_show_options_print)  (Implementation)


---
### cmd_show_options_all<!-- {{#callable_declaration:cmd_show_options_all}} -->
Displays all options within a specified scope.
- **Description**: Use this function to display all options that fall within a specified scope, which can include global, session, or window options. It is typically called when you need to list all available options and their current values. The function requires a valid command structure and command queue item, and it operates within the context of a specified options scope. It handles both regular and array options, printing their values to the command queue. Ensure that the options structure provided is correctly initialized and corresponds to the desired scope.
- **Inputs**:
    - `self`: A pointer to the command structure representing the current command. Must not be null.
    - `item`: A pointer to the command queue item where output will be printed. Must not be null.
    - `scope`: An integer representing the scope of options to display. Valid values are determined by the options system and typically include global, session, or window scopes.
    - `oo`: A pointer to the options structure containing the options to be displayed. Must not be null and should be initialized to the correct scope.
- **Output**: Returns CMD_RETURN_NORMAL to indicate successful execution.
- **See also**: [`cmd_show_options_all`](#cmd_show_options_all)  (Implementation)


