# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it implements the functionality to source configuration files. The primary purpose of this code is to read and execute commands from specified configuration files, allowing users to customize their tmux environment. The code defines a command entry, `cmd_source_file_entry`, which includes the command name, alias, arguments, usage, and execution function. The execution function, [`cmd_source_file_exec`](#cmd_source_file_exec), handles the logic for processing the command, including managing nested file sourcing with a depth limit to prevent excessive recursion.

The code is structured around several key components, including the `cmd_source_file_data` structure, which holds information about the command execution context, such as the list of files to be sourced and flags indicating command behavior. The code uses functions like [`cmd_source_file_add`](#cmd_source_file_add) to manage file paths and [`cmd_source_file_done`](#cmd_source_file_done) to handle the completion of file reading. It also employs the `glob` function to expand file patterns, allowing for flexible file specification. The implementation ensures that errors are logged and handled appropriately, and it supports various command flags to modify behavior, such as quiet mode or verbose output. This file is intended to be part of the tmux codebase, providing a specific feature rather than serving as a standalone executable or library.
# Imports and Dependencies

---
- `sys/types.h`
- `errno.h`
- `glob.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### cmd_source_file_depth
- **Type**: `u_int`
- **Description**: The `cmd_source_file_depth` is a static unsigned integer variable that tracks the depth of nested configuration file sourcing operations. It is used to prevent excessive recursion when sourcing configuration files.
- **Use**: This variable is incremented each time a configuration file is sourced and decremented when the sourcing is complete, ensuring the depth does not exceed a predefined limit.


---
### cmd_source_file_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_source_file_entry` is a constant structure of type `cmd_entry` that defines the command 'source-file' and its alias 'source'. It specifies the command's arguments, usage, target, flags, and the function to execute when the command is invoked.
- **Use**: This variable is used to register and define the behavior of the 'source-file' command within the application, allowing it to source configuration files.


# Data Structures

---
### cmd_source_file_data
- **Type**: `struct`
- **Members**:
    - `item`: A pointer to a cmdq_item structure, representing a command queue item.
    - `flags`: An integer representing various flags for command parsing.
    - `after`: A pointer to a cmdq_item structure, indicating the item after which new items are inserted.
    - `retval`: An enum cmd_retval indicating the return value of the command execution.
    - `current`: An unsigned integer representing the current index in the files array.
    - `files`: A pointer to an array of strings, each representing a file path to be sourced.
    - `nfiles`: An unsigned integer representing the number of files in the files array.
- **Description**: The `cmd_source_file_data` structure is used to manage the state and data associated with sourcing configuration files in the tmux application. It holds information about the command queue item being processed, flags for command parsing, the current file being processed, and a list of file paths to be sourced. This structure facilitates the execution of the 'source-file' command by tracking the progress and handling the completion of file sourcing operations.


# Functions

---
### cmd_source_file_complete_cb<!-- {{#callable:cmd_source_file_complete_cb}} -->
The function `cmd_source_file_complete_cb` decrements the source file depth counter for a client or globally and logs the current depth, then prints configuration causes and returns a normal command return value.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item associated with the source file operation.
    - `data`: An unused pointer, typically used for passing additional data, but not utilized in this function.
- **Control Flow**:
    - Retrieve the client associated with the command queue item using [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client).
    - Check if the client is `NULL`.
    - If the client is `NULL`, decrement the global `cmd_source_file_depth` counter and log the new depth.
    - If the client is not `NULL`, decrement the client's `source_file_depth` counter and log the new depth.
    - Call [`cfg_print_causes`](cfg.c.driver.md#cfg_print_causes) to print any configuration causes associated with the command queue item.
    - Return `CMD_RETURN_NORMAL` to indicate normal command completion.
- **Output**:
    - The function returns `CMD_RETURN_NORMAL`, indicating that the command completed normally.
- **Functions called**:
    - [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client)
    - [`cfg_print_causes`](cfg.c.driver.md#cfg_print_causes)


---
### cmd_source_file_complete<!-- {{#callable:cmd_source_file_complete}} -->
The function `cmd_source_file_complete` finalizes the processing of sourced files by handling errors, queuing a callback, and freeing allocated resources.
- **Inputs**:
    - `c`: A pointer to a `client` structure, representing the client context in which the command is executed.
    - `cdata`: A pointer to a `cmd_source_file_data` structure, containing data related to the source file command execution, including file paths and execution state.
- **Control Flow**:
    - Check if the configuration processing is finished (`cfg_finished`).
    - If there was an error (`cdata->retval == CMD_RETURN_ERROR`) and the client `c` is not null and has no session, set the client's return value to 1.
    - Create a new command queue item with a callback to `cmd_source_file_complete_cb` and insert it after the current command queue item (`cdata->after`).
    - Iterate over the list of files in `cdata`, freeing each file path string.
    - Free the array of file paths and the `cdata` structure itself.
- **Output**:
    - The function does not return a value; it performs cleanup and state management tasks.
- **Functions called**:
    - [`cmdq_insert_after`](cmd-queue.c.driver.md#cmdq_insert_after)


---
### cmd_source_file_done<!-- {{#callable:cmd_source_file_done}} -->
The `cmd_source_file_done` function handles the completion of reading a source file, processes its content, and manages the continuation of command queue execution.
- **Inputs**:
    - `c`: A pointer to the client structure, representing the client context in which the command is executed.
    - `path`: A constant character pointer representing the path of the source file being processed.
    - `error`: An integer indicating if there was an error during file reading; 0 means no error.
    - `closed`: An integer flag indicating whether the file reading is complete (non-zero if closed).
    - `buffer`: A pointer to an evbuffer structure containing the data read from the file.
    - `data`: A void pointer to user-defined data, specifically a pointer to a `cmd_source_file_data` structure in this context.
- **Control Flow**:
    - Check if the file is closed; if not, return immediately.
    - If there is an error, log the error message using `cmdq_error`.
    - If there is no error and the buffer size is non-zero, attempt to load the configuration from the buffer using [`load_cfg_from_buffer`](cfg.c.driver.md#load_cfg_from_buffer).
    - If loading the configuration fails, set the return value to `CMD_RETURN_ERROR`; otherwise, update `cdata->after` if a new command queue item is created.
    - Increment the current file index in `cdata` and check if there are more files to process.
    - If more files exist, initiate reading the next file using [`file_read`](file.c.driver.md#file_read).
    - If all files have been processed, call [`cmd_source_file_complete`](#cmd_source_file_complete) and continue the command queue with [`cmdq_continue`](cmd-queue.c.driver.md#cmdq_continue).
- **Output**:
    - The function does not return a value but modifies the state of the `cmd_source_file_data` structure and potentially the command queue.
- **Functions called**:
    - [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target)
    - [`load_cfg_from_buffer`](cfg.c.driver.md#load_cfg_from_buffer)
    - [`file_read`](file.c.driver.md#file_read)
    - [`cmd_source_file_complete`](#cmd_source_file_complete)
    - [`cmdq_continue`](cmd-queue.c.driver.md#cmdq_continue)


---
### cmd_source_file_add<!-- {{#callable:cmd_source_file_add}} -->
The `cmd_source_file_add` function resolves a file path and adds it to a list of files in a `cmd_source_file_data` structure.
- **Inputs**:
    - `cdata`: A pointer to a `cmd_source_file_data` structure that holds information about the files to be sourced.
    - `path`: A constant character pointer representing the file path to be added to the `cdata` structure.
- **Control Flow**:
    - Attempt to resolve the absolute path of the given `path` using `realpath` and store it in the `resolved` buffer.
    - If `realpath` fails, log a debug message indicating the failure and the error message.
    - If `realpath` succeeds, update `path` to point to the resolved path.
    - Log a debug message with the function name and the (possibly resolved) path.
    - Reallocate memory for the `files` array in `cdata` to accommodate one more file.
    - Duplicate the (possibly resolved) path string and store it in the `files` array at the current index, then increment the `nfiles` counter.
- **Output**:
    - The function does not return a value; it modifies the `cdata` structure by adding a new file path to its `files` array.
- **Functions called**:
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### cmd_source_file_exec<!-- {{#callable:cmd_source_file_exec}} -->
The `cmd_source_file_exec` function processes and executes commands from specified configuration files, handling file path expansion, globbing, and nested file depth limits.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve the command arguments using [`cmd_get_args`](cmd.c.driver.md#cmd_get_args) and the client using [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client).
    - Check if the client is NULL and manage the global or client-specific source file depth, returning an error if the depth limit is exceeded.
    - Allocate and initialize a `cmd_source_file_data` structure to store command execution data.
    - Set flags in `cdata` based on the presence of specific command arguments ('q', 'n', 'v').
    - Get the current working directory of the client and store it in `cwd`.
    - Iterate over each argument, expanding paths if the 'F' flag is set, and handle '-' as a special case to add directly to `cdata`.
    - For each path, construct a pattern for globbing, either using the absolute path or combining with `cwd`.
    - Perform globbing on the pattern and handle errors, adding matched files to `cdata` or logging errors if no matches are found.
    - Free the expanded path and pattern memory after use.
    - If files are added to `cdata`, initiate reading the first file with [`file_read`](file.c.driver.md#file_read), otherwise complete the command execution with [`cmd_source_file_complete`](#cmd_source_file_complete).
    - Free the `cwd` memory and return the command execution result.
- **Output**:
    - Returns an `enum cmd_retval` indicating the result of the command execution, which can be `CMD_RETURN_NORMAL`, `CMD_RETURN_ERROR`, or `CMD_RETURN_WAIT`.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`utf8_stravis`](utf8.c.driver.md#utf8_stravis)
    - [`server_client_get_cwd`](server-client.c.driver.md#server_client_get_cwd)
    - [`args_count`](arguments.c.driver.md#args_count)
    - [`args_string`](arguments.c.driver.md#args_string)
    - [`format_single_from_target`](format.c.driver.md#format_single_from_target)
    - [`cmd_source_file_add`](#cmd_source_file_add)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`file_read`](file.c.driver.md#file_read)
    - [`cmd_source_file_complete`](#cmd_source_file_complete)


