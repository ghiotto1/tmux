# Purpose
This C source code file is part of the `tmux` project, a terminal multiplexer, and it implements the functionality to source configuration files. The primary purpose of this code is to read and execute commands from specified configuration files, allowing users to customize their `tmux` environment. The code defines a command entry, `cmd_source_file_entry`, which includes the command name, alias, arguments, usage, and execution function. The execution function, [`cmd_source_file_exec`](#cmd_source_file_exec), handles the logic for processing the command, including resolving file paths, handling glob patterns, and managing nested file sourcing with a depth limit to prevent excessive recursion.

The code is structured around several key components, including the `cmd_source_file_data` structure, which holds information about the command execution context, such as the list of files to be sourced and flags indicating command options. The code also includes functions for handling the completion of file sourcing ([`cmd_source_file_complete`](#cmd_source_file_complete)), managing the reading of files ([`cmd_source_file_done`](#cmd_source_file_done)), and adding files to the list of files to be sourced ([`cmd_source_file_add`](#cmd_source_file_add)). The implementation ensures that errors are logged and reported appropriately, and it supports various command-line options to control the behavior of the sourcing process, such as quiet mode, parse-only mode, and verbose mode. This file is intended to be part of the broader `tmux` codebase, providing a specific functionality related to configuration management.
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
- **Description**: The `cmd_source_file_depth` is a static unsigned integer variable that tracks the current depth of nested configuration file sourcing operations. It is used to prevent excessive recursion when sourcing configuration files.
- **Use**: This variable is incremented each time a configuration file is sourced and decremented when the operation completes, ensuring the depth does not exceed a predefined limit.


---
### cmd_source_file_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_source_file_entry` is a constant structure of type `cmd_entry` that defines the command 'source-file' and its alias 'source'. It specifies the command's arguments, usage, target, flags, and execution function.
- **Use**: This variable is used to register and define the behavior of the 'source-file' command within the application, allowing it to source configuration files.


# Data Structures

---
### cmd_source_file_data
- **Type**: `struct`
- **Members**:
    - `item`: A pointer to a cmdq_item structure, representing the current command queue item.
    - `flags`: An integer representing various flags for command parsing.
    - `after`: A pointer to a cmdq_item structure, representing the command queue item to execute after the current one.
    - `retval`: An enum value of type cmd_retval, indicating the return value of the command execution.
    - `current`: An unsigned integer representing the current index in the files array.
    - `files`: A pointer to an array of strings, each representing a file path to be sourced.
    - `nfiles`: An unsigned integer representing the number of files in the files array.
- **Description**: The `cmd_source_file_data` structure is used to manage the state and data required for sourcing configuration files in a command execution context. It holds pointers to command queue items, flags for parsing options, a return value for the command execution, and an array of file paths to be processed. This structure facilitates the execution of commands that involve reading and processing multiple configuration files, maintaining the current state and progress through the files.


# Functions

---
### cmd_source_file_complete_cb<!-- {{#callable:cmd_source_file_complete_cb}} -->
The function `cmd_source_file_complete_cb` decrements the source file depth counter for a client or globally and logs the current depth, then prints configuration causes and returns a normal command return value.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item associated with the callback.
    - `data`: A void pointer to additional data, which is unused in this function.
- **Control Flow**:
    - Retrieve the client associated with the command queue item using `cmdq_get_client`.
    - Check if the client is NULL; if so, decrement the global `cmd_source_file_depth` counter and log the new depth.
    - If the client is not NULL, decrement the client's `source_file_depth` counter and log the new depth.
    - Call `cfg_print_causes` to print any configuration causes associated with the item.
    - Return `CMD_RETURN_NORMAL` to indicate normal command completion.
- **Output**:
    - The function returns an `enum cmd_retval` value, specifically `CMD_RETURN_NORMAL`, indicating normal command completion.


---
### cmd_source_file_complete<!-- {{#callable:cmd_source_file_complete}} -->
The `cmd_source_file_complete` function finalizes the processing of sourced configuration files by handling return values, queuing a callback, and freeing allocated resources.
- **Inputs**:
    - `c`: A pointer to a `client` structure, representing the client context in which the command is executed.
    - `cdata`: A pointer to a `cmd_source_file_data` structure, containing data related to the source file command execution, such as file paths and return values.
- **Control Flow**:
    - Check if the configuration processing is finished (`cfg_finished`).
    - If the return value (`retval`) in `cdata` is `CMD_RETURN_ERROR`, and the client `c` is not null and has no session, set the client's return value (`retval`) to 1.
    - Create a new command queue item (`new_item`) with a callback to `cmd_source_file_complete_cb`.
    - Insert the new command queue item after the item specified in `cdata->after`.
    - Iterate over each file in `cdata->files` and free the memory allocated for each file path.
    - Free the memory allocated for the `files` array and the `cdata` structure itself.
- **Output**:
    - The function does not return a value; it performs cleanup and queuing operations as side effects.


---
### cmd_source_file_done<!-- {{#callable:cmd_source_file_done}} -->
The `cmd_source_file_done` function handles the completion of reading a source file, processes its content, and manages the continuation of command queue execution.
- **Inputs**:
    - `c`: A pointer to the client structure, representing the client context in which the command is executed.
    - `path`: A string representing the path of the file that was read.
    - `error`: An integer indicating if there was an error during the file reading process.
    - `closed`: An integer flag indicating whether the file reading process is complete.
    - `buffer`: A pointer to an evbuffer structure containing the data read from the file.
    - `data`: A pointer to a cmd_source_file_data structure containing metadata and state information for the file sourcing process.
- **Control Flow**:
    - Check if the file reading process is not closed; if so, return immediately.
    - If there is an error, log the error message using cmdq_error.
    - If there is no error and the buffer size is not zero, attempt to load the configuration from the buffer using load_cfg_from_buffer.
    - If loading the configuration fails, set the return value to CMD_RETURN_ERROR; otherwise, update the 'after' field with the new command queue item if it exists.
    - Increment the current file index in cdata and check if there are more files to read.
    - If more files exist, initiate reading the next file using file_read.
    - If no more files exist, call cmd_source_file_complete to finalize the process and continue the command queue execution with cmdq_continue.
- **Output**:
    - The function does not return a value but updates the state of the cmd_source_file_data structure and potentially modifies the command queue execution flow.
- **Functions called**:
    - [`cmd_source_file_complete`](#cmd_source_file_complete)


---
### cmd_source_file_add<!-- {{#callable:cmd_source_file_add}} -->
The `cmd_source_file_add` function adds a file path to a list of files in a `cmd_source_file_data` structure, resolving the path to an absolute path if possible.
- **Inputs**:
    - `cdata`: A pointer to a `cmd_source_file_data` structure where the file path will be added.
    - `path`: A string representing the file path to be added to the `cdata` structure.
- **Control Flow**:
    - Attempt to resolve the given `path` to an absolute path using `realpath`; if it fails, log an error message.
    - If `realpath` succeeds, update `path` to the resolved absolute path.
    - Log the path being added for debugging purposes.
    - Reallocate memory for the `files` array in `cdata` to accommodate one more file path.
    - Duplicate the `path` string and add it to the `files` array in `cdata`.
    - Increment the `nfiles` counter in `cdata` to reflect the addition of the new file path.
- **Output**:
    - The function does not return a value; it modifies the `cdata` structure by adding a new file path to its `files` array.


---
### cmd_source_file_exec<!-- {{#callable:cmd_source_file_exec}} -->
The `cmd_source_file_exec` function processes and executes a list of configuration files, handling file path expansion, globbing, and error reporting, while managing nested file depth limits.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item associated with this execution.
- **Control Flow**:
    - Retrieve the command arguments using `cmd_get_args` and the client using `cmdq_get_client`.
    - Check if the client is NULL and manage the global or client-specific source file depth, returning an error if the depth limit is exceeded.
    - Allocate and initialize a `cmd_source_file_data` structure to store execution data.
    - Set flags in `cdata` based on the presence of specific command-line options ('q', 'n', 'v').
    - Get the current working directory of the client and store it in `cwd`.
    - Iterate over each argument provided to the command, expanding paths if the 'F' option is set.
    - For each path, determine if it is absolute or relative, and construct a glob pattern accordingly.
    - Use `glob` to expand the pattern into a list of files, handling errors and reporting them if necessary.
    - Add each resolved file path to the `cdata` structure using [`cmd_source_file_add`](#cmd_source_file_add).
    - If files were added to `cdata`, initiate reading the first file with `file_read`, otherwise complete the command execution with [`cmd_source_file_complete`](#cmd_source_file_complete).
    - Free allocated memory for `expanded` and `cwd` before returning the final command return value.
- **Output**:
    - The function returns an `enum cmd_retval` indicating the result of the command execution, which can be `CMD_RETURN_NORMAL`, `CMD_RETURN_ERROR`, or `CMD_RETURN_WAIT` depending on the processing outcome.
- **Functions called**:
    - [`cmd_source_file_add`](#cmd_source_file_add)
    - [`cmd_source_file_complete`](#cmd_source_file_complete)


# Function Declarations (Public API)

---
### cmd_source_file_exec<!-- {{#callable_declaration:cmd_source_file_exec}} -->
Sources a configuration file.
- **Description**: This function is used to source a configuration file, allowing the execution of commands specified within the file. It supports various options to control the behavior of the sourcing process, such as quiet mode, parse-only mode, and verbose mode. The function handles file path expansion and globbing, and it manages nested file sourcing with a depth limit to prevent excessive recursion. It should be called with a valid command and command queue item, and it expects the paths to be valid and accessible. The function returns a status indicating the success or failure of the operation.
- **Inputs**:
    - `self`: A pointer to a struct cmd representing the command to be executed. Must not be null.
    - `item`: A pointer to a struct cmdq_item representing the command queue item. Must not be null.
- **Output**: Returns an enum cmd_retval indicating the result of the operation, which can be CMD_RETURN_NORMAL, CMD_RETURN_ERROR, or CMD_RETURN_WAIT.
- **See also**: [`cmd_source_file_exec`](#cmd_source_file_exec)  (Implementation)


