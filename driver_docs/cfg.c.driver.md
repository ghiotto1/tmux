# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file is responsible for handling the configuration loading process of tmux. It defines functions to load configuration files and buffers, manage configuration-related errors, and ensure that the initial client is blocked until the configuration is fully loaded. The code includes functions such as [`start_cfg`](#start_cfg), [`load_cfg`](#load_cfg), and [`load_cfg_from_buffer`](#load_cfg_from_buffer), which are used to parse and execute commands from configuration files or buffers. It also manages error reporting through functions like [`cfg_add_cause`](#cfg_add_cause), [`cfg_print_causes`](#cfg_print_causes), and [`cfg_show_causes`](#cfg_show_causes), which collect and display any issues encountered during the configuration loading process.

The file is not an executable but rather a component of the tmux codebase that provides specific functionality related to configuration management. It interacts with other parts of the tmux system, such as command queues and client management, to ensure that configuration commands are executed in the correct order and context. The code uses several static and global variables to track the state of the configuration process, such as `cfg_client`, `cfg_finished`, and `cfg_causes`. This file does not define public APIs or external interfaces but is intended to be used internally within the tmux project to facilitate the configuration loading and error handling processes.
# Imports and Dependencies

---
- `sys/types.h`
- `ctype.h`
- `errno.h`
- `stdio.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### cfg_client
- **Type**: `struct client*`
- **Description**: `cfg_client` is a global pointer variable of type `struct client*`. It is used to reference the initial client in the system, which is crucial for managing the execution of commands after configuration files are loaded.
- **Use**: This variable is used to block the initial client so that its command runs after the configuration is loaded, ensuring proper command execution order.


---
### cfg_finished
- **Type**: `int`
- **Description**: The `cfg_finished` variable is a global integer that indicates whether the configuration process has been completed. It is used as a flag to control the flow of the configuration loading process.
- **Use**: This variable is used to determine if the configuration loading process is finished, allowing subsequent operations to proceed.


---
### cfg_causes
- **Type**: `char **`
- **Description**: `cfg_causes` is a static global variable that is a pointer to an array of strings (character pointers). It is used to store error messages or causes that occur during the configuration process.
- **Use**: This variable is used to accumulate and store error messages that are generated when loading configuration files, allowing them to be displayed or logged later.


---
### cfg_ncauses
- **Type**: `u_int`
- **Description**: The `cfg_ncauses` variable is a static unsigned integer that keeps track of the number of configuration error messages stored in the `cfg_causes` array. It is used to manage the dynamic allocation and reallocation of the `cfg_causes` array as new error messages are added.
- **Use**: This variable is incremented each time a new configuration error message is added to the `cfg_causes` array, ensuring accurate tracking of the number of error messages.


---
### cfg_item
- **Type**: `struct cmdq_item *`
- **Description**: `cfg_item` is a static pointer to a `struct cmdq_item`, which is part of the command queue system in the tmux codebase. It is used to manage command execution order and dependencies, particularly during the configuration loading process.
- **Use**: `cfg_item` is used to hold a reference to a command queue item that ensures the initial client command runs after configuration files are loaded.


---
### cfg_quiet
- **Type**: `int`
- **Description**: The `cfg_quiet` variable is an integer that is initialized to 1, indicating that the configuration loading process should operate in a quiet mode by default. This means that non-critical errors, such as missing configuration files, will not produce error messages.
- **Use**: This variable is used to determine whether the configuration loading process should suppress non-critical error messages.


---
### cfg_files
- **Type**: `char **`
- **Description**: The `cfg_files` variable is a global pointer to an array of strings, where each string represents the path to a configuration file. It is used to store the list of configuration files that need to be loaded by the program.
- **Use**: This variable is used in the `start_cfg` function to iterate over and load each configuration file specified in the array.


---
### cfg_nfiles
- **Type**: `u_int`
- **Description**: The `cfg_nfiles` variable is a global variable of type `u_int` (unsigned integer) that holds the number of configuration files to be processed. It is used to iterate over the list of configuration files stored in the `cfg_files` array.
- **Use**: `cfg_nfiles` is used in a loop to load each configuration file specified in the `cfg_files` array during the initialization process.


# Functions

---
### cfg_client_done<!-- {{#callable:cfg_client_done}} -->
The `cfg_client_done` function checks if the configuration process is finished and returns a command return value accordingly.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which is unused in this function.
    - `data`: A pointer to any additional data, which is unused in this function.
- **Control Flow**:
    - The function checks the global variable `cfg_finished` to determine if the configuration process is complete.
    - If `cfg_finished` is false (indicating the configuration is not yet finished), the function returns `CMD_RETURN_WAIT`.
    - If `cfg_finished` is true (indicating the configuration is complete), the function returns `CMD_RETURN_NORMAL`.
- **Output**:
    - The function returns an `enum cmd_retval`, which is either `CMD_RETURN_WAIT` if the configuration is not finished, or `CMD_RETURN_NORMAL` if it is finished.


---
### cfg_done<!-- {{#callable:cfg_done}} -->
The `cfg_done` function finalizes the configuration process by marking it as finished, displaying any configuration errors, continuing any pending command queue items, and loading the command history.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which is unused in this function.
    - `data`: A void pointer to any additional data, which is unused in this function.
- **Control Flow**:
    - Check if the configuration process is already finished by evaluating `cfg_finished`; if true, return `CMD_RETURN_NORMAL`.
    - Set `cfg_finished` to 1 to mark the configuration process as completed.
    - Call `cfg_show_causes(NULL)` to display any configuration errors that occurred.
    - If `cfg_item` is not NULL, call `cmdq_continue(cfg_item)` to continue processing the command queue item.
    - Call `status_prompt_load_history()` to load the command history.
    - Return `CMD_RETURN_NORMAL` to indicate normal completion.
- **Output**:
    - The function returns an `enum cmd_retval` value, specifically `CMD_RETURN_NORMAL`, indicating the normal completion of the function.
- **Functions called**:
    - [`cfg_show_causes`](#cfg_show_causes)


---
### start_cfg<!-- {{#callable:start_cfg}} -->
The `start_cfg` function initializes the configuration process by loading configuration files and setting up command queues for clients.
- **Inputs**:
    - None
- **Control Flow**:
    - Initialize a client pointer `c` and an integer `flags` to 0.
    - Retrieve the first client from the global `clients` list and assign it to `cfg_client` and `c`.
    - If a client exists, create a command queue item with a callback `cfg_client_done` and append it to the client's command queue.
    - Set `flags` to `CMD_PARSE_QUIET` if `cfg_quiet` is true.
    - Iterate over each configuration file in `cfg_files` and call [`load_cfg`](#load_cfg) with the file path, client, and flags.
    - Append a command queue item with a callback `cfg_done` to the global command queue.
- **Output**:
    - The function does not return a value; it sets up command queues and loads configuration files.
- **Functions called**:
    - [`load_cfg`](#load_cfg)


---
### load_cfg<!-- {{#callable:load_cfg}} -->
The `load_cfg` function loads and parses a configuration file, executing commands from it and optionally returning a new command queue item.
- **Inputs**:
    - `path`: A string representing the file path to the configuration file to be loaded.
    - `c`: A pointer to a `struct client` representing the client context for the configuration loading.
    - `item`: A pointer to a `struct cmdq_item` representing the current command queue item, which can be used to insert new items.
    - `current`: A pointer to a `struct cmd_find_state` representing the current command find state, used for state copying.
    - `flags`: An integer representing flags that modify the behavior of the function, such as quiet mode or parse-only mode.
    - `new_item`: A pointer to a pointer to a `struct cmdq_item` where a new command queue item will be stored if created.
- **Control Flow**:
    - Initialize the `new_item` to NULL if it is not NULL.
    - Log the loading of the configuration file path.
    - Attempt to open the file at the given path in binary read mode.
    - If the file cannot be opened and the error is ENOENT with CMD_PARSE_QUIET flag, return 0; otherwise, log the error and return -1.
    - Initialize a `cmd_parse_input` structure with the provided parameters and default values.
    - Parse the file using `cmd_parse_from_file` and close the file.
    - If parsing results in an error, log the error, free the error message, and return -1.
    - If CMD_PARSE_PARSEONLY flag is set, free the command list and return 0.
    - Create a new command queue state based on the current item or a new state if item is NULL.
    - Add the current file format to the state.
    - Get a new command queue item from the parsed command list and state.
    - Insert the new item after the current item if it exists, otherwise append it to the queue.
    - Free the command list and the command queue state.
    - If `new_item` is not NULL, set it to the newly created command queue item.
    - Return 0 to indicate successful execution.
- **Output**:
    - The function returns 0 on success or -1 on failure, and optionally sets `new_item` to a new command queue item if provided.
- **Functions called**:
    - [`cfg_add_cause`](#cfg_add_cause)


---
### load_cfg_from_buffer<!-- {{#callable:load_cfg_from_buffer}} -->
The `load_cfg_from_buffer` function parses a configuration from a buffer and processes the resulting command list, optionally inserting it into a command queue.
- **Inputs**:
    - `buf`: A pointer to the buffer containing the configuration data to be parsed.
    - `len`: The length of the buffer in bytes.
    - `path`: A string representing the path of the configuration file, used for logging and error reporting.
    - `c`: A pointer to a `struct client` representing the client context for the configuration.
    - `item`: A pointer to a `struct cmdq_item` representing the current command queue item, used for state management.
    - `current`: A pointer to a `struct cmd_find_state` representing the current command find state, used for state copying.
    - `flags`: An integer representing flags that modify the parsing behavior, such as `CMD_PARSE_PARSEONLY`.
    - `new_item`: A pointer to a pointer to a `struct cmdq_item` where the new command queue item will be stored if created.
- **Control Flow**:
    - Initialize the `new_item` to NULL if it is not NULL.
    - Log the loading of the configuration using the provided path.
    - Initialize a `cmd_parse_input` structure with the provided flags, path, line number, item, and client.
    - Parse the buffer using `cmd_parse_from_buffer` and check the result status.
    - If parsing results in an error, log the error, free the error message, and return -1.
    - If the `CMD_PARSE_PARSEONLY` flag is set, free the command list and return 0.
    - Determine the command queue state based on whether `item` is NULL or not, and add the current file format to the state.
    - Get a new command queue item from the parsed command list and state.
    - Insert the new command queue item after the current item if `item` is not NULL, otherwise append it to the queue.
    - Free the command list and command queue state.
    - If `new_item` is not NULL, set it to the newly created command queue item.
    - Return 0 to indicate successful processing.
- **Output**:
    - The function returns 0 on success, or -1 if there is a parsing error. If `new_item` is not NULL, it will point to the newly created command queue item.
- **Functions called**:
    - [`cfg_add_cause`](#cfg_add_cause)


---
### cfg_add_cause<!-- {{#callable:cfg_add_cause}} -->
The `cfg_add_cause` function formats a message and appends it to a dynamically growing list of configuration error causes.
- **Inputs**:
    - `fmt`: A format string that specifies how subsequent arguments are converted for output.
    - `...`: A variable number of arguments that are formatted according to the format string `fmt`.
- **Control Flow**:
    - Initialize a variable argument list `ap` using `va_start` with the format string `fmt`.
    - Use `xvasprintf` to format the message into a dynamically allocated string `msg` using the argument list `ap`.
    - End the variable argument list using `va_end`.
    - Increment the global counter `cfg_ncauses` to reflect the addition of a new cause.
    - Reallocate the `cfg_causes` array to accommodate the new message by increasing its size by one element.
    - Assign the newly formatted message `msg` to the last position in the `cfg_causes` array.
- **Output**:
    - The function does not return a value; it modifies the global `cfg_causes` array and `cfg_ncauses` counter to include the new formatted message.


---
### cfg_print_causes<!-- {{#callable:cfg_print_causes}} -->
The `cfg_print_causes` function prints all configuration error messages stored in `cfg_causes` to a command queue item and then frees the memory allocated for these messages.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure where the configuration error messages will be printed.
- **Control Flow**:
    - Iterates over each error message stored in the `cfg_causes` array using a for loop.
    - For each message, it prints the message to the provided `cmdq_item` using `cmdq_print`.
    - Frees the memory allocated for each individual error message after printing.
    - Frees the memory allocated for the `cfg_causes` array itself.
    - Resets `cfg_causes` to `NULL` and `cfg_ncauses` to `0` to indicate that there are no more error messages.
- **Output**:
    - The function does not return a value; it performs operations on the `cfg_causes` array and the `cmdq_item`.


---
### cfg_show_causes<!-- {{#callable:cfg_show_causes}} -->
The `cfg_show_causes` function displays configuration error messages to a client in control mode or to the active window pane of a session.
- **Inputs**:
    - `s`: A pointer to a `session` structure, representing the session in which to display configuration errors.
- **Control Flow**:
    - Check if there are any configuration causes (`cfg_ncauses` is 0), and return immediately if none exist.
    - Retrieve the first client from the list of clients.
    - If a client exists and is in control mode, iterate over the configuration causes, writing each to the client and freeing the memory, then jump to cleanup.
    - If no session is provided (`s` is NULL), attempt to use the session of the first client or the first session in the session tree.
    - If the session is still NULL or not attached, return without doing anything.
    - Retrieve the active window pane of the current window in the session.
    - Check if the active window pane is in view mode; if not, set it to view mode.
    - Iterate over the configuration causes, adding each to the window pane and freeing the memory.
    - Free the memory allocated for configuration causes and reset the count to zero.
- **Output**:
    - The function does not return a value, but it outputs configuration error messages to either a client in control mode or the active window pane of a session.


