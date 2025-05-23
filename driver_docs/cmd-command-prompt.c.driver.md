# Purpose
This C source code file is part of the tmux terminal multiplexer project and implements the functionality for a command prompt within a tmux client. The primary purpose of this code is to handle user input prompts, allowing users to input commands or data interactively. The file defines a command entry for "command-prompt," which includes parsing arguments, executing the command, and managing the lifecycle of the prompt interaction. The code is structured around several static functions that handle argument parsing, command execution, callback processing for user input, and cleanup of allocated resources.

The key components of this file include the `cmd_command_prompt_entry` structure, which defines the command's name, usage, and execution logic, and the `cmd_command_prompt_cdata` structure, which holds the state and data necessary for managing multiple prompts. The code supports various prompt types and flags, such as single input, numeric input, and incremental input, which are specified through command-line arguments. The file does not define a public API but rather integrates into the larger tmux application, providing a specific feature for user interaction within the tmux environment.
# Imports and Dependencies

---
- `sys/types.h`
- `ctype.h`
- `stdlib.h`
- `string.h`
- `time.h`
- `tmux.h`


# Global Variables

---
### cmd_command_prompt_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_command_prompt_entry` is a constant structure of type `cmd_entry` that defines the properties and behavior of the 'command-prompt' command in the application. It includes the command's name, alias, argument specifications, usage instructions, flags, and the function to execute the command.
- **Use**: This variable is used to register and define the 'command-prompt' command, specifying how it should be parsed, executed, and displayed to the user.


# Data Structures

---
### cmd_command_prompt_prompt
- **Type**: `struct`
- **Members**:
    - `input`: A pointer to a character array representing the input string.
    - `prompt`: A pointer to a character array representing the prompt string.
- **Description**: The `cmd_command_prompt_prompt` structure is used to store a pair of strings: one for the input and one for the prompt. This structure is part of a command-prompt mechanism in a client application, where it holds the current input and prompt strings that are displayed to the user. It is typically used in conjunction with other structures and functions to manage user input and command execution in a terminal-based interface.


---
### cmd_command_prompt_cdata
- **Type**: `struct`
- **Members**:
    - `item`: A pointer to a cmdq_item structure, representing a command queue item.
    - `state`: A pointer to an args_command_state structure, representing the state of command arguments.
    - `flags`: An integer representing various flags for the command prompt.
    - `prompt_type`: An enum value representing the type of prompt.
    - `prompts`: A pointer to an array of cmd_command_prompt_prompt structures, each containing a prompt and its input.
    - `count`: An unsigned integer representing the number of prompts.
    - `current`: An unsigned integer representing the current prompt index.
    - `argc`: An integer representing the number of arguments.
    - `argv`: A pointer to an array of strings representing the arguments.
- **Description**: The `cmd_command_prompt_cdata` structure is used to manage the state and data associated with a command prompt in a client. It holds information about the command queue item, the state of command arguments, and various flags that modify the behavior of the prompt. It also contains an array of prompts, each with its own input, and tracks the number of prompts and the current prompt being processed. Additionally, it manages the arguments passed to the command prompt, storing them in an array of strings.


# Functions

---
### cmd_command_prompt_args_parse<!-- {{#callable:cmd_command_prompt_args_parse}} -->
The function `cmd_command_prompt_args_parse` returns a constant value indicating the type of argument parsing to be used for command prompts.
- **Inputs**:
    - `args`: A pointer to a struct of type `args`, which is not used in this function.
    - `idx`: An unsigned integer representing the index, which is not used in this function.
    - `cause`: A pointer to a character pointer for error messages, which is not used in this function.
- **Control Flow**:
    - The function immediately returns the constant `ARGS_PARSE_COMMANDS_OR_STRING`.
- **Output**:
    - The function returns an enum value of type `args_parse_type`, specifically `ARGS_PARSE_COMMANDS_OR_STRING`.


---
### cmd_command_prompt_exec<!-- {{#callable:cmd_command_prompt_exec}} -->
The `cmd_command_prompt_exec` function sets up and executes a command prompt in a client, handling user input and command execution based on provided arguments.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item associated with this execution.
- **Control Flow**:
    - Retrieve the arguments for the command using [`cmd_get_args`](cmd.c.driver.md#cmd_get_args) and the target client using [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client).
    - Check if the target client already has a prompt string; if so, return `CMD_RETURN_NORMAL`.
    - Determine if the command should wait for user input based on the presence of the 'b' and 'i' flags in the arguments.
    - Allocate and initialize a `cmd_command_prompt_cdata` structure to store prompt data and state information.
    - Prepare the command state using [`args_make_commands_prepare`](arguments.c.driver.md#args_make_commands_prepare), considering the 'F' flag for formatting.
    - Determine the prompt string(s) from the 'p' argument or default to a generated or default prompt if not provided.
    - Determine initial input values from the 'I' argument if provided.
    - Iterate over the prompts, separating them by commas, and store each prompt and its corresponding input in the `cdata` structure.
    - Free temporary memory allocations for inputs and prompts.
    - Determine the prompt type from the 'T' argument, defaulting to `PROMPT_TYPE_COMMAND` if not specified, and handle invalid types by returning an error.
    - Set flags in `cdata` based on the presence of '1', 'N', 'i', and 'k' flags in the arguments.
    - Set the status prompt in the target client using [`status_prompt_set`](status.c.driver.md#status_prompt_set), passing the first prompt and input, along with callback and free functions.
    - Return `CMD_RETURN_NORMAL` if not waiting for input, otherwise return `CMD_RETURN_WAIT`.
- **Output**:
    - Returns an `enum cmd_retval` indicating the result of the command execution, either `CMD_RETURN_NORMAL` or `CMD_RETURN_WAIT`.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client)
    - [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target)
    - [`args_count`](arguments.c.driver.md#args_count)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`args_make_commands_prepare`](arguments.c.driver.md#args_make_commands_prepare)
    - [`args_get`](arguments.c.driver.md#args_get)
    - [`args_make_commands_get_command`](arguments.c.driver.md#args_make_commands_get_command)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`strsep`](compat/strsep.c.driver.md#strsep)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`status_prompt_type`](status.c.driver.md#status_prompt_type)
    - [`cmd_command_prompt_free`](#cmd_command_prompt_free)
    - [`status_prompt_set`](status.c.driver.md#status_prompt_set)


---
### cmd_command_prompt_callback<!-- {{#callable:cmd_command_prompt_callback}} -->
The `cmd_command_prompt_callback` function handles the input from a command prompt in a client, processing the input and executing commands based on the input and prompt state.
- **Inputs**:
    - `c`: A pointer to the client structure, representing the client that initiated the command prompt.
    - `data`: A pointer to the command prompt callback data structure (`cmd_command_prompt_cdata`), which contains information about the current prompt state and arguments.
    - `s`: A string representing the input received from the command prompt.
    - `done`: An integer flag indicating whether the prompt input is complete (non-zero) or still ongoing (zero).
- **Control Flow**:
    - Check if the input string `s` is NULL; if so, jump to the cleanup section labeled `out`.
    - If the input is complete (`done` is non-zero), check if the prompt is incremental; if so, jump to the cleanup section.
    - Append the input string `s` to the argument list in `cdata` if the prompt is not incremental.
    - If the current prompt is not the last one, update the prompt to the next one and return 1 to indicate more input is expected.
    - Copy the current argument list from `cdata` to a local variable `argv` and update it with the new input if not done.
    - If the input is complete, free the old argument list in `cdata` and replace it with the updated list.
    - Create a command list from the arguments using [`args_make_commands`](arguments.c.driver.md#args_make_commands); if successful, append or insert the command into the command queue, otherwise append an error message.
    - Free the local copy of the argument list.
    - If the client's prompt input callback is not this function, return 1 to indicate the prompt is still active.
    - In the cleanup section, if there is an associated command queue item, continue processing it.
- **Output**:
    - Returns 1 if more input is expected or the prompt is still active, otherwise returns 0 to indicate completion.
- **Functions called**:
    - [`cmd_append_argv`](cmd.c.driver.md#cmd_append_argv)
    - [`status_prompt_update`](status.c.driver.md#status_prompt_update)
    - [`cmd_copy_argv`](cmd.c.driver.md#cmd_copy_argv)
    - [`cmd_free_argv`](cmd.c.driver.md#cmd_free_argv)
    - [`args_make_commands`](arguments.c.driver.md#args_make_commands)
    - [`cmdq_append`](cmd-queue.c.driver.md#cmdq_append)
    - [`cmdq_get_error`](cmd-queue.c.driver.md#cmdq_get_error)
    - [`cmdq_get_command`](cmd-queue.c.driver.md#cmdq_get_command)
    - [`cmdq_get_state`](cmd-queue.c.driver.md#cmdq_get_state)
    - [`cmdq_insert_after`](cmd-queue.c.driver.md#cmdq_insert_after)
    - [`cmdq_continue`](cmd-queue.c.driver.md#cmdq_continue)


---
### cmd_command_prompt_free<!-- {{#callable:cmd_command_prompt_free}} -->
The `cmd_command_prompt_free` function deallocates memory associated with a `cmd_command_prompt_cdata` structure, including its prompts, arguments, and state.
- **Inputs**:
    - `data`: A pointer to a `cmd_command_prompt_cdata` structure that contains data to be freed.
- **Control Flow**:
    - Cast the input `data` to a `cmd_command_prompt_cdata` pointer named `cdata`.
    - Iterate over each prompt in `cdata->prompts` using a loop from 0 to `cdata->count`.
    - For each prompt, free the memory allocated for `prompt` and `input` strings.
    - Free the memory allocated for the `prompts` array itself.
    - Call [`cmd_free_argv`](cmd.c.driver.md#cmd_free_argv) to free the argument vector (`argv`) and its count (`argc`).
    - Call [`args_make_commands_free`](arguments.c.driver.md#args_make_commands_free) to free the command state (`state`).
    - Finally, free the `cdata` structure itself.
- **Output**:
    - The function does not return any value; it performs memory deallocation for cleanup purposes.
- **Functions called**:
    - [`cmd_free_argv`](cmd.c.driver.md#cmd_free_argv)
    - [`args_make_commands_free`](arguments.c.driver.md#args_make_commands_free)


