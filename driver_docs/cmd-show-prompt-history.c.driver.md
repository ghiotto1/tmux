# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it provides functionality for managing prompt history within the application. The file defines two command entries: `show-prompt-history` and `clear-prompt-history`, which are used to display and clear the history of prompts, respectively. These commands are registered with specific names and aliases, and they share a common execution function, [`cmd_show_prompt_history_exec`](#cmd_show_prompt_history_exec), which handles both showing and clearing the prompt history based on the command invoked. The commands can optionally take a prompt type as an argument, allowing users to target specific types of prompts.

The code is structured to handle both the display and clearing of prompt history by iterating over the available prompt types and their respective histories. It uses a set of predefined prompt types and manages their histories through arrays that store the history entries and their sizes. The execution function checks for the validity of the prompt type and performs the appropriate action, either printing the history entries or clearing them. This file is a focused component of the tmux codebase, providing a specific utility for managing prompt histories, and it is intended to be integrated into the larger tmux application rather than serving as a standalone executable.
# Imports and Dependencies

---
- `tmux.h`
- `stdlib.h`


# Global Variables

---
### cmd_show_prompt_history_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_show_prompt_history_entry` is a constant structure of type `cmd_entry` that defines a command for showing the prompt history in the application. It includes fields such as the command name, alias, arguments, usage, flags, and the execution function associated with the command.
- **Use**: This variable is used to register and define the behavior of the 'show-prompt-history' command within the application, allowing users to view the history of prompts.


---
### cmd_clear_prompt_history_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_clear_prompt_history_entry` is a constant structure of type `cmd_entry` that defines a command for clearing the prompt history in the application. It specifies the command name as "clear-prompt-history" with an alias "clearphist", and it accepts an optional argument for specifying the prompt type to clear. The command is executed using the `cmd_show_prompt_history_exec` function.
- **Use**: This variable is used to define and register the command for clearing prompt history, allowing users to clear all or specific types of prompt history entries.


# Functions

---
### cmd_show_prompt_history_exec<!-- {{#callable:cmd_show_prompt_history_exec}} -->
The `cmd_show_prompt_history_exec` function either displays or clears the prompt history based on the command entry and optional type argument.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve the arguments from the command using `cmd_get_args` and get the type string using `args_get`.
    - Check if the command entry is `cmd_clear_prompt_history_entry`; if true, proceed to clear the history.
    - If `typestr` is NULL, iterate over all prompt types and clear their histories by freeing memory and setting sizes to zero.
    - If `typestr` is not NULL, convert it to a prompt type using `status_prompt_type`; if invalid, return an error.
    - If the command is not `cmd_clear_prompt_history_entry`, display the history.
    - If `typestr` is NULL, iterate over all prompt types and print their histories.
    - If `typestr` is not NULL, convert it to a prompt type and print the history for that specific type.
    - Return `CMD_RETURN_NORMAL` after processing.
- **Output**:
    - The function returns an `enum cmd_retval`, which is either `CMD_RETURN_NORMAL` for successful execution or `CMD_RETURN_ERROR` if an invalid type is specified.


# Function Declarations (Public API)

---
### cmd_show_prompt_history_exec<!-- {{#callable_declaration:cmd_show_prompt_history_exec}} -->
Show or clear prompt history based on command entry.
- **Description**: This function is used to either display or clear the history of prompts, depending on the command entry it is associated with. It can handle different types of prompts specified by the '-T' option. If no type is specified, it operates on all prompt types. When used with the 'show-prompt-history' command, it prints the history of prompts to the command queue item. When used with the 'clear-prompt-history' command, it clears the history. The function handles invalid prompt types by returning an error.
- **Inputs**:
    - `self`: A pointer to the command structure representing the current command. Must not be null. The function uses this to determine the command entry and associated arguments.
    - `item`: A pointer to the command queue item where output messages are printed. Must not be null. The function uses this to print history or error messages.
- **Output**: Returns an enum cmd_retval indicating success (CMD_RETURN_NORMAL) or error (CMD_RETURN_ERROR) based on the validity of the prompt type.
- **See also**: [`cmd_show_prompt_history_exec`](#cmd_show_prompt_history_exec)  (Implementation)


