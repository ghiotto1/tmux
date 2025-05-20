# Purpose
This C source code file is part of the `tmux` project, a terminal multiplexer, and it provides functionality for capturing the contents of a terminal pane. The file defines two primary command entries: `capture-pane` and `clear-history`. The `capture-pane` command is responsible for capturing the text displayed in a terminal pane, either from the current screen or from the pane's history, and writing it to a buffer or directly to standard output. The `clear-history` command, on the other hand, clears the history of a specified pane, effectively resetting its state.

The file includes several static functions that implement the core logic for these commands. The [`cmd_capture_pane_exec`](#cmd_capture_pane_exec) function is the main execution function for both commands, determining whether to capture the pane's current content or clear its history based on the command entry. The [`cmd_capture_pane_pending`](#cmd_capture_pane_pending) and [`cmd_capture_pane_history`](#cmd_capture_pane_history) functions handle the retrieval of the pane's current and historical content, respectively, and format it according to the specified options. The code is structured to be part of a larger application, with command entries and execution functions that integrate with the `tmux` command queue and argument handling system. This file does not define public APIs or external interfaces but rather contributes to the internal command processing of the `tmux` application.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### cmd_capture_pane_append
- **Type**: `function pointer`
- **Description**: `cmd_capture_pane_append` is a static function that appends a line of text to a buffer, reallocating memory as necessary to accommodate the new data. It takes a buffer, its current length, a line to append, and the line's length as parameters.
- **Use**: This function is used to build a complete buffer by appending lines of text, typically when capturing the contents of a pane.


---
### cmd_capture_pane_pending
- **Type**: `function pointer`
- **Description**: `cmd_capture_pane_pending` is a static function pointer that takes three parameters: a pointer to an `args` structure, a pointer to a `window_pane` structure, and a pointer to a `size_t` variable. It returns a pointer to a character array (string).
- **Use**: This function is used to capture the pending input data from a window pane and return it as a string, potentially modifying the string based on the provided arguments.


---
### cmd_capture_pane_history
- **Type**: `function pointer`
- **Description**: `cmd_capture_pane_history` is a static function that captures the history of a specified window pane in a terminal multiplexer environment. It processes the grid data of the pane, applying various flags and options to format the output, and returns the captured content as a dynamically allocated string.
- **Use**: This function is used to retrieve and format the historical content of a window pane, which can then be output or stored as needed.


---
### cmd_capture_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_capture_pane_entry` is a constant structure of type `cmd_entry` that defines the command entry for the 'capture-pane' command in the tmux application. It includes the command's name, alias, argument specifications, usage instructions, target specifications, flags, and the function to execute the command.
- **Use**: This variable is used to register and define the behavior of the 'capture-pane' command within the tmux command framework.


---
### cmd_clear_history_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_clear_history_entry` is a constant structure of type `cmd_entry` that defines the command for clearing the history of a pane in the tmux terminal multiplexer. It includes fields such as the command name, alias, arguments, usage, target, flags, and the execution function. This structure is used to register and handle the 'clear-history' command within the tmux application.
- **Use**: This variable is used to define and execute the 'clear-history' command in tmux, which clears the history of a specified pane.


# Functions

---
### cmd_capture_pane_append<!-- {{#callable:cmd_capture_pane_append}} -->
The function `cmd_capture_pane_append` appends a line of text to a buffer, reallocating memory as needed.
- **Inputs**:
    - `buf`: A pointer to the buffer where the line will be appended.
    - `len`: A pointer to the size of the buffer, which will be updated to reflect the new size after appending the line.
    - `line`: A pointer to the line of text to be appended to the buffer.
    - `linelen`: The length of the line to be appended.
- **Control Flow**:
    - Reallocate memory for the buffer to accommodate the new line plus a null terminator using [`xrealloc`](xmalloc.c.driver.md#xrealloc).
    - Copy the line into the buffer at the position indicated by the current length using `memcpy`.
    - Update the length of the buffer to include the newly appended line.
- **Output**:
    - Returns a pointer to the reallocated buffer containing the appended line.
- **Functions called**:
    - [`xrealloc`](xmalloc.c.driver.md#xrealloc)


---
### cmd_capture_pane_pending<!-- {{#callable:cmd_capture_pane_pending}} -->
The function `cmd_capture_pane_pending` captures the pending input data from a window pane and formats it into a string buffer, optionally escaping non-printable characters.
- **Inputs**:
    - `args`: A pointer to a struct `args` which contains command-line arguments that may modify the function's behavior.
    - `wp`: A pointer to a struct `window_pane` representing the pane from which pending input data is captured.
    - `len`: A pointer to a size_t variable where the length of the resulting buffer will be stored.
- **Control Flow**:
    - Retrieve the pending input buffer from the window pane's input context using [`input_pending`](input.c.driver.md#input_pending) function.
    - If the pending buffer is NULL, return an empty string using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - Extract the data and length of the pending buffer using `EVBUFFER_DATA` and `EVBUFFER_LENGTH`.
    - Initialize an empty string buffer using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - Check if the 'C' flag is present in the arguments using [`args_has`](arguments.c.driver.md#args_has).
    - If the 'C' flag is present, iterate over each character in the pending data.
    - For each character, if it is a printable character and not a backslash, append it directly to the buffer.
    - If it is not a printable character or is a backslash, format it as an escaped octal sequence and append it to the buffer.
    - If the 'C' flag is not present, append the entire pending data to the buffer without modification.
    - Return the constructed buffer.
- **Output**:
    - A dynamically allocated string containing the formatted pending input data from the window pane, with its length stored in the provided `len` pointer.
- **Functions called**:
    - [`input_pending`](input.c.driver.md#input_pending)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)
    - [`cmd_capture_pane_append`](#cmd_capture_pane_append)


---
### cmd_capture_pane_history<!-- {{#callable:cmd_capture_pane_history}} -->
The `cmd_capture_pane_history` function captures the history of a specified window pane and returns it as a string buffer, optionally formatted according to various flags.
- **Inputs**:
    - `args`: A pointer to a struct containing command-line arguments that specify options for capturing the pane history.
    - `item`: A pointer to a command queue item, used for error reporting and context.
    - `wp`: A pointer to the window pane whose history is to be captured.
    - `len`: A pointer to a size_t variable where the length of the captured buffer will be stored.
- **Control Flow**:
    - Determine the grid to capture from based on the presence of the 'a' or 'M' flags, defaulting to the base grid if neither is present.
    - Calculate the starting line (`top`) based on the 'S' flag, defaulting to the top of the grid if the flag is not set or invalid.
    - Calculate the ending line (`bottom`) based on the 'E' flag, defaulting to the bottom of the grid if the flag is not set or invalid.
    - Swap `top` and `bottom` if `bottom` is less than `top`.
    - Set flags for line joining and formatting based on the presence of 'J', 'e', 'C', 'T', and 'N' flags.
    - Iterate over each line from `top` to `bottom`, converting each line to a string and appending it to the buffer, adding a newline unless lines are joined or wrapped.
    - Return the constructed buffer containing the captured pane history.
- **Output**:
    - A dynamically allocated string buffer containing the captured pane history, formatted according to the specified flags, or NULL if an error occurs.
- **Functions called**:
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`args_get`](arguments.c.driver.md#args_get)
    - [`args_strtonum_and_expand`](arguments.c.driver.md#args_strtonum_and_expand)
    - [`grid_string_cells`](grid.c.driver.md#grid_string_cells)
    - [`cmd_capture_pane_append`](#cmd_capture_pane_append)
    - [`grid_peek_line`](grid.c.driver.md#grid_peek_line)


---
### cmd_capture_pane_exec<!-- {{#callable:cmd_capture_pane_exec}} -->
The `cmd_capture_pane_exec` function captures the contents of a window pane and either writes it to a buffer or outputs it to a client, depending on the provided arguments.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve the command arguments, client, and target window pane using helper functions.
    - Check if the command entry is `cmd_clear_history_entry`; if so, reset the pane's mode, clear its history, and reset hyperlinks if specified, then return `CMD_RETURN_NORMAL`.
    - Initialize the buffer length to zero and determine whether to capture pending input or pane history based on the presence of the 'P' argument.
    - If the buffer is NULL, return `CMD_RETURN_ERROR`.
    - If the 'p' argument is present, adjust the buffer length if it ends with a newline, and write the buffer to the client or print it to the file, handling errors appropriately.
    - If the 'p' argument is not present, optionally retrieve a buffer name from the arguments and attempt to set the paste buffer, handling errors appropriately.
    - Return `CMD_RETURN_NORMAL` if successful.
- **Output**:
    - The function returns an `enum cmd_retval`, which is either `CMD_RETURN_NORMAL` on success or `CMD_RETURN_ERROR` on failure.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client)
    - [`cmdq_get_target`](cmd-queue.c.driver.md#cmdq_get_target)
    - [`cmd_get_entry`](cmd.c.driver.md#cmd_get_entry)
    - [`window_pane_reset_mode_all`](window.c.driver.md#window_pane_reset_mode_all)
    - [`grid_clear_history`](grid.c.driver.md#grid_clear_history)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`screen_reset_hyperlinks`](screen.c.driver.md#screen_reset_hyperlinks)
    - [`cmd_capture_pane_pending`](#cmd_capture_pane_pending)
    - [`cmd_capture_pane_history`](#cmd_capture_pane_history)
    - [`file_can_print`](file.c.driver.md#file_can_print)
    - [`file_print_buffer`](file.c.driver.md#file_print_buffer)
    - [`args_get`](arguments.c.driver.md#args_get)
    - [`paste_set`](paste.c.driver.md#paste_set)


