# Purpose
This C source code file is part of the `tmux` project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file implements the functionality for a command prompt within a `tmux` client, allowing users to input commands interactively. The primary component of this file is the `cmd_command_prompt_entry` structure, which defines the command prompt's name, usage, and execution behavior. The command prompt can be customized with various flags and options, such as specifying input prompts, prompt types, and whether the prompt should wait for user input.

The file includes several static functions that handle parsing arguments, executing the command prompt, and managing the lifecycle of the prompt data. The [`cmd_command_prompt_exec`](#cmd_command_prompt_exec) function is responsible for setting up the prompt, including handling user inputs and managing the prompt's state. The [`cmd_command_prompt_callback`](#cmd_command_prompt_callback) function processes user input and executes the corresponding commands, while [`cmd_command_prompt_free`](#cmd_command_prompt_free) cleans up allocated resources. This code provides a focused functionality within the broader `tmux` application, specifically enabling interactive command input and execution in a client session.
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
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_command_prompt_entry` is a constant structure of type `cmd_entry` that defines the properties and behavior of the 'command-prompt' command in the application. It includes the command's name, alias, argument specifications, usage instructions, flags, and the function to execute the command.
- **Use**: This variable is used to register and define the 'command-prompt' command within the application, specifying how it should be parsed and executed.


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
The function `cmd_command_prompt_args_parse` determines the type of argument parsing to be used for the command prompt, returning a constant value indicating commands or string parsing.
- **Inputs**:
    - `args`: A pointer to a struct of type `args`, which is not used in this function.
    - `idx`: An unsigned integer representing the index, which is not used in this function.
    - `cause`: A pointer to a character pointer for error messages, which is not used in this function.
- **Control Flow**:
    - The function is defined as static, meaning it is limited to the file scope.
    - The function takes three parameters, all marked as unused, indicating they are not utilized within the function body.
    - The function immediately returns a constant value `ARGS_PARSE_COMMANDS_OR_STRING`.
- **Output**:
    - The function returns an enum value of type `args_parse_type`, specifically `ARGS_PARSE_COMMANDS_OR_STRING`, indicating the parsing strategy for command arguments.


---
### cmd_command_prompt_exec<!-- {{#callable:cmd_command_prompt_exec}} -->
The `cmd_command_prompt_exec` function sets up and executes a command prompt in a client, handling various prompt configurations and input options.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve the command arguments and target client from the provided structures.
    - Check if the target client already has a prompt string; if so, return `CMD_RETURN_NORMAL`.
    - Determine if the command should wait for input based on the presence of certain arguments.
    - Allocate and initialize a `cmd_command_prompt_cdata` structure to store prompt data.
    - Prepare the command state for execution using `args_make_commands_prepare`.
    - Determine the prompt string(s) and initial input(s) based on the provided arguments.
    - Iterate over the prompt strings, storing each prompt and its corresponding input in the `cdata` structure.
    - Free any temporary memory allocations for inputs and prompts.
    - Determine the prompt type from the arguments, defaulting to `PROMPT_TYPE_COMMAND` if not specified.
    - Set flags for the prompt based on the presence of specific arguments.
    - Set the status prompt for the target client using `status_prompt_set`.
    - Return `CMD_RETURN_NORMAL` if not waiting for input, otherwise return `CMD_RETURN_WAIT`.
- **Output**:
    - The function returns an `enum cmd_retval`, which can be `CMD_RETURN_NORMAL` if the prompt is set without waiting, or `CMD_RETURN_WAIT` if the command should wait for input.
- **Functions called**:
    - [`cmd_command_prompt_free`](#cmd_command_prompt_free)


---
### cmd_command_prompt_callback<!-- {{#callable:cmd_command_prompt_callback}} -->
The `cmd_command_prompt_callback` function handles the completion of a command prompt interaction in a client, processing user input and executing commands accordingly.
- **Inputs**:
    - `c`: A pointer to the client structure, representing the client interacting with the command prompt.
    - `data`: A pointer to the command prompt callback data structure (`cmd_command_prompt_cdata`), which contains information about the current command prompt session.
    - `s`: A string representing the user input received from the command prompt.
    - `done`: An integer flag indicating whether the command prompt interaction is complete (non-zero) or still ongoing (zero).
- **Control Flow**:
    - Check if the input string `s` is NULL; if so, jump to the cleanup section labeled 'out'.
    - If the interaction is marked as done, check if the prompt is incremental; if so, jump to the cleanup section.
    - Append the input string `s` to the argument list in `cdata` if the interaction is done.
    - If there are more prompts to process, update the prompt with the next one and return 1 to indicate continuation.
    - Copy the current argument list from `cdata` to a local variable `argv` and append `s` if the interaction is not done.
    - If the interaction is done, free the old argument list in `cdata` and update it with the new one.
    - Create a command list from the arguments; if successful, append or insert the command into the command queue, otherwise append an error message.
    - Free the local argument list `argv`.
    - If the client's prompt input callback is not this function, return 1 to indicate continuation.
    - In the cleanup section, if there is an associated command queue item, continue processing it.
- **Output**:
    - Returns 1 if the prompt interaction should continue, or 0 if it is complete and cleanup is performed.


---
### cmd_command_prompt_free<!-- {{#callable:cmd_command_prompt_free}} -->
The `cmd_command_prompt_free` function deallocates memory associated with a `cmd_command_prompt_cdata` structure, including its prompts, arguments, and state.
- **Inputs**:
    - `data`: A pointer to a `cmd_command_prompt_cdata` structure that contains the data to be freed.
- **Control Flow**:
    - Cast the `data` pointer to a `cmd_command_prompt_cdata` pointer named `cdata`.
    - Iterate over each prompt in `cdata->prompts` using a loop that runs from 0 to `cdata->count`.
    - For each prompt, free the memory allocated for `prompt` and `input` strings.
    - Free the memory allocated for the `prompts` array itself.
    - Call `cmd_free_argv` to free the argument vector (`argv`) and its count (`argc`).
    - Call `args_make_commands_free` to free the command state (`state`).
    - Finally, free the `cdata` structure itself.
- **Output**:
    - The function does not return any value; it performs memory deallocation to prevent memory leaks.


# Function Declarations (Public API)

---
### cmd_command_prompt_args_parse<!-- {{#callable_declaration:cmd_command_prompt_args_parse}} -->
Parse command prompt arguments.
- **Description**: This function is used to parse arguments for a command prompt within a client. It determines the type of parsing to be applied to the arguments, which can be either commands or strings. This function is typically called as part of the command entry setup and is expected to handle argument parsing consistently. It does not perform any operations on the arguments themselves but returns a constant indicating the parsing type.
- **Inputs**:
    - `args`: A pointer to a struct args, representing the arguments to be parsed. This parameter is not used in the function and can be null.
    - `idx`: An unsigned integer representing the index of the argument to parse. This parameter is not used in the function.
    - `cause`: A pointer to a char pointer for storing error messages. This parameter is not used in the function and can be null.
- **Output**: Returns an enum value ARGS_PARSE_COMMANDS_OR_STRING, indicating the type of parsing to be applied.
- **See also**: [`cmd_command_prompt_args_parse`](#cmd_command_prompt_args_parse)  (Implementation)


---
### cmd_command_prompt_exec<!-- {{#callable_declaration:cmd_command_prompt_exec}} -->
Prompts the user for command input in a client.
- **Description**: This function is used to prompt the user for command input within a client session. It sets up the necessary prompts and inputs based on the provided arguments and manages the interaction with the user. The function should be called when a command prompt is needed, and it handles various prompt types and input configurations. It requires a valid client and command queue item, and it manages the prompt lifecycle, including handling user input and executing commands based on that input. The function returns immediately or waits for user input based on the provided arguments.
- **Inputs**:
    - `self`: A pointer to the command structure. Must not be null. The caller retains ownership.
    - `item`: A pointer to the command queue item. Must not be null. The caller retains ownership.
- **Output**: Returns CMD_RETURN_NORMAL if the prompt is set successfully and does not wait for input, CMD_RETURN_WAIT if it waits for user input, or CMD_RETURN_ERROR if an error occurs (e.g., invalid prompt type).
- **See also**: [`cmd_command_prompt_exec`](#cmd_command_prompt_exec)  (Implementation)


---
### cmd_command_prompt_callback<!-- {{#callable_declaration:cmd_command_prompt_callback}} -->
Handle command prompt input and execute commands.
- **Description**: This function is used to process input from a command prompt in a client context, handling both incremental and final input submissions. It is typically called as a callback when a user provides input to a prompt. The function manages multiple prompts, updating the prompt as needed, and executes commands once all input is received. It is important to ensure that the client and data parameters are correctly initialized and that the function is used in the context of a command prompt session.
- **Inputs**:
    - `c`: A pointer to a client structure representing the client session. Must not be null. The function uses this to manage prompt updates and command execution.
    - `data`: A pointer to user-defined data, expected to be of type `struct cmd_command_prompt_cdata`. This contains the state and context for the command prompt session. Must not be null.
    - `s`: A string representing the current input from the user. Can be null, in which case the function will handle it as an exit condition.
    - `done`: An integer flag indicating whether the input session is complete (non-zero) or ongoing (zero). Determines if the function should finalize input processing and execute commands.
- **Output**: Returns an integer indicating whether the prompt session should continue (1) or is complete (0).
- **See also**: [`cmd_command_prompt_callback`](#cmd_command_prompt_callback)  (Implementation)


---
### cmd_command_prompt_free<!-- {{#callable_declaration:cmd_command_prompt_free}} -->
Frees resources associated with command prompt data.
- **Description**: This function is used to release all dynamically allocated memory associated with a command prompt data structure. It should be called when the command prompt data is no longer needed to prevent memory leaks. The function assumes that the provided data pointer is valid and points to a properly initialized command prompt data structure. It is typically used as a cleanup callback after command prompt operations are completed.
- **Inputs**:
    - `data`: A pointer to the command prompt data structure to be freed. Must not be null and should point to a valid, initialized structure. The function will handle any internal null checks and free operations.
- **Output**: None
- **See also**: [`cmd_command_prompt_free`](#cmd_command_prompt_free)  (Implementation)


