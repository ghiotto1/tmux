# Purpose
The provided C source code file is part of the `tmux` terminal multiplexer project and implements the `if-shell` command functionality. This command allows users to execute a `tmux` command conditionally based on the success or failure of a shell command. The file defines the command's entry point, argument parsing, execution logic, and associated callback functions. The `if-shell` command can be used to run a specified `tmux` command if a given shell command returns true (success) or optionally run a different command if the shell command returns false (failure).

Key components of the code include the [`cmd_if_shell_exec`](#cmd_if_shell_exec) function, which handles the execution of the `if-shell` command, and the [`cmd_if_shell_callback`](#cmd_if_shell_callback) function, which processes the result of the shell command execution. The code also defines a structure, `cmd_if_shell_data`, to manage the state and data associated with the command execution. The command supports options such as running in the background (`-b`) and interpreting the shell command's output as a boolean (`-F`). The file is designed to be part of the `tmux` command execution framework, integrating with other components like job management and command queues, and does not define a standalone executable or public API.
# Imports and Dependencies

---
- `sys/types.h`
- `sys/wait.h`
- `ctype.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### cmd_if_shell_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_if_shell_entry` is a constant structure of type `cmd_entry` that defines the properties and behavior of the 'if-shell' command in the tmux application. It includes the command's name, alias, argument specifications, usage instructions, target specifications, flags, and the function to execute the command. This structure is used to register and handle the 'if-shell' command, which conditionally executes tmux commands based on the result of a shell command.
- **Use**: This variable is used to define and register the 'if-shell' command within the tmux command framework.


# Data Structures

---
### cmd_if_shell_data
- **Type**: `struct`
- **Members**:
    - `cmd_if`: Pointer to an args_command_state structure representing the command to execute if the shell command returns true.
    - `cmd_else`: Pointer to an args_command_state structure representing the command to execute if the shell command returns false.
    - `client`: Pointer to a client structure representing the client associated with the command.
    - `item`: Pointer to a cmdq_item structure representing the command queue item associated with the command.
- **Description**: The `cmd_if_shell_data` structure is used to store the state and context for executing a conditional command in the tmux environment. It holds pointers to the commands to be executed based on the result of a shell command, as well as references to the client and command queue item involved in the execution. This structure is integral to the `if-shell` command functionality, allowing tmux to conditionally execute commands based on the success or failure of a shell command.


# Functions

---
### cmd_if_shell_args_parse<!-- {{#callable:cmd_if_shell_args_parse}} -->
The function `cmd_if_shell_args_parse` determines the type of argument parsing required based on the index of the argument.
- **Inputs**:
    - `args`: A pointer to a struct of type `args`, which is not used in this function.
    - `idx`: An unsigned integer representing the index of the argument to be parsed.
    - `cause`: A pointer to a character pointer, which is not used in this function.
- **Control Flow**:
    - The function checks if the `idx` is either 1 or 2.
    - If `idx` is 1 or 2, it returns `ARGS_PARSE_COMMANDS_OR_STRING`.
    - If `idx` is not 1 or 2, it returns `ARGS_PARSE_STRING`.
- **Output**:
    - The function returns an `enum args_parse_type` which indicates the type of parsing to be applied to the argument at the given index.


---
### cmd_if_shell_exec<!-- {{#callable:cmd_if_shell_exec}} -->
The `cmd_if_shell_exec` function executes a tmux command based on the result of a shell command, with options for synchronous or asynchronous execution.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command to be executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve arguments and target information from the command and command queue item.
    - Format the shell command string from the target and arguments.
    - Check if the 'F' flag is set; if so, execute commands immediately based on the shell command's result and return.
    - Allocate memory for `cmd_if_shell_data` structure to store command states and client information.
    - Prepare the 'if' and 'else' command states based on the number of arguments.
    - Determine whether to wait for the shell command to complete based on the 'b' flag.
    - Run the shell command using [`job_run`](job.c.driver.md#job_run), passing the necessary callbacks and data.
    - If the shell command fails to run, log an error and return an error status.
    - Free the shell command string after use.
    - Return a wait status if waiting for the shell command to complete, otherwise return normal status.
- **Output**:
    - The function returns an `enum cmd_retval` indicating the result of the command execution, which can be `CMD_RETURN_NORMAL`, `CMD_RETURN_ERROR`, or `CMD_RETURN_WAIT`.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target)
    - [`cmdq_get_target_client`](cmd-queue.c.driver.md#cmdq_get_target_client)
    - [`args_count`](arguments.c.driver.md#args_count)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`format_single_from_target`](format.c.driver.md#format_single_from_target)
    - [`args_string`](arguments.c.driver.md#args_string)
    - [`args_make_commands_now`](arguments.c.driver.md#args_make_commands_now)
    - [`cmdq_get_command`](cmd-queue.c.driver.md#cmdq_get_command)
    - [`cmdq_get_state`](cmd-queue.c.driver.md#cmdq_get_state)
    - [`cmdq_insert_after`](cmd-queue.c.driver.md#cmdq_insert_after)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`args_make_commands_prepare`](arguments.c.driver.md#args_make_commands_prepare)
    - [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client)
    - [`job_run`](job.c.driver.md#job_run)
    - [`server_client_get_cwd`](server-client.c.driver.md#server_client_get_cwd)


---
### cmd_if_shell_callback<!-- {{#callable:cmd_if_shell_callback}} -->
The `cmd_if_shell_callback` function processes the result of a shell command executed in a job and determines which tmux command list to execute based on the shell command's exit status.
- **Inputs**:
    - `job`: A pointer to a `job` structure representing the shell command execution whose result is being processed.
- **Control Flow**:
    - Retrieve the `cmd_if_shell_data` associated with the job to access the client, item, and command states.
    - Get the exit status of the shell command from the job.
    - Determine which command state (`cmd_if` or `cmd_else`) to use based on the exit status: if the command exited successfully, use `cmd_if`; otherwise, use `cmd_else`.
    - If the chosen command state is NULL, exit the function.
    - Attempt to create a command list from the chosen command state using [`args_make_commands`](arguments.c.driver.md#args_make_commands).
    - If command list creation fails, report the error to the client or item, and free the error message.
    - If the command list is successfully created, either append or insert the new command item into the command queue, depending on whether the original item is NULL.
    - If the original item is not NULL, continue processing the command queue from that item.
- **Output**:
    - The function does not return a value but modifies the command queue based on the shell command's exit status and the associated command states.
- **Functions called**:
    - [`job_get_data`](job.c.driver.md#job_get_data)
    - [`job_get_status`](job.c.driver.md#job_get_status)
    - [`args_make_commands`](arguments.c.driver.md#args_make_commands)
    - [`cmdq_get_command`](cmd-queue.c.driver.md#cmdq_get_command)
    - [`cmdq_append`](cmd-queue.c.driver.md#cmdq_append)
    - [`cmdq_get_state`](cmd-queue.c.driver.md#cmdq_get_state)
    - [`cmdq_insert_after`](cmd-queue.c.driver.md#cmdq_insert_after)
    - [`cmdq_continue`](cmd-queue.c.driver.md#cmdq_continue)


---
### cmd_if_shell_free<!-- {{#callable:cmd_if_shell_free}} -->
The `cmd_if_shell_free` function releases resources associated with a `cmd_if_shell_data` structure, including client references and command states.
- **Inputs**:
    - `data`: A pointer to a `cmd_if_shell_data` structure that contains the resources to be freed.
- **Control Flow**:
    - Cast the input `data` to a `cmd_if_shell_data` structure pointer `cdata`.
    - Check if `cdata->client` is not NULL; if so, call [`server_client_unref`](server-client.c.driver.md#server_client_unref) to decrement the reference count of the client.
    - Check if `cdata->cmd_else` is not NULL; if so, call [`args_make_commands_free`](arguments.c.driver.md#args_make_commands_free) to free the command state for the 'else' command.
    - Call [`args_make_commands_free`](arguments.c.driver.md#args_make_commands_free) to free the command state for the 'if' command.
    - Free the memory allocated for `cdata`.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the provided data.
- **Functions called**:
    - [`server_client_unref`](server-client.c.driver.md#server_client_unref)
    - [`args_make_commands_free`](arguments.c.driver.md#args_make_commands_free)


