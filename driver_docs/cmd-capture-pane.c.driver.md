# Purpose
This C source code file is part of the `tmux` project, a terminal multiplexer, and it provides functionality for capturing the contents of a terminal pane. The file defines two primary command entries: `capture-pane` and `clear-history`. The `capture-pane` command is responsible for capturing the text displayed in a terminal pane, either to a buffer or directly to standard output, with options to specify the range of lines to capture and whether to include escape sequences. The `clear-history` command clears the scrollback history of a pane, effectively resetting its state. The code includes functions to handle the capture of both pending input and historical data from the pane, and it supports various options to customize the output format and destination.

The file is structured around the [`cmd_capture_pane_exec`](#cmd_capture_pane_exec) function, which executes the capture or clear operation based on the command entry invoked. It utilizes helper functions like [`cmd_capture_pane_append`](#cmd_capture_pane_append), [`cmd_capture_pane_pending`](#cmd_capture_pane_pending), and [`cmd_capture_pane_history`](#cmd_capture_pane_history) to manage the data retrieval and formatting processes. The code is designed to be integrated into the larger `tmux` application, as indicated by its inclusion of the `tmux.h` header and its use of `tmux`-specific data structures and functions. The file defines public command entries that extend the functionality of `tmux`, allowing users to interact with terminal panes programmatically through the `capture-pane` and `clear-history` commands.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### cmd_capture_pane_append
- **Type**: `function`
- **Description**: The `cmd_capture_pane_append` function is a static function that appends a line of text to a buffer, reallocating memory as necessary to accommodate the new data. It takes a buffer, its current length, a line to append, and the line's length as parameters.
- **Use**: This function is used to build up a buffer by appending lines of text, typically when capturing the contents of a pane.


---
### cmd_capture_pane_pending
- **Type**: `function pointer`
- **Description**: `cmd_capture_pane_pending` is a static function pointer that takes three parameters: a pointer to an `args` structure, a pointer to a `window_pane` structure, and a pointer to a `size_t` variable. It returns a pointer to a character array (string).
- **Use**: This function is used to capture the pending input data from a specified window pane and return it as a string, potentially modifying the length of the captured data.


---
### cmd_capture_pane_history
- **Type**: `function pointer`
- **Description**: `cmd_capture_pane_history` is a static function pointer that captures the history of a window pane in the tmux application. It takes arguments related to command execution, the window pane to capture, and a pointer to store the size of the captured data.
- **Use**: This function is used to retrieve and format the historical content of a window pane, which can then be output or stored as needed.


---
### cmd_capture_pane_entry
- **Type**: `const struct cmd_entry`
- **Description**: The `cmd_capture_pane_entry` is a constant structure of type `cmd_entry` that defines the command entry for the 'capture-pane' command in the tmux application. It includes the command's name, alias, argument specifications, usage instructions, target specifications, flags, and the function to execute the command.
- **Use**: This variable is used to register and define the behavior of the 'capture-pane' command, which captures the contents of a tmux pane to a buffer or stdout.


---
### cmd_clear_history_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_clear_history_entry` is a constant structure of type `cmd_entry` that defines the command entry for the 'clear-history' command in the application. It includes fields such as the command name, alias, arguments, usage, target, flags, and the execution function associated with the command.
- **Use**: This variable is used to define and register the 'clear-history' command, which clears the history of a specified pane in the application.


# Functions

---
### cmd_capture_pane_append<!-- {{#callable:cmd_capture_pane_append}} -->
The `cmd_capture_pane_append` function appends a line of text to a buffer, reallocating memory as needed.
- **Inputs**:
    - `buf`: A pointer to the current buffer where the line will be appended.
    - `len`: A pointer to the size of the current buffer, which will be updated to reflect the new size after appending the line.
    - `line`: A pointer to the line of text that needs to be appended to the buffer.
    - `linelen`: The length of the line of text to be appended.
- **Control Flow**:
    - Reallocate memory for the buffer to accommodate the new line, plus one additional byte for a null terminator.
    - Copy the line into the buffer at the position indicated by the current length of the buffer.
    - Update the length of the buffer to include the length of the newly appended line.
    - Return the updated buffer pointer.
- **Output**:
    - The function returns a pointer to the updated buffer containing the appended line.


---
### cmd_capture_pane_pending<!-- {{#callable:cmd_capture_pane_pending}} -->
The function `cmd_capture_pane_pending` captures and returns the pending input data from a window pane, optionally encoding non-printable characters.
- **Inputs**:
    - `args`: A pointer to a struct `args` which contains command-line arguments that may modify the function's behavior.
    - `wp`: A pointer to a struct `window_pane` representing the pane from which pending input data is to be captured.
    - `len`: A pointer to a size_t variable where the length of the captured data will be stored.
- **Control Flow**:
    - Retrieve the pending input buffer from the window pane's input context using `input_pending` function.
    - If the pending buffer is NULL, return an empty string using `xstrdup`.
    - Extract the data and its length from the pending buffer using `EVBUFFER_DATA` and `EVBUFFER_LENGTH`.
    - Initialize an empty string buffer using `xstrdup`.
    - Check if the 'C' flag is present in the arguments using `args_has`.
    - If the 'C' flag is present, iterate over each character in the pending data.
    - For each character, if it is a printable character and not a backslash, append it directly to the buffer.
    - If it is not a printable character or is a backslash, encode it as an octal escape sequence and append it to the buffer.
    - If the 'C' flag is not present, append the entire pending data to the buffer without modification.
    - Return the constructed buffer containing the captured data.
- **Output**:
    - The function returns a dynamically allocated string containing the captured pending input data from the window pane, with its length stored in the provided `len` pointer.
- **Functions called**:
    - [`cmd_capture_pane_append`](#cmd_capture_pane_append)


---
### cmd_capture_pane_history<!-- {{#callable:cmd_capture_pane_history}} -->
The `cmd_capture_pane_history` function captures the history of a specified window pane and returns it as a string buffer.
- **Inputs**:
    - `args`: A pointer to a struct containing command-line arguments that modify the behavior of the function.
    - `item`: A pointer to a command queue item, used for error reporting and context.
    - `wp`: A pointer to the window pane whose history is to be captured.
    - `len`: A pointer to a size_t variable where the length of the captured buffer will be stored.
- **Control Flow**:
    - Determine the screen and grid to use based on the presence of certain flags ('a' for alternate screen, 'M' for mode screen).
    - Calculate the top and bottom line indices to capture based on 'S' and 'E' flags, defaulting to the entire grid if not specified.
    - Adjust the top and bottom indices if they are out of bounds or if bottom is less than top.
    - Set flags for capturing based on the presence of 'J', 'e', 'C', 'T', and 'N' flags, which control line joining, escape sequences, and trimming.
    - Iterate over the lines from top to bottom, capturing each line as a string and appending it to the buffer.
    - Return the buffer containing the captured pane history.
- **Output**:
    - A dynamically allocated string buffer containing the captured pane history, or NULL if an error occurs.
- **Functions called**:
    - [`cmd_capture_pane_append`](#cmd_capture_pane_append)


---
### cmd_capture_pane_exec<!-- {{#callable:cmd_capture_pane_exec}} -->
The `cmd_capture_pane_exec` function captures the contents of a window pane and either writes it to a buffer or outputs it to a client, depending on the provided arguments.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item.
- **Control Flow**:
    - Retrieve the command arguments, client, and target window pane using helper functions.
    - Check if the command is `clear-history`; if so, reset the pane's mode and clear its history, then return `CMD_RETURN_NORMAL`.
    - Initialize the buffer length to zero and determine whether to capture pending input or pane history based on the presence of the 'P' argument.
    - If capturing fails (buffer is NULL), return `CMD_RETURN_ERROR`.
    - If the 'p' argument is present, write the buffer to the client or print it, handling errors if the client cannot be written to.
    - If the 'p' argument is not present, set the buffer as a paste buffer, handling errors if the operation fails.
    - Return `CMD_RETURN_NORMAL` if the operation completes successfully.
- **Output**:
    - The function returns an `enum cmd_retval`, which is either `CMD_RETURN_NORMAL` on success or `CMD_RETURN_ERROR` on failure.
- **Functions called**:
    - [`cmd_capture_pane_pending`](#cmd_capture_pane_pending)
    - [`cmd_capture_pane_history`](#cmd_capture_pane_history)


# Function Declarations (Public API)

---
### cmd_capture_pane_exec<!-- {{#callable_declaration:cmd_capture_pane_exec}} -->
Capture the contents of a pane and write them to a buffer or stdout.
- **Description**: This function captures the contents of a specified pane and either writes them to a buffer or directly to the standard output, depending on the provided arguments. It can be used to capture the current state of a pane for logging, debugging, or further processing. The function must be called with a valid command and command queue item, and it operates on the target pane specified in the command queue. It handles various options for capturing pending input, history, and formatting the output. The function also supports clearing the pane's history if invoked with the appropriate command entry.
- **Inputs**:
    - `self`: A pointer to the command structure representing the command to be executed. Must not be null.
    - `item`: A pointer to the command queue item that contains context for the command execution, including the target pane. Must not be null.
- **Output**: Returns CMD_RETURN_NORMAL on success or CMD_RETURN_ERROR on failure, indicating whether the capture operation was successful.
- **See also**: [`cmd_capture_pane_exec`](#cmd_capture_pane_exec)  (Implementation)


---
### cmd_capture_pane_append<!-- {{#callable_declaration:cmd_capture_pane_append}} -->
Appends a line to a buffer, reallocating as necessary.
- **Description**: This function appends a given line to an existing buffer, expanding the buffer's memory allocation to accommodate the new data if necessary. It is useful when building a larger string or data block from multiple smaller pieces. The function updates the length of the buffer to reflect the addition of the new line. It is important to ensure that the initial buffer is properly allocated and that the length pointer is valid before calling this function.
- **Inputs**:
    - `buf`: A pointer to the current buffer where the line will be appended. It can be null if the buffer is initially empty. The caller retains ownership, and the buffer is reallocated as needed.
    - `len`: A pointer to a size_t variable representing the current length of the buffer. This value is updated to reflect the new length after the line is appended. Must not be null.
    - `line`: A pointer to the line of data to append to the buffer. The caller retains ownership, and the data is copied into the buffer. Must not be null.
    - `linelen`: The length of the line to append. Must be a valid size_t value representing the number of bytes to copy from the line into the buffer.
- **Output**: Returns a pointer to the reallocated buffer containing the appended data.
- **See also**: [`cmd_capture_pane_append`](#cmd_capture_pane_append)  (Implementation)


---
### cmd_capture_pane_pending<!-- {{#callable_declaration:cmd_capture_pane_pending}} -->
Capture pending input from a window pane into a buffer.
- **Description**: This function captures any pending input from a specified window pane and stores it in a dynamically allocated buffer. It is typically used when you need to retrieve input that has not yet been processed or displayed. The function can optionally escape non-printable characters if the 'C' flag is set in the provided arguments. It is important to ensure that the window pane is valid and that the length pointer is not null. The caller is responsible for freeing the returned buffer.
- **Inputs**:
    - `args`: A pointer to an 'args' structure containing command-line arguments. Must not be null. The 'C' flag, if present, indicates that non-printable characters should be escaped.
    - `wp`: A pointer to a 'window_pane' structure representing the pane from which to capture input. Must not be null.
    - `len`: A pointer to a size_t variable where the length of the captured data will be stored. Must not be null.
- **Output**: Returns a dynamically allocated string containing the captured input. The caller is responsible for freeing this string.
- **See also**: [`cmd_capture_pane_pending`](#cmd_capture_pane_pending)  (Implementation)


---
### cmd_capture_pane_history<!-- {{#callable_declaration:cmd_capture_pane_history}} -->
Capture the history of a window pane into a buffer.
- **Description**: This function captures the history of a specified window pane and stores it in a dynamically allocated buffer. It is typically used to retrieve the contents of a pane for further processing or display. The function requires a valid `args` structure to specify options, a `cmdq_item` for error reporting, and a `window_pane` from which to capture the history. The length of the captured data is returned via a pointer. The function handles various options such as capturing alternate screens, joining lines, and including escape sequences. It returns a null-terminated string containing the captured data or NULL in case of an error.
- **Inputs**:
    - `args`: A pointer to an `args` structure containing command-line arguments that specify options for capturing the pane history. Must not be null.
    - `item`: A pointer to a `cmdq_item` used for error reporting. Must not be null.
    - `wp`: A pointer to the `window_pane` from which the history is to be captured. Must not be null.
    - `len`: A pointer to a `size_t` variable where the length of the captured data will be stored. Must not be null.
- **Output**: Returns a pointer to a dynamically allocated buffer containing the captured pane history, or NULL if an error occurs. The caller is responsible for freeing the returned buffer.
- **See also**: [`cmd_capture_pane_history`](#cmd_capture_pane_history)  (Implementation)


