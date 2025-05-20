# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to switch between several programs in one terminal. The file is responsible for handling the configuration loading process of tmux. It defines functions to load configuration files and buffers, manage configuration-related errors, and ensure that the initial client is blocked until the configuration is fully loaded. The code includes functions such as [`start_cfg`](#start_cfg), [`load_cfg`](#load_cfg), and [`load_cfg_from_buffer`](#load_cfg_from_buffer), which are central to reading and parsing configuration files and buffers, respectively. These functions utilize a command queue system to manage the execution of commands parsed from the configuration files.

The file also manages error reporting related to configuration loading through functions like [`cfg_add_cause`](#cfg_add_cause), [`cfg_print_causes`](#cfg_print_causes), and [`cfg_show_causes`](#cfg_show_causes). These functions collect, store, and display any errors encountered during the configuration loading process. The code is structured to ensure that configuration errors are communicated to the user, either through the client interface or by displaying them in a window pane. This file is not an executable on its own but is intended to be part of the larger tmux codebase, providing specific functionality related to configuration management. It does not define public APIs or external interfaces but rather operates within the internal architecture of tmux.
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
- **Use**: This variable is used to determine if the configuration loading is finished, allowing subsequent operations to proceed.


---
### cfg_causes
- **Type**: `char **`
- **Description**: `cfg_causes` is a static global variable that is a pointer to an array of strings (character pointers). It is used to store error messages or causes that occur during the configuration process.
- **Use**: This variable is used to accumulate and store error messages that are generated when loading configuration files, allowing them to be displayed or logged later.


---
### cfg_ncauses
- **Type**: `u_int`
- **Description**: The `cfg_ncauses` variable is a static unsigned integer that keeps track of the number of configuration error messages stored in the `cfg_causes` array. It is used to manage the dynamic allocation and reallocation of the `cfg_causes` array as new error messages are added.
- **Use**: `cfg_ncauses` is incremented each time a new configuration error message is added to the `cfg_causes` array, ensuring the array size matches the number of stored messages.


---
### cfg_item
- **Type**: `struct cmdq_item *`
- **Description**: The `cfg_item` is a static pointer to a `struct cmdq_item`, which is part of the command queue system in the tmux codebase. It is used to manage the execution of commands in a queue, specifically related to configuration loading.
- **Use**: This variable is used to store a command queue item that ensures the initial client command runs after the configuration files are loaded.


---
### cfg_quiet
- **Type**: `int`
- **Description**: The `cfg_quiet` variable is an integer that is initialized to 1. It is used to control the verbosity of the configuration loading process in the program.
- **Use**: This variable is used to determine whether to suppress error messages when loading configuration files.


---
### cfg_files
- **Type**: `char **`
- **Description**: `cfg_files` is a global variable that is a pointer to an array of strings, where each string represents a path to a configuration file. It is used to store the list of configuration files that need to be loaded by the program.
- **Use**: This variable is used in the `start_cfg` function to iterate over and load each configuration file specified in the array.


---
### cfg_nfiles
- **Type**: `u_int`
- **Description**: The `cfg_nfiles` variable is a global variable of type `u_int` (unsigned integer) that holds the number of configuration files to be processed. It is used to iterate over the list of configuration files stored in `cfg_files`. This variable is crucial for determining how many configuration files need to be loaded during the initialization process.
- **Use**: `cfg_nfiles` is used in a loop to load each configuration file specified in the `cfg_files` array.


# Functions

---
### cfg_client_done<!-- {{#callable:cfg_client_done}} -->
The `cfg_client_done` function checks if the configuration process is finished and returns a command return value accordingly.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which is unused in this function.
    - `data`: A void pointer to any additional data, which is also unused in this function.
- **Control Flow**:
    - The function checks the global variable `cfg_finished` to determine if the configuration process is complete.
    - If `cfg_finished` is false, the function returns `CMD_RETURN_WAIT`, indicating that the process should wait.
    - If `cfg_finished` is true, the function returns `CMD_RETURN_NORMAL`, indicating that the process can proceed normally.
- **Output**:
    - The function returns an `enum cmd_retval` value, either `CMD_RETURN_WAIT` or `CMD_RETURN_NORMAL`, based on the state of the `cfg_finished` variable.


---
### cfg_done<!-- {{#callable:cfg_done}} -->
The `cfg_done` function marks the configuration process as finished, displays any configuration errors, continues any pending command queue items, and loads the command history.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which is unused in this function.
    - `data`: A void pointer to any additional data, which is unused in this function.
- **Control Flow**:
    - Check if the configuration process is already finished by evaluating `cfg_finished`.
    - If `cfg_finished` is true, return `CMD_RETURN_NORMAL`.
    - Set `cfg_finished` to 1 to mark the configuration process as complete.
    - Call [`cfg_show_causes`](#cfg_show_causes) to display any configuration errors.
    - If `cfg_item` is not NULL, call [`cmdq_continue`](cmd-queue.c.driver.md#cmdq_continue) to continue processing the command queue item.
    - Call [`status_prompt_load_history`](status.c.driver.md#status_prompt_load_history) to load the command history.
    - Return `CMD_RETURN_NORMAL`.
- **Output**:
    - The function returns `CMD_RETURN_NORMAL`, indicating the normal completion of the command.
- **Functions called**:
    - [`cfg_show_causes`](#cfg_show_causes)
    - [`cmdq_continue`](cmd-queue.c.driver.md#cmdq_continue)
    - [`status_prompt_load_history`](status.c.driver.md#status_prompt_load_history)


---
### start_cfg<!-- {{#callable:start_cfg}} -->
The `start_cfg` function initializes the configuration loading process by setting up the initial client and loading configuration files into the command queue.
- **Inputs**:
    - None
- **Control Flow**:
    - Initialize a client pointer `c` and set `flags` to 0.
    - Check if there is an initial client in the `clients` queue; if so, set up a callback `cfg_client_done` to block the client until configuration is loaded.
    - Set `flags` to `CMD_PARSE_QUIET` if `cfg_quiet` is true.
    - Iterate over each configuration file in `cfg_files` and call [`load_cfg`](#load_cfg) to load them with the specified flags.
    - Append a callback `cfg_done` to the command queue to finalize the configuration process.
- **Output**:
    - The function does not return a value; it sets up the configuration loading process and modifies global state.
- **Functions called**:
    - [`cmdq_append`](cmd-queue.c.driver.md#cmdq_append)
    - [`load_cfg`](#load_cfg)


---
### load_cfg<!-- {{#callable:load_cfg}} -->
The `load_cfg` function loads a configuration file, parses its commands, and optionally appends them to a command queue.
- **Inputs**:
    - `path`: A string representing the file path to the configuration file to be loaded.
    - `c`: A pointer to a `struct client` representing the client context for the configuration loading.
    - `item`: A pointer to a `struct cmdq_item` representing the current command queue item, which can be used to determine the state of the command queue.
    - `current`: A pointer to a `struct cmd_find_state` representing the current command find state, used for state copying.
    - `flags`: An integer representing flags that modify the behavior of the function, such as `CMD_PARSE_QUIET` or `CMD_PARSE_PARSEONLY`.
    - `new_item`: A pointer to a pointer to a `struct cmdq_item` where the new command queue item will be stored if created.
- **Control Flow**:
    - Initialize `new_item` to NULL if it is not NULL.
    - Log the loading of the configuration file using the provided path.
    - Attempt to open the file at the given path in binary read mode.
    - If the file cannot be opened and the error is `ENOENT` with `CMD_PARSE_QUIET` flag set, return 0; otherwise, log the error and return -1.
    - Initialize a `cmd_parse_input` structure with the provided parameters and default values.
    - Parse the file using `cmd_parse_from_file` and close the file afterwards.
    - If parsing results in an error, log the error, free the error message, and return -1.
    - If the `CMD_PARSE_PARSEONLY` flag is set, free the command list and return 0.
    - Determine the command queue state based on whether `item` is NULL or not, and add the current file format to the state.
    - Get a new command queue item from the parsed command list and state.
    - Insert the new command queue item after the current item if `item` is not NULL, otherwise append it to the queue.
    - Free the command list and the command queue state.
    - If `new_item` is not NULL, set it to the newly created command queue item.
    - Return 0 to indicate successful execution.
- **Output**:
    - The function returns 0 on success, or -1 if an error occurs during file opening or command parsing.
- **Functions called**:
    - [`cfg_add_cause`](#cfg_add_cause)
    - [`cmd_list_free`](cmd.c.driver.md#cmd_list_free)
    - [`cmdq_copy_state`](cmd-queue.c.driver.md#cmdq_copy_state)
    - [`cmdq_get_state`](cmd-queue.c.driver.md#cmdq_get_state)
    - [`cmdq_new_state`](cmd-queue.c.driver.md#cmdq_new_state)
    - [`cmdq_get_command`](cmd-queue.c.driver.md#cmdq_get_command)
    - [`cmdq_insert_after`](cmd-queue.c.driver.md#cmdq_insert_after)
    - [`cmdq_append`](cmd-queue.c.driver.md#cmdq_append)
    - [`cmdq_free_state`](cmd-queue.c.driver.md#cmdq_free_state)


---
### load_cfg_from_buffer<!-- {{#callable:load_cfg_from_buffer}} -->
The `load_cfg_from_buffer` function parses a configuration from a buffer and processes the resulting command list, optionally inserting it into a command queue.
- **Inputs**:
    - `buf`: A pointer to the buffer containing the configuration data to be parsed.
    - `len`: The length of the buffer in bytes.
    - `path`: A string representing the path associated with the buffer, used for logging and error reporting.
    - `c`: A pointer to a `struct client` representing the client context for the command execution.
    - `item`: A pointer to a `struct cmdq_item` representing the current command queue item, used for state management.
    - `current`: A pointer to a `struct cmd_find_state` representing the current command find state, used for state copying.
    - `flags`: An integer representing flags that modify the parsing behavior, such as `CMD_PARSE_PARSEONLY`.
    - `new_item`: A pointer to a pointer to a `struct cmdq_item` where the new command queue item will be stored if not NULL.
- **Control Flow**:
    - Initialize the `new_item` to NULL if it is not NULL.
    - Log the loading of the configuration using the provided path.
    - Initialize a `cmd_parse_input` structure with the provided flags, path, line number, item, and client.
    - Parse the buffer using `cmd_parse_from_buffer` and store the result in `pr`.
    - If parsing results in an error, add the error to the configuration causes, free the error string, and return -1.
    - If the `CMD_PARSE_PARSEONLY` flag is set, free the command list and return 0.
    - Determine the command queue state based on whether `item` is NULL or not, copying or creating a new state accordingly.
    - Add the current file format to the command queue state.
    - Get a new command queue item from the parsed command list and state.
    - Insert or append the new command queue item into the queue based on whether `item` is NULL or not.
    - Free the command list and the command queue state.
    - If `new_item` is not NULL, set it to the new command queue item.
    - Return 0 to indicate success.
- **Output**:
    - The function returns 0 on success or -1 if there is a parsing error, and optionally sets `new_item` to the newly created command queue item.
- **Functions called**:
    - [`cfg_add_cause`](#cfg_add_cause)
    - [`cmd_list_free`](cmd.c.driver.md#cmd_list_free)
    - [`cmdq_copy_state`](cmd-queue.c.driver.md#cmdq_copy_state)
    - [`cmdq_get_state`](cmd-queue.c.driver.md#cmdq_get_state)
    - [`cmdq_new_state`](cmd-queue.c.driver.md#cmdq_new_state)
    - [`cmdq_get_command`](cmd-queue.c.driver.md#cmdq_get_command)
    - [`cmdq_insert_after`](cmd-queue.c.driver.md#cmdq_insert_after)
    - [`cmdq_append`](cmd-queue.c.driver.md#cmdq_append)
    - [`cmdq_free_state`](cmd-queue.c.driver.md#cmdq_free_state)


---
### cfg_add_cause<!-- {{#callable:cfg_add_cause}} -->
The `cfg_add_cause` function formats a message and appends it to a dynamically allocated array of configuration error messages.
- **Inputs**:
    - `fmt`: A format string that specifies how subsequent arguments are converted for output.
    - `...`: A variable number of arguments that are formatted according to the format string `fmt`.
- **Control Flow**:
    - Initialize a `va_list` variable `ap` to retrieve the additional arguments passed to the function.
    - Use [`xvasprintf`](xmalloc.c.driver.md#xvasprintf) to format the message according to `fmt` and store the result in `msg`.
    - Increment the global variable `cfg_ncauses` to reflect the addition of a new cause.
    - Reallocate memory for the `cfg_causes` array to accommodate the new message using [`xreallocarray`](xmalloc.c.driver.md#xreallocarray).
    - Store the formatted message `msg` in the last position of the `cfg_causes` array.
- **Output**:
    - The function does not return a value; it modifies the global `cfg_causes` array and `cfg_ncauses` counter to include the new formatted message.
- **Functions called**:
    - [`xvasprintf`](xmalloc.c.driver.md#xvasprintf)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)


---
### cfg_print_causes<!-- {{#callable:cfg_print_causes}} -->
The `cfg_print_causes` function prints all configuration error messages stored in `cfg_causes` to a command queue item and then frees the memory used by these messages.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure where the configuration error messages will be printed.
- **Control Flow**:
    - Iterates over each error message stored in the `cfg_causes` array using a for loop.
    - For each message, it calls `cmdq_print` to print the message to the provided `cmdq_item`.
    - Frees the memory allocated for each message in the `cfg_causes` array after printing.
    - Frees the memory allocated for the `cfg_causes` array itself.
    - Sets `cfg_causes` to `NULL` and `cfg_ncauses` to `0` to reset the state.
- **Output**:
    - The function does not return a value; it performs operations on the `cfg_causes` array and the `cmdq_item`.


---
### cfg_show_causes<!-- {{#callable:cfg_show_causes}} -->
The `cfg_show_causes` function displays configuration error messages to either a control client or an active window pane in a session.
- **Inputs**:
    - `s`: A pointer to a `session` structure, which represents the current session context where configuration errors might be displayed.
- **Control Flow**:
    - Check if there are any configuration causes (`cfg_ncauses` is 0), and return immediately if none exist.
    - Retrieve the first client from the global clients list.
    - If a control client is present, iterate over each configuration cause, write it to the control client, and free the cause memory.
    - If no session is provided (`s` is NULL), attempt to assign the session from the first client or the first session in the global sessions list.
    - If the session is still NULL or not attached, return without doing anything further.
    - Retrieve the active window pane from the current session's window.
    - Check if the window pane is in view mode; if not, set it to view mode.
    - Iterate over each configuration cause, add it to the window pane's copy buffer, and free the cause memory.
    - Free the memory allocated for the causes array and reset the causes count to zero.
- **Output**:
    - The function does not return a value but outputs configuration error messages to either a control client or a window pane, and it frees the memory allocated for the configuration causes.
- **Functions called**:
    - [`window_pane_set_mode`](window.c.driver.md#window_pane_set_mode)


