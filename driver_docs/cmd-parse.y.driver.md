# Purpose
This source code file is part of the `tmux` project, specifically handling the parsing of command input. It is a parser implementation that uses a combination of C and Yacc/Bison syntax to define the grammar and parsing logic for commands. The file is responsible for interpreting command strings, handling conditional logic, and managing command arguments and their execution. It defines a set of tokens and grammar rules to parse command lines, which can include conditions, assignments, and command sequences.

The file includes several key components: structures to represent parsing states, commands, and arguments; functions to manage these structures; and a lexer to tokenize input strings. The lexer (`yylex`) processes input to identify tokens such as commands, conditions, and special characters, while the parser (`yyparse`) uses these tokens to build a structured representation of the commands. The code also includes functions to handle command execution, error reporting, and alias expansion, allowing for flexible command processing.

Overall, this file provides a comprehensive framework for parsing and executing commands within the `tmux` environment. It supports complex command structures, including nested commands and conditional execution, making it a critical component for interpreting user input and automating tasks in `tmux`.
# Imports and Dependencies

---
- `sys/types.h`
- `ctype.h`
- `errno.h`
- `pwd.h`
- `stdlib.h`
- `string.h`
- `unistd.h`
- `wchar.h`
- `tmux.h`


# Global Variables

---
### parse_state
- **Type**: `struct cmd_parse_state`
- **Description**: The `parse_state` variable is a static instance of the `cmd_parse_state` structure, which is used to maintain the state of the command parsing process. This structure includes fields for managing input sources, tracking parsing progress, handling errors, and storing parsed commands and scopes.
- **Use**: This variable is used throughout the parsing functions to keep track of the current state and context of the command parsing operation.


# Data Structures

---
### cmd_parse_scope
- **Type**: `struct`
- **Members**:
    - `flag`: An integer flag used to indicate the state or condition of the scope.
    - `entry`: A TAILQ_ENTRY that links cmd_parse_scope structures in a queue.
- **Description**: The `cmd_parse_scope` structure is a component of a command parsing system, used to manage and track the state of command parsing scopes. It contains a flag to indicate the current state or condition of the scope and a TAILQ_ENTRY to allow it to be part of a linked list, enabling the management of multiple parsing scopes in a queue. This structure is essential for handling conditional parsing logic, such as if-else statements, within the command parsing process.


---
### cmd_parse_argument
- **Type**: `struct`
- **Members**:
    - `type`: Specifies the type of the argument, which can be a string, commands, or parsed commands.
    - `string`: Holds a string value if the argument type is CMD_PARSE_STRING.
    - `commands`: Points to a list of commands if the argument type is CMD_PARSE_COMMANDS.
    - `cmdlist`: Points to a list of parsed commands if the argument type is CMD_PARSE_PARSED_COMMANDS.
    - `entry`: Links the argument into a tail queue for ordered processing.
- **Description**: The `cmd_parse_argument` structure is designed to represent an argument in a command parsing context, supporting different types of arguments such as strings, command lists, and parsed command lists. It uses a union-like approach to store different types of data based on the argument type, and it is linked into a queue for sequential processing. This structure is part of a larger command parsing system, likely used in a shell or scripting environment, to handle complex command inputs and their associated arguments.


---
### cmd_parse_arguments
- **Type**: `TAILQ_HEAD`
- **Members**:
    - `entry`: A linked list entry for cmd_parse_argument.
- **Description**: The `cmd_parse_arguments` is a linked list data structure that holds a collection of `cmd_parse_argument` elements. Each element in the list represents an argument parsed from a command, which can be a string, a set of commands, or a list of parsed commands. This structure is used to manage and iterate over the arguments associated with a command in the parsing process.


---
### cmd_parse_command
- **Type**: `struct`
- **Members**:
    - `line`: Stores the line number of the command in the input.
    - `arguments`: Holds the list of arguments associated with the command.
    - `entry`: Links the command in a tail queue for ordered processing.
- **Description**: The `cmd_parse_command` structure represents a parsed command within a command parsing system, encapsulating the line number where the command appears and a list of its arguments. It is part of a tail queue, allowing for efficient insertion and removal of commands in a sequence, which is crucial for processing command lists in order. This structure is used in conjunction with other parsing components to manage and execute commands in a controlled and organized manner.


---
### cmd_parse_commands
- **Type**: `TAILQ_HEAD`
- **Members**:
    - `cmd_parse_command`: Represents a single command with its line number and associated arguments.
- **Description**: The `cmd_parse_commands` data structure is a tail queue that holds a list of `cmd_parse_command` structures, each representing a parsed command with its line number and associated arguments. This structure is used to manage and iterate over a collection of commands that have been parsed from input, allowing for operations such as command execution or further processing.


---
### cmd_parse_state
- **Type**: `struct`
- **Members**:
    - `f`: A pointer to a FILE object used for file input.
    - `buf`: A constant character pointer to a buffer for input data.
    - `len`: A size_t representing the length of the buffer.
    - `off`: A size_t representing the current offset in the buffer.
    - `condition`: An integer indicating the current parsing condition.
    - `eol`: An integer flag indicating end-of-line status.
    - `eof`: An integer flag indicating end-of-file status.
    - `input`: A pointer to a cmd_parse_input structure for input data.
    - `escapes`: An unsigned integer tracking escape sequences.
    - `error`: A character pointer for storing error messages.
    - `commands`: A pointer to a cmd_parse_commands structure holding parsed commands.
    - `scope`: A pointer to a cmd_parse_scope structure for the current parsing scope.
    - `stack`: A TAILQ_HEAD for managing a stack of cmd_parse_scope entries.
- **Description**: The cmd_parse_state structure is used to maintain the state of a command parsing process, including input sources (file or buffer), parsing conditions, and error handling. It manages the current position in the input, tracks escape sequences, and holds the parsed commands and scope information. This structure is essential for parsing commands in a structured and stateful manner, allowing for complex command processing and error management.


# Functions

---
### cmd_parse_get_error
The `cmd_parse_get_error` function constructs and returns a formatted error message string based on the provided file name, line number, and error message.
- **Inputs**:
    - `file`: A constant character pointer representing the name of the file where the error occurred; it can be NULL if the file name is not applicable.
    - `line`: An unsigned integer representing the line number in the file where the error occurred.
    - `error`: A constant character pointer representing the error message to be included in the formatted string.
- **Control Flow**:
    - Check if the `file` argument is NULL.
    - If `file` is NULL, duplicate the `error` string using `xstrdup` and assign it to `s`.
    - If `file` is not NULL, format the string to include the file name, line number, and error message using `xasprintf`, and assign it to `s`.
    - Return the formatted string `s`.
- **Output**:
    - A dynamically allocated string containing the formatted error message, which includes the file name and line number if provided.


---
### cmd_parse_print_commands
The `cmd_parse_print_commands` function prints a list of commands to a specified output if verbose mode is enabled.
- **Inputs**:
    - `pi`: A pointer to a `cmd_parse_input` structure that contains input parsing information, including the output item and flags.
    - `cmdlist`: A pointer to a `cmd_list` structure that contains the list of commands to be printed.
- **Control Flow**:
    - Check if the `item` in `pi` is NULL or if the `CMD_PARSE_VERBOSE` flag is not set in `pi->flags`; if either condition is true, return immediately without printing.
    - Call `cmd_list_print` to convert the command list into a string representation.
    - Check if `pi->file` is not NULL; if true, print the file name, line number, and command string using `cmdq_print`.
    - If `pi->file` is NULL, print only the line number and command string using `cmdq_print`.
    - Free the memory allocated for the command string.
- **Output**:
    - The function does not return a value; it performs output operations to print the command list.


---
### cmd_parse_free_argument
The `cmd_parse_free_argument` function deallocates memory associated with a `cmd_parse_argument` structure based on its type.
- **Inputs**:
    - `arg`: A pointer to a `cmd_parse_argument` structure that needs to be freed.
- **Control Flow**:
    - The function checks the type of the `cmd_parse_argument` structure using a switch statement.
    - If the type is `CMD_PARSE_STRING`, it frees the `string` member of the structure.
    - If the type is `CMD_PARSE_COMMANDS`, it calls `cmd_parse_free_commands` to free the `commands` member.
    - If the type is `CMD_PARSE_PARSED_COMMANDS`, it calls `cmd_list_free` to free the `cmdlist` member.
    - Finally, it frees the `cmd_parse_argument` structure itself.
- **Output**:
    - The function does not return any value; it performs memory deallocation.


---
### cmd_parse_free_arguments
The `cmd_parse_free_arguments` function deallocates memory for a list of command parse arguments.
- **Inputs**:
    - `args`: A pointer to a `cmd_parse_arguments` structure, which is a list of command parse arguments to be freed.
- **Control Flow**:
    - Iterate over each `cmd_parse_argument` in the `args` list using `TAILQ_FOREACH_SAFE` to safely traverse and modify the list.
    - For each argument, remove it from the list using `TAILQ_REMOVE`.
    - Call `cmd_parse_free_argument` to free the memory associated with the current argument.
    - Continue until all arguments in the list have been processed and freed.
- **Output**:
    - The function does not return a value; it performs memory deallocation for the provided list of command parse arguments.


---
### cmd_parse_free_command
The `cmd_parse_free_command` function deallocates memory associated with a `cmd_parse_command` structure, including its arguments.
- **Inputs**:
    - `cmd`: A pointer to a `cmd_parse_command` structure that needs to be freed.
- **Control Flow**:
    - The function calls `cmd_parse_free_arguments` to free all arguments associated with the command.
    - It then frees the memory allocated for the `cmd_parse_command` structure itself.
- **Output**:
    - The function does not return any value; it performs memory deallocation for the given command structure.


---
### cmd_parse_new_commands
The `cmd_parse_new_commands` function initializes and returns a new `cmd_parse_commands` structure.
- **Inputs**:
    - None
- **Control Flow**:
    - Allocate memory for a new `cmd_parse_commands` structure.
    - Initialize the TAILQ (doubly linked list) within the structure using `TAILQ_INIT`.
    - Return the newly created `cmd_parse_commands` structure.
- **Output**:
    - A pointer to a newly allocated and initialized `cmd_parse_commands` structure.


---
### cmd_parse_free_commands
The `cmd_parse_free_commands` function deallocates memory associated with a list of parsed commands.
- **Inputs**:
    - `cmds`: A pointer to a `cmd_parse_commands` structure, which is a list of parsed commands to be freed.
- **Control Flow**:
    - Iterate over each command in the `cmds` list using `TAILQ_FOREACH_SAFE` to safely traverse and remove elements.
    - For each command, remove it from the list using `TAILQ_REMOVE`.
    - Call `cmd_parse_free_command` to free the memory associated with the command's arguments and the command itself.
    - After all commands are removed and freed, free the `cmds` list structure itself.
- **Output**:
    - The function does not return any value; it performs memory deallocation for the provided list of commands.


---
### cmd_parse_run_parser
The `cmd_parse_run_parser` function executes the parsing process for commands and returns a list of parsed commands or an error cause.
- **Inputs**:
    - `cause`: A pointer to a character pointer where the error message will be stored if parsing fails.
- **Control Flow**:
    - Initialize the `parse_state` structure and its command list and stack.
    - Call the `yyparse` function to perform the parsing process.
    - Iterate over the `parse_state` stack to free any remaining scope entries.
    - Check the return value of `yyparse`; if non-zero, set the `cause` to the error message and return `NULL`.
    - If parsing is successful and `parse_state.commands` is `NULL`, create a new command list.
    - Return the parsed command list.
- **Output**:
    - Returns a pointer to a `cmd_parse_commands` structure containing the parsed commands, or `NULL` if an error occurred.


---
### cmd_parse_do_file
The `cmd_parse_do_file` function parses commands from a given file and returns a list of parsed commands.
- **Inputs**:
    - `f`: A pointer to a FILE object from which commands are to be read and parsed.
    - `pi`: A pointer to a `cmd_parse_input` structure that provides context and settings for the parsing process.
    - `cause`: A pointer to a char pointer where an error message will be stored if parsing fails.
- **Control Flow**:
    - Initialize the `cmd_parse_state` structure to zero.
    - Set the `input` field of `cmd_parse_state` to the provided `cmd_parse_input` structure.
    - Set the `f` field of `cmd_parse_state` to the provided FILE pointer `f`.
    - Call `cmd_parse_run_parser` to parse the commands from the file.
    - Return the parsed commands or NULL if an error occurred.
- **Output**:
    - Returns a pointer to a `cmd_parse_commands` structure containing the parsed commands, or NULL if an error occurred.


---
### cmd_parse_do_buffer
The `cmd_parse_do_buffer` function parses a buffer containing command data and returns a list of parsed commands.
- **Inputs**:
    - `buf`: A pointer to the buffer containing the command data to be parsed.
    - `len`: The length of the buffer.
    - `pi`: A pointer to a `cmd_parse_input` structure containing input parameters for parsing.
    - `cause`: A pointer to a string that will be set to an error message if parsing fails.
- **Control Flow**:
    - Initialize the `cmd_parse_state` structure to zero and set its input and buffer fields.
    - Call `cmd_parse_run_parser` to parse the buffer and obtain the parsed commands.
    - If parsing fails, set the `cause` to the error message and return `NULL`.
    - If parsing succeeds, return the parsed commands.
- **Output**:
    - Returns a pointer to a `cmd_parse_commands` structure containing the parsed commands, or `NULL` if an error occurs.


---
### cmd_parse_log_commands
The `cmd_parse_log_commands` function logs the details of parsed commands and their arguments with a specified prefix.
- **Inputs**:
    - `cmds`: A pointer to a `cmd_parse_commands` structure, which is a list of parsed commands to be logged.
    - `prefix`: A string prefix to be prepended to each log entry for identification.
- **Control Flow**:
    - Initialize a counter `i` to track the command index.
    - Iterate over each command in the `cmds` list using a `TAILQ_FOREACH` loop.
    - For each command, initialize a counter `j` to track the argument index.
    - Iterate over each argument in the command's arguments list using a `TAILQ_FOREACH` loop.
    - Depending on the argument type, log the argument details with the prefix, command index `i`, and argument index `j`.
    - If the argument type is `CMD_PARSE_COMMANDS`, recursively call `cmd_parse_log_commands` with the nested commands and a new prefix.
    - Increment the argument index `j` after processing each argument.
    - Increment the command index `i` after processing each command.
- **Output**:
    - The function does not return a value; it logs information to the debug log.


---
### cmd_parse_expand_alias
The `cmd_parse_expand_alias` function expands command aliases within a command parse structure, updating the command list with the expanded commands.
- **Inputs**:
    - `cmd`: A pointer to a `cmd_parse_command` structure representing the command to be expanded.
    - `pi`: A pointer to a `cmd_parse_input` structure containing input parsing information and flags.
    - `pr`: A pointer to a `cmd_parse_result` structure where the result of the parsing operation will be stored.
- **Control Flow**:
    - Check if alias expansion is disabled via the `CMD_PARSE_NOALIAS` flag in `pi`; if so, return 0 without expanding.
    - Retrieve the first argument from the command's argument list and check if it is a string; if not, set the parse result to success with an empty command list and return 1.
    - Get the alias for the command name using `cmd_get_alias`; if no alias is found, return 0.
    - Parse the alias string into a command list using `cmd_parse_do_buffer`; if parsing fails, set the parse result to error with the cause and return 1.
    - Remove the first argument from the command's argument list and free it.
    - Transfer remaining arguments from the original command to the last command in the parsed alias command list.
    - Log the expanded commands for debugging purposes.
    - Set the `CMD_PARSE_NOALIAS` flag to prevent recursive alias expansion, build the commands from the parsed alias, and then clear the flag.
    - Return 1 to indicate that alias expansion was performed.
- **Output**:
    - Returns an integer indicating whether alias expansion was performed (1) or not (0).


---
### cmd_parse_build_command
The `cmd_parse_build_command` function processes a single command by expanding any aliases, parsing its arguments, and constructing a command list for execution.
- **Inputs**:
    - `cmd`: A pointer to a `cmd_parse_command` structure representing the command to be processed.
    - `pi`: A pointer to a `cmd_parse_input` structure containing input parameters and state for parsing.
    - `pr`: A pointer to a `cmd_parse_result` structure where the result of the command parsing will be stored.
- **Control Flow**:
    - Initialize the `cmd_parse_result` structure `pr` to zero.
    - Check if the command can be expanded as an alias using `cmd_parse_expand_alias`; if so, return immediately.
    - Iterate over each argument in the command's argument list.
    - For each argument, allocate or reallocate memory for `values` to store the argument data.
    - Depending on the argument type, either copy the string, build commands from nested commands, or reference parsed commands.
    - Attempt to parse the command using `cmd_parse` and store the result in `pr`.
    - If parsing fails, set the error status and message in `pr` and clean up resources.
    - If successful, append the parsed command to a new command list in `pr`.
    - Free any allocated resources for argument values.
- **Output**:
    - The function outputs a `cmd_parse_result` structure `pr` that contains the status of the parsing operation and a command list if successful, or an error message if parsing failed.


---
### cmd_parse_build_commands
The `cmd_parse_build_commands` function processes a list of parsed commands and constructs a command list for execution, handling command grouping and error checking.
- **Inputs**:
    - `cmds`: A pointer to a `cmd_parse_commands` structure, which is a list of parsed commands to be processed.
    - `pi`: A pointer to a `cmd_parse_input` structure, which contains input parameters and state information for parsing.
    - `pr`: A pointer to a `cmd_parse_result` structure, which will store the result of the command parsing process, including status and any generated command list.
- **Control Flow**:
    - Initialize the `pr` structure to zero to clear any previous state.
    - Check if the `cmds` list is empty; if so, set the result status to success and return an empty command list.
    - Log the commands for debugging purposes using `cmd_parse_log_commands`.
    - Create a new command list `result` to store the parsed commands.
    - Iterate over each command in the `cmds` list.
    - For each command, check if a new command list should be started based on the line number and flags.
    - Build each command using `cmd_parse_build_command`, updating the `pr` structure with the result.
    - If a command fails to parse, free the current and result command lists and return.
    - Append successfully parsed commands to the current command list.
    - After processing all commands, move the current command list to the result list and free the current list.
    - Log the final command list for debugging purposes.
    - Set the result status to success and assign the result command list to the `pr` structure.
- **Output**:
    - The function outputs a `cmd_parse_result` structure with a status indicating success or error, and a command list containing the parsed commands ready for execution.


---
### cmd_parse_from_file
The `cmd_parse_from_file` function parses commands from a given file and returns the result of the parsing process.
- **Inputs**:
    - `f`: A pointer to a FILE object from which commands are to be read and parsed.
    - `pi`: A pointer to a `cmd_parse_input` structure that provides context and settings for the parsing process.
- **Control Flow**:
    - Initialize a static `cmd_parse_result` structure `pr` and a `cmd_parse_input` structure `input` if `pi` is NULL.
    - Call `cmd_parse_do_file` to parse commands from the file `f` using the input `pi`, storing any error message in `cause`.
    - If `cmd_parse_do_file` returns NULL, set the status of `pr` to `CMD_PARSE_ERROR` and assign the error message to `pr.error`.
    - If commands are successfully parsed, call `cmd_parse_build_commands` to build the command list from the parsed commands.
    - Free the parsed commands using `cmd_parse_free_commands`.
    - Return a pointer to the `cmd_parse_result` structure `pr`.
- **Output**:
    - A pointer to a `cmd_parse_result` structure containing the status of the parsing operation and any resulting command list or error message.


---
### cmd_parse_from_string
The `cmd_parse_from_string` function parses a command string into a structured command list, handling command grouping and parsing errors.
- **Inputs**:
    - `s`: A string containing the commands to be parsed.
    - `pi`: A pointer to a `cmd_parse_input` structure that provides context for the parsing operation, such as flags and line numbers.
- **Control Flow**:
    - If `pi` is NULL, initialize a local `cmd_parse_input` structure and assign it to `pi`.
    - Set the `CMD_PARSE_ONEGROUP` flag in `pi` to ensure commands are grouped together even if they span multiple lines.
    - Call `cmd_parse_from_buffer` with the command string `s`, its length, and the `pi` structure to perform the parsing.
    - Return the result of `cmd_parse_from_buffer`, which is a pointer to a `cmd_parse_result` structure containing the parsing outcome.
- **Output**:
    - A pointer to a `cmd_parse_result` structure, which contains the status of the parsing operation and any resulting command list or error message.


---
### cmd_parse_and_insert
The `cmd_parse_and_insert` function parses a command string and inserts the resulting command list into a command queue after a specified item.
- **Inputs**:
    - `s`: A string containing the command to be parsed.
    - `pi`: A pointer to a `cmd_parse_input` structure containing parsing context and options.
    - `after`: A pointer to a `cmdq_item` structure after which the parsed command list will be inserted.
    - `state`: A pointer to a `cmdq_state` structure representing the current state of the command queue.
    - `error`: A pointer to a character pointer where an error message will be stored if parsing fails.
- **Control Flow**:
    - Call `cmd_parse_from_string` to parse the command string `s` into a `cmd_parse_result` structure `pr`.
    - Check the status of `pr` to determine if parsing was successful or if an error occurred.
    - If an error occurred, store the error message in `*error` if `error` is not NULL, otherwise free the error message.
    - If parsing was successful, retrieve a command queue item from the parsed command list using `cmdq_get_command`.
    - Insert the retrieved command queue item into the command queue after the specified `after` item using `cmdq_insert_after`.
    - Free the parsed command list.
- **Output**:
    - Returns an `enum cmd_parse_status` indicating the result of the parsing operation, either `CMD_PARSE_SUCCESS` or `CMD_PARSE_ERROR`.


---
### cmd_parse_and_append
The `cmd_parse_and_append` function parses a command string and appends the resulting command queue item to a client's command queue.
- **Inputs**:
    - `s`: A string containing the command to be parsed.
    - `pi`: A pointer to a `cmd_parse_input` structure containing parsing context and options.
    - `c`: A pointer to a `client` structure representing the client to which the command will be appended.
    - `state`: A pointer to a `cmdq_state` structure representing the state of the command queue.
    - `error`: A pointer to a char pointer where an error message will be stored if parsing fails.
- **Control Flow**:
    - The function begins by calling `cmd_parse_from_string` to parse the command string `s` into a `cmd_parse_result` structure `pr`.
    - It checks the status of `pr` to determine if parsing was successful or if an error occurred.
    - If an error occurred (`CMD_PARSE_ERROR`), it assigns the error message to `*error` if `error` is not NULL, otherwise it frees the error message.
    - If parsing is successful (`CMD_PARSE_SUCCESS`), it retrieves a command queue item from the parsed command list using `cmdq_get_command`.
    - The command queue item is then appended to the client's command queue using `cmdq_append`.
    - Finally, the function frees the command list in `pr`.
- **Output**:
    - The function returns an `enum cmd_parse_status` indicating the result of the parsing operation, either `CMD_PARSE_SUCCESS` or `CMD_PARSE_ERROR`.


---
### cmd_parse_from_buffer
The `cmd_parse_from_buffer` function parses a buffer containing command strings into a structured command list, handling syntax and errors.
- **Inputs**:
    - `buf`: A pointer to the buffer containing the command strings to be parsed.
    - `len`: The length of the buffer.
    - `pi`: A pointer to a `cmd_parse_input` structure that provides context for the parsing operation.
- **Control Flow**:
    - Initialize a static `cmd_parse_result` structure and a local `cmd_parse_input` structure if `pi` is NULL.
    - Check if the buffer length `len` is zero; if so, return a successful parse result with an empty command list.
    - Call `cmd_parse_do_buffer` to parse the buffer into a `cmd_parse_commands` structure, capturing any error in `cause`.
    - If parsing fails, set the parse result status to error and store the error message.
    - If parsing succeeds, build the command list from the parsed commands using `cmd_parse_build_commands`.
    - Free the parsed commands structure after building the command list.
    - Return the parse result.
- **Output**:
    - Returns a pointer to a `cmd_parse_result` structure containing the status of the parse operation and the resulting command list or error message.


---
### cmd_parse_from_arguments
The `cmd_parse_from_arguments` function parses a list of command arguments into a structured command list.
- **Inputs**:
    - `values`: An array of `args_value` structures representing the command arguments to be parsed.
    - `count`: The number of elements in the `values` array.
    - `pi`: A pointer to a `cmd_parse_input` structure that provides context for the parsing operation.
- **Control Flow**:
    - Initialize a `cmd_parse_input` structure if `pi` is NULL.
    - Create a new `cmd_parse_commands` structure to hold the parsed commands.
    - Iterate over each argument in the `values` array.
    - For each argument, determine its type and handle it accordingly: if it's a string, check for command separators (';') and split commands accordingly; if it's a command list, add it directly to the current command.
    - If a command separator is found, finalize the current command and start a new one.
    - After processing all arguments, finalize any remaining command.
    - Build the command list from the parsed commands and store the result in the `cmd_parse_result` structure.
    - Free the memory allocated for the parsed commands.
- **Output**:
    - Returns a pointer to a `cmd_parse_result` structure containing the status of the parsing operation and the resulting command list.


---
### yyerror
The `yyerror` function formats and stores an error message in the global parse state if no error message is already present.
- **Inputs**:
    - `fmt`: A format string similar to those used in printf-style functions.
- **Control Flow**:
    - Check if the current parse state's error is already set; if so, return immediately.
    - Initialize a variable argument list to handle the variable arguments passed to the function.
    - Use `xvasprintf` to format the error message according to the provided format string and arguments.
    - Store the formatted error message in the parse state's error field using `cmd_parse_get_error`.
    - Free the temporary error string created by `xvasprintf`.
    - Return 0 to indicate the function executed successfully.
- **Output**:
    - The function returns 0, indicating successful execution, but its primary purpose is to set the error message in the parse state.


---
### yylex_is_var
The `yylex_is_var` function checks if a character is valid as part of a variable name in a lexical analysis context.
- **Inputs**:
    - `ch`: The character to be checked.
    - `first`: A flag indicating if the character is the first in the variable name.
- **Control Flow**:
    - Check if the character is an equal sign ('='), return 0 if true.
    - If 'first' is true, check if the character is a digit, return 0 if true.
    - Return 1 if the character is alphanumeric or an underscore ('_'), otherwise return 0.
- **Output**:
    - Returns 1 if the character is valid as part of a variable name, otherwise returns 0.


---
### yylex_append
The `yylex_append` function appends a specified string to a buffer, reallocating memory as needed to accommodate the new data.
- **Inputs**:
    - `buf`: A pointer to a character buffer where the string will be appended.
    - `len`: A pointer to a size_t variable representing the current length of the buffer.
    - `add`: A pointer to the string to be appended to the buffer.
    - `addlen`: The length of the string to be appended.
- **Control Flow**:
    - Check if the combined length of the current buffer and the new string exceeds the maximum allowable size, and if so, trigger a fatal error.
    - Reallocate the buffer to accommodate the new string plus a null terminator.
    - Copy the new string into the buffer at the position immediately following the current content.
    - Update the length of the buffer to reflect the addition of the new string.
- **Output**:
    - The function does not return a value; it modifies the buffer and its length in place.


---
### yylex_append1
The `yylex_append1` function appends a single character to a dynamically allocated buffer, updating its length.
- **Inputs**:
    - `buf`: A pointer to a dynamically allocated buffer where the character will be appended.
    - `len`: A pointer to a size_t variable representing the current length of the buffer, which will be updated.
    - `add`: The character to be appended to the buffer.
- **Control Flow**:
    - The function calls `yylex_append`, passing the buffer, its length, the address of the character to append, and the length of 1.
    - `yylex_append` handles the actual appending of the character to the buffer and updates the length.
- **Output**:
    - The function does not return a value; it modifies the buffer and its length in place.


---
### yylex_getc1
The `yylex_getc1` function retrieves the next character from the input source, which can be either a file or a buffer, and handles the end-of-file condition.
- **Inputs**:
    - `None`: This function does not take any input arguments directly; it operates on the global `parse_state` structure.
- **Control Flow**:
    - Check if the input source is a file (`ps->f` is not NULL).
    - If it is a file, use `getc` to read the next character from the file.
    - If it is not a file, check if the current offset (`ps->off`) has reached the length of the buffer (`ps->len`).
    - If the offset has reached the length, return `EOF` to indicate the end of the buffer.
    - If not, return the character at the current offset and increment the offset.
- **Output**:
    - The function returns the next character from the input source or `EOF` if the end of the input is reached.


---
### yylex_ungetc
The `yylex_ungetc` function pushes a character back onto the input stream for the lexical analyzer.
- **Inputs**:
    - `ch`: The character to be pushed back onto the input stream.
- **Control Flow**:
    - Check if the input stream is a file; if so, use `ungetc` to push the character back onto the file stream.
    - If the input stream is a buffer, check if the offset is greater than zero and the character is not EOF.
    - If conditions are met, decrement the offset to effectively push the character back onto the buffer.
- **Output**:
    - The function does not return a value; it modifies the input stream state by pushing a character back.


---
### yylex_getc
The `yylex_getc` function reads characters from a file or buffer, handling escape sequences and line continuations, and returns the next character to be processed by the lexer.
- **Inputs**:
    - `None`: The function does not take any explicit input parameters, but it operates on the global `parse_state` structure.
- **Control Flow**:
    - Check if there are any escape characters pending; if so, return a backslash and decrement the escape count.
    - Enter a loop to read characters using `yylex_getc1`.
    - If a backslash is encountered, increment the escape count and continue the loop.
    - If a newline is encountered and the escape count is odd, increment the line number, decrement the escape count, and continue the loop.
    - If there are pending escapes, unget the current character, decrement the escape count, and return a backslash.
    - Return the current character if no special conditions are met.
- **Output**:
    - The function returns an integer representing the next character to be processed, handling escape sequences and line continuations appropriately.


---
### yylex_get_word
The `yylex_get_word` function reads characters from input until a delimiter is encountered, forming and returning a word.
- **Inputs**:
    - `ch`: The initial character to start forming the word.
- **Control Flow**:
    - Initialize a buffer to store the word and set its length to zero.
    - Append the initial character to the buffer.
    - Enter a loop to read characters using `yylex_getc` until EOF or a delimiter (space, tab, newline) is encountered.
    - Append each character to the buffer until a delimiter is found.
    - Use `yylex_ungetc` to put back the delimiter character into the input stream.
    - Terminate the buffer with a null character to form a valid C string.
    - Log the formed word for debugging purposes.
- **Output**:
    - Returns a dynamically allocated string containing the word formed from the input characters.


---
### yylex
The `yylex` function is a lexical analyzer that processes input to identify and return tokens for parsing.
- **Inputs**:
    - None
- **Control Flow**:
    - Initialize the line number if end-of-line (EOL) is reached.
    - Loop to read characters from input using `yylex_getc`.
    - Handle end-of-file (EOF) by returning a newline character if not already marked as EOF.
    - Skip whitespace characters and handle carriage return followed by newline as a single newline.
    - Return specific characters like semicolon, braces, and newline as tokens directly.
    - Handle comments by skipping characters until the end of the line.
    - Identify and process conditions and formats starting with `%` and `#{` respectively.
    - Process tokens by reading words and checking for variable assignments using `=`.
    - Return tokens or errors based on the identified patterns and conditions.
- **Output**:
    - The function returns an integer representing a token type or a specific character, or 0 if EOF is reached.


---
### yylex_format
The `yylex_format` function reads and processes a format string enclosed in `#{}` from the input, handling nested formats and returning the complete format string.
- **Inputs**:
    - None
- **Control Flow**:
    - Initialize a buffer to store the format string and set the initial bracket count to 1.
    - Append the initial `#{` to the buffer.
    - Enter a loop to read characters from the input until the format string is complete or an error occurs.
    - If a `#` is encountered, check if the next character is `{` to handle nested formats, incrementing the bracket count if so.
    - If a `}` is encountered, decrement the bracket count and check if it reaches zero, indicating the end of the format string.
    - Append each character to the buffer as it is read.
    - If the loop exits with unmatched brackets, free the buffer and return `NULL` to indicate an error.
    - If the format string is successfully read, terminate the buffer with a null character and return it.
- **Output**:
    - Returns a dynamically allocated string containing the complete format string, or `NULL` if an error occurs.


---
### yylex_token_escape
The `yylex_token_escape` function processes escape sequences in a token and appends the corresponding character to a buffer.
- **Inputs**:
    - `buf`: A pointer to a character buffer where the processed escape sequence will be appended.
    - `len`: A pointer to a size_t variable representing the current length of the buffer.
- **Control Flow**:
    - Retrieve the next character using `yylex_getc()`.
    - Check if the character is an octal digit between '0' and '3'.
    - If it is, attempt to read two more octal digits to form a complete octal escape sequence.
    - If a valid octal escape sequence is formed, convert it to a character and append it to the buffer.
    - If the character is not part of a valid octal escape sequence, report an error using `yyerror()`.
    - For non-octal escape sequences, map specific characters to their escape equivalents (e.g., '\n' to newline) and append them to the buffer.
    - For Unicode escape sequences (\u or \U), read the specified number of hexadecimal digits, convert them to a wide character, and append the multibyte representation to the buffer.
    - Return 1 if the escape sequence is successfully processed and appended, otherwise return 0.
- **Output**:
    - Returns 1 if the escape sequence is successfully processed and appended to the buffer, otherwise returns 0.


---
### yylex_token_variable
The `yylex_token_variable` function processes and expands environment variable tokens within a given input stream.
- **Inputs**:
    - `buf`: A pointer to a character buffer where the expanded variable value will be appended.
    - `len`: A pointer to a size_t variable representing the current length of the buffer.
- **Control Flow**:
    - Read the next character from the input stream to determine if it starts a variable name or is a '{' indicating a bracketed variable name.
    - If the character is not a valid start for a variable name, append the '$' character to the buffer and return.
    - Iterate over the input stream to read the variable name, checking for valid characters and handling bracketed names appropriately.
    - If the variable name is too long, report an error and return.
    - Look up the variable name in the environment and, if found, append its value to the buffer.
    - Return success if the variable was expanded, or failure if an error occurred.
- **Output**:
    - Returns 1 if the variable was successfully expanded and appended to the buffer, or 0 if an error occurred.


---
### yylex_token_tilde
The `yylex_token_tilde` function processes a tilde character in a token, expanding it to the corresponding home directory path.
- **Inputs**:
    - `buf`: A pointer to a character buffer where the expanded token will be stored.
    - `len`: A pointer to a size_t variable representing the current length of the buffer.
- **Control Flow**:
    - Initialize a character array `name` to store the username and a size_t `namelen` to track its length.
    - Read characters using `yylex_getc` until a delimiter or EOF is encountered, appending valid characters to `name`.
    - If `name` is empty, attempt to retrieve the home directory from the environment variable `HOME` or the current user's password entry.
    - If `name` is not empty, attempt to retrieve the home directory for the specified username using `getpwnam`.
    - If a home directory is found, append it to the buffer `buf` and return success; otherwise, return failure.
- **Output**:
    - Returns 1 on success, indicating the tilde was successfully expanded to a home directory path, or 0 on failure.


---
### yylex_token
The `yylex_token` function reads characters from input, processes them according to specific rules, and returns a token string.
- **Inputs**:
    - `ch`: The initial character to start processing the token.
- **Control Flow**:
    - Initialize a buffer to store the token and set the initial state to NONE.
    - Enter a loop to process each character until EOF or a newline is encountered.
    - Check if the character is a special character (e.g., whitespace, semicolon, brace) and handle it according to the current state (e.g., inside quotes or not).
    - Handle escape sequences, tilde expansion, and variable expansion if not inside single quotes.
    - Manage quote states by toggling between NONE, SINGLE_QUOTES, and DOUBLE_QUOTES when encountering quote characters.
    - Append valid characters to the buffer and continue processing the next character.
    - Terminate the loop when a token-ending condition is met and return the constructed token.
- **Output**:
    - Returns a dynamically allocated string containing the processed token, or NULL if an error occurs during token processing.


