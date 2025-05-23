# Purpose
The provided C source code file is part of the `tmux` terminal multiplexer project, specifically implementing the `if-shell` command functionality. This command allows users to execute a `tmux` command conditionally based on the success or failure of a shell command. The file defines the command's entry point, argument parsing, execution logic, and callback handling for asynchronous job execution. The `if-shell` command can be used to run a specified `tmux` command if a given shell command returns true (success) or optionally run a different command if the shell command returns false (failure).

Key components of the code include the [`cmd_if_shell_exec`](#cmd_if_shell_exec) function, which handles the execution of the `if-shell` command by preparing the shell command and determining whether to wait for its completion. The [`cmd_if_shell_callback`](#cmd_if_shell_callback) function processes the result of the shell command, deciding which `tmux` command to execute based on the shell command's exit status. The code also manages memory and resources associated with the command execution, such as freeing allocated data structures and handling client references. This file is a part of the broader `tmux` codebase, providing a specific feature that enhances the scripting capabilities within `tmux` sessions.
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
- **Description**: The `cmd_if_shell_entry` is a constant structure of type `cmd_entry` that defines the properties and behavior of the 'if-shell' command in the tmux application. It includes the command's name, alias, argument specifications, usage instructions, target specifications, flags, and the function to execute the command.
- **Use**: This variable is used to register and define the 'if-shell' command within the tmux command framework, allowing conditional execution of commands based on shell command results.


# Data Structures

---
### cmd_if_shell_data
- **Type**: `struct`
- **Members**:
    - `cmd_if`: Pointer to an args_command_state structure representing the command to execute if the shell command returns true.
    - `cmd_else`: Pointer to an args_command_state structure representing the command to execute if the shell command returns false.
    - `client`: Pointer to a client structure representing the client associated with the command.
    - `item`: Pointer to a cmdq_item structure representing the command queue item associated with the command.
- **Description**: The `cmd_if_shell_data` structure is used to store the state and context for executing a conditional command in the tmux environment. It holds pointers to the commands to be executed based on the result of a shell command, as well as references to the client and command queue item involved in the execution. This structure facilitates the execution flow of conditional commands by maintaining the necessary state and references required for command execution and callback handling.


# Functions

---
### cmd_if_shell_args_parse<!-- {{#callable:cmd_if_shell_args_parse}} -->
The `cmd_if_shell_args_parse` function determines the type of argument parsing required based on the index of the argument.
- **Inputs**:
    - `args`: A pointer to a struct of type `args`, which is not used in this function.
    - `idx`: An unsigned integer representing the index of the argument to be parsed.
    - `cause`: A pointer to a character pointer, which is not used in this function.
- **Control Flow**:
    - The function checks if the `idx` is either 1 or 2.
    - If `idx` is 1 or 2, the function returns `ARGS_PARSE_COMMANDS_OR_STRING`.
    - If `idx` is not 1 or 2, the function returns `ARGS_PARSE_STRING`.
- **Output**:
    - The function returns an `enum args_parse_type`, which indicates the type of parsing to be applied to the argument at the given index.


---
### cmd_if_shell_exec<!-- {{#callable:cmd_if_shell_exec}} -->
The `cmd_if_shell_exec` function executes a tmux command based on the result of a shell command, with options for synchronous or asynchronous execution.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command to be executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve the arguments and target client/session from the command and item structures.
    - Determine if the command should wait for the shell command to complete based on the presence of the 'b' flag.
    - Format the shell command string from the arguments and target.
    - If the 'F' flag is present, execute the shell command immediately and determine which tmux command to run based on the shell command's output.
    - If the 'F' flag is not present, prepare the command data structure for asynchronous execution and run the shell command as a job.
    - If the job fails to start, log an error and return an error status.
    - Free the shell command string after use.
    - Return the appropriate command return value based on whether the command is waiting for the shell command to complete.
- **Output**:
    - The function returns an `enum cmd_retval` indicating the result of the command execution, which can be `CMD_RETURN_NORMAL`, `CMD_RETURN_ERROR`, or `CMD_RETURN_WAIT`.


---
### cmd_if_shell_callback<!-- {{#callable:cmd_if_shell_callback}} -->
The `cmd_if_shell_callback` function processes the result of a shell command executed in a job and determines which tmux command list to execute based on the shell command's exit status.
- **Inputs**:
    - `job`: A pointer to a `job` structure representing the shell command execution whose result is being processed.
- **Control Flow**:
    - Retrieve the `cmd_if_shell_data` structure associated with the job to access the client, item, and command states.
    - Get the exit status of the shell command from the job.
    - Determine which command state (`cmd_if` or `cmd_else`) to use based on whether the shell command exited successfully.
    - If the chosen command state is NULL, exit the function.
    - Attempt to create a command list from the chosen command state; handle errors by displaying a message to the client or item if command list creation fails.
    - If a command list is successfully created, append or insert the new command item into the command queue based on whether the original item is NULL.
    - Continue processing the original command queue item if it exists.
- **Output**:
    - The function does not return a value but modifies the command queue by appending or inserting new command items based on the shell command's result.


---
### cmd_if_shell_free<!-- {{#callable:cmd_if_shell_free}} -->
The `cmd_if_shell_free` function deallocates resources associated with a `cmd_if_shell_data` structure, including client references and command states.
- **Inputs**:
    - `data`: A pointer to a `cmd_if_shell_data` structure that contains the resources to be freed.
- **Control Flow**:
    - Cast the `data` pointer to a `cmd_if_shell_data` structure pointer `cdata`.
    - Check if `cdata->client` is not NULL; if so, call `server_client_unref` to decrement the reference count of the client.
    - Check if `cdata->cmd_else` is not NULL; if so, call `args_make_commands_free` to free the command state for the 'else' command.
    - Call `args_make_commands_free` to free the command state for the 'if' command.
    - Free the memory allocated for the `cdata` structure itself.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the provided data.


# Function Declarations (Public API)

---
### cmd_if_shell_args_parse<!-- {{#callable_declaration:cmd_if_shell_args_parse}} -->
Determine the type of argument parsing required based on the argument index.
- **Description**: This function is used to decide how to parse command arguments based on their position in the argument list. It is typically called during the setup of command parsing to determine whether the argument at a given index should be treated as a command or a string. This function is particularly useful in scenarios where the command syntax allows for different types of arguments depending on their position. It should be used when setting up command parsing logic that needs to differentiate between command and string arguments.
- **Inputs**:
    - `args`: A pointer to a struct args, which represents the arguments to be parsed. This parameter is not used in the function and can be ignored.
    - `idx`: An unsigned integer representing the index of the argument to be parsed. Valid values are typically 1 or 2, which will result in different parsing types.
    - `cause`: A pointer to a char pointer, intended to store an error message if needed. This parameter is not used in the function and can be ignored.
- **Output**: Returns an enum value of type args_parse_type, indicating whether the argument should be parsed as a command or a string.
- **See also**: [`cmd_if_shell_args_parse`](#cmd_if_shell_args_parse)  (Implementation)


---
### cmd_if_shell_exec<!-- {{#callable_declaration:cmd_if_shell_exec}} -->
Execute a tmux command based on the result of a shell command.
- **Description**: This function is used to conditionally execute tmux commands depending on the result of a shell command. It is typically called within the tmux environment when a user wants to execute a command if a certain shell condition is met. The function supports options to run the shell command in the background and to interpret the shell command's output as a boolean. It requires a valid command structure and a command queue item, and it may execute additional commands based on the shell command's success or failure. The function handles both synchronous and asynchronous execution based on the provided arguments.
- **Inputs**:
    - `self`: A pointer to the cmd structure representing the command to be executed. Must not be null.
    - `item`: A pointer to the cmdq_item structure representing the command queue item. Must not be null.
- **Output**: Returns an enum cmd_retval indicating the result of the command execution, which can be CMD_RETURN_NORMAL, CMD_RETURN_ERROR, or CMD_RETURN_WAIT.
- **See also**: [`cmd_if_shell_exec`](#cmd_if_shell_exec)  (Implementation)


---
### cmd_if_shell_callback<!-- {{#callable_declaration:cmd_if_shell_callback}} -->
Execute a tmux command based on the result of a shell command.
- **Description**: This function is used to execute a tmux command conditionally, depending on the exit status of a shell command executed as a job. It is typically called as a callback when the shell command completes. If the shell command exits successfully, the 'if' command is executed; otherwise, the 'else' command is executed if provided. This function should be used in contexts where asynchronous job execution is required, and it handles the continuation of command queues appropriately.
- **Inputs**:
    - `job`: A pointer to a struct job representing the shell command execution. The job must have been previously created and must not be null. The function uses this to determine the exit status of the shell command and to retrieve associated data.
- **Output**: None
- **See also**: [`cmd_if_shell_callback`](#cmd_if_shell_callback)  (Implementation)


---
### cmd_if_shell_free<!-- {{#callable_declaration:cmd_if_shell_free}} -->
Releases resources associated with a shell command execution.
- **Description**: This function is used to free all resources allocated for executing a shell command within the context of a tmux command. It should be called when the command execution is complete or no longer needed. The function ensures that any associated client references and command states are properly released, preventing memory leaks. It is important to ensure that the data parameter is valid and points to a properly initialized cmd_if_shell_data structure before calling this function.
- **Inputs**:
    - `data`: A pointer to a cmd_if_shell_data structure containing resources to be freed. Must not be null and should point to a valid structure. The function will handle null checks for internal pointers within the structure.
- **Output**: None
- **See also**: [`cmd_if_shell_free`](#cmd_if_shell_free)  (Implementation)


