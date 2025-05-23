# Purpose
The provided C source code is part of the `tmux` terminal multiplexer, specifically handling terminal (TTY) interactions. This file is responsible for managing the terminal's display and input/output operations, including setting up terminal attributes, handling cursor movements, managing terminal regions and margins, and processing input/output buffers. It includes functions for initializing and resizing the terminal, handling terminal events, and managing terminal modes and features. The code also deals with terminal-specific capabilities, such as color handling, cursor styles, and special terminal sequences, ensuring compatibility with various terminal types.

The file defines a set of static and non-static functions that encapsulate the logic for interacting with the terminal. It includes functions for setting terminal attributes, such as colors and cursor styles, and for handling terminal-specific operations like scrolling, clearing lines, and drawing text. The code also manages input and output buffers, ensuring efficient data transfer between the terminal and the `tmux` application. Additionally, it includes mechanisms for handling terminal events, such as resizing and input/output readiness, and for managing terminal-specific features, such as sixel graphics and clipboard operations. Overall, this file is a critical component of `tmux`, providing the necessary functionality to interact with and control the terminal environment.
# Imports and Dependencies

---
- `sys/types.h`
- `sys/ioctl.h`
- `netinet/in.h`
- `curses.h`
- `errno.h`
- `fcntl.h`
- `resolv.h`
- `stdlib.h`
- `string.h`
- `termios.h`
- `time.h`
- `unistd.h`
- `tmux.h`


# Global Variables

---
### tty_log_fd
- **Type**: `int`
- **Description**: The `tty_log_fd` is a static integer variable initialized to -1, which typically indicates an invalid or uninitialized file descriptor. It is used to store the file descriptor for a log file that records terminal output.
- **Use**: This variable is used to manage the file descriptor for logging terminal output to a file.


# Functions

---
### tty_create_log<!-- {{#callable:tty_create_log}} -->
Creates a log file for the terminal session.
- **Inputs**:
    - None
- **Control Flow**:
    - Generates a log file name using the process ID.
    - Attempts to open the log file with write permissions, creating it if it does not exist.
    - If the file descriptor is valid, sets the close-on-exec flag; otherwise, calls `fatal` to handle the error.
- **Output**:
    - The function does not return a value but creates a log file and sets the global `tty_log_fd` to the file descriptor of the opened log file.


---
### tty_init<!-- {{#callable:tty_init}} -->
Initializes a `tty` structure for a terminal client.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure that will be initialized.
    - `c`: A pointer to a `client` structure representing the terminal client.
- **Control Flow**:
    - Checks if the file descriptor `c->fd` is associated with a terminal using `isatty`.
    - If `isatty` returns false, the function returns -1 indicating failure.
    - Clears the `tty` structure by setting its memory to zero using `memset`.
    - Assigns the `client` pointer to the `tty` structure.
    - Sets default values for cursor style and colors in the `tty` structure.
    - Retrieves the terminal attributes using `tcgetattr` and stores them in `tty->tio`.
    - If `tcgetattr` fails, the function returns -1.
    - If all operations succeed, the function returns 0.
- **Output**:
    - Returns 0 on success, or -1 if an error occurs during initialization.


---
### tty_resize<!-- {{#callable:tty_resize}} -->
The `tty_resize` function adjusts the terminal's window size based on the current dimensions reported by the terminal.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal to be resized.
- **Control Flow**:
    - The function retrieves the current window size using the `ioctl` system call with the `TIOCGWINSZ` request.
    - If the window size is successfully retrieved, it checks the number of columns and rows.
    - If the number of columns or rows is zero, it defaults to 80 columns and 24 rows.
    - It calculates the pixel dimensions based on the retrieved window size.
    - If the pixel dimensions are zero and certain conditions are met, it sends escape sequences to query the window size.
    - Finally, it logs the new size and calls [`tty_set_size`](#tty_set_size) to update the terminal's size and [`tty_invalidate`](#tty_invalidate) to refresh the terminal state.
- **Output**:
    - The function does not return a value but updates the terminal's size and state based on the current window dimensions.
- **Functions called**:
    - [`tty_puts`](#tty_puts)
    - [`tty_set_size`](#tty_set_size)
    - [`tty_invalidate`](#tty_invalidate)


---
### tty_set_size<!-- {{#callable:tty_set_size}} -->
Sets the size and pixel dimensions of a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal interface whose size is being set.
    - `sx`: An unsigned integer representing the new width of the terminal in character cells.
    - `sy`: An unsigned integer representing the new height of the terminal in character cells.
    - `xpixel`: An unsigned integer representing the new width of the terminal in pixels.
    - `ypixel`: An unsigned integer representing the new height of the terminal in pixels.
- **Control Flow**:
    - The function directly assigns the input values for width, height, and pixel dimensions to the corresponding fields in the `tty` structure.
    - No conditional logic or loops are present, making the function straightforward and efficient.
- **Output**:
    - The function does not return a value; it modifies the `tty` structure in place to reflect the new size and pixel dimensions.


---
### tty_read_callback<!-- {{#callable:tty_read_callback}} -->
The `tty_read_callback` function handles reading data from a terminal input buffer and processes it accordingly.
- **Inputs**:
    - `fd`: An integer file descriptor for the terminal, which is unused in this function.
    - `events`: A short integer representing the events that triggered the callback, which is also unused.
    - `data`: A pointer to a `tty` structure that contains information about the terminal.
- **Control Flow**:
    - The function retrieves the `tty` structure from the `data` pointer.
    - It accesses the associated `client` structure and retrieves the client's name.
    - The function checks the length of the input buffer to determine how many bytes can be read.
    - It attempts to read data from the terminal's input buffer into the `tty->in` buffer.
    - If no bytes are read or an error occurs, it logs the appropriate message and calls `server_client_lost` to handle the client disconnection.
    - If data is successfully read, it logs the number of bytes read and continues processing the input by calling `tty_keys_next` in a loop until all keys are processed.
- **Output**:
    - The function does not return a value; instead, it modifies the state of the terminal and logs messages based on the read operation's success or failure.


---
### tty_timer_callback<!-- {{#callable:tty_timer_callback}} -->
The `tty_timer_callback` function handles timer events for a terminal, managing the state of discarded output and triggering redraws as necessary.
- **Inputs**:
    - `fd`: An unused file descriptor, typically for event handling.
    - `events`: An unused event mask indicating the type of events that triggered the callback.
    - `data`: A pointer to a `tty` structure containing terminal state information.
- **Control Flow**:
    - Logs the number of discarded output bytes for the associated client.
    - Updates the client's redraw flags and total discarded count.
    - Checks if the number of discarded bytes is below a threshold; if so, it clears the block flag and invalidates the tty state.
    - If the discarded count exceeds the threshold, it resets the discarded count and re-adds the timer with a specified interval.
- **Output**:
    - The function does not return a value but modifies the state of the `tty` and `client` structures, potentially triggering a redraw of the terminal output.
- **Functions called**:
    - [`tty_invalidate`](#tty_invalidate)


---
### tty_block_maybe<!-- {{#callable:tty_block_maybe}} -->
The `tty_block_maybe` function manages the blocking state of a terminal output buffer based on its current size.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
- **Control Flow**:
    - Check if the output buffer size is zero; if so, clear the `TTY_NOBLOCK` flag.
    - If the `TTY_NOBLOCK` flag is set, return 0 immediately.
    - If the output buffer size is less than the threshold defined by `TTY_BLOCK_START`, return 0.
    - If the `TTY_BLOCK` flag is already set, return 1.
    - Set the `TTY_BLOCK` flag to indicate that the terminal is now in a blocking state.
    - Log a debug message indicating that the terminal cannot keep up and the size of discarded data.
    - Drain the output buffer to discard all data.
    - Increment the discarded count for the client associated with the terminal.
    - Reset the `discarded` count in the `tty` structure.
    - Set a timer for blocking operations using `evtimer_add`.
- **Output**:
    - Returns 1 if the terminal is now in a blocking state, otherwise returns 0.


---
### tty_write_callback<!-- {{#callable:tty_write_callback}} -->
The `tty_write_callback` function handles writing data from a `tty` output buffer to a client file descriptor.
- **Inputs**:
    - `fd`: An unused file descriptor, typically representing the output stream.
    - `events`: An unused event mask indicating the events that triggered the callback.
    - `data`: A pointer to a `tty` structure containing the output buffer and associated client.
- **Control Flow**:
    - Retrieve the `tty` structure from the `data` pointer.
    - Get the associated `client` structure from the `tty`.
    - Determine the size of the output buffer using `EVBUFFER_LENGTH`.
    - Attempt to write the contents of the output buffer to the client's file descriptor using `evbuffer_write`.
    - If the write operation fails, exit the function.
    - Log the number of bytes written and the original size of the output buffer.
    - Check if the client requires a redraw; if so, adjust the redraw count accordingly.
    - If the redraw count is zero, set it to zero; otherwise, decrement it by the number of bytes written.
    - Log the remaining bytes to be redrawn if applicable.
    - Check if the `tty` is blocked; if so, exit the function.
    - If there are still bytes in the output buffer, re-add the output event to the event loop.
- **Output**:
    - The function does not return a value; it modifies the state of the `tty` and `client` structures and manages the output buffer.
- **Functions called**:
    - [`tty_block_maybe`](#tty_block_maybe)


---
### tty_open<!-- {{#callable:tty_open}} -->
The `tty_open` function initializes a terminal interface for a given client.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure that represents the terminal interface to be opened.
    - `cause`: A pointer to a string that will hold an error message if the terminal creation fails.
- **Control Flow**:
    - The function retrieves the associated client from the `tty` structure.
    - It attempts to create a terminal using `tty_term_create` with the client's terminal name and capabilities.
    - If terminal creation fails (returns NULL), it calls [`tty_close`](#tty_close) to clean up and returns -1.
    - If successful, it sets the `TTY_OPENED` flag in the `tty` structure.
    - It clears specific flags related to cursor visibility and freezing.
    - It sets up event handlers for reading and writing data using `event_set`.
    - It allocates new input and output buffers using `evbuffer_new`, and checks for memory allocation failure.
    - It initializes a timer for handling events with `evtimer_set`.
    - Finally, it starts the terminal interface with [`tty_start_tty`](#tty_start_tty) and builds key mappings with `tty_keys_build`.
- **Output**:
    - The function returns 0 on success, indicating that the terminal has been successfully opened and initialized.
- **Functions called**:
    - [`tty_close`](#tty_close)
    - [`tty_start_tty`](#tty_start_tty)


---
### tty_start_timer_callback<!-- {{#callable:tty_start_timer_callback}} -->
The `tty_start_timer_callback` function is a callback that is triggered when a timer event occurs, updating terminal features and request flags.
- **Inputs**:
    - `fd`: An unused file descriptor, typically associated with the event that triggered the callback.
    - `events`: An unused short integer representing the events that triggered the callback.
    - `data`: A pointer to the `tty` structure containing terminal state information.
- **Control Flow**:
    - Logs a debug message indicating that the timer has fired for the associated client.
    - Checks if the `tty` flags do not indicate that certain features have been set (TTY_HAVEDA, TTY_HAVEDA2, TTY_HAVEXDA).
    - If the features have not been set, it calls [`tty_update_features`](#tty_update_features) to update the terminal features.
    - Sets the `tty` flags to indicate that all request flags are now set.
- **Output**:
    - The function does not return a value but modifies the state of the `tty` structure by updating its flags and potentially its features.
- **Functions called**:
    - [`tty_update_features`](#tty_update_features)


---
### tty_start_tty<!-- {{#callable:tty_start_tty}} -->
Initializes and configures a terminal interface for a client.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal interface to be started.
- **Control Flow**:
    - Sets the file descriptor of the client to non-blocking mode.
    - Adds an event listener for input events on the terminal.
    - Copies the terminal attributes from the `tty` structure and modifies them to disable certain input and output processing features.
    - Applies the modified terminal attributes using `tcsetattr` and flushes the output buffer if successful.
    - Sends terminal control codes to set up the terminal display, including enabling alternate character sets and clearing the screen.
    - Checks if additional capabilities for ACS (Alternate Character Set) are needed and logs the appropriate message.
    - Sets up mouse tracking if supported by the terminal.
    - Configures theme changes for VT100-like terminals.
    - Initializes a timer for starting the terminal and adds it to the event loop.
    - Marks the terminal as started and invalidates its state.
    - Resets mouse drag flags and updates cursor color if necessary.
- **Output**:
    - The function does not return a value but modifies the state of the `tty` structure and configures the terminal for interaction.
- **Functions called**:
    - [`tty_putcode`](#tty_putcode)
    - [`tty_puts`](#tty_puts)
    - [`tty_invalidate`](#tty_invalidate)
    - [`tty_force_cursor_colour`](#tty_force_cursor_colour)


---
### tty_send_requests<!-- {{#callable:tty_send_requests}} -->
The `tty_send_requests` function sends terminal requests based on the state of the `tty` structure.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal state.
- **Control Flow**:
    - Checks if the `TTY_STARTED` flag is not set; if not, the function returns immediately.
    - If the terminal type is VT100-like, it checks various flags and sends specific escape sequences to the terminal.
    - If the terminal is not VT100-like, it sets the `TTY_ALL_REQUEST_FLAGS` flag.
    - Updates the `last_requests` timestamp to the current time.
- **Output**:
    - The function does not return a value but modifies the state of the `tty` structure and sends escape sequences to the terminal.
- **Functions called**:
    - [`tty_puts`](#tty_puts)


---
### tty_repeat_requests<!-- {{#callable:tty_repeat_requests}} -->
The `tty_repeat_requests` function sends terminal requests to query the foreground and background colors if certain conditions are met.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal context.
- **Control Flow**:
    - The function retrieves the current time and checks if the `TTY_STARTED` flag is set in the `tty` structure.
    - If the `TTY_STARTED` flag is not set, the function returns immediately.
    - It then checks if the time since the last request is less than the `TTY_REQUEST_LIMIT` constant.
    - If the time limit has not been exceeded, the function returns without sending any requests.
    - If the time limit has been exceeded, it updates the `last_requests` timestamp.
    - Finally, if the terminal type supports VT100-like features, it sends escape sequences to query the foreground and background colors.
- **Output**:
    - The function does not return a value; it sends escape sequences to the terminal to request color information.
- **Functions called**:
    - [`tty_puts`](#tty_puts)


---
### tty_stop_tty<!-- {{#callable:tty_stop_tty}} -->
The `tty_stop_tty` function stops the terminal interface associated with a given `tty` structure.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal interface to be stopped.
- **Control Flow**:
    - Checks if the `TTY_STARTED` flag is set in the `tty` structure; if not, the function returns immediately.
    - Clears the `TTY_STARTED` flag to indicate that the terminal is no longer active.
    - Deletes the start timer and any active event timers associated with input and output.
    - Attempts to retrieve the current window size using `ioctl`; if it fails, the function returns.
    - Sets the terminal attributes back to the original state using `tcsetattr`.
    - Sends various terminal control sequences to reset the terminal display and cursor settings.
    - Restores the terminal to a normal state and clears any active mouse tracking settings.
- **Output**:
    - The function does not return a value; it modifies the state of the terminal and sends control sequences to the terminal.
- **Functions called**:
    - [`tty_raw`](#tty_raw)


---
### tty_close<!-- {{#callable:tty_close}} -->
Closes a terminal interface and releases associated resources.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal interface to be closed.
- **Control Flow**:
    - Checks if the `key_timer` event is initialized and deletes it if so.
    - Calls [`tty_stop_tty`](#tty_stop_tty) to stop the terminal interface.
    - Checks if the `TTY_OPENED` flag is set in the `tty` structure.
    - If `TTY_OPENED` is set, frees the input and output buffers, deletes the input and output events, frees the terminal structure, and frees the key bindings.
    - Clears the `TTY_OPENED` flag.
- **Output**:
    - The function does not return a value; it performs cleanup operations on the terminal interface.
- **Functions called**:
    - [`tty_stop_tty`](#tty_stop_tty)


---
### tty_free<!-- {{#callable:tty_free}} -->
The `tty_free` function releases resources associated with a `tty` structure.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal interface to be freed.
- **Control Flow**:
    - Calls the [`tty_close`](#tty_close) function to close the terminal and release associated resources.
- **Output**:
    - This function does not return a value; it performs cleanup operations on the `tty` structure.
- **Functions called**:
    - [`tty_close`](#tty_close)


---
### tty_update_features<!-- {{#callable:tty_update_features}} -->
Updates terminal features based on the current `tty` and client settings.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal interface.
- **Control Flow**:
    - Calls `tty_apply_features` to apply the terminal features from the client to the terminal.
    - If features are applied, it calls `tty_term_apply_overrides` to apply any necessary overrides.
    - Checks if the terminal uses margins and sends the appropriate code if true.
    - Checks global options for extended keys and focus events, sending the corresponding terminal codes if enabled.
    - If the terminal is VT100-like, it sends a specific escape sequence to enable certain features.
    - Calls `server_redraw_client` to redraw the client if features have changed since the initial draw.
    - Finally, it calls [`tty_invalidate`](#tty_invalidate) to mark the terminal as needing an update.
- **Output**:
    - The function does not return a value but updates the terminal's features and state based on the current settings.
- **Functions called**:
    - [`tty_putcode`](#tty_putcode)
    - [`tty_puts`](#tty_puts)
    - [`tty_invalidate`](#tty_invalidate)


---
### tty_raw<!-- {{#callable:tty_raw}} -->
The `tty_raw` function writes a string to a terminal device in a non-blocking manner, attempting multiple times to ensure all data is sent.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal context.
    - `s`: A pointer to a constant character string that contains the data to be written to the terminal.
- **Control Flow**:
    - Calculate the length of the input string `s` using `strlen`.
    - Loop up to 5 times to attempt writing the string to the terminal.
    - In each iteration, call `write` to send the data to the file descriptor associated with the terminal.
    - If `write` returns a positive number, adjust the pointer `s` and the length `slen` accordingly.
    - If `write` returns -1 and the error is not `EAGAIN`, break the loop.
    - If the entire string is written (i.e., `slen` becomes 0`), break the loop.
    - If the write operation is not successful, sleep for 100 microseconds before the next attempt.
- **Output**:
    - The function does not return a value; it writes the specified string to the terminal, handling partial writes and errors gracefully.


---
### tty_putcode<!-- {{#callable:tty_putcode}} -->
The `tty_putcode` function sends a terminal control code to the specified terminal.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal to which the code will be sent.
    - `code`: An enumeration value of type `enum tty_code_code` that specifies the terminal control code to be sent.
- **Control Flow**:
    - The function calls `tty_term_string` to retrieve the string representation of the terminal control code based on the provided `tty` and `code`.
    - The resulting string is then passed to the [`tty_puts`](#tty_puts) function, which handles the actual sending of the string to the terminal.
- **Output**:
    - The function does not return a value; it sends the specified terminal control code to the terminal, affecting its behavior or appearance.
- **Functions called**:
    - [`tty_puts`](#tty_puts)


---
### tty_putcode_i<!-- {{#callable:tty_putcode_i}} -->
The `tty_putcode_i` function sends a terminal control sequence to the specified `tty` if the integer argument `a` is non-negative.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal to which the control sequence will be sent.
    - `code`: An enumeration value of type `enum tty_code_code` that specifies the control sequence to be sent.
    - `a`: An integer that is used as an argument for the control sequence; if it is negative, the function does nothing.
- **Control Flow**:
    - The function first checks if the value of `a` is less than 0; if so, it returns immediately without performing any action.
    - If `a` is non-negative, it calls the `tty_term_string_i` function to get the appropriate control sequence string for the given `code` and `a`.
    - Finally, it sends the resulting string to the terminal using the [`tty_puts`](#tty_puts) function.
- **Output**:
    - The function does not return a value; it outputs a terminal control sequence to the specified `tty` if `a` is non-negative.
- **Functions called**:
    - [`tty_puts`](#tty_puts)


---
### tty_putcode_ii<!-- {{#callable:tty_putcode_ii}} -->
The `tty_putcode_ii` function sends a terminal control sequence with two integer parameters.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `code`: An enumeration value of type `tty_code_code` that specifies the control code to be sent.
    - `a`: An integer parameter that is part of the control sequence.
    - `b`: Another integer parameter that is part of the control sequence.
- **Control Flow**:
    - The function first checks if either `a` or `b` is negative; if so, it returns immediately without doing anything.
    - If both parameters are non-negative, it calls `tty_term_string_ii` to generate the appropriate control sequence string using the provided `code`, `a`, and `b`.
    - Finally, it sends the generated string to the terminal using the [`tty_puts`](#tty_puts) function.
- **Output**:
    - The function does not return a value; it outputs a control sequence to the terminal based on the provided parameters.
- **Functions called**:
    - [`tty_puts`](#tty_puts)


---
### tty_putcode_iii<!-- {{#callable:tty_putcode_iii}} -->
The `tty_putcode_iii` function sends a terminal control sequence with three integer parameters.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal context.
    - `code`: An enumeration value of type `enum tty_code_code` that specifies the control code to be sent.
    - `a`: An integer parameter that is part of the control sequence.
    - `b`: An integer parameter that is part of the control sequence.
    - `c`: An integer parameter that is part of the control sequence.
- **Control Flow**:
    - The function first checks if any of the parameters `a`, `b`, or `c` are negative.
    - If any parameter is negative, the function returns immediately without performing any action.
    - If all parameters are non-negative, it calls the `tty_term_string_iii` function to generate the appropriate control sequence string.
    - Finally, it sends the generated string to the terminal using the [`tty_puts`](#tty_puts) function.
- **Output**:
    - The function does not return a value; it outputs a terminal control sequence based on the provided parameters.
- **Functions called**:
    - [`tty_puts`](#tty_puts)


---
### tty_putcode_s<!-- {{#callable:tty_putcode_s}} -->
The `tty_putcode_s` function sends a terminal control sequence to the specified `tty` if the provided string is not NULL.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal to which the control sequence will be sent.
    - `code`: An enumeration value of type `enum tty_code_code` that specifies the type of control sequence to be sent.
    - `a`: A pointer to a constant character string that contains additional data for the control sequence.
- **Control Flow**:
    - The function first checks if the string `a` is not NULL.
    - If `a` is not NULL, it calls the `tty_term_string_s` function to generate the appropriate control sequence string based on the `code` and `a`.
    - The generated string is then sent to the terminal using the [`tty_puts`](#tty_puts) function.
- **Output**:
    - The function does not return a value; it performs an action by sending a control sequence to the terminal if the input string is not NULL.
- **Functions called**:
    - [`tty_puts`](#tty_puts)


---
### tty_putcode_ss<!-- {{#callable:tty_putcode_ss}} -->
The `tty_putcode_ss` function sends a terminal control sequence that includes two string arguments to the specified terminal.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal to which the control sequence will be sent.
    - `code`: An enumeration value of type `enum tty_code_code` that specifies the control sequence to be sent.
    - `a`: A pointer to a string that is the first argument to be included in the control sequence.
    - `b`: A pointer to a string that is the second argument to be included in the control sequence.
- **Control Flow**:
    - The function first checks if both input strings `a` and `b` are not NULL.
    - If both strings are valid, it calls the `tty_term_string_ss` function to generate the appropriate control sequence string using the provided `code`, `a`, and `b`.
    - Finally, it sends the generated string to the terminal using the [`tty_puts`](#tty_puts) function.
- **Output**:
    - The function does not return a value; it sends a control sequence to the terminal if the input strings are valid.
- **Functions called**:
    - [`tty_puts`](#tty_puts)


---
### tty_add<!-- {{#callable:tty_add}} -->
The `tty_add` function adds a buffer of data to the output of a terminal interface, handling various conditions such as blocking and logging.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal interface to which the data will be added.
    - `buf`: A pointer to a character buffer containing the data to be added.
    - `len`: A size_t value representing the length of the data in the buffer.
- **Control Flow**:
    - Check if the `TTY_BLOCK` flag is set in the `tty` structure; if so, increment the `discarded` count by `len` and return immediately.
    - If not blocked, add the data from `buf` to the `out` buffer of the `tty` structure using `evbuffer_add`.
    - Log the added data using `log_debug`, including the client's name and the content of the buffer.
    - Increment the `written` count of the associated `client` structure by `len`.
    - If `tty_log_fd` is valid (not -1), write the data to the log file.
    - If the `TTY_STARTED` flag is set, add the `event_out` event to the event loop.
- **Output**:
    - The function does not return a value; it modifies the state of the `tty` and `client` structures and may produce side effects such as logging data.


---
### tty_puts<!-- {{#callable:tty_puts}} -->
The `tty_puts` function sends a string to a terminal interface if the string is not empty.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal interface to which the string will be sent.
    - `s`: A pointer to a constant character string that contains the text to be sent to the terminal.
- **Control Flow**:
    - The function first checks if the first character of the string `s` is not the null terminator ('\0').
    - If the string is not empty, it calls the [`tty_add`](#tty_add) function to send the string to the terminal, along with its length.
- **Output**:
    - The function does not return a value; it modifies the state of the terminal by sending the string to it.
- **Functions called**:
    - [`tty_add`](#tty_add)


---
### tty_putc<!-- {{#callable:tty_putc}} -->
The `tty_putc` function outputs a single character to the terminal, handling character set attributes and cursor positioning.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal context.
    - `ch`: A character of type `u_char` to be output to the terminal.
- **Control Flow**:
    - Checks if automatic margins are disabled and if the character is printable; if so, it returns early if the cursor is at the end of the line.
    - If the terminal supports character sets, it retrieves the appropriate ACS (Alternate Character Set) representation for the character.
    - If an ACS representation is found, it adds that to the output buffer; otherwise, it adds the character itself.
    - If the character is printable and the cursor is at the end of the line, it wraps the cursor to the next line and updates the cursor position accordingly.
    - If automatic margins are disabled, it sends a cursor positioning command to the terminal.
- **Output**:
    - The function does not return a value; instead, it modifies the terminal's output buffer and cursor position based on the character processed.
- **Functions called**:
    - [`tty_add`](#tty_add)
    - [`tty_putcode_ii`](#tty_putcode_ii)


---
### tty_putn<!-- {{#callable:tty_putn}} -->
The `tty_putn` function outputs a specified number of characters from a buffer to a terminal, managing cursor position and line wrapping.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal context.
    - `buf`: A pointer to the buffer containing the data to be written to the terminal.
    - `len`: The number of bytes to write from the buffer.
    - `width`: The width of the output, which affects cursor positioning.
- **Control Flow**:
    - Check if the terminal has the `TERM_NOAM` flag set and if the cursor is at the bottom of the screen, adjusting the length to prevent overflow.
    - Call [`tty_add`](#tty_add) to add the specified number of bytes from the buffer to the terminal's output buffer.
    - Check if the new cursor position exceeds the terminal's width, and if so, adjust the cursor position and potentially increment the line number.
    - If the cursor position exceeds the terminal width, set the cursor position to the start of the next line or mark it as invalid.
- **Output**:
    - The function does not return a value but modifies the state of the `tty` structure, specifically the cursor position and the output buffer.
- **Functions called**:
    - [`tty_add`](#tty_add)


---
### tty_set_italics<!-- {{#callable:tty_set_italics}} -->
Sets the italics mode for the terminal if supported.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
- **Control Flow**:
    - Checks if the terminal supports italics mode using `tty_term_has` with the `TTYC_SITM` capability.
    - Retrieves the default terminal type from global options.
    - If the terminal type is not 'screen' or does not start with 'screen-', it sends the italics start code using [`tty_putcode`](#tty_putcode) with `TTYC_SITM`.
    - If the terminal type is 'screen', it sends the standout mode code using [`tty_putcode`](#tty_putcode) with `TTYC_SMSO`.
- **Output**:
    - The function does not return a value; it modifies the terminal's display attributes based on the terminal type and capabilities.
- **Functions called**:
    - [`tty_putcode`](#tty_putcode)


---
### tty_set_title<!-- {{#callable:tty_set_title}} -->
Sets the terminal title if supported by the terminal.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal.
    - `title`: A string containing the title to be set for the terminal.
- **Control Flow**:
    - Checks if the terminal supports title setting capabilities using `tty_term_has` for `TTYC_TSL` and `TTYC_FSL`.
    - If either capability is not supported, the function returns early without making any changes.
    - If both capabilities are supported, it sends the escape sequence to set the title, outputs the title string, and then sends the escape sequence to finalize the title setting.
- **Output**:
    - The function does not return a value; it modifies the terminal's title if the capabilities are supported.
- **Functions called**:
    - [`tty_putcode`](#tty_putcode)
    - [`tty_puts`](#tty_puts)


---
### tty_set_path<!-- {{#callable:tty_set_path}} -->
Sets the terminal's current working directory path in the title bar.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal.
    - `title`: A string representing the title to be set in the terminal's path.
- **Control Flow**:
    - Checks if the terminal supports the `TTYC_SWD` and `TTYC_FSL` capabilities.
    - If either capability is not supported, the function returns early without making changes.
    - If both capabilities are supported, it sends the `TTYC_SWD` code to set the working directory.
    - Then, it sends the `title` string to display it in the terminal.
    - Finally, it sends the `TTYC_FSL` code to finalize the title setting.
- **Output**:
    - The function does not return a value; it modifies the terminal's display directly.
- **Functions called**:
    - [`tty_putcode`](#tty_putcode)
    - [`tty_puts`](#tty_puts)


---
### tty_force_cursor_colour<!-- {{#callable:tty_force_cursor_colour}} -->
The `tty_force_cursor_colour` function sets the cursor color for a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal interface.
    - `c`: An integer representing the color value to set for the cursor, or -1 to reset the cursor color.
- **Control Flow**:
    - If `c` is not -1, it is converted to an RGB color value using `colour_force_rgb`.
    - If the new color is the same as the current cursor color (`tty->ccolour`), the function returns early.
    - If `c` is -1, the function sends a command to reset the cursor color using [`tty_putcode`](#tty_putcode).
    - If `c` is a valid color, it splits the RGB value into its red, green, and blue components using `colour_split_rgb`.
    - The RGB values are formatted into a string and sent to the terminal using [`tty_putcode_s`](#tty_putcode_s).
    - Finally, the current cursor color is updated to the new value.
- **Output**:
    - The function does not return a value; it modifies the state of the terminal by changing the cursor color.
- **Functions called**:
    - [`tty_putcode`](#tty_putcode)
    - [`tty_putcode_s`](#tty_putcode_s)


---
### tty_update_cursor<!-- {{#callable:tty_update_cursor}} -->
Updates the cursor appearance and behavior in a terminal based on the provided mode and screen settings.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal.
    - `mode`: An integer representing the cursor mode, which can include visibility and blinking options.
    - `s`: A pointer to a `screen` structure that contains cursor style and color information.
- **Control Flow**:
    - Checks if the `screen` structure is not NULL to set the cursor color based on the provided screen settings.
    - If the cursor mode indicates it should be off, it sets the cursor to invisible and returns the current mode.
    - Determines the cursor style based on the provided screen settings or defaults to the current tty settings.
    - If there are no changes in cursor mode or style, the function exits early.
    - Sets the cursor style using terminal control codes based on the determined style and mode, including handling blinking and visibility.
- **Output**:
    - Returns the updated cursor mode after applying any changes.
- **Functions called**:
    - [`tty_force_cursor_colour`](#tty_force_cursor_colour)
    - [`tty_putcode`](#tty_putcode)
    - [`tty_putcode_i`](#tty_putcode_i)


---
### tty_update_mode<!-- {{#callable:tty_update_mode}} -->
Updates the terminal mode and cursor settings for a given `tty`.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal to be updated.
    - `mode`: An integer representing the new mode settings to apply to the terminal.
    - `s`: A pointer to a `struct screen` that may contain cursor style and color information.
- **Control Flow**:
    - Checks if the `TTY_NOCURSOR` flag is set in `tty` and modifies the `mode` to disable the cursor if necessary.
    - Calls [`tty_update_cursor`](#tty_update_cursor) to update the cursor settings based on the new `mode` and the provided `screen`.
    - Calculates the `changed` variable to determine which mode settings have changed compared to the current settings.
    - Logs the current and new mode settings if logging is enabled and changes have occurred.
    - If mouse mode settings have changed, sends escape sequences to the terminal to reset and apply the new mouse modes.
    - Updates the `tty`'s mode with the new settings.
- **Output**:
    - The function does not return a value; it modifies the state of the `tty` and sends commands to the terminal.
- **Functions called**:
    - [`tty_update_cursor`](#tty_update_cursor)
    - [`tty_puts`](#tty_puts)


---
### tty_emulate_repeat<!-- {{#callable:tty_emulate_repeat}} -->
The `tty_emulate_repeat` function sends terminal control codes to emulate a repeat action based on the terminal's capabilities.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `code`: An `enum tty_code_code` value representing the primary control code to be sent.
    - `code1`: An `enum tty_code_code` value representing the alternative control code to be sent if the primary code is not supported.
    - `n`: An unsigned integer representing the number of times to repeat the primary control code.
- **Control Flow**:
    - The function first checks if the terminal supports the primary control code using `tty_term_has`.
    - If the primary control code is supported, it sends the code with a repeat count using [`tty_putcode_i`](#tty_putcode_i).
    - If the primary control code is not supported, it enters a loop that decrements `n` and sends the alternative control code `code1` until `n` reaches zero.
- **Output**:
    - The function does not return a value; it directly sends control codes to the terminal based on its capabilities.
- **Functions called**:
    - [`tty_putcode_i`](#tty_putcode_i)
    - [`tty_putcode`](#tty_putcode)


---
### tty_repeat_space<!-- {{#callable:tty_repeat_space}} -->
The `tty_repeat_space` function outputs a specified number of space characters to a terminal.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `n`: An unsigned integer representing the total number of space characters to output.
- **Control Flow**:
    - A static character array `s` of size 500 is defined to hold space characters.
    - If the first character of `s` is not a space, the array is filled with spaces using `memset`.
    - A loop checks if `n` is greater than the size of `s` and repeatedly calls [`tty_putn`](#tty_putn) to output `s` until `n` is reduced to a value less than or equal to the size of `s`.
    - If `n` is not zero after the loop, [`tty_putn`](#tty_putn) is called one last time to output the remaining spaces.
- **Output**:
    - The function does not return a value; it directly outputs space characters to the terminal.
- **Functions called**:
    - [`tty_putn`](#tty_putn)


---
### tty_window_bigger<!-- {{#callable:tty_window_bigger}} -->
The `tty_window_bigger` function checks if the terminal window is smaller than the current window dimensions.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that contains information about the terminal.
- **Control Flow**:
    - The function retrieves the `client` associated with the `tty` structure.
    - It then accesses the current window from the client's session.
    - The function compares the terminal's width (`tty->sx`) with the window's width (`w->sx`).
    - It also checks if the terminal's height (`tty->sy`) minus the status line size is less than the window's height (`w->sy`).
    - If either condition is true, it returns a non-zero value indicating the terminal is smaller.
- **Output**:
    - Returns a non-zero value if the terminal window is smaller than the current window, otherwise returns zero.


---
### tty_window_offset<!-- {{#callable:tty_window_offset}} -->
The `tty_window_offset` function retrieves the current window offsets and size for a given `tty` structure.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that contains information about the terminal.
    - `ox`: A pointer to a `u_int` variable where the x-offset of the window will be stored.
    - `oy`: A pointer to a `u_int` variable where the y-offset of the window will be stored.
    - `sx`: A pointer to a `u_int` variable where the width of the window will be stored.
    - `sy`: A pointer to a `u_int` variable where the height of the window will be stored.
- **Control Flow**:
    - The function dereferences the pointers `ox`, `oy`, `sx`, and `sy` to assign them the values of `tty->oox`, `tty->ooy`, `tty->osx`, and `tty->osy`, respectively.
    - Finally, it returns the value of `tty->oflag`.
- **Output**:
    - The function returns an integer representing the `oflag` value from the `tty` structure.


---
### tty_window_offset1<!-- {{#callable:tty_window_offset1}} -->
Calculates the offsets and sizes for a terminal window based on the current terminal and window dimensions.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal.
    - `ox`: A pointer to an unsigned integer where the x-offset will be stored.
    - `oy`: A pointer to an unsigned integer where the y-offset will be stored.
    - `sx`: A pointer to an unsigned integer where the width of the window will be stored.
    - `sy`: A pointer to an unsigned integer where the height of the window will be stored.
- **Control Flow**:
    - Calculates the number of lines in the status line using `status_line_size(c)`.
    - Checks if the terminal size is greater than or equal to the window size, adjusting offsets and sizes accordingly.
    - If the terminal is in a 'pan' state, it adjusts the offsets based on the current pane's offsets.
    - If the cursor mode is not set, it resets the offsets to zero.
    - If the cursor mode is set, it calculates the offsets based on the cursor's position within the window.
- **Output**:
    - Returns 0 if the terminal fits the window, 1 if it does not, and updates the output parameters with the calculated offsets and sizes.


---
### tty_update_window_offset<!-- {{#callable:tty_update_window_offset}} -->
Updates the window offsets for all clients associated with a specified window.
- **Inputs**:
    - `w`: A pointer to a `window` structure representing the window whose offsets are to be updated.
- **Control Flow**:
    - Iterates through the list of clients using `TAILQ_FOREACH`.
    - For each client, checks if the client has an active session and if the current window of that session matches the specified window.
    - If both conditions are met, calls the [`tty_update_client_offset`](#tty_update_client_offset) function to update the client's window offset.
- **Output**:
    - The function does not return a value; it modifies the offsets of the clients' associated windows directly.
- **Functions called**:
    - [`tty_update_client_offset`](#tty_update_client_offset)


---
### tty_update_client_offset<!-- {{#callable:tty_update_client_offset}} -->
Updates the terminal offsets for a client based on the current window size.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client whose terminal offsets are to be updated.
- **Control Flow**:
    - Checks if the `CLIENT_TERMINAL` flag is set for the client; if not, the function returns immediately.
    - Calls [`tty_window_offset1`](#tty_window_offset1) to retrieve the new offsets (ox, oy) and sizes (sx, sy) for the terminal window.
    - Compares the new offsets and sizes with the previously stored values in the client structure.
    - If the values have changed, logs the change using `log_debug`.
    - Updates the client's stored offsets and sizes with the new values.
    - Sets the `CLIENT_REDRAWWINDOW` and `CLIENT_REDRAWSTATUS` flags to indicate that the window needs to be redrawn.
- **Output**:
    - The function does not return a value; it modifies the state of the `client` structure directly.
- **Functions called**:
    - [`tty_window_offset1`](#tty_window_offset1)


---
### tty_large_region<!-- {{#callable:tty_large_region}} -->
Determines if a specified region in a terminal context is large enough to warrant a redraw.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal.
    - `ctx`: A pointer to a `tty_ctx` structure containing context information about the terminal display.
- **Control Flow**:
    - Calculates the difference between the `orlower` and `orupper` fields of the `ctx` structure.
    - Compares this difference to half of the `sy` field in the `ctx` structure.
    - Returns 1 (true) if the difference is greater than or equal to half of `sy`, indicating a large region; otherwise returns 0 (false).
- **Output**:
    - Returns an integer value indicating whether the region is considered large enough for a redraw.


---
### tty_fake_bce<!-- {{#callable:tty_fake_bce}} -->
The `tty_fake_bce` function determines if backspace character erase (BCE) emulation is needed based on terminal capabilities and color settings.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `gc`: A pointer to a `struct grid_cell` representing the grid cell attributes.
    - `bg`: An unsigned integer representing the background color.
- **Control Flow**:
    - The function first checks if the terminal supports BCE by calling `tty_term_flag` with the `TTYC_BCE` capability.
    - If BCE is supported, the function returns 0, indicating no emulation is needed.
    - Next, it checks if the background color (`bg`) or the background color of the grid cell (`gc->bg`) is not the default color using `COLOUR_DEFAULT`.
    - If either color is not default, it returns 1, indicating that BCE emulation is required.
    - If both colors are default, it returns 0, indicating no emulation is needed.
- **Output**:
    - The function returns an integer: 0 if BCE is supported or both colors are default, and 1 if BCE emulation is required.


---
### tty_redraw_region<!-- {{#callable:tty_redraw_region}} -->
The `tty_redraw_region` function redraws a specified region of a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing the context for the redraw operation, including the region to be redrawn.
- **Control Flow**:
    - The function first checks if the region defined by `ctx` is large enough to warrant a redraw operation using the [`tty_large_region`](#tty_large_region) function.
    - If the region is large, it logs a debug message and calls the `redraw_cb` callback function provided in the `ctx` structure to handle the redraw.
    - If the region is not large, it enters a loop that iterates from `ctx->orupper` to `ctx->orlower`, calling [`tty_draw_pane`](#tty_draw_pane) for each line in the specified range to redraw the content.
- **Output**:
    - The function does not return a value; it performs redraw operations directly on the terminal output.
- **Functions called**:
    - [`tty_large_region`](#tty_large_region)
    - [`tty_draw_pane`](#tty_draw_pane)


---
### tty_is_visible<!-- {{#callable:tty_is_visible}} -->
The `tty_is_visible` function checks if a specified area is visible within a terminal's window.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal.
    - `ctx`: A pointer to a `struct tty_ctx` containing context information about the terminal's drawing area.
    - `px`: An unsigned integer representing the x-coordinate of the area to check.
    - `py`: An unsigned integer representing the y-coordinate of the area to check.
    - `nx`: An unsigned integer representing the width of the area to check.
    - `ny`: An unsigned integer representing the height of the area to check.
- **Control Flow**:
    - The function first calculates the x and y offsets by adding the respective offsets from the context to the provided coordinates.
    - It checks if the `bigger` flag in the context is not set; if so, it returns 1, indicating the area is visible.
    - Next, it checks if the calculated area defined by the offsets and dimensions is outside the bounds of the terminal's window.
    - If the area is outside the bounds, it returns 0, indicating the area is not visible; otherwise, it returns 1.
- **Output**:
    - The function returns an integer: 1 if the specified area is visible within the terminal's window, and 0 if it is not.


---
### tty_clamp_line<!-- {{#callable:tty_clamp_line}} -->
The `tty_clamp_line` function adjusts the position and size of a line within a terminal context to ensure it is visible.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing the context information for the terminal.
    - `px`: The x-coordinate of the line to be clamped.
    - `py`: The y-coordinate of the line to be clamped.
    - `nx`: The width of the line to be clamped.
    - `i`: A pointer to an unsigned integer where the starting index of the visible line will be stored.
    - `x`: A pointer to an unsigned integer where the adjusted x-coordinate will be stored.
    - `rx`: A pointer to an unsigned integer where the width of the visible portion of the line will be stored.
    - `ry`: A pointer to an unsigned integer where the adjusted y-coordinate will be stored.
- **Control Flow**:
    - The function first calculates the x-offset by adding the `rxoff` from the context to the `px` input.
    - It checks if the line is visible using the [`tty_is_visible`](#tty_is_visible) function; if not, it returns 0.
    - The adjusted y-coordinate is calculated and stored in `ry`.
    - The function then checks various conditions to determine how much of the line is visible and adjusts the indices accordingly:
    - 1. If the entire line is visible, it sets `i`, `x`, and `rx` to represent the full line.
    - 2. If both ends of the line are outside the visible area, it sets `i`, `x`, and `rx` to represent the maximum visible width.
    - 3. If only the left end is outside, it calculates the visible portion from the right.
    - 4. If only the right end is outside, it calculates the visible portion from the left.
    - Finally, it checks if the calculated `rx` exceeds `nx`, and if so, it triggers a fatal error.
- **Output**:
    - The function returns 1 if the line is successfully clamped and visible, or 0 if it is not visible.
- **Functions called**:
    - [`tty_is_visible`](#tty_is_visible)


---
### tty_clear_line<!-- {{#callable:tty_clear_line}} -->
Clears a specified line in a terminal, using escape sequences or spaces based on the terminal's capabilities.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `defaults`: A pointer to a `struct grid_cell` containing default cell attributes.
    - `py`: An unsigned integer representing the y-coordinate (row) of the line to clear.
    - `px`: An unsigned integer representing the x-coordinate (column) where the clearing starts.
    - `nx`: An unsigned integer indicating the number of characters to clear.
    - `bg`: An unsigned integer representing the background color to use when clearing.
- **Control Flow**:
    - Logs the function call with the client's name and parameters.
    - Checks if `nx` is zero; if so, returns immediately as there is nothing to clear.
    - Checks if genuine Background Color Erasure (BCE) is available; if not, attempts to use escape sequences based on the terminal's capabilities.
    - If the clearing area extends beyond the terminal width, uses the EL escape sequence to clear to the end of the line.
    - If clearing starts at the beginning of the line, uses the EL1 escape sequence.
    - If clearing a section of the line, uses the ECH escape sequence to clear the specified number of characters.
    - If escape sequences cannot be used, checks for overlays and clears the visible parts using spaces.
- **Output**:
    - The function does not return a value; it modifies the terminal display by clearing the specified line.
- **Functions called**:
    - [`tty_fake_bce`](#tty_fake_bce)
    - [`tty_cursor`](#tty_cursor)
    - [`tty_putcode`](#tty_putcode)
    - [`tty_putcode_i`](#tty_putcode_i)
    - [`tty_check_overlay_range`](#tty_check_overlay_range)
    - [`tty_repeat_space`](#tty_repeat_space)


---
### tty_clear_pane_line<!-- {{#callable:tty_clear_pane_line}} -->
Clears a specified line in a terminal pane, adjusting for visibility.
- **Inputs**:
    - `tty`: A pointer to the `tty` structure representing the terminal.
    - `ctx`: A pointer to the `tty_ctx` structure containing context information for the terminal pane.
    - `py`: The y-coordinate of the line to be cleared.
    - `px`: The x-coordinate of the starting position for clearing.
    - `nx`: The number of characters to clear from the line.
    - `bg`: The background color to use when clearing the line.
- **Control Flow**:
    - Logs the action of clearing the line with the client's name and parameters.
    - Calls [`tty_clamp_line`](#tty_clamp_line) to determine the visible portion of the line to clear.
    - If the line is visible, it calls [`tty_clear_line`](#tty_clear_line) to perform the actual clearing operation.
- **Output**:
    - The function does not return a value; it modifies the terminal display by clearing the specified line.
- **Functions called**:
    - [`tty_clamp_line`](#tty_clamp_line)
    - [`tty_clear_line`](#tty_clear_line)


---
### tty_clamp_area<!-- {{#callable:tty_clamp_area}} -->
The `tty_clamp_area` function adjusts the coordinates and dimensions of a specified area to ensure it fits within the visible region of a terminal pane.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing the context of the terminal pane.
    - `px`: The x-coordinate of the top-left corner of the area to clamp.
    - `py`: The y-coordinate of the top-left corner of the area to clamp.
    - `nx`: The width of the area to clamp.
    - `ny`: The height of the area to clamp.
    - `i`: A pointer to an unsigned integer to store the adjusted x-offset.
    - `j`: A pointer to an unsigned integer to store the adjusted y-offset.
    - `x`: A pointer to an unsigned integer to store the adjusted x-coordinate.
    - `y`: A pointer to an unsigned integer to store the adjusted y-coordinate.
    - `rx`: A pointer to an unsigned integer to store the clamped width.
    - `ry`: A pointer to an unsigned integer to store the clamped height.
- **Control Flow**:
    - The function first calculates the absolute offsets for the x and y coordinates based on the context offsets.
    - It checks if the specified area is visible using the [`tty_is_visible`](#tty_is_visible) function.
    - If the area is fully visible, it sets the output parameters accordingly.
    - If the area extends beyond the left and right bounds, it adjusts the parameters to fit within the visible width.
    - If the area extends beyond the top and bottom bounds, it adjusts the parameters to fit within the visible height.
    - The function ensures that the calculated dimensions do not exceed the original dimensions.
- **Output**:
    - The function returns 1 if the area is clamped successfully, or 0 if the area is not visible.
- **Functions called**:
    - [`tty_is_visible`](#tty_is_visible)


---
### tty_clear_area<!-- {{#callable:tty_clear_area}} -->
The `tty_clear_area` function clears a specified rectangular area on a terminal screen.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `defaults`: A pointer to a `struct grid_cell` containing default cell attributes.
    - `py`: The starting y-coordinate (row) of the area to clear.
    - `ny`: The number of rows to clear.
    - `px`: The starting x-coordinate (column) of the area to clear.
    - `nx`: The number of columns to clear.
    - `bg`: The background color to use for clearing.
- **Control Flow**:
    - Logs the function call with parameters for debugging purposes.
    - Checks if the area to clear is valid (non-zero dimensions).
    - If genuine Background Color Erasure (BCE) is available, attempts to use escape sequences to clear the area.
    - If the area is at the bottom of the terminal, uses the ED escape sequence to clear it.
    - If the terminal supports DECFRA and the background color is not default, uses a specific escape sequence to clear the area.
    - If the area can be scrolled away, uses scrolling commands to clear it.
    - If none of the above methods are applicable, iterates over each line in the specified area and calls [`tty_clear_line`](#tty_clear_line) to clear it.
- **Output**:
    - The function does not return a value; it directly modifies the terminal display by clearing the specified area.
- **Functions called**:
    - [`tty_fake_bce`](#tty_fake_bce)
    - [`tty_cursor`](#tty_cursor)
    - [`tty_putcode`](#tty_putcode)
    - [`tty_puts`](#tty_puts)
    - [`tty_region`](#tty_region)
    - [`tty_margin_off`](#tty_margin_off)
    - [`tty_putcode_i`](#tty_putcode_i)
    - [`tty_margin`](#tty_margin)
    - [`tty_clear_line`](#tty_clear_line)


---
### tty_clear_pane_area<!-- {{#callable:tty_clear_pane_area}} -->
Clears a specified area of a terminal pane.
- **Inputs**:
    - `tty`: A pointer to the `tty` structure representing the terminal.
    - `ctx`: A pointer to the `tty_ctx` structure containing context information for the terminal pane.
    - `py`: The starting y-coordinate (row) of the area to clear.
    - `ny`: The number of rows to clear.
    - `px`: The starting x-coordinate (column) of the area to clear.
    - `nx`: The number of columns to clear.
    - `bg`: The background color to use when clearing the area.
- **Control Flow**:
    - Calls [`tty_clamp_area`](#tty_clamp_area) to determine the visible portion of the area to clear.
    - If the area is valid (i.e., it is visible), it calls [`tty_clear_area`](#tty_clear_area) to perform the actual clearing.
- **Output**:
    - The function does not return a value; it modifies the terminal display by clearing the specified area.
- **Functions called**:
    - [`tty_clamp_area`](#tty_clamp_area)
    - [`tty_clear_area`](#tty_clear_area)


---
### tty_draw_pane<!-- {{#callable:tty_draw_pane}} -->
The `tty_draw_pane` function draws a specific line of a pane in a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing the drawing context, including screen and pane dimensions.
    - `py`: An unsigned integer representing the y-coordinate of the line to be drawn.
- **Control Flow**:
    - Logs the function call with the client's name and the y-coordinate.
    - Checks if the pane is not bigger than the terminal; if so, it calls [`tty_draw_line`](#tty_draw_line) to draw the line directly.
    - If the pane is bigger, it calls [`tty_clamp_line`](#tty_clamp_line) to determine the visible portion of the line and then draws it using [`tty_draw_line`](#tty_draw_line).
- **Output**:
    - The function does not return a value; it directly modifies the terminal display by drawing the specified line of the pane.
- **Functions called**:
    - [`tty_draw_line`](#tty_draw_line)
    - [`tty_clamp_line`](#tty_clamp_line)


---
### tty_check_codeset<!-- {{#callable:tty_check_codeset}} -->
The `tty_check_codeset` function validates and potentially modifies a `grid_cell` based on the terminal's capabilities.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal context.
    - `gc`: A pointer to a `grid_cell` structure containing the character data and attributes to be checked.
- **Control Flow**:
    - Checks if the character size is 1 and its value is less than 0x7f, returning the original `grid_cell` if true.
    - If the `grid_cell` has the `GRID_FLAG_TAB` flag set, it returns the original `grid_cell`.
    - If the terminal supports UTF-8 and the character is valid UTF-8, it returns the original `grid_cell`.
    - Copies the original `grid_cell` to a new static `grid_cell` for modification.
    - Attempts to map the character to an ACS (Alternate Character Set) character using `tty_acs_reverse_get`.
    - If a valid ACS character is found, it updates the new `grid_cell` with this character and returns it.
    - If no valid ACS character is found, it replaces the character data with underscores based on the width of the original character and returns the modified `grid_cell`.
- **Output**:
    - Returns a pointer to a potentially modified `grid_cell`, which may represent the original character, an ACS character, or a series of underscores.


---
### tty_check_overlay<!-- {{#callable:tty_check_overlay}} -->
Checks if a specific position on a `tty` is obstructed by an overlay.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal.
    - `px`: An unsigned integer representing the x-coordinate position to check.
    - `py`: An unsigned integer representing the y-coordinate position to check.
- **Control Flow**:
    - Calls [`tty_check_overlay_range`](#tty_check_overlay_range) to check the overlay ranges at the specified position (px, py) with a width of 1.
    - If the sum of the first two entries in the `overlay_ranges` structure is zero, it indicates no overlay is present at the position.
    - Returns 0 if no overlay is present, otherwise returns 1.
- **Output**:
    - Returns 1 if the position is obstructed by an overlay, otherwise returns 0.
- **Functions called**:
    - [`tty_check_overlay_range`](#tty_check_overlay_range)


---
### tty_check_overlay_range<!-- {{#callable:tty_check_overlay_range}} -->
The `tty_check_overlay_range` function checks the visibility of a specified range of characters in a terminal overlay.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `px`: An unsigned integer representing the starting x-coordinate of the range to check.
    - `py`: An unsigned integer representing the y-coordinate of the range to check.
    - `nx`: An unsigned integer representing the width of the range to check.
    - `r`: A pointer to a `struct overlay_ranges` where the results of the visibility check will be stored.
- **Control Flow**:
    - The function retrieves the `client` associated with the `tty` structure.
    - It checks if the `overlay_check` function pointer of the `client` is NULL.
    - If `overlay_check` is NULL, it sets the first entry of the `overlay_ranges` structure to the input range and the rest to zero, then returns.
    - If `overlay_check` is not NULL, it calls the `overlay_check` function with the provided parameters to fill the `overlay_ranges` structure.
- **Output**:
    - The function does not return a value but populates the `overlay_ranges` structure with the visible parts of the specified range.


---
### tty_draw_line<!-- {{#callable:tty_draw_line}} -->
The `tty_draw_line` function draws a line on a terminal screen based on specified parameters.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `s`: A pointer to a `struct screen` representing the screen to draw on.
    - `px`: The starting x-coordinate in the grid from where to draw.
    - `py`: The y-coordinate in the grid where the line will be drawn.
    - `nx`: The width of the line to be drawn.
    - `atx`: The x-coordinate on the terminal where the line will be drawn.
    - `aty`: The y-coordinate on the terminal where the line will be drawn.
    - `defaults`: A pointer to a `struct grid_cell` containing default attributes for the line.
    - `palette`: A pointer to a `struct colour_palette` for color management.
- **Control Flow**:
    - The function begins by logging the input parameters for debugging purposes.
    - It sets the terminal to a no-cursor mode and updates the terminal mode based on the current screen.
    - The width of the line to be drawn is clamped to the screen size and the cell size.
    - If the line is not wrapped and certain conditions are met, it clears the line using terminal escape codes.
    - The function iterates over the specified width, retrieving grid cells and checking their attributes.
    - It handles drawing characters, checking for overlays, and managing visibility based on the terminal's capabilities.
    - Finally, it clears any remaining space on the line if necessary and restores the cursor state.
- **Output**:
    - The function does not return a value; it directly modifies the terminal display by drawing the specified line.
- **Functions called**:
    - [`tty_update_mode`](#tty_update_mode)
    - [`tty_region_off`](#tty_region_off)
    - [`tty_margin_off`](#tty_margin_off)
    - [`tty_fake_bce`](#tty_fake_bce)
    - [`tty_default_attributes`](#tty_default_attributes)
    - [`tty_cursor`](#tty_cursor)
    - [`tty_putcode`](#tty_putcode)
    - [`tty_check_codeset`](#tty_check_codeset)
    - [`tty_check_overlay`](#tty_check_overlay)
    - [`tty_attributes`](#tty_attributes)
    - [`tty_clear_line`](#tty_clear_line)
    - [`tty_putn`](#tty_putn)
    - [`tty_check_overlay_range`](#tty_check_overlay_range)
    - [`tty_repeat_space`](#tty_repeat_space)
    - [`tty_putc`](#tty_putc)


---
### tty_set_client_cb<!-- {{#callable:tty_set_client_cb}} -->
The `tty_set_client_cb` function updates the properties of a `tty_ctx` based on the current client and its associated window.
- **Inputs**:
    - `ttyctx`: A pointer to a `tty_ctx` structure that holds the terminal context information.
    - `c`: A pointer to a `client` structure representing the current client.
- **Control Flow**:
    - Checks if the current client's window matches the window associated with the `tty_ctx`; if not, returns 0.
    - Checks if the layout cell of the window pane is NULL; if so, returns 0.
    - Calls [`tty_window_offset`](#tty_window_offset) to set the offsets and dimensions of the window in the `tty_ctx`.
    - Sets the vertical offsets in the `tty_ctx` based on the window pane's y-offset and the status line size.
- **Output**:
    - Returns 1 if the properties were successfully set, otherwise returns 0.
- **Functions called**:
    - [`tty_window_offset`](#tty_window_offset)


---
### tty_draw_images<!-- {{#callable:tty_draw_images}} -->
The `tty_draw_images` function draws images on a terminal screen for a specified client and window pane.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client for which the images are being drawn.
    - `wp`: A pointer to a `struct window_pane` representing the window pane where the images will be drawn.
    - `s`: A pointer to a `struct screen` containing the images to be drawn.
- **Control Flow**:
    - Iterates over each image in the `screen`'s images list using `TAILQ_FOREACH`.
    - For each image, initializes a `tty_ctx` structure to hold drawing context.
    - Sets various properties of `ttyctx` based on the image's position and the window pane's dimensions.
    - Calls [`tty_write_one`](#tty_write_one) to send the drawing command for each image to the client.
- **Output**:
    - The function does not return a value; instead, it sends drawing commands to the terminal to render the images on the specified window pane.
- **Functions called**:
    - [`tty_write_one`](#tty_write_one)


---
### tty_sync_start<!-- {{#callable:tty_sync_start}} -->
The `tty_sync_start` function initiates synchronization for a terminal interface if it is not already blocked or syncing.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal interface to be synchronized.
- **Control Flow**:
    - Check if the `TTY_BLOCK` flag is set in `tty->flags`; if so, exit the function immediately.
    - Check if the `TTY_SYNCING` flag is set in `tty->flags`; if so, exit the function immediately.
    - Set the `TTY_SYNCING` flag in `tty->flags` to indicate that synchronization has started.
    - Check if the terminal supports the `TTYC_SYNC` capability; if it does, log a debug message indicating the start of synchronization.
    - Send a synchronization command to the terminal using [`tty_putcode_i`](#tty_putcode_i).
- **Output**:
    - The function does not return a value; it modifies the state of the `tty` structure and may send a synchronization command to the terminal.
- **Functions called**:
    - [`tty_putcode_i`](#tty_putcode_i)


---
### tty_sync_end<!-- {{#callable:tty_sync_end}} -->
The `tty_sync_end` function ends a synchronization process for a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal interface.
- **Control Flow**:
    - Checks if the `TTY_BLOCK` flag is set in the `tty` structure; if so, the function returns immediately.
    - Checks if the `TTY_SYNCING` flag is not set; if so, the function returns immediately.
    - Clears the `TTY_SYNCING` flag from the `tty` structure.
    - Checks if the terminal supports the `TTYC_SYNC` capability; if so, it logs a debug message and sends a synchronization end code to the terminal.
- **Output**:
    - The function does not return a value; it modifies the state of the `tty` structure and may send output to the terminal.
- **Functions called**:
    - [`tty_putcode_i`](#tty_putcode_i)


---
### tty_client_ready<!-- {{#callable:tty_client_ready}} -->
The `tty_client_ready` function checks if a terminal client is ready for interaction.
- **Inputs**:
    - `ctx`: A pointer to a `tty_ctx` structure that contains context information for the terminal.
    - `c`: A pointer to a `client` structure representing the terminal client.
- **Control Flow**:
    - The function first checks if the client's session or terminal is NULL, returning 0 if either is true.
    - It then checks if the client is suspended, returning 0 if it is.
    - If invisible panes are allowed, the function returns 1, indicating the client is ready.
    - Next, it checks if the client has the `CLIENT_REDRAWWINDOW` flag set, returning 0 if true.
    - It checks if the terminal is frozen (indicated by the `TTY_FREEZE` flag), returning 0 if it is.
    - If none of the above conditions are met, the function returns 1, indicating the client is ready.
- **Output**:
    - The function returns an integer: 0 if the client is not ready, and 1 if the client is ready for interaction.


---
### tty_write<!-- {{#callable:tty_write}} -->
The `tty_write` function sends commands to all ready clients based on a provided context.
- **Inputs**:
    - `cmdfn`: A function pointer to a command function that takes a `struct tty *` and a `struct tty_ctx *` as parameters.
    - `ctx`: A pointer to a `struct tty_ctx` that contains context information for the command execution.
- **Control Flow**:
    - Checks if the `set_client_cb` callback in the context is NULL; if so, it returns immediately.
    - Iterates over all clients in the `clients` list using `TAILQ_FOREACH`.
    - For each client, it checks if the client is ready using the [`tty_client_ready`](#tty_client_ready) function.
    - If the client is ready, it calls the `set_client_cb` callback with the context and client, storing the return state.
    - If the state is -1, the loop breaks, indicating an error or stop condition.
    - If the state is 0, it continues to the next client without executing the command.
    - If the state is positive, it calls the command function `cmdfn` with the client's `tty` and the context.
- **Output**:
    - The function does not return a value; it executes the command function for each ready client based on the context provided.
- **Functions called**:
    - [`tty_client_ready`](#tty_client_ready)


---
### tty_write_one<!-- {{#callable:tty_write_one}} -->
The `tty_write_one` function writes a command to a specific client terminal if the client is ready.
- **Inputs**:
    - `cmdfn`: A function pointer that takes a `struct tty *` and a `const struct tty_ctx *` as parameters, representing the command to execute.
    - `c`: A pointer to a `struct client`, representing the client whose terminal will receive the command.
    - `ctx`: A pointer to a `struct tty_ctx`, containing context information for the terminal operation.
- **Control Flow**:
    - Check if the `set_client_cb` function pointer in `ctx` is NULL; if it is, return immediately.
    - Call the `set_client_cb` function with `ctx` and `c` as arguments, storing the return value in `state`.
    - If `state` equals 1, invoke the command function `cmdfn` with the `tty` associated with the client `c` and the context `ctx`.
- **Output**:
    - The function does not return a value; it performs an action based on the command function and the client state.


---
### tty_cmd_insertcharacter<!-- {{#callable:tty_cmd_insertcharacter}} -->
Inserts a character at the current cursor position in a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal.
    - `ctx`: A pointer to a `tty_ctx` structure containing context information such as cursor position and attributes.
- **Control Flow**:
    - Checks if the terminal context indicates a larger size or if the terminal is not full width.
    - Checks if background color emulation is needed and if the terminal supports the insert character command.
    - If any of the above checks fail, it redraws the pane at the current cursor Y position and exits.
    - Sets the default attributes for the terminal based on the context provided.
    - Moves the cursor to the specified position in the pane.
    - Emulates the insertion of the character using the appropriate terminal codes.
- **Output**:
    - No return value; the function modifies the terminal display by inserting a character at the cursor position.
- **Functions called**:
    - [`tty_fake_bce`](#tty_fake_bce)
    - [`tty_draw_pane`](#tty_draw_pane)
    - [`tty_default_attributes`](#tty_default_attributes)
    - [`tty_cursor_pane`](#tty_cursor_pane)
    - [`tty_emulate_repeat`](#tty_emulate_repeat)


---
### tty_cmd_deletecharacter<!-- {{#callable:tty_cmd_deletecharacter}} -->
The `tty_cmd_deletecharacter` function deletes a character at the current cursor position in a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing the context for the operation, including cursor position and attributes.
- **Control Flow**:
    - The function first retrieves the associated `client` from the `tty` structure.
    - It checks several conditions to determine if the character deletion can proceed, including whether the context is larger than the terminal, if the terminal supports the delete character operation, and if there are any overlays present.
    - If any of these conditions are met, it calls [`tty_draw_pane`](#tty_draw_pane) to redraw the pane at the current cursor y-coordinate and exits the function.
    - If the conditions are not met, it sets the default attributes for the terminal using [`tty_default_attributes`](#tty_default_attributes).
    - The cursor position is updated using [`tty_cursor_pane`](#tty_cursor_pane) to reflect the current context.
    - Finally, it calls [`tty_emulate_repeat`](#tty_emulate_repeat) to perform the actual character deletion operation, potentially repeating it based on the context's `num` value.
- **Output**:
    - The function does not return a value; it modifies the terminal display by deleting a character at the cursor position.
- **Functions called**:
    - [`tty_fake_bce`](#tty_fake_bce)
    - [`tty_draw_pane`](#tty_draw_pane)
    - [`tty_default_attributes`](#tty_default_attributes)
    - [`tty_cursor_pane`](#tty_cursor_pane)
    - [`tty_emulate_repeat`](#tty_emulate_repeat)


---
### tty_cmd_clearcharacter<!-- {{#callable:tty_cmd_clearcharacter}} -->
Clears a specified number of characters from the terminal at the current cursor position.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing the context for the operation, including cursor position and attributes.
- **Control Flow**:
    - Calls [`tty_default_attributes`](#tty_default_attributes) to set the terminal's default attributes based on the provided context.
    - Invokes [`tty_clear_pane_line`](#tty_clear_pane_line) to clear the specified number of characters from the line at the cursor's current position.
- **Output**:
    - The function does not return a value; it modifies the terminal display by clearing characters.
- **Functions called**:
    - [`tty_default_attributes`](#tty_default_attributes)
    - [`tty_clear_pane_line`](#tty_clear_pane_line)


---
### tty_cmd_insertline<!-- {{#callable:tty_cmd_insertline}} -->
Inserts a new line in the terminal at the current cursor position.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing the context for the operation, including cursor position and dimensions.
- **Control Flow**:
    - Checks if the terminal context indicates a larger size or if the terminal does not support full width.
    - Verifies if the terminal supports the necessary capabilities for inserting lines.
    - If any of the above checks fail, it calls [`tty_redraw_region`](#tty_redraw_region) to redraw the affected region.
    - Sets the default attributes for the terminal using [`tty_default_attributes`](#tty_default_attributes).
    - Defines the region for the pane using [`tty_region_pane`](#tty_region_pane).
    - Disables margins with [`tty_margin_off`](#tty_margin_off).
    - Moves the cursor to the specified position using [`tty_cursor_pane`](#tty_cursor_pane).
    - Emulates the line insertion using [`tty_emulate_repeat`](#tty_emulate_repeat) with the appropriate terminal codes.
    - Resets the cursor position to an invalid state.
- **Output**:
    - The function does not return a value but modifies the terminal display by inserting a line at the current cursor position.
- **Functions called**:
    - [`tty_fake_bce`](#tty_fake_bce)
    - [`tty_redraw_region`](#tty_redraw_region)
    - [`tty_default_attributes`](#tty_default_attributes)
    - [`tty_region_pane`](#tty_region_pane)
    - [`tty_margin_off`](#tty_margin_off)
    - [`tty_cursor_pane`](#tty_cursor_pane)
    - [`tty_emulate_repeat`](#tty_emulate_repeat)


---
### tty_cmd_deleteline<!-- {{#callable:tty_cmd_deleteline}} -->
The `tty_cmd_deleteline` function deletes a line in a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing the context for the operation, including cursor position and dimensions.
- **Control Flow**:
    - The function first retrieves the associated `client` from the `tty` structure.
    - It checks several conditions to determine if the line deletion can proceed, including terminal capabilities and context dimensions.
    - If any condition fails, it calls [`tty_redraw_region`](#tty_redraw_region) to refresh the display instead of deleting the line.
    - If conditions are met, it sets the default attributes for the terminal using [`tty_default_attributes`](#tty_default_attributes).
    - It defines the region for the pane using [`tty_region_pane`](#tty_region_pane) and disables margins with [`tty_margin_off`](#tty_margin_off).
    - The cursor position is set using [`tty_cursor_pane`](#tty_cursor_pane).
    - Finally, it emulates the line deletion using [`tty_emulate_repeat`](#tty_emulate_repeat) and resets the cursor position to an invalid state.
- **Output**:
    - The function does not return a value; it modifies the terminal display by deleting a line or refreshing the display based on the conditions checked.
- **Functions called**:
    - [`tty_fake_bce`](#tty_fake_bce)
    - [`tty_redraw_region`](#tty_redraw_region)
    - [`tty_default_attributes`](#tty_default_attributes)
    - [`tty_region_pane`](#tty_region_pane)
    - [`tty_margin_off`](#tty_margin_off)
    - [`tty_cursor_pane`](#tty_cursor_pane)
    - [`tty_emulate_repeat`](#tty_emulate_repeat)


---
### tty_cmd_clearline<!-- {{#callable:tty_cmd_clearline}} -->
Clears the current line in the terminal.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal.
    - `ctx`: A pointer to a `struct tty_ctx` containing context information such as default attributes and cursor position.
- **Control Flow**:
    - Calls [`tty_default_attributes`](#tty_default_attributes) to set the terminal's default attributes based on the provided context.
    - Invokes [`tty_clear_pane_line`](#tty_clear_pane_line) to clear the specified line in the terminal using the attributes set earlier.
- **Output**:
    - The function does not return a value; it modifies the terminal display by clearing the current line.
- **Functions called**:
    - [`tty_default_attributes`](#tty_default_attributes)
    - [`tty_clear_pane_line`](#tty_clear_pane_line)


---
### tty_cmd_clearendofline<!-- {{#callable:tty_cmd_clearendofline}} -->
Clears the end of the current line in a terminal.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal.
    - `ctx`: A pointer to a `struct tty_ctx` containing context information such as cursor position and screen dimensions.
- **Control Flow**:
    - Calculates the number of columns to clear from the current cursor position to the end of the line.
    - Calls [`tty_default_attributes`](#tty_default_attributes) to set the terminal attributes based on the provided context.
    - Calls [`tty_clear_pane_line`](#tty_clear_pane_line) to perform the actual clearing of the line from the cursor position to the end.
- **Output**:
    - The function does not return a value; it modifies the terminal display by clearing part of the line.
- **Functions called**:
    - [`tty_default_attributes`](#tty_default_attributes)
    - [`tty_clear_pane_line`](#tty_clear_pane_line)


---
### tty_cmd_clearstartofline<!-- {{#callable:tty_cmd_clearstartofline}} -->
Clears the start of the current line in a terminal context.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing the context for the terminal operations, including cursor position and default attributes.
- **Control Flow**:
    - Calls [`tty_default_attributes`](#tty_default_attributes) to set the terminal's default attributes based on the provided context.
    - Invokes [`tty_clear_pane_line`](#tty_clear_pane_line) to clear the line from the start up to the current cursor position, effectively erasing any text in that range.
- **Output**:
    - The function does not return a value; it modifies the terminal display by clearing part of the line.
- **Functions called**:
    - [`tty_default_attributes`](#tty_default_attributes)
    - [`tty_clear_pane_line`](#tty_clear_pane_line)


---
### tty_cmd_reverseindex<!-- {{#callable:tty_cmd_reverseindex}} -->
The `tty_cmd_reverseindex` function handles the reverse index operation for a terminal, adjusting the cursor position and redrawing the region if necessary.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing the current context, including cursor position and screen dimensions.
- **Control Flow**:
    - Checks if the cursor's y-coordinate (`ocy`) matches the upper boundary of the region (`orupper`); if not, the function returns immediately.
    - Evaluates several conditions to determine if the terminal can perform a reverse index operation, including checks for terminal capabilities and dimensions.
    - If any of the conditions are not met, it calls [`tty_redraw_region`](#tty_redraw_region) to redraw the affected region and returns.
    - Sets the default attributes for the terminal using [`tty_default_attributes`](#tty_default_attributes).
    - Defines the region for the pane using [`tty_region_pane`](#tty_region_pane) and sets the margin using [`tty_margin_pane`](#tty_margin_pane).
    - Moves the cursor to the specified position using [`tty_cursor_pane`](#tty_cursor_pane).
    - Finally, it sends the appropriate escape code for reverse indexing using [`tty_putcode`](#tty_putcode) or [`tty_putcode_i`](#tty_putcode_i).
- **Output**:
    - The function does not return a value but modifies the terminal's state and cursor position based on the operations performed.
- **Functions called**:
    - [`tty_fake_bce`](#tty_fake_bce)
    - [`tty_redraw_region`](#tty_redraw_region)
    - [`tty_default_attributes`](#tty_default_attributes)
    - [`tty_region_pane`](#tty_region_pane)
    - [`tty_margin_pane`](#tty_margin_pane)
    - [`tty_cursor_pane`](#tty_cursor_pane)
    - [`tty_putcode`](#tty_putcode)
    - [`tty_putcode_i`](#tty_putcode_i)


---
### tty_cmd_linefeed<!-- {{#callable:tty_cmd_linefeed}} -->
The `tty_cmd_linefeed` function handles the line feed operation in a terminal context.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing the current terminal context information.
- **Control Flow**:
    - The function first checks if the cursor's y-coordinate (`ctx->ocy`) is equal to the lower boundary of the region (`ctx->orlower`); if not, it returns immediately.
    - It then checks several conditions related to the terminal's capabilities and the context state, such as whether the terminal is in a larger state, if it can use margins, if it supports certain terminal capabilities, and if overlays are present.
    - If any of these conditions are met, it calls [`tty_redraw_region`](#tty_redraw_region) to redraw the region instead of performing a line feed.
    - If the conditions are not met, it sets the default attributes for the terminal using [`tty_default_attributes`](#tty_default_attributes).
    - Next, it sets the region pane and margin for the terminal using [`tty_region_pane`](#tty_region_pane) and [`tty_margin_pane`](#tty_margin_pane).
    - The function then checks if the cursor position exceeds the right margin; if so, it adjusts the cursor position accordingly.
    - Finally, it outputs a newline character to the terminal using [`tty_putc`](#tty_putc).
- **Output**:
    - The function does not return a value; it performs actions directly on the terminal context, resulting in a line feed operation being executed in the terminal.
- **Functions called**:
    - [`tty_fake_bce`](#tty_fake_bce)
    - [`tty_redraw_region`](#tty_redraw_region)
    - [`tty_default_attributes`](#tty_default_attributes)
    - [`tty_region_pane`](#tty_region_pane)
    - [`tty_margin_pane`](#tty_margin_pane)
    - [`tty_cursor`](#tty_cursor)
    - [`tty_cursor_pane`](#tty_cursor_pane)
    - [`tty_putc`](#tty_putc)


---
### tty_cmd_scrollup<!-- {{#callable:tty_cmd_scrollup}} -->
The `tty_cmd_scrollup` function scrolls the terminal display up by a specified number of lines.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing context information such as cursor position and scrolling parameters.
- **Control Flow**:
    - Checks if the terminal context indicates a larger display or if certain terminal capabilities are not supported.
    - If any of the conditions are met, it calls [`tty_redraw_region`](#tty_redraw_region) to redraw the current region instead of scrolling.
    - Sets default attributes for the terminal using [`tty_default_attributes`](#tty_default_attributes).
    - Defines the scroll region using [`tty_region_pane`](#tty_region_pane) and sets margins with [`tty_margin_pane`](#tty_margin_pane).
    - If the number of lines to scroll is 1 or the terminal does not support the `INDN` capability, it moves the cursor to the appropriate position and outputs new line characters.
    - If the terminal supports the `INDN` capability, it uses [`tty_putcode_i`](#tty_putcode_i) to scroll up the specified number of lines.
- **Output**:
    - The function does not return a value but modifies the terminal display by scrolling up the content and adjusting the cursor position accordingly.
- **Functions called**:
    - [`tty_fake_bce`](#tty_fake_bce)
    - [`tty_redraw_region`](#tty_redraw_region)
    - [`tty_default_attributes`](#tty_default_attributes)
    - [`tty_region_pane`](#tty_region_pane)
    - [`tty_margin_pane`](#tty_margin_pane)
    - [`tty_cursor`](#tty_cursor)
    - [`tty_putc`](#tty_putc)
    - [`tty_putcode_i`](#tty_putcode_i)


---
### tty_cmd_scrolldown<!-- {{#callable:tty_cmd_scrolldown}} -->
The `tty_cmd_scrolldown` function scrolls the terminal display down by a specified number of lines.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing the context for the operation, including attributes like cursor position and scrolling parameters.
- **Control Flow**:
    - The function first checks several conditions to determine if scrolling can be performed, such as terminal capabilities and context attributes.
    - If any of the conditions indicate that scrolling cannot be performed, it calls [`tty_redraw_region`](#tty_redraw_region) to refresh the display instead.
    - If scrolling is possible, it sets the default attributes for the terminal using [`tty_default_attributes`](#tty_default_attributes).
    - It defines the scroll region using [`tty_region_pane`](#tty_region_pane) and sets the cursor position with [`tty_cursor_pane`](#tty_cursor_pane).
    - Finally, it either sends a single scroll command using [`tty_putcode_i`](#tty_putcode_i) if the terminal supports it, or sends multiple individual scroll commands using a loop with [`tty_putcode`](#tty_putcode).
- **Output**:
    - The function does not return a value but modifies the terminal display by scrolling it down, or refreshing the display if scrolling is not possible.
- **Functions called**:
    - [`tty_fake_bce`](#tty_fake_bce)
    - [`tty_redraw_region`](#tty_redraw_region)
    - [`tty_default_attributes`](#tty_default_attributes)
    - [`tty_region_pane`](#tty_region_pane)
    - [`tty_margin_pane`](#tty_margin_pane)
    - [`tty_cursor_pane`](#tty_cursor_pane)
    - [`tty_putcode_i`](#tty_putcode_i)
    - [`tty_putcode`](#tty_putcode)


---
### tty_cmd_clearendofscreen<!-- {{#callable:tty_cmd_clearendofscreen}} -->
Clears the end of the screen from the current cursor position to the bottom.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal.
    - `ctx`: A pointer to a `tty_ctx` structure containing context information such as cursor position and screen dimensions.
- **Control Flow**:
    - Sets the default attributes for the terminal using [`tty_default_attributes`](#tty_default_attributes).
    - Defines the region of the pane to be cleared using [`tty_region_pane`](#tty_region_pane).
    - Disables margins with [`tty_margin_off`](#tty_margin_off).
    - Calculates the starting position (px, py) and dimensions (nx, ny) for the area to be cleared.
    - Calls [`tty_clear_pane_area`](#tty_clear_pane_area) to clear the area from the cursor position to the bottom of the screen.
    - Calculates the position and width for the line at the cursor's current row and clears it using [`tty_clear_pane_line`](#tty_clear_pane_line).
- **Output**:
    - The function does not return a value; it modifies the terminal display by clearing the specified area.
- **Functions called**:
    - [`tty_default_attributes`](#tty_default_attributes)
    - [`tty_region_pane`](#tty_region_pane)
    - [`tty_margin_off`](#tty_margin_off)
    - [`tty_clear_pane_area`](#tty_clear_pane_area)
    - [`tty_clear_pane_line`](#tty_clear_pane_line)


---
### tty_cmd_clearstartofscreen<!-- {{#callable:tty_cmd_clearstartofscreen}} -->
Clears the start of the screen in a terminal context.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal.
    - `ctx`: A pointer to a `struct tty_ctx` containing context information such as dimensions and attributes.
- **Control Flow**:
    - Sets default attributes for the terminal using [`tty_default_attributes`](#tty_default_attributes).
    - Defines the region of the terminal pane to be cleared using [`tty_region_pane`](#tty_region_pane).
    - Disables margins with [`tty_margin_off`](#tty_margin_off).
    - Calculates the coordinates for the area to be cleared, including the start of the screen to the current cursor position.
    - Calls [`tty_clear_pane_area`](#tty_clear_pane_area) to clear the area from the top of the screen to the current cursor Y position.
    - Calls [`tty_clear_pane_line`](#tty_clear_pane_line) to clear the line from the start of the line to the current cursor X position.
- **Output**:
    - The function does not return a value; it modifies the terminal display by clearing specified areas.
- **Functions called**:
    - [`tty_default_attributes`](#tty_default_attributes)
    - [`tty_region_pane`](#tty_region_pane)
    - [`tty_margin_off`](#tty_margin_off)
    - [`tty_clear_pane_area`](#tty_clear_pane_area)
    - [`tty_clear_pane_line`](#tty_clear_pane_line)


---
### tty_cmd_clearscreen<!-- {{#callable:tty_cmd_clearscreen}} -->
Clears the entire screen in a terminal context.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal.
    - `ctx`: A pointer to a `tty_ctx` structure containing context information such as dimensions and attributes.
- **Control Flow**:
    - Calls [`tty_default_attributes`](#tty_default_attributes) to set the default attributes for the terminal based on the context.
    - Sets the region for the entire pane using [`tty_region_pane`](#tty_region_pane).
    - Disables any margins with [`tty_margin_off`](#tty_margin_off).
    - Defines the coordinates for the area to clear: starting from (0,0) to (ctx->sx, ctx->sy).
    - Calls [`tty_clear_pane_area`](#tty_clear_pane_area) to clear the defined area with the specified background color.
- **Output**:
    - The function does not return a value; it modifies the terminal display by clearing the screen.
- **Functions called**:
    - [`tty_default_attributes`](#tty_default_attributes)
    - [`tty_region_pane`](#tty_region_pane)
    - [`tty_margin_off`](#tty_margin_off)
    - [`tty_clear_pane_area`](#tty_clear_pane_area)


---
### tty_cmd_alignmenttest<!-- {{#callable:tty_cmd_alignmenttest}} -->
The `tty_cmd_alignmenttest` function fills the terminal screen with the character 'E' for alignment testing.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing the context for the terminal operations, including dimensions and attributes.
- **Control Flow**:
    - The function first checks if the `ctx->bigger` flag is set; if so, it calls the `redraw_cb` callback function and returns.
    - It sets the terminal attributes using [`tty_attributes`](#tty_attributes) with default cell attributes and the context's defaults.
    - The function defines a region for the entire height of the terminal using [`tty_region_pane`](#tty_region_pane).
    - It disables margins with [`tty_margin_off`](#tty_margin_off).
    - A nested loop iterates over each row (`j`) and each column (`i`) of the terminal dimensions, placing the character 'E' at each position using [`tty_putc`](#tty_putc).
- **Output**:
    - The function does not return a value; it directly modifies the terminal display by filling it with the character 'E' across the specified dimensions.
- **Functions called**:
    - [`tty_attributes`](#tty_attributes)
    - [`tty_region_pane`](#tty_region_pane)
    - [`tty_margin_off`](#tty_margin_off)
    - [`tty_cursor_pane`](#tty_cursor_pane)
    - [`tty_putc`](#tty_putc)


---
### tty_cmd_cell<!-- {{#callable:tty_cmd_cell}} -->
The `tty_cmd_cell` function handles the rendering of a single cell in a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing the context for the cell to be drawn, including cell attributes and position.
- **Control Flow**:
    - Calculates the terminal position of the cell based on the context offsets.
    - Checks if the cell is visible in the terminal; if not, the function returns early.
    - Handles partially obstructed wide characters by checking overlay ranges and drawing the visible part.
    - If the cursor position exceeds the terminal width and is at the lower edge, it adjusts the pane region.
    - Sets the margin off and positions the cursor unless it would wrap.
    - Draws the cell using the specified attributes and palette.
    - Invalidates the terminal if the context number is 1.
- **Output**:
    - The function does not return a value; it directly modifies the terminal display by rendering the specified cell.
- **Functions called**:
    - [`tty_is_visible`](#tty_is_visible)
    - [`tty_check_overlay`](#tty_check_overlay)
    - [`tty_check_overlay_range`](#tty_check_overlay_range)
    - [`tty_draw_line`](#tty_draw_line)
    - [`tty_region_pane`](#tty_region_pane)
    - [`tty_margin_off`](#tty_margin_off)
    - [`tty_cursor_pane_unless_wrap`](#tty_cursor_pane_unless_wrap)
    - [`tty_cell`](#tty_cell)
    - [`tty_invalidate`](#tty_invalidate)


---
### tty_cmd_cells<!-- {{#callable:tty_cmd_cells}} -->
The `tty_cmd_cells` function updates the terminal display with a specified number of character cells at a given position.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing the context for the operation, including position and attributes.
- **Control Flow**:
    - Checks if the specified cell position is visible on the terminal using [`tty_is_visible`](#tty_is_visible).
    - If the context indicates a larger area and the specified cells exceed the terminal's width, it decides whether to redraw the pane or call a redraw callback.
    - Disables margins with [`tty_margin_off`](#tty_margin_off) and positions the cursor unless wrapping is needed.
    - Sets the terminal attributes for the specified cell using [`tty_attributes`](#tty_attributes).
    - Calculates the terminal position for overlay checks and checks for any overlays that might obstruct the cells.
    - Iterates through the overlay ranges and prints the visible parts of the cells using [`tty_putn`](#tty_putn).
- **Output**:
    - The function does not return a value but updates the terminal display by drawing the specified character cells at the given position, taking into account visibility and overlays.
- **Functions called**:
    - [`tty_is_visible`](#tty_is_visible)
    - [`tty_draw_pane`](#tty_draw_pane)
    - [`tty_margin_off`](#tty_margin_off)
    - [`tty_cursor_pane_unless_wrap`](#tty_cursor_pane_unless_wrap)
    - [`tty_attributes`](#tty_attributes)
    - [`tty_check_overlay_range`](#tty_check_overlay_range)
    - [`tty_putn`](#tty_putn)


---
### tty_cmd_setselection<!-- {{#callable:tty_cmd_setselection}} -->
Sets the selection in the terminal based on the provided context.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal.
    - `ctx`: A pointer to a `struct tty_ctx` containing selection parameters such as pointers and length.
- **Control Flow**:
    - The function calls [`tty_set_selection`](#tty_set_selection) with the terminal pointer and selection parameters extracted from the context.
    - The parameters passed to [`tty_set_selection`](#tty_set_selection) include the selection flags, the pointer to the selection buffer, and the length of the selection.
- **Output**:
    - The function does not return a value; it modifies the terminal's selection state.
- **Functions called**:
    - [`tty_set_selection`](#tty_set_selection)


---
### tty_set_selection<!-- {{#callable:tty_set_selection}} -->
The `tty_set_selection` function encodes a given buffer into base64 and sends it to the terminal for selection purposes.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `flags`: A string containing flags that modify the selection behavior.
    - `buf`: A pointer to the buffer containing the data to be selected.
    - `len`: The length of the data in the buffer.
- **Control Flow**:
    - Check if the terminal has started by verifying the `TTY_STARTED` flag in `tty->flags`.
    - Return early if the terminal does not support mouse selection by checking `tty_term_has(tty->term, TTYC_MS)`.
    - Calculate the size needed for the base64 encoded string based on the length of the input buffer.
    - Allocate memory for the encoded string using `xmalloc`.
    - Encode the input buffer into base64 format using `b64_ntop`.
    - Set the `TTY_NOBLOCK` flag in `tty->flags` to indicate non-blocking operation.
    - Send the encoded selection data to the terminal using [`tty_putcode_ss`](#tty_putcode_ss) with the appropriate flags.
    - Free the allocated memory for the encoded string.
- **Output**:
    - The function does not return a value but sends the encoded selection data to the terminal, allowing the user to interact with the selected text.
- **Functions called**:
    - [`tty_putcode_ss`](#tty_putcode_ss)


---
### tty_cmd_rawstring<!-- {{#callable:tty_cmd_rawstring}} -->
The `tty_cmd_rawstring` function adds a raw string to the output buffer of a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal interface.
    - `ctx`: A pointer to a `struct tty_ctx` containing the context for the operation, including a pointer to the string and its length.
- **Control Flow**:
    - The function sets the `TTY_NOBLOCK` flag on the `tty` structure to indicate that the terminal should not block on output.
    - It calls the [`tty_add`](#tty_add) function to add the raw string from the context to the terminal's output buffer.
    - Finally, it calls [`tty_invalidate`](#tty_invalidate) to mark the terminal as needing to be redrawn.
- **Output**:
    - The function does not return a value; it modifies the state of the terminal by adding the raw string to its output buffer and marking it for redrawing.
- **Functions called**:
    - [`tty_add`](#tty_add)
    - [`tty_invalidate`](#tty_invalidate)


---
### tty_cmd_sixelimage<!-- {{#callable:tty_cmd_sixelimage}} -->
The `tty_cmd_sixelimage` function processes and displays a sixel image in a terminal context.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context where the image will be displayed.
    - `ctx`: A pointer to a `struct tty_ctx` containing the context for the terminal operation, including image data and cursor position.
- **Control Flow**:
    - Checks if the terminal supports sixel graphics; if not, sets a fallback flag.
    - Validates the terminal's pixel dimensions; if either dimension is zero, sets the fallback flag.
    - Calculates the size of the sixel image in cells and logs the dimensions.
    - Clamps the area where the image will be drawn to ensure it fits within the terminal's visible area.
    - If fallback is triggered, duplicates the fallback image data; otherwise, scales the sixel image to fit the terminal's pixel dimensions.
    - If the image data is valid, logs the size of the data, disables the terminal region and margin, and sets the cursor position.
    - Adds the image data to the terminal output buffer and invalidates the terminal state.
    - Frees the scaled image data if it was created.
- **Output**:
    - The function does not return a value but modifies the terminal's output buffer to display the sixel image or its fallback representation.
- **Functions called**:
    - [`tty_clamp_area`](#tty_clamp_area)
    - [`tty_region_off`](#tty_region_off)
    - [`tty_margin_off`](#tty_margin_off)
    - [`tty_cursor`](#tty_cursor)
    - [`tty_add`](#tty_add)
    - [`tty_invalidate`](#tty_invalidate)


---
### tty_cmd_syncstart<!-- {{#callable:tty_cmd_syncstart}} -->
The `tty_cmd_syncstart` function initiates synchronized updates for a terminal based on the context provided.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing context information, including the command number.
- **Control Flow**:
    - The function first checks if the `num` field of `ctx` is equal to 0x11, indicating an overlay command.
    - If it is an overlay command, it calls `tty_sync_start(tty)` to start synchronized updates.
    - If `ctx->num` is not 0x11 and the bitwise negation of `ctx->num` AND 0x10 is true, it checks if `ctx->num` is non-zero or if the terminal's client has an overlay draw function.
    - If either condition is true, it calls `tty_sync_start(tty)` to start synchronized updates.
- **Output**:
    - The function does not return a value; it modifies the state of the terminal to begin synchronized updates based on the conditions evaluated.
- **Functions called**:
    - [`tty_sync_start`](#tty_sync_start)


---
### tty_cell<!-- {{#callable:tty_cell}} -->
The `tty_cell` function writes a character or string to the terminal, applying the appropriate attributes and handling special cases.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `gc`: A pointer to a `struct grid_cell` containing the character data and its attributes.
    - `defaults`: A pointer to a `struct grid_cell` containing default attributes for the terminal.
    - `palette`: A pointer to a `struct colour_palette` used for color management.
    - `hl`: A pointer to a `struct hyperlinks` for managing hyperlink attributes.
- **Control Flow**:
    - Checks if the terminal is in a state where the last character should be skipped due to terminal limitations.
    - Returns early if the character is a padding character.
    - Calls [`tty_check_codeset`](#tty_check_codeset) to validate and possibly modify the character based on the terminal's output codeset.
    - Applies the attributes of the character using [`tty_attributes`](#tty_attributes).
    - If the character is a single byte, it checks if it is a control character and writes it using [`tty_putc`](#tty_putc) if valid.
    - If the character is multi-byte, it writes the data using [`tty_putn`](#tty_putn).
- **Output**:
    - The function does not return a value; instead, it directly writes to the terminal output based on the provided character and its attributes.
- **Functions called**:
    - [`tty_check_codeset`](#tty_check_codeset)
    - [`tty_attributes`](#tty_attributes)
    - [`tty_putc`](#tty_putc)
    - [`tty_putn`](#tty_putn)


---
### tty_reset<!-- {{#callable:tty_reset}} -->
The `tty_reset` function resets the terminal's current cell attributes to default values.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal to be reset.
- **Control Flow**:
    - The function first retrieves the current cell attributes from the `tty` structure.
    - It checks if the current cell attributes are different from the default cell attributes.
    - If the current cell has a link attribute, it sends a command to highlight the link.
    - If the current cell uses a character set and the terminal requires ACS (Alternate Character Set), it sends a command to reset the ACS.
    - It sends a command to reset all graphics attributes to default.
    - The current cell attributes are then copied from the default cell attributes.
    - Finally, it updates the last cell attributes to the default cell attributes.
- **Output**:
    - The function does not return a value; it modifies the state of the terminal by resetting its attributes.
- **Functions called**:
    - [`tty_putcode_ss`](#tty_putcode_ss)
    - [`tty_putcode`](#tty_putcode)


---
### tty_invalidate<!-- {{#callable:tty_invalidate}} -->
The `tty_invalidate` function resets the terminal state and cursor position for a given `tty` structure.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal state to be invalidated.
- **Control Flow**:
    - Copies the default cell attributes from `grid_default_cell` to the `tty`'s current cell and last cell.
    - Sets the cursor position (`cx`, `cy`) and the region boundaries (`rupper`, `rleft`, `rlower`, `rright`) to `UINT_MAX`.
    - Checks if the `tty` has been started by evaluating the `TTY_STARTED` flag.
    - If started, it checks for margin usage and sends the appropriate escape codes to enable margins and reset graphics.
    - Updates the mode of the `tty` to `ALL_MODES` and updates the cursor mode.
    - Moves the cursor to the top-left corner of the terminal (0, 0) and disables any active regions and margins.
- **Output**:
    - The function does not return a value; it modifies the state of the `tty` structure directly.
- **Functions called**:
    - [`tty_putcode`](#tty_putcode)
    - [`tty_update_mode`](#tty_update_mode)
    - [`tty_cursor`](#tty_cursor)
    - [`tty_region_off`](#tty_region_off)
    - [`tty_margin_off`](#tty_margin_off)


---
### tty_region_off<!-- {{#callable:tty_region_off}} -->
The `tty_region_off` function sets the terminal's region to cover the entire height of the terminal.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal context.
- **Control Flow**:
    - Calls the [`tty_region`](#tty_region) function with parameters 0 and `tty->sy - 1`.
    - The parameters define the upper and lower bounds of the region to be set.
- **Output**:
    - This function does not return a value; it modifies the terminal's region settings directly.
- **Functions called**:
    - [`tty_region`](#tty_region)


---
### tty_region_pane<!-- {{#callable:tty_region_pane}} -->
The `tty_region_pane` function sets the scrolling region for a specific pane in a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing context information for the terminal display.
    - `rupper`: An unsigned integer representing the upper boundary of the scrolling region.
    - `rlower`: An unsigned integer representing the lower boundary of the scrolling region.
- **Control Flow**:
    - The function calls [`tty_region`](#tty_region), passing the calculated upper and lower boundaries of the scrolling region based on the context and the pane's offsets.
    - The upper boundary is calculated as `ctx->yoff + rupper - ctx->woy`.
    - The lower boundary is calculated as `ctx->yoff + rlower - ctx->woy`.
- **Output**:
    - The function does not return a value; it modifies the terminal's scrolling region settings based on the provided parameters.
- **Functions called**:
    - [`tty_region`](#tty_region)


---
### tty_region<!-- {{#callable:tty_region}} -->
The `tty_region` function sets the scrolling region for a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `rupper`: An unsigned integer representing the upper boundary of the scrolling region.
    - `rlower`: An unsigned integer representing the lower boundary of the scrolling region.
- **Control Flow**:
    - The function first checks if the new region boundaries are the same as the current ones; if so, it returns immediately.
    - It then checks if the terminal supports the `CS` capability (cursor scrolling) using `tty_term_has`.
    - If the terminal does not support scrolling regions, it returns without making changes.
    - The function updates the `rupper` and `rlower` fields of the `tty` structure to the new values.
    - If the current cursor position is beyond the last column, it moves the cursor to the start of the line or retains its current row.
    - Finally, it sends the new scrolling region settings to the terminal using [`tty_putcode_ii`](#tty_putcode_ii).
- **Output**:
    - The function does not return a value; it modifies the terminal's scrolling region and cursor position as needed.
- **Functions called**:
    - [`tty_cursor`](#tty_cursor)
    - [`tty_putcode_ii`](#tty_putcode_ii)


---
### tty_margin_off<!-- {{#callable:tty_margin_off}} -->
The `tty_margin_off` function disables the margin feature for a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal interface.
- **Control Flow**:
    - Calls the [`tty_margin`](#tty_margin) function with parameters 0 and `tty->sx - 1` to disable the margin.
    - The [`tty_margin`](#tty_margin) function adjusts the left and right margins of the terminal to the specified values.
- **Output**:
    - The function does not return a value; it modifies the state of the terminal's margins.
- **Functions called**:
    - [`tty_margin`](#tty_margin)


---
### tty_margin_pane<!-- {{#callable:tty_margin_pane}} -->
The `tty_margin_pane` function sets the margin for a terminal pane based on the provided context.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal.
    - `ctx`: A pointer to a `struct tty_ctx` containing the context information for the terminal pane.
- **Control Flow**:
    - Calls the [`tty_margin`](#tty_margin) function with calculated left and right margin values based on the `ctx` structure.
    - The left margin is calculated as `ctx->xoff - ctx->wox`.
    - The right margin is calculated as `ctx->xoff + ctx->sx - 1 - ctx->wox`.
- **Output**:
    - The function does not return a value; it modifies the terminal's margin settings based on the provided context.
- **Functions called**:
    - [`tty_margin`](#tty_margin)


---
### tty_margin<!-- {{#callable:tty_margin}} -->
The `tty_margin` function sets the left and right margins for a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal.
    - `rleft`: An unsigned integer representing the left margin position.
    - `rright`: An unsigned integer representing the right margin position.
- **Control Flow**:
    - Checks if margins are supported for the terminal using `tty_use_margin(tty)`.
    - If margins are not supported, the function returns immediately.
    - If the current left and right margins are the same as the provided values, the function returns.
    - Calls [`tty_putcode_ii`](#tty_putcode_ii) to set the cursor scroll region based on the current upper and lower margins.
    - Updates the `rleft` and `rright` fields of the `tty` structure with the new margin values.
    - If the left margin is 0 and the right margin is equal to the terminal width, it calls [`tty_putcode`](#tty_putcode) to clear the margins.
    - Otherwise, it calls [`tty_putcode_ii`](#tty_putcode_ii) to set the new left and right margins.
    - Finally, it resets the cursor position by setting `cx` and `cy` to `UINT_MAX`.
- **Output**:
    - The function does not return a value but modifies the state of the `tty` structure to reflect the new margin settings.
- **Functions called**:
    - [`tty_putcode_ii`](#tty_putcode_ii)
    - [`tty_putcode`](#tty_putcode)


---
### tty_cursor_pane_unless_wrap<!-- {{#callable:tty_cursor_pane_unless_wrap}} -->
Moves the cursor within a pane unless it would wrap.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal.
    - `ctx`: A pointer to a `tty_ctx` structure containing context information for the terminal.
    - `cx`: The desired x-coordinate (column) for the cursor.
    - `cy`: The desired y-coordinate (row) for the cursor.
- **Control Flow**:
    - Checks if the cursor should wrap based on the `ctx->wrapped` flag.
    - Verifies if the terminal is in full width mode using `tty_full_width(tty, ctx)`.
    - Checks if the terminal has the `TERM_NOAM` flag set, which indicates automatic margins are disabled.
    - Validates the cursor's intended position against the current cursor position and terminal dimensions.
    - If all conditions are met, calls [`tty_cursor_pane`](#tty_cursor_pane) to move the cursor.
    - If wrapping would occur, logs a debug message indicating the cursor will wrap.
- **Output**:
    - The function does not return a value; it either moves the cursor or logs a message.
- **Functions called**:
    - [`tty_cursor_pane`](#tty_cursor_pane)


---
### tty_cursor_pane<!-- {{#callable:tty_cursor_pane}} -->
Moves the cursor to a specified position within a terminal pane.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal.
    - `ctx`: A pointer to a `tty_ctx` structure containing context information for the terminal.
    - `cx`: An unsigned integer representing the x-coordinate (column) to move the cursor to.
    - `cy`: An unsigned integer representing the y-coordinate (row) to move the cursor to.
- **Control Flow**:
    - The function calls [`tty_cursor`](#tty_cursor) with adjusted coordinates based on the context offsets.
    - The x-coordinate is calculated as `ctx->xoff + cx - ctx->wox`.
    - The y-coordinate is calculated as `ctx->yoff + cy - ctx->woy`.
- **Output**:
    - The function does not return a value; it directly modifies the cursor position in the terminal.
- **Functions called**:
    - [`tty_cursor`](#tty_cursor)


---
### tty_cursor<!-- {{#callable:tty_cursor}} -->
The `tty_cursor` function moves the cursor to a specified position in a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal context.
    - `cx`: An unsigned integer representing the target x-coordinate (column) for the cursor.
    - `cy`: An unsigned integer representing the target y-coordinate (row) for the cursor.
- **Control Flow**:
    - Checks if the terminal is in a blocked state and returns early if so.
    - Stores the current cursor position in `thisx` and `thisy`.
    - If the target position is the same as the current position, it returns early.
    - If the target x-coordinate exceeds the terminal width, it adjusts it to the maximum width.
    - Handles special cases for moving to the home position or the next line.
    - Determines if the cursor movement is horizontal or vertical and executes the appropriate movement commands.
    - If the movement is larger than a single step, it uses absolute positioning commands.
- **Output**:
    - The function does not return a value but updates the cursor position in the terminal based on the specified coordinates.
- **Functions called**:
    - [`tty_putcode`](#tty_putcode)
    - [`tty_putc`](#tty_putc)
    - [`tty_putcode_i`](#tty_putcode_i)
    - [`tty_putcode_ii`](#tty_putcode_ii)


---
### tty_hyperlink<!-- {{#callable:tty_hyperlink}} -->
The `tty_hyperlink` function updates the hyperlink information for a terminal cell.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `gc`: A pointer to a `struct grid_cell` that contains the cell's attributes including the hyperlink.
    - `hl`: A pointer to a `struct hyperlinks` that holds the hyperlink data.
- **Control Flow**:
    - Check if the hyperlink in the current cell (`gc->link`) is the same as the one stored in the terminal (`tty->cell.link`); if so, return immediately.
    - Update the terminal's current cell link to the new link from `gc`.
    - If the `hl` (hyperlinks) structure is NULL, return as there is no hyperlink data to process.
    - Check if the link is zero or if the hyperlink data cannot be retrieved using `hyperlinks_get`; if so, call [`tty_putcode_ss`](#tty_putcode_ss) to clear the hyperlink.
    - If the hyperlink data is valid, call [`tty_putcode_ss`](#tty_putcode_ss) to set the hyperlink with the retrieved `id` and `uri`.
- **Output**:
    - The function does not return a value; it modifies the terminal's state by updating the hyperlink attributes for the current cell.
- **Functions called**:
    - [`tty_putcode_ss`](#tty_putcode_ss)


---
### tty_attributes<!-- {{#callable:tty_attributes}} -->
The `tty_attributes` function updates the terminal's display attributes based on the provided grid cell attributes.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `gc`: A pointer to a `struct grid_cell` containing the attributes to be applied.
    - `defaults`: A pointer to a `struct grid_cell` containing default attributes for fallback.
    - `palette`: A pointer to a `struct colour_palette` used for color mapping.
    - `hl`: A pointer to a `struct hyperlinks` for managing hyperlink attributes.
- **Control Flow**:
    - The function begins by copying the attributes from the input `gc` to a local variable `gc2`.
    - It checks if the `GRID_FLAG_NOPALETTE` flag is not set and updates the foreground and background colors if they are set to default values.
    - If the new attributes are the same as the last attributes applied, the function returns early to avoid unnecessary updates.
    - It checks for terminal capabilities and adjusts the attributes accordingly, particularly for reverse colors.
    - The function then verifies and updates the foreground, background, and underline attributes using helper functions.
    - If any attributes are cleared or the underline color is set to default, it resets the terminal attributes.
    - The new colors are applied, and any changes in attributes are processed to send the appropriate escape codes to the terminal.
    - Finally, it updates the last cell attributes stored in the `tty` structure.
- **Output**:
    - The function does not return a value but updates the terminal's display attributes based on the provided grid cell attributes.
- **Functions called**:
    - [`tty_check_fg`](#tty_check_fg)
    - [`tty_check_bg`](#tty_check_bg)
    - [`tty_check_us`](#tty_check_us)
    - [`tty_reset`](#tty_reset)
    - [`tty_colours`](#tty_colours)
    - [`tty_putcode`](#tty_putcode)
    - [`tty_set_italics`](#tty_set_italics)
    - [`tty_putcode_i`](#tty_putcode_i)
    - [`tty_hyperlink`](#tty_hyperlink)


---
### tty_colours<!-- {{#callable:tty_colours}} -->
The `tty_colours` function updates the foreground, background, and underscore colors of a terminal's current cell based on the provided grid cell attributes.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `gc`: A pointer to a `struct grid_cell` containing the desired color attributes (foreground, background, and underline) to be set.
- **Control Flow**:
    - Checks if the new colors are the same as the current colors; if so, it returns early without making changes.
    - If either the foreground or background color is set to default, it handles this case specially, potentially resetting both colors to default.
    - If the terminal does not support the AX feature, it resets the terminal colors to default.
    - If the foreground color is not default and different from the current foreground, it calls [`tty_colours_fg`](#tty_colours_fg) to set the new foreground color.
    - If the background color is not default and different from the current background, it calls [`tty_colours_bg`](#tty_colours_bg) to set the new background color.
    - If the underline color is different from the current underline color, it calls [`tty_colours_us`](#tty_colours_us) to set the new underline color.
- **Output**:
    - The function does not return a value; instead, it modifies the terminal's current cell attributes directly based on the provided grid cell attributes.
- **Functions called**:
    - [`tty_reset`](#tty_reset)
    - [`tty_puts`](#tty_puts)
    - [`tty_colours_fg`](#tty_colours_fg)
    - [`tty_colours_bg`](#tty_colours_bg)
    - [`tty_colours_us`](#tty_colours_us)


---
### tty_check_fg<!-- {{#callable:tty_check_fg}} -->
The `tty_check_fg` function adjusts the foreground color of a grid cell based on terminal capabilities and color palettes.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `palette`: A pointer to a `struct colour_palette` used for color substitution.
    - `gc`: A pointer to a `struct grid_cell` representing the cell whose foreground color is to be checked and potentially modified.
- **Control Flow**:
    - Checks if the grid cell has a palette flag set; if so, it adjusts the foreground color based on the palette and bright attribute.
    - Determines if the foreground color is a 24-bit RGB color and translates it to a 256-color palette if necessary.
    - Checks the number of colors supported by the terminal and adjusts the foreground color if it is a 256-color value.
    - Handles special cases for aixterm colors and ensures that the final foreground color is valid for the terminal.
- **Output**:
    - The function modifies the foreground color of the provided grid cell (`gc`) based on the terminal's capabilities and the specified color palette.


---
### tty_check_bg<!-- {{#callable:tty_check_bg}} -->
The `tty_check_bg` function adjusts the background color of a grid cell based on the terminal's capabilities and color palette.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `palette`: A pointer to a `struct colour_palette` used for color substitutions.
    - `gc`: A pointer to a `struct grid_cell` representing the grid cell whose background color is to be checked and potentially modified.
- **Control Flow**:
    - Checks if the grid cell has a palette flag set; if so, it attempts to substitute the background color using the provided palette.
    - Determines if the background color is a 24-bit RGB color; if the terminal does not support RGB colors, it translates the color to a 256-color palette.
    - Checks how many colors the terminal supports (256 or fewer) and adjusts the background color accordingly if it is a 256-color value.
    - If the background color is an aixterm color and the terminal supports fewer than 16 colors, it adjusts the color to fit within the supported range.
- **Output**:
    - The function modifies the `gc->bg` field of the `grid_cell` structure to reflect the appropriate background color based on the terminal's capabilities and the color palette.


---
### tty_check_us<!-- {{#callable:tty_check_us}} -->
The `tty_check_us` function updates the underline color in a `grid_cell` based on the terminal's capabilities and a provided color palette.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `palette`: A pointer to a `struct colour_palette` that contains color mappings.
    - `gc`: A pointer to a `struct grid_cell` that holds the current cell's attributes including underline color.
- **Control Flow**:
    - Checks if the `GRID_FLAG_NOPALETTE` flag is not set in `gc->flags` to determine if color substitution is allowed.
    - If substitution is allowed, it attempts to get the color from the palette using `colour_palette_get` and updates `gc->us` if a valid color is found.
    - Checks if the terminal supports setting underline colors with `TTYC_SETULC1`.
    - If the terminal does not support it, it attempts to convert the underline color to RGB using `colour_force_rgb` and updates `gc->us` accordingly.
- **Output**:
    - The function does not return a value; it modifies the `gc->us` field directly to reflect the updated underline color based on the terminal's capabilities and the provided palette.


---
### tty_colours_fg<!-- {{#callable:tty_colours_fg}} -->
Sets the foreground color for a terminal based on the provided grid cell.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `gc`: A pointer to a `struct grid_cell` containing the color information to be set.
- **Control Flow**:
    - Checks if the current foreground color is a bright aixterm color and the new color is not, and resets if necessary.
    - Determines if the new color is a 24-bit or 256-color and attempts to set it using [`tty_try_colour`](#tty_try_colour).
    - If the color is an aixterm bright color, it sets the color using the appropriate escape sequence.
    - If the color is neither bright nor 24-bit/256-color, it sets the foreground color using a standard escape sequence.
    - Saves the new foreground color in the terminal's current cell.
- **Output**:
    - The function does not return a value; it modifies the terminal's state by changing the foreground color and updates the terminal's current cell with the new color.
- **Functions called**:
    - [`tty_reset`](#tty_reset)
    - [`tty_try_colour`](#tty_try_colour)
    - [`tty_puts`](#tty_puts)
    - [`tty_putcode_i`](#tty_putcode_i)


---
### tty_colours_bg<!-- {{#callable:tty_colours_bg}} -->
The `tty_colours_bg` function sets the background color of a terminal cell based on the provided grid cell attributes.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `gc`: A pointer to a `struct grid_cell` containing the color attributes, specifically the background color.
- **Control Flow**:
    - Checks if the background color is a 24-bit or 256-color value using bitwise operations.
    - If it is a 24-bit or 256-color, attempts to set the color using [`tty_try_colour`](#tty_try_colour) and saves the new value if successful.
    - If the background color is within the range of aixterm bright colors (90-97), it sets the color accordingly based on terminal capabilities.
    - If the background color is a standard color, it sets the background color directly using [`tty_putcode_i`](#tty_putcode_i).
    - Finally, it saves the new background color in the terminal's current cell.
- **Output**:
    - The function does not return a value but updates the terminal's background color and the current cell's background color attribute.
- **Functions called**:
    - [`tty_try_colour`](#tty_try_colour)
    - [`tty_puts`](#tty_puts)
    - [`tty_putcode_i`](#tty_putcode_i)


---
### tty_colours_us<!-- {{#callable:tty_colours_us}} -->
The `tty_colours_us` function sets the underline color for a terminal's text display.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `gc`: A pointer to a `struct grid_cell` containing color information, specifically the underline color.
- **Control Flow**:
    - The function first checks if the underline color is set to default; if so, it sends a command to clear the underline color.
    - If the underline color is not an RGB color, it adjusts the color value and sends a command to set the underline color using a single argument.
    - If the underline color is an RGB color, it splits the RGB values and calculates a single color value, then sends the appropriate command to set the underline color.
    - Finally, it saves the new underline color value in the terminal's current cell.
- **Output**:
    - The function does not return a value but modifies the terminal's display by setting the underline color and updates the terminal's current cell with the new underline color.
- **Functions called**:
    - [`tty_putcode`](#tty_putcode)
    - [`tty_putcode_i`](#tty_putcode_i)


---
### tty_try_colour<!-- {{#callable:tty_try_colour}} -->
Attempts to set the terminal's foreground or background color based on the provided color value and type.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `colour`: An integer representing the color value, which can indicate RGB or 256-color.
    - `type`: A string indicating the type of color setting, either foreground ('3') or background.
- **Control Flow**:
    - Checks if the `colour` has the `COLOUR_FLAG_256` flag set.
    - If true, it checks if the `type` indicates foreground and if the terminal supports setting foreground color, then sends the appropriate escape code.
    - If the `type` indicates background and the terminal supports setting background color, it sends the appropriate escape code.
    - If the `colour` has the `COLOUR_FLAG_RGB` flag set, it splits the RGB value into its components and checks if the terminal supports RGB foreground or background colors, sending the appropriate escape codes.
    - If neither condition is met, it returns -1 indicating failure.
- **Output**:
    - Returns 0 on success if the color was set, or -1 if the color could not be set.
- **Functions called**:
    - [`tty_putcode_i`](#tty_putcode_i)
    - [`tty_putcode_iii`](#tty_putcode_iii)


---
### tty_window_default_style<!-- {{#callable:tty_window_default_style}} -->
Sets the default style for a terminal window pane.
- **Inputs**:
    - `gc`: A pointer to a `struct grid_cell` that will be modified to represent the default style.
    - `wp`: A pointer to a `struct window_pane` that contains the palette information used to set foreground and background colors.
- **Control Flow**:
    - The function begins by copying the contents of `grid_default_cell` into the `gc` structure, which initializes it with default values.
    - Next, it sets the foreground color of `gc` to the foreground color defined in the `palette` of the `window_pane` (wp).
    - Finally, it sets the background color of `gc` to the background color defined in the `palette` of the `window_pane` (wp).
- **Output**:
    - The function does not return a value; instead, it modifies the `gc` structure to reflect the default style, including the foreground and background colors based on the specified window pane's palette.


---
### tty_default_colours<!-- {{#callable:tty_default_colours}} -->
Sets the default colors for a terminal window pane.
- **Inputs**:
    - `gc`: A pointer to a `struct grid_cell` that will be modified to hold the default colors.
    - `wp`: A pointer to a `struct window_pane` that contains options and state information for the pane.
- **Control Flow**:
    - Copies the default grid cell colors into the provided `gc` structure.
    - Checks if the pane's style has changed by examining the `PANE_STYLECHANGED` flag.
    - If the style has changed, it logs the change, clears the flag, and updates the active and cached styles using format trees.
    - If the foreground color of `gc` is set to 8 (default), it updates it based on the active or cached styles depending on whether the pane is active.
    - Similarly, if the background color of `gc` is set to 8, it updates it based on the active or cached styles.
- **Output**:
    - The function modifies the `gc` structure in place to reflect the default colors for the terminal pane, based on the pane's current state and style.
- **Functions called**:
    - [`tty_window_default_style`](#tty_window_default_style)


---
### tty_default_attributes<!-- {{#callable:tty_default_attributes}} -->
Sets the default attributes for a terminal's `tty` based on provided defaults and background color.
- **Inputs**:
    - `tty`: A pointer to the `tty` structure representing the terminal.
    - `defaults`: A pointer to a `grid_cell` structure containing default attributes.
    - `palette`: A pointer to a `colour_palette` structure for color management.
    - `bg`: An unsigned integer representing the background color.
    - `hl`: A pointer to a `hyperlinks` structure for managing hyperlink attributes.
- **Control Flow**:
    - A `grid_cell` structure `gc` is created and initialized by copying the default grid cell attributes.
    - The background color of `gc` is set to the provided `bg` value.
    - The [`tty_attributes`](#tty_attributes) function is called with the `tty`, `gc`, `defaults`, `palette`, and `hl` to apply the attributes.
- **Output**:
    - The function does not return a value; it modifies the terminal's attributes based on the provided parameters.
- **Functions called**:
    - [`tty_attributes`](#tty_attributes)


---
### tty_clipboard_query_callback<!-- {{#callable:tty_clipboard_query_callback}} -->
The `tty_clipboard_query_callback` function handles the callback for clipboard query events in a terminal context.
- **Inputs**:
    - `fd`: An unused file descriptor, typically for the terminal.
    - `events`: An unused short representing event types.
    - `data`: A pointer to a `tty` structure containing terminal state information.
- **Control Flow**:
    - The function retrieves the `tty` structure from the `data` pointer.
    - It accesses the associated `client` structure from the `tty`.
    - The `CLIENT_CLIPBOARDBUFFER` flag is cleared from the client's flags.
    - The clipboard panes are freed and set to NULL.
    - The number of clipboard panes is reset to zero.
    - The `TTY_OSC52QUERY` flag is cleared from the `tty` structure.
- **Output**:
    - The function does not return a value; it modifies the state of the `client` and `tty` structures to reflect the completion of a clipboard query.


---
### tty_clipboard_query<!-- {{#callable:tty_clipboard_query}} -->
The `tty_clipboard_query` function initiates a clipboard query for a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
- **Control Flow**:
    - Checks if the terminal has started and if a clipboard query is already in progress.
    - If the checks pass, it sends a clipboard query command to the terminal.
    - Sets a flag indicating that a clipboard query is in progress.
    - Configures a timer to handle the response for the clipboard query.
- **Output**:
    - The function does not return a value but modifies the state of the `tty` structure and sets up a timer for handling the clipboard query response.
- **Functions called**:
    - [`tty_putcode_ss`](#tty_putcode_ss)


# Function Declarations (Public API)

---
### tty_set_italics<!-- {{#callable_declaration:tty_set_italics}} -->
Sets the terminal to use italics or standout mode.
- **Description**: This function configures the terminal to display text in italics if supported by the terminal and the current terminal type is not 'screen' or prefixed with 'screen-'. If italics are not supported or the terminal type is 'screen', it falls back to using standout mode. This function should be called when you want to apply text styling in a terminal session, ensuring that the terminal supports the desired styling mode.
- **Inputs**:
    - `tty`: A pointer to a 'struct tty' representing the terminal state. Must not be null. The function assumes that the terminal has been properly initialized and is in a state where it can accept styling commands.
- **Output**: None
- **See also**: [`tty_set_italics`](#tty_set_italics)  (Implementation)


---
### tty_try_colour<!-- {{#callable_declaration:tty_try_colour}} -->
Attempts to set the terminal color based on the given color code and type.
- **Description**: This function is used to set the color of a terminal based on a specified color code and type. It supports both 256-color and RGB color modes, depending on the capabilities of the terminal. The function should be called when there is a need to change the terminal's foreground or background color. It returns 0 if the color was successfully set, or -1 if the color could not be set due to unsupported terminal capabilities. The function expects the terminal to be initialized and capable of handling the specified color mode.
- **Inputs**:
    - `tty`: A pointer to a 'struct tty' representing the terminal. Must not be null and should be properly initialized before calling this function.
    - `colour`: An integer representing the color code. It can be a 256-color code or an RGB color code, indicated by the COLOUR_FLAG_256 or COLOUR_FLAG_RGB flags, respectively.
    - `type`: A string representing the type of color to set, where '3' indicates a foreground color and any other value indicates a background color. Must not be null.
- **Output**: Returns 0 if the color was successfully set, or -1 if the terminal does not support the specified color mode.
- **See also**: [`tty_try_colour`](#tty_try_colour)  (Implementation)


---
### tty_force_cursor_colour<!-- {{#callable_declaration:tty_force_cursor_colour}} -->
Sets the cursor color for a terminal.
- **Description**: This function is used to set the cursor color of a terminal to a specified color. It should be called whenever there is a need to change the cursor color, ensuring that the new color is different from the current one to avoid unnecessary operations. If the color is set to -1, the cursor color is reset to the default. This function must be used with a valid `tty` structure, and it assumes that the terminal supports RGB color settings.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal. Must not be null, and the terminal should be properly initialized before calling this function.
    - `c`: An integer representing the desired color. It can be a valid RGB color value or -1 to reset to the default color. If the color is invalid or unchanged, the function will not perform any operation.
- **Output**: None
- **See also**: [`tty_force_cursor_colour`](#tty_force_cursor_colour)  (Implementation)


---
### tty_cursor_pane<!-- {{#callable_declaration:tty_cursor_pane}} -->
Adjusts the cursor position within a pane based on offsets.
- **Description**: This function is used to move the cursor to a specific position within a pane, taking into account the pane's offset within the terminal. It should be called when you need to position the cursor relative to a pane's coordinates, rather than the terminal's absolute coordinates. This function assumes that the pane's offsets and the desired cursor position are correctly calculated and provided. It does not return any value or indicate success or failure, so it is important to ensure that the inputs are valid and within expected ranges.
- **Inputs**:
    - `tty`: A pointer to a 'struct tty' representing the terminal interface. Must not be null.
    - `ctx`: A pointer to a 'struct tty_ctx' containing context information about the pane, including offsets. Must not be null.
    - `cx`: An unsigned integer representing the x-coordinate within the pane. Should be within the pane's width.
    - `cy`: An unsigned integer representing the y-coordinate within the pane. Should be within the pane's height.
- **Output**: None
- **See also**: [`tty_cursor_pane`](#tty_cursor_pane)  (Implementation)


---
### tty_cursor_pane_unless_wrap<!-- {{#callable_declaration:tty_cursor_pane_unless_wrap}} -->
Moves the cursor within a terminal pane unless it would cause a line wrap.
- **Description**: This function is used to position the cursor within a terminal pane, but it avoids moving the cursor if doing so would result in a line wrap. It is typically used in terminal emulation contexts where precise cursor positioning is required, and line wrapping needs to be controlled. The function should be called with valid cursor coordinates and a context that describes the current state of the terminal pane. It is important to ensure that the terminal and context are properly initialized before calling this function.
- **Inputs**:
    - `tty`: A pointer to a 'struct tty' representing the terminal state. Must not be null.
    - `ctx`: A pointer to a 'struct tty_ctx' containing the context of the terminal pane. Must not be null.
    - `cx`: An unsigned integer representing the x-coordinate for the cursor position. Should be within the valid range of the pane's width.
    - `cy`: An unsigned integer representing the y-coordinate for the cursor position. Should be within the valid range of the pane's height.
- **Output**: None
- **See also**: [`tty_cursor_pane_unless_wrap`](#tty_cursor_pane_unless_wrap)  (Implementation)


---
### tty_colours<!-- {{#callable_declaration:tty_colours}} -->
Set the terminal's foreground, background, and underscore colors based on the provided grid cell.
- **Description**: This function updates the terminal's color settings to match those specified in the provided grid cell structure. It should be used when the terminal's color attributes need to be changed, such as when rendering text with different colors. The function checks if the new colors are different from the current settings and applies changes only if necessary. It handles default colors specially, potentially resetting both colors to default if required. This function must be called with a valid tty structure and a grid cell containing the desired color settings.
- **Inputs**:
    - `tty`: A pointer to a struct tty, representing the terminal to be updated. Must not be null.
    - `gc`: A pointer to a const struct grid_cell, containing the desired foreground, background, and underscore colors. Must not be null.
- **Output**: None
- **See also**: [`tty_colours`](#tty_colours)  (Implementation)


---
### tty_check_fg<!-- {{#callable_declaration:tty_check_fg}} -->
Adjusts the foreground color of a grid cell based on terminal capabilities and color palette.
- **Description**: This function is used to adjust the foreground color of a grid cell to match the capabilities of the terminal and the specified color palette. It should be called when rendering text to ensure that the correct color is displayed, taking into account the terminal's color support and any palette substitutions. This function handles various color formats, including 24-bit RGB and 256-color modes, and makes adjustments for terminals with limited color support.
- **Inputs**:
    - `tty`: A pointer to a 'struct tty' representing the terminal. Must not be null.
    - `palette`: A pointer to a 'struct colour_palette' used for color substitutions. Must not be null.
    - `gc`: A pointer to a 'struct grid_cell' whose foreground color will be adjusted. Must not be null.
- **Output**: None
- **See also**: [`tty_check_fg`](#tty_check_fg)  (Implementation)


---
### tty_check_bg<!-- {{#callable_declaration:tty_check_bg}} -->
Adjusts the background color of a grid cell for compatibility with the terminal's color capabilities.
- **Description**: This function modifies the background color of a grid cell to ensure it is compatible with the terminal's color capabilities. It should be used when rendering grid cells to a terminal, particularly when the terminal may not support 24-bit or 256-color modes. The function checks if the background color is a 24-bit or 256-color value and translates it to a compatible color if necessary. It also applies any color substitutions from a provided color palette. This function must be called with a valid `tty`, `palette`, and `gc` structure.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal. Must not be null. The function uses this to determine the terminal's color capabilities.
    - `palette`: A pointer to a `struct colour_palette` used for color substitution. Must not be null. The function uses this to apply any palette-based color substitutions.
    - `gc`: A pointer to a `struct grid_cell` representing the grid cell whose background color is to be adjusted. Must not be null. The function modifies the `bg` field of this structure.
- **Output**: None
- **See also**: [`tty_check_bg`](#tty_check_bg)  (Implementation)


---
### tty_check_us<!-- {{#callable_declaration:tty_check_us}} -->
Adjusts the underscore color in a grid cell based on a color palette and terminal capabilities.
- **Description**: This function is used to adjust the underscore color of a grid cell, potentially substituting it with a color from a provided palette if the grid cell does not have the GRID_FLAG_NOPALETTE flag set. It also ensures compatibility with terminals that do not support direct underscore color setting by converting the color to RGB if necessary. This function should be called when preparing a grid cell for display on a terminal, ensuring that the underscore color is correctly set according to the terminal's capabilities and any active color palette.
- **Inputs**:
    - `tty`: A pointer to a tty structure representing the terminal. This parameter is unused in this function.
    - `palette`: A pointer to a colour_palette structure. This palette is used to substitute the underscore color if the grid cell does not have the GRID_FLAG_NOPALETTE flag set. Must not be null.
    - `gc`: A pointer to a grid_cell structure representing the cell whose underscore color is to be adjusted. The function modifies the us field of this structure based on the palette and terminal capabilities. Must not be null.
- **Output**: None
- **See also**: [`tty_check_us`](#tty_check_us)  (Implementation)


---
### tty_colours_fg<!-- {{#callable_declaration:tty_colours_fg}} -->
Sets the foreground color of a terminal cell.
- **Description**: This function is used to set the foreground color of a terminal cell based on the specified grid cell's foreground color. It handles different color formats, including 24-bit RGB and 256-color modes, and adjusts the terminal's current cell color accordingly. It should be called when updating the terminal display to ensure the correct foreground color is applied. The function also manages terminal-specific quirks, such as resetting bright colors when necessary.
- **Inputs**:
    - `tty`: A pointer to a 'struct tty' representing the terminal state. Must not be null. The function updates the terminal's current cell color based on this state.
    - `gc`: A pointer to a 'const struct grid_cell' containing the desired foreground color. Must not be null. The function uses this to determine the new foreground color for the terminal cell.
- **Output**: None
- **See also**: [`tty_colours_fg`](#tty_colours_fg)  (Implementation)


---
### tty_colours_bg<!-- {{#callable_declaration:tty_colours_bg}} -->
Sets the background color of a terminal cell.
- **Description**: This function is used to set the background color of a terminal cell based on the specified grid cell's background color. It handles different color formats, including 24-bit RGB and 256-color modes, and adjusts the terminal's current cell background color accordingly. This function should be called when updating the terminal display to reflect changes in background color. It assumes that the terminal supports the necessary color codes and that the grid cell's background color has been validated prior to calling this function.
- **Inputs**:
    - `tty`: A pointer to a 'struct tty' representing the terminal interface. Must not be null. The function updates the terminal's current cell background color.
    - `gc`: A pointer to a 'struct grid_cell' containing the background color information. Must not be null. The function uses the background color from this structure to set the terminal cell's background color.
- **Output**: None
- **See also**: [`tty_colours_bg`](#tty_colours_bg)  (Implementation)


---
### tty_colours_us<!-- {{#callable_declaration:tty_colours_us}} -->
Sets the underline color for a terminal cell.
- **Description**: This function is used to set the underline color of a terminal cell based on the provided grid cell's attributes. It should be called when the underline color needs to be updated, typically during rendering operations. The function handles both RGB and non-RGB color formats, and it will clear the underline color if the default is specified. It is important to ensure that the `tty` and `gc` parameters are valid and properly initialized before calling this function.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal state. Must not be null and should be properly initialized before use.
    - `gc`: A pointer to a `struct grid_cell` containing the color attributes for the underline. Must not be null and should contain valid color information.
- **Output**: None
- **See also**: [`tty_colours_us`](#tty_colours_us)  (Implementation)


---
### tty_region_pane<!-- {{#callable_declaration:tty_region_pane}} -->
Sets the scroll region for a terminal pane.
- **Description**: This function is used to define the vertical scroll region within a terminal pane, which is a subsection of the terminal screen. It should be called when you need to restrict scrolling to a specific region of the terminal pane, typically before performing operations that involve scrolling, such as inserting or deleting lines. The function requires the pane's offset and the desired upper and lower bounds of the scroll region. It is important to ensure that the terminal has been properly initialized and that the provided offsets and bounds are within the valid range of the terminal pane.
- **Inputs**:
    - `tty`: A pointer to a 'struct tty' representing the terminal interface. Must not be null.
    - `ctx`: A pointer to a 'struct tty_ctx' containing context information for the terminal pane. Must not be null.
    - `rupper`: An unsigned integer specifying the upper bound of the scroll region relative to the pane's offset. Must be within the pane's vertical range.
    - `rlower`: An unsigned integer specifying the lower bound of the scroll region relative to the pane's offset. Must be within the pane's vertical range.
- **Output**: None
- **See also**: [`tty_region_pane`](#tty_region_pane)  (Implementation)


---
### tty_region<!-- {{#callable_declaration:tty_region}} -->
Sets the scroll region of a terminal.
- **Description**: This function configures the scroll region of a terminal to the specified upper and lower bounds. It should be used when you need to define a specific area of the terminal for scrolling operations. The function checks if the terminal supports the necessary capabilities before making changes. It is important to ensure that the terminal has been properly initialized and supports the required capabilities before calling this function. The function does not perform any operations if the specified region is already set.
- **Inputs**:
    - `tty`: A pointer to a 'struct tty' representing the terminal. Must not be null and should be properly initialized.
    - `rupper`: An unsigned integer specifying the upper bound of the scroll region. Must be within the valid range of the terminal's rows.
    - `rlower`: An unsigned integer specifying the lower bound of the scroll region. Must be within the valid range of the terminal's rows.
- **Output**: None
- **See also**: [`tty_region`](#tty_region)  (Implementation)


---
### tty_margin_pane<!-- {{#callable_declaration:tty_margin_pane}} -->
Sets the margin for a pane in a terminal.
- **Description**: This function is used to set the left and right margins for a specific pane within a terminal. It should be called when you need to adjust the margins of a pane, typically in preparation for operations that require specific margin settings. The function expects valid `tty` and `ctx` structures, and it calculates the margin based on the offset and size information provided in the `ctx` structure. Ensure that the `tty` and `ctx` pointers are not null before calling this function.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal. Must not be null. The caller retains ownership.
    - `ctx`: A pointer to a `struct tty_ctx` containing context information for the pane, including offsets and size. Must not be null. The caller retains ownership.
- **Output**: None
- **See also**: [`tty_margin_pane`](#tty_margin_pane)  (Implementation)


---
### tty_margin<!-- {{#callable_declaration:tty_margin}} -->
Sets the left and right margins for a terminal.
- **Description**: This function configures the left and right margins of a terminal if the terminal supports margin settings. It should be called when you need to adjust the margins for text output on a terminal that supports DECSTBM (Set Top and Bottom Margins) and DECSLRM (Set Left and Right Margins). The function checks if the current margins are different from the desired margins and updates them accordingly. It also resets the cursor position to an undefined state. This function must be used in conjunction with a terminal that supports margin settings.
- **Inputs**:
    - `tty`: A pointer to a 'struct tty' representing the terminal. Must not be null. The function assumes the terminal supports margin settings.
    - `rleft`: An unsigned integer specifying the left margin position. Must be within the valid range of the terminal's width.
    - `rright`: An unsigned integer specifying the right margin position. Must be within the valid range of the terminal's width.
- **Output**: None
- **See also**: [`tty_margin`](#tty_margin)  (Implementation)


---
### tty_large_region<!-- {{#callable_declaration:tty_large_region}} -->
Determines if a region is large enough to warrant a single redraw.
- **Description**: This function evaluates whether a given region, defined by the context, is large enough to justify redrawing it once later rather than multiple times immediately. It is typically used to optimize rendering by deferring redraws for large regions. The function should be called with a valid `tty_ctx` structure, and it assumes that the `sy` field in the context is non-zero to avoid division by zero errors.
- **Inputs**:
    - `tty`: A pointer to a `struct tty`, which is not used in this function and can be null.
    - `ctx`: A pointer to a `struct tty_ctx` that must not be null. It contains the region's upper and lower bounds (`orupper` and `orlower`) and the screen height (`sy`). The function expects `sy` to be non-zero.
- **Output**: Returns a non-zero integer if the region is large enough to be considered for a single redraw, otherwise returns zero.
- **See also**: [`tty_large_region`](#tty_large_region)  (Implementation)


---
### tty_fake_bce<!-- {{#callable_declaration:tty_fake_bce}} -->
Determine if background color erase (BCE) needs to be emulated.
- **Description**: This function checks whether the terminal associated with the given tty supports background color erase (BCE) and whether the specified background colors are default. It is used to decide if BCE needs to be emulated when the terminal does not support it. This function should be called when rendering operations require knowledge of BCE support. It assumes that the tty and grid_cell pointers are valid and that the background color is specified as an unsigned integer.
- **Inputs**:
    - `tty`: A pointer to a tty structure representing the terminal. Must not be null. The function checks the terminal's capabilities using this parameter.
    - `gc`: A pointer to a grid_cell structure representing the cell attributes, including background color. Must not be null.
    - `bg`: An unsigned integer representing the background color. It is expected to be a valid color value.
- **Output**: Returns 1 if BCE needs to be emulated (i.e., the terminal does not support BCE or the background colors are not default), otherwise returns 0.
- **See also**: [`tty_fake_bce`](#tty_fake_bce)  (Implementation)


---
### tty_redraw_region<!-- {{#callable_declaration:tty_redraw_region}} -->
Redraws a specified region of the terminal.
- **Description**: This function is used to redraw a specific region of the terminal associated with a tty context. It should be called when a portion of the terminal needs to be updated or refreshed. If the region to be redrawn is large, the function will schedule a complete redraw using a callback function provided in the context. This function does not return any value and is intended to be used internally within the terminal handling logic.
- **Inputs**:
    - `tty`: A pointer to a struct tty representing the terminal to be redrawn. This must not be null, and the caller retains ownership.
    - `ctx`: A pointer to a struct tty_ctx containing the context for the redraw operation, including the region boundaries and a callback for large redraws. This must not be null, and the caller retains ownership.
- **Output**: None
- **See also**: [`tty_redraw_region`](#tty_redraw_region)  (Implementation)


---
### tty_emulate_repeat<!-- {{#callable_declaration:tty_emulate_repeat}} -->
Emulates a terminal command repeat operation.
- **Description**: This function is used to emulate the repetition of a terminal command code a specified number of times. It checks if the terminal supports a specific command code and uses it if available; otherwise, it falls back to an alternative command code, repeating it the specified number of times. This function is useful when you need to ensure a command is repeated on terminals with varying capabilities.
- **Inputs**:
    - `tty`: A pointer to a 'struct tty' representing the terminal interface. Must not be null.
    - `code`: An enum value of type 'tty_code_code' representing the primary command code to be used if supported by the terminal.
    - `code1`: An enum value of type 'tty_code_code' representing the fallback command code to be used if the primary code is not supported.
    - `n`: An unsigned integer specifying the number of times the command should be repeated. Must be greater than zero.
- **Output**: None
- **See also**: [`tty_emulate_repeat`](#tty_emulate_repeat)  (Implementation)


---
### tty_repeat_space<!-- {{#callable_declaration:tty_repeat_space}} -->
Outputs a specified number of space characters to a tty.
- **Description**: This function is used to output a specified number of space characters to a tty (teletypewriter) interface. It is useful when you need to insert a series of spaces in the output stream, such as for formatting purposes. The function handles large numbers of spaces by outputting them in chunks of up to 500 characters at a time. It must be called with a valid tty structure, and the number of spaces to output must be specified. The function does not return a value and does not modify the input parameters.
- **Inputs**:
    - `tty`: A pointer to a tty structure representing the teletypewriter interface. Must not be null.
    - `n`: The number of space characters to output. Must be a non-negative integer.
- **Output**: None
- **See also**: [`tty_repeat_space`](#tty_repeat_space)  (Implementation)


---
### tty_draw_pane<!-- {{#callable_declaration:tty_draw_pane}} -->
Draws a line of a pane on the terminal.
- **Description**: This function is used to render a specific line of a pane onto the terminal. It should be called when a line within a pane needs to be displayed, taking into account the current terminal and pane context. The function handles visibility checks and adjusts the drawing based on the pane's size relative to the terminal. It is important to ensure that the `tty` and `ctx` structures are properly initialized and that the line index `py` is within the bounds of the pane's height.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal where the pane line will be drawn. Must not be null.
    - `ctx`: A pointer to a `struct tty_ctx` containing the context for drawing, including pane size and offsets. Must not be null and should be properly initialized.
    - `py`: An unsigned integer representing the y-coordinate (line index) within the pane to be drawn. It should be within the valid range of the pane's height.
- **Output**: None
- **See also**: [`tty_draw_pane`](#tty_draw_pane)  (Implementation)


---
### tty_default_attributes<!-- {{#callable_declaration:tty_default_attributes}} -->
Sets the default attributes for a terminal.
- **Description**: This function is used to set the default attributes for a terminal, such as background color, using a grid cell structure. It should be called when you need to reset or initialize the terminal's attributes to a default state. This function does not return any value and does not modify the input parameters.
- **Inputs**:
    - `tty`: A pointer to a 'struct tty' which represents the terminal. Must not be null.
    - `defaults`: A pointer to a 'struct grid_cell' containing default attributes. Must not be null.
    - `palette`: A pointer to a 'struct colour_palette' used for color settings. Must not be null.
    - `bg`: An unsigned integer representing the background color. Valid range depends on the color system used.
    - `hl`: A pointer to a 'struct hyperlinks' for hyperlink settings. Must not be null.
- **Output**: None
- **See also**: [`tty_default_attributes`](#tty_default_attributes)  (Implementation)


---
### tty_check_overlay<!-- {{#callable_declaration:tty_check_overlay}} -->
Checks if a specific position on the terminal is covered by an overlay.
- **Description**: This function determines whether a given position on the terminal, specified by its x and y coordinates, is obscured by an overlay. It is useful for rendering logic where visibility of terminal content needs to be assessed. The function returns a boolean indicating the presence of an overlay at the specified position. It should be called with valid terminal coordinates and a properly initialized tty structure.
- **Inputs**:
    - `tty`: A pointer to a tty structure representing the terminal state. Must not be null and should be properly initialized before calling this function.
    - `px`: The x-coordinate on the terminal to check for overlay coverage. Should be within the valid range of terminal width.
    - `py`: The y-coordinate on the terminal to check for overlay coverage. Should be within the valid range of terminal height.
- **Output**: Returns 1 if the specified position is covered by an overlay, otherwise returns 0.
- **See also**: [`tty_check_overlay`](#tty_check_overlay)  (Implementation)


---
### tty_check_overlay_range<!-- {{#callable_declaration:tty_check_overlay_range}} -->
Determine visible parts of a range considering overlays.
- **Description**: This function is used to determine which parts of a specified range on a terminal are visible, taking into account any overlays that might obscure parts of the range. It is typically called when rendering or updating the display to ensure that only visible parts are processed. The function requires a valid `tty` structure and expects the `overlay_check` function to be set in the associated client structure to perform the visibility check. If `overlay_check` is not set, the entire range is considered visible.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal state. Must not be null.
    - `px`: An unsigned integer representing the starting x-coordinate of the range. Must be within the terminal's width.
    - `py`: An unsigned integer representing the y-coordinate of the range. Must be within the terminal's height.
    - `nx`: An unsigned integer representing the width of the range. Must be a positive value.
    - `r`: A pointer to a `struct overlay_ranges` where the function will store the visible parts of the range. Must not be null.
- **Output**: None
- **See also**: [`tty_check_overlay_range`](#tty_check_overlay_range)  (Implementation)


---
### tty_write_one<!-- {{#callable_declaration:tty_write_one}} -->
Executes a command function for a specific client and context.
- **Description**: This function is used to execute a command function for a specific client and context, provided that the context's client callback function is set and returns a positive result. It is typically used in scenarios where a command needs to be executed for a particular client, such as rendering or updating a terminal display. The function should be called with a valid client and context, and it will only execute the command if the context's client callback function is defined and returns 1.
- **Inputs**:
    - `cmdfn`: A function pointer to the command function to be executed. The function should accept a pointer to a tty structure and a pointer to a tty_ctx structure as arguments.
    - `c`: A pointer to a client structure representing the client for which the command function should be executed. Must not be null.
    - `ctx`: A pointer to a tty_ctx structure containing the context for the command function. Must not be null and must have a valid set_client_cb function.
- **Output**: None
- **See also**: [`tty_write_one`](#tty_write_one)  (Implementation)


