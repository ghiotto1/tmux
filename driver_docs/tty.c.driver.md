# Purpose
The provided C source code is part of the `tmux` terminal multiplexer, specifically handling terminal (TTY) interactions. This file is responsible for managing the terminal's display and input/output operations, including setting up terminal attributes, handling cursor movements, managing color and style attributes, and processing input/output buffers. It includes functions for initializing and configuring the terminal, handling terminal resizing, and managing terminal-specific features like margins and regions. The code also includes mechanisms for handling terminal-specific quirks and capabilities, such as color support and cursor styles, and it provides functions for drawing and updating the terminal display based on the current state of the `tmux` session.

The file is a collection of functions that interact with the terminal at a low level, using system calls and terminal control sequences to achieve its functionality. It defines several static functions for internal use, such as [`tty_set_italics`](#tty_set_italics), [`tty_try_colour`](#tty_try_colour), and [`tty_cursor_pane`](#tty_cursor_pane), which are used to manage specific aspects of terminal behavior. The code also includes macros and constants that define terminal behavior, such as `TTY_BLOCK_INTERVAL` and `TTY_QUERY_TIMEOUT`. This file is not intended to be an executable on its own but rather a component of the larger `tmux` application, providing essential terminal handling capabilities that are used throughout the application to manage terminal sessions and user interactions.
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
Creates a log file for the terminal output.
- **Inputs**:
    - None
- **Control Flow**:
    - Generates a log file name using the process ID.
    - Attempts to open the log file with write permissions, creating it if it doesn't exist.
    - If the file descriptor is valid, sets the close-on-exec flag; otherwise, calls `fatal` to handle the error.
- **Output**:
    - The function does not return a value but creates a log file for terminal output.
- **Functions called**:
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)


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
    - The function retrieves the current window size using the `ioctl` system call with the `TIOCGWINSZ` command.
    - If the window size is successfully retrieved, it checks if the number of columns (sx) or rows (sy) is zero, and assigns default values if necessary.
    - It calculates the pixel dimensions (xpixel and ypixel) based on the retrieved window size.
    - If either pixel dimension is zero and certain conditions are met, it sends escape sequences to query the window size again.
    - If the `ioctl` call fails, default values for sx, sy, xpixel, and ypixel are set.
    - The function logs the new dimensions and calls [`tty_set_size`](#tty_set_size) to update the terminal's size attributes.
    - Finally, it calls [`tty_invalidate`](#tty_invalidate) to mark the terminal for redrawing.
- **Output**:
    - The function does not return a value; it modifies the state of the terminal represented by the `tty` structure and updates its size attributes.
- **Functions called**:
    - [`tty_puts`](#tty_puts)
    - [`tty_set_size`](#tty_set_size)
    - [`tty_invalidate`](#tty_invalidate)


---
### tty_set_size<!-- {{#callable:tty_set_size}} -->
Sets the size parameters of a `tty` structure.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that holds terminal size and pixel information.
    - `sx`: An unsigned integer representing the number of columns (width) of the terminal.
    - `sy`: An unsigned integer representing the number of rows (height) of the terminal.
    - `xpixel`: An unsigned integer representing the width in pixels of the terminal.
    - `ypixel`: An unsigned integer representing the height in pixels of the terminal.
- **Control Flow**:
    - The function directly assigns the input values to the corresponding fields in the `tty` structure.
    - No conditional logic or loops are present, making the function straightforward and efficient.
- **Output**:
    - The function does not return a value; it modifies the `tty` structure in place to reflect the new size and pixel dimensions.


---
### tty_read_callback<!-- {{#callable:tty_read_callback}} -->
The `tty_read_callback` function handles reading data from a terminal input buffer.
- **Inputs**:
    - `fd`: An integer file descriptor for the terminal, which is unused in this function.
    - `events`: A short integer representing the events that triggered the callback, which is also unused.
    - `data`: A pointer to a `tty` structure that contains information about the terminal.
- **Control Flow**:
    - The function retrieves the `tty` structure from the `data` pointer.
    - It accesses the associated `client` structure and retrieves the client's name.
    - The function checks the length of the input buffer and attempts to read data from the terminal file descriptor into the buffer.
    - If no data is read (nread == 0) or an error occurs (nread == -1), it logs the appropriate message and calls [`server_client_lost`](server-client.c.driver.md#server_client_lost) to handle the lost client.
    - If data is successfully read, it logs the number of bytes read and the current buffer size.
    - Finally, it enters a loop that processes the input keys by calling [`tty_keys_next`](tty-keys.c.driver.md#tty_keys_next) until there are no more keys to process.
- **Output**:
    - The function does not return a value; it performs actions based on the read data and logs the results.
- **Functions called**:
    - [`server_client_lost`](server-client.c.driver.md#server_client_lost)
    - [`tty_keys_next`](tty-keys.c.driver.md#tty_keys_next)


---
### tty_timer_callback<!-- {{#callable:tty_timer_callback}} -->
The `tty_timer_callback` function handles timer events for a terminal, managing the state of discarded output and scheduling further timer events.
- **Inputs**:
    - `fd`: An unused file descriptor, typically for event handling.
    - `events`: An unused short representing event types.
    - `data`: A pointer to a `tty` structure containing terminal state information.
- **Control Flow**:
    - Logs the number of discarded output for the associated client.
    - Updates the client's redraw flags and total discarded count.
    - Checks if the number of discarded outputs is below a threshold; if so, it clears the block flag and invalidates the tty.
    - If the threshold is exceeded, resets the discarded count and schedules the next timer event.
- **Output**:
    - The function does not return a value; it modifies the state of the `tty` and schedules future timer events based on the current state.
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
    - Set the `TTY_BLOCK` flag to indicate that the terminal is now in a blocked state.
    - Log a debug message indicating that the terminal cannot keep up and the amount of data discarded.
    - Drain the output buffer to discard all data.
    - Increment the discarded count for the client associated with the terminal.
    - Reset the discarded count for the terminal and set a timer for blocking interval.
    - Return 1 to indicate that the terminal is now blocked.
- **Output**:
    - Returns 1 if the terminal is now in a blocked state, 0 otherwise.


---
### tty_write_callback<!-- {{#callable:tty_write_callback}} -->
The `tty_write_callback` function handles writing data from a `tty` output buffer to a client file descriptor.
- **Inputs**:
    - `fd`: An unused file descriptor, typically representing the terminal.
    - `events`: An unused event mask indicating the type of events that triggered the callback.
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
    - The function does not return a value; it modifies the state of the `tty` and client, and may trigger further events based on the output buffer's state.
- **Functions called**:
    - [`tty_block_maybe`](#tty_block_maybe)


---
### tty_open<!-- {{#callable:tty_open}} -->
The `tty_open` function initializes a terminal interface for a given client.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure that represents the terminal interface to be opened.
    - `cause`: A pointer to a string that will hold an error message if the terminal creation fails.
- **Control Flow**:
    - The function retrieves the associated `client` structure from the `tty` object.
    - It attempts to create a terminal object using [`tty_term_create`](tty-term.c.driver.md#tty_term_create) with the client's terminal name and capabilities.
    - If terminal creation fails (returns NULL), it calls [`tty_close`](#tty_close) to clean up and returns -1.
    - If successful, it sets the `TTY_OPENED` flag in the `tty` structure and clears several other flags.
    - It sets up input and output event handlers using `event_set` for reading and writing data.
    - It allocates new input and output buffers using `evbuffer_new`, and checks for memory allocation failure.
    - A timer is set up using `evtimer_set` for handling time-based events.
    - Finally, it calls [`tty_start_tty`](#tty_start_tty) to start the terminal interface and [`tty_keys_build`](tty-keys.c.driver.md#tty_keys_build) to initialize key mappings.
- **Output**:
    - The function returns 0 on success, indicating that the terminal interface has been successfully opened.
- **Functions called**:
    - [`tty_term_create`](tty-term.c.driver.md#tty_term_create)
    - [`tty_close`](#tty_close)
    - [`tty_start_tty`](#tty_start_tty)
    - [`tty_keys_build`](tty-keys.c.driver.md#tty_keys_build)


---
### tty_start_timer_callback<!-- {{#callable:tty_start_timer_callback}} -->
The `tty_start_timer_callback` function handles the timer event for a terminal, updating features and setting request flags.
- **Inputs**:
    - `fd`: An unused file descriptor, typically for event handling.
    - `events`: An unused short representing event types.
    - `data`: A pointer to the `tty` structure containing terminal information.
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
The `tty_start_tty` function initializes and configures a terminal interface for a given `tty` structure.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal interface to be initialized.
- **Control Flow**:
    - Sets the terminal file descriptor to non-blocking mode using [`setblocking`](tmux.c.driver.md#setblocking).
    - Adds an event listener for input events on the terminal using `event_add`.
    - Copies the current terminal attributes from `tty->tio` to a local `tio` variable.
    - Modifies the `tio` structure to disable certain input and output flags, and sets the minimum number of characters to read and the timeout.
    - Applies the modified terminal attributes using `tcsetattr` and flushes the output buffer with `tcflush`.
    - Sends terminal control codes to set the terminal's state, including enabling alternate screen buffer and clearing the screen.
    - Checks if alternate character set (ACS) capabilities are needed and sends the appropriate control code.
    - Sets the cursor to normal visibility and configures mouse reporting if supported by the terminal.
    - Sets up a timer for starting the terminal with a specified timeout.
    - Updates the `tty` structure to indicate that it has been started and invalidates its state.
    - Resets the cursor color if it is set, and initializes mouse drag flags.
- **Output**:
    - The function does not return a value but modifies the state of the `tty` structure to reflect the initialized terminal settings.
- **Functions called**:
    - [`setblocking`](tmux.c.driver.md#setblocking)
    - [`tty_putcode`](#tty_putcode)
    - [`tty_acs_needed`](tty-acs.c.driver.md#tty_acs_needed)
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
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
    - If the terminal type is VT100-like, it checks various flags (`TTY_HAVEDA`, `TTY_HAVEDA2`, `TTY_HAVEXDA`) and sends specific escape sequences to the terminal.
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
    - Retrieve the current time using `time(NULL)`.
    - Check if the `TTY_STARTED` flag is not set; if so, exit the function.
    - Check if the time since the last requests is less than `TTY_REQUEST_LIMIT`; if so, exit the function.
    - Update `tty->last_requests` with the current time.
    - If the terminal type supports VT100-like features, send escape sequences to query the foreground and background colors.
- **Output**:
    - The function does not return a value; it sends escape sequences to the terminal to request color information.
- **Functions called**:
    - [`tty_puts`](#tty_puts)


---
### tty_stop_tty<!-- {{#callable:tty_stop_tty}} -->
The `tty_stop_tty` function stops the terminal interface by cleaning up resources and resetting terminal settings.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal interface to be stopped.
- **Control Flow**:
    - Checks if the terminal has been started by evaluating the `TTY_STARTED` flag; if not, it returns immediately.
    - Clears the `TTY_STARTED` flag to indicate that the terminal is no longer active.
    - Removes any active timers associated with the terminal.
    - Deactivates input and output events for the terminal.
    - Attempts to retrieve the current window size using `ioctl` and stores it in a `winsize` structure; if it fails, it returns.
    - Restores terminal attributes using `tcsetattr` to reset the terminal to its previous state.
    - Sends various terminal control sequences to reset visual settings, such as cursor visibility and color modes.
    - Checks for specific terminal capabilities and sends additional control sequences as needed.
    - Sets the terminal to blocking mode at the end of the function.
- **Output**:
    - The function does not return a value; it modifies the state of the terminal and cleans up resources associated with it.
- **Functions called**:
    - [`tty_raw`](#tty_raw)
    - [`tty_term_string_ii`](tty-term.c.driver.md#tty_term_string_ii)
    - [`tty_acs_needed`](tty-acs.c.driver.md#tty_acs_needed)
    - [`tty_term_string`](tty-term.c.driver.md#tty_term_string)
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
    - [`tty_term_string_i`](tty-term.c.driver.md#tty_term_string_i)
    - [`setblocking`](tmux.c.driver.md#setblocking)


---
### tty_close<!-- {{#callable:tty_close}} -->
The `tty_close` function cleans up and closes a terminal interface represented by the `tty` structure.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal interface to be closed.
- **Control Flow**:
    - Checks if the `key_timer` event is initialized and deletes it if so.
    - Calls [`tty_stop_tty`](#tty_stop_tty) to stop the terminal interface.
    - Checks if the `TTY_OPENED` flag is set in the `tty` structure.
    - If `TTY_OPENED` is set, frees the input and output buffers, deletes the input and output events, frees the terminal structure, and frees the key bindings.
    - Clears the `TTY_OPENED` flag.
- **Output**:
    - The function does not return a value; it performs cleanup operations on the `tty` structure and releases associated resources.
- **Functions called**:
    - [`tty_stop_tty`](#tty_stop_tty)
    - [`tty_term_free`](tty-term.c.driver.md#tty_term_free)
    - [`tty_keys_free`](tty-keys.c.driver.md#tty_keys_free)


---
### tty_free<!-- {{#callable:tty_free}} -->
The `tty_free` function releases resources associated with a `tty` structure.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal interface to be freed.
- **Control Flow**:
    - Calls the [`tty_close`](#tty_close) function to close the terminal and free associated resources.
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
    - Checks if terminal features can be applied using [`tty_apply_features`](tty-features.c.driver.md#tty_apply_features) and applies overrides if successful.
    - If the terminal uses margins, sends the `TTYC_ENMG` code to enable margins.
    - If the global option for extended keys is enabled, sends the corresponding terminal string.
    - If the global option for focus events is enabled, sends the corresponding terminal string.
    - If the terminal is VT100-like, sends a specific escape sequence to enable certain features.
    - Calls [`server_redraw_client`](server-fn.c.driver.md#server_redraw_client) to update the client display based on the new features.
    - Invalidates the current `tty` state to ensure it reflects the updated features.
- **Output**:
    - The function does not return a value but updates the state of the terminal and client based on the applied features.
- **Functions called**:
    - [`tty_apply_features`](tty-features.c.driver.md#tty_apply_features)
    - [`tty_term_apply_overrides`](tty-term.c.driver.md#tty_term_apply_overrides)
    - [`tty_putcode`](#tty_putcode)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`tty_puts`](#tty_puts)
    - [`tty_term_string`](tty-term.c.driver.md#tty_term_string)
    - [`server_redraw_client`](server-fn.c.driver.md#server_redraw_client)
    - [`tty_invalidate`](#tty_invalidate)


---
### tty_raw<!-- {{#callable:tty_raw}} -->
The `tty_raw` function writes a string to a terminal device in a non-blocking manner, attempting multiple times to ensure all data is sent.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal context.
    - `s`: A pointer to a constant character string that contains the data to be written to the terminal.
- **Control Flow**:
    - The length of the string `s` is calculated using `strlen`.
    - A loop runs up to 5 times to attempt writing the string to the terminal.
    - Within the loop, `write` is called to send the data to the file descriptor associated with the terminal.
    - If `write` returns a positive number, the pointer `s` is advanced by that number, and the remaining length is decremented.
    - If the entire string is written (i.e., `slen` becomes 0), the loop breaks.
    - If `write` returns -1 and the error is not `EAGAIN`, the loop breaks.
    - If the write operation is not successful, the function sleeps for 100 microseconds before retrying.
- **Output**:
    - The function does not return a value; it writes data directly to the terminal device.


---
### tty_putcode<!-- {{#callable:tty_putcode}} -->
The `tty_putcode` function sends a terminal control code to the specified terminal.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal to which the code will be sent.
    - `code`: An enumeration value of type `enum tty_code_code` that specifies the terminal control code to be sent.
- **Control Flow**:
    - The function calls [`tty_term_string`](tty-term.c.driver.md#tty_term_string) to retrieve the string representation of the terminal control code based on the provided `tty` and `code`.
    - The resulting string is then passed to the [`tty_puts`](#tty_puts) function, which handles the actual sending of the string to the terminal.
- **Output**:
    - The function does not return a value; it sends the specified terminal control code to the terminal, affecting its behavior or appearance.
- **Functions called**:
    - [`tty_puts`](#tty_puts)
    - [`tty_term_string`](tty-term.c.driver.md#tty_term_string)


---
### tty_putcode_i<!-- {{#callable:tty_putcode_i}} -->
The `tty_putcode_i` function sends a terminal control sequence to the specified `tty` if the integer argument `a` is non-negative.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal to which the control sequence will be sent.
    - `code`: An enumeration value of type `enum tty_code_code` that specifies the control sequence to be sent.
    - `a`: An integer value that is used as an argument for the control sequence; if this value is negative, the function will not send any sequence.
- **Control Flow**:
    - The function first checks if the value of `a` is less than 0.
    - If `a` is negative, the function returns immediately without performing any action.
    - If `a` is non-negative, it calls the [`tty_term_string_i`](tty-term.c.driver.md#tty_term_string_i) function to get the appropriate control sequence string for the given `code` and `a`.
    - Finally, it sends the resulting string to the terminal using the [`tty_puts`](#tty_puts) function.
- **Output**:
    - The function does not return a value; it sends a control sequence to the terminal if the conditions are met.
- **Functions called**:
    - [`tty_puts`](#tty_puts)
    - [`tty_term_string_i`](tty-term.c.driver.md#tty_term_string_i)


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
    - If both parameters are non-negative, it calls [`tty_term_string_ii`](tty-term.c.driver.md#tty_term_string_ii) to generate the appropriate control string using the provided `code`, `a`, and `b`.
    - Finally, it sends the generated string to the terminal using the [`tty_puts`](#tty_puts) function.
- **Output**:
    - The function does not return a value; it outputs a control sequence to the terminal based on the provided parameters.
- **Functions called**:
    - [`tty_puts`](#tty_puts)
    - [`tty_term_string_ii`](tty-term.c.driver.md#tty_term_string_ii)


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
    - If all parameters are non-negative, it calls the [`tty_term_string_iii`](tty-term.c.driver.md#tty_term_string_iii) function to generate the appropriate terminal string using the provided `code` and parameters.
    - Finally, it sends the generated string to the terminal using the [`tty_puts`](#tty_puts) function.
- **Output**:
    - The function does not return a value; it outputs a terminal control sequence based on the provided parameters.
- **Functions called**:
    - [`tty_puts`](#tty_puts)
    - [`tty_term_string_iii`](tty-term.c.driver.md#tty_term_string_iii)


---
### tty_putcode_s<!-- {{#callable:tty_putcode_s}} -->
The `tty_putcode_s` function sends a terminal control sequence to the specified `tty` if the provided string is not NULL.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal to which the control sequence will be sent.
    - `code`: An enumeration value of type `enum tty_code_code` that specifies the type of control sequence to be sent.
    - `a`: A pointer to a string that contains additional data for the control sequence; if this is NULL, no action is taken.
- **Control Flow**:
    - The function first checks if the string `a` is not NULL.
    - If `a` is not NULL, it calls the [`tty_term_string_s`](tty-term.c.driver.md#tty_term_string_s) function to generate the appropriate control sequence string based on the `code` and `a`.
    - The generated string is then sent to the terminal using the [`tty_puts`](#tty_puts) function.
- **Output**:
    - The function does not return a value; it sends a control sequence to the terminal if the input string is not NULL.
- **Functions called**:
    - [`tty_puts`](#tty_puts)
    - [`tty_term_string_s`](tty-term.c.driver.md#tty_term_string_s)


---
### tty_putcode_ss<!-- {{#callable:tty_putcode_ss}} -->
The `tty_putcode_ss` function sends a terminal control sequence with two string parameters to a specified terminal.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal to which the control sequence will be sent.
    - `code`: An enumeration value of type `enum tty_code_code` that specifies the control sequence to be sent.
    - `a`: A pointer to a string that serves as the first parameter for the control sequence.
    - `b`: A pointer to a string that serves as the second parameter for the control sequence.
- **Control Flow**:
    - The function first checks if both input strings `a` and `b` are not NULL.
    - If both strings are valid, it calls the [`tty_term_string_ss`](tty-term.c.driver.md#tty_term_string_ss) function to generate the appropriate terminal control sequence using the provided `code`, `a`, and `b`.
    - The generated control sequence is then sent to the terminal using the [`tty_puts`](#tty_puts) function.
- **Output**:
    - The function does not return a value; it sends a control sequence to the terminal if the input strings are valid.
- **Functions called**:
    - [`tty_puts`](#tty_puts)
    - [`tty_term_string_ss`](tty-term.c.driver.md#tty_term_string_ss)


---
### tty_add<!-- {{#callable:tty_add}} -->
The `tty_add` function adds data to the output buffer of a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal interface.
    - `buf`: A pointer to a character buffer containing the data to be added.
    - `len`: The size of the data in bytes to be added from the buffer.
- **Control Flow**:
    - Checks if the `TTY_BLOCK` flag is set in the `tty` structure.
    - If `TTY_BLOCK` is set, increments the `discarded` count by `len` and returns early.
    - If not blocked, adds the data from `buf` to the `out` buffer of the `tty` structure.
    - Logs the added data using `log_debug`.
    - Increments the `written` count of the associated `client` structure.
    - If `tty_log_fd` is valid, writes the data to the log file.
    - If the `TTY_STARTED` flag is set, adds the output event to the event loop.
- **Output**:
    - The function does not return a value; it modifies the state of the `tty` structure and may log data or write to a log file.


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
    - The function does not return a value; it modifies the state of the terminal by sending the string to it if the string is not empty.
- **Functions called**:
    - [`tty_add`](#tty_add)


---
### tty_putc<!-- {{#callable:tty_putc}} -->
The `tty_putc` function outputs a single character to a terminal, handling character set attributes and cursor positioning.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal context.
    - `ch`: A character of type `u_char` to be output to the terminal.
- **Control Flow**:
    - Checks if the terminal is in a mode that prevents outputting characters at the current cursor position.
    - If the terminal supports alternate character sets, it retrieves the corresponding ACS representation of the character.
    - If an ACS representation is found, it adds that to the output buffer; otherwise, it adds the character itself.
    - If the character is a printable character, it updates the cursor position, handling line wrapping if necessary.
- **Output**:
    - The function does not return a value but modifies the terminal's output buffer and cursor position based on the character processed.
- **Functions called**:
    - [`tty_acs_get`](tty-acs.c.driver.md#tty_acs_get)
    - [`tty_add`](#tty_add)
    - [`tty_putcode_ii`](#tty_putcode_ii)


---
### tty_putn<!-- {{#callable:tty_putn}} -->
The `tty_putn` function writes a specified number of bytes from a buffer to a terminal, handling cursor positioning and line wrapping.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `buf`: A pointer to the data buffer containing the bytes to be written.
    - `len`: The number of bytes to write from the buffer.
    - `width`: The width of the text to be written, used for cursor positioning.
- **Control Flow**:
    - Checks if the terminal is in a mode that prevents automatic line wrapping and adjusts the length of data to be written if necessary.
    - Calls the [`tty_add`](#tty_add) function to add the specified data to the terminal's output buffer.
    - Calculates the new cursor position based on the width of the text written and the terminal's dimensions.
    - If the new cursor position exceeds the terminal's width, it wraps to the next line and adjusts the cursor position accordingly.
- **Output**:
    - The function does not return a value but modifies the terminal's output buffer and cursor position based on the written data.
- **Functions called**:
    - [`tty_add`](#tty_add)


---
### tty_set_italics<!-- {{#callable:tty_set_italics}} -->
Sets the terminal to italics mode if supported.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
- **Control Flow**:
    - Checks if the terminal supports italics mode using [`tty_term_has`](tty-term.c.driver.md#tty_term_has) with the `TTYC_SITM` capability.
    - Retrieves the default terminal type from global options.
    - If the terminal type is not 'screen' or does not start with 'screen-', it sends the italics enable code using [`tty_putcode`](#tty_putcode).
    - If the terminal type is 'screen', it sends the stand out mode code using [`tty_putcode`](#tty_putcode) instead.
- **Output**:
    - No return value; the function modifies the terminal's display attributes directly.
- **Functions called**:
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
    - [`options_get_string`](options.c.driver.md#options_get_string)
    - [`tty_putcode`](#tty_putcode)


---
### tty_set_title<!-- {{#callable:tty_set_title}} -->
Sets the terminal title if supported by the terminal.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal.
    - `title`: A string containing the title to be set for the terminal.
- **Control Flow**:
    - Checks if the terminal supports the title setting capabilities using [`tty_term_has`](tty-term.c.driver.md#tty_term_has) for `TTYC_TSL` and `TTYC_FSL`.
    - If either capability is not supported, the function returns early without making any changes.
    - If both capabilities are supported, it sends the escape sequence to start setting the title using [`tty_putcode`](#tty_putcode) with `TTYC_TSL`.
    - The provided title string is sent to the terminal using [`tty_puts`](#tty_puts).
    - Finally, it sends the escape sequence to end setting the title using [`tty_putcode`](#tty_putcode) with `TTYC_FSL`.
- **Output**:
    - The function does not return a value; it modifies the terminal's title if the terminal supports the operation.
- **Functions called**:
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
    - [`tty_putcode`](#tty_putcode)
    - [`tty_puts`](#tty_puts)


---
### tty_set_path<!-- {{#callable:tty_set_path}} -->
Sets the terminal's current working directory path in the title bar.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal.
    - `title`: A string representing the title to be set in the terminal's title bar.
- **Control Flow**:
    - Checks if the terminal supports the `SWD` (Set Window Title) and `FSL` (Final String) capabilities.
    - If either capability is not supported, the function returns early without making any changes.
    - If both capabilities are supported, it sends the `SWD` code to the terminal.
    - Then, it sends the provided `title` string to be displayed in the title bar.
    - Finally, it sends the `FSL` code to indicate the end of the title string.
- **Output**:
    - The function does not return a value; it modifies the terminal's title bar to display the specified path.
- **Functions called**:
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
    - [`tty_putcode`](#tty_putcode)
    - [`tty_puts`](#tty_puts)


---
### tty_force_cursor_colour<!-- {{#callable:tty_force_cursor_colour}} -->
The `tty_force_cursor_colour` function sets the cursor color for a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `c`: An integer representing the color value to set for the cursor, or -1 to reset the cursor color.
- **Control Flow**:
    - If the input color `c` is not -1, it is converted to an RGB color using [`colour_force_rgb`](colour.c.driver.md#colour_force_rgb).
    - If the new color is the same as the current cursor color, the function returns early.
    - If the new color is -1, the function sends a command to reset the cursor color using [`tty_putcode`](#tty_putcode).
    - If the new color is valid, it is split into its RGB components using [`colour_split_rgb`](colour.c.driver.md#colour_split_rgb), formatted into a string, and sent to the terminal using [`tty_putcode_s`](#tty_putcode_s).
    - Finally, the current cursor color in the `tty` structure is updated to the new color.
- **Output**:
    - The function does not return a value; it modifies the terminal's cursor color based on the provided input.
- **Functions called**:
    - [`colour_force_rgb`](colour.c.driver.md#colour_force_rgb)
    - [`tty_putcode`](#tty_putcode)
    - [`colour_split_rgb`](colour.c.driver.md#colour_split_rgb)
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)
    - [`tty_putcode_s`](#tty_putcode_s)


---
### tty_update_cursor<!-- {{#callable:tty_update_cursor}} -->
Updates the cursor appearance and behavior in a terminal based on the provided mode and screen settings.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal.
    - `mode`: An integer representing the cursor mode, which can include visibility and blinking settings.
    - `s`: A pointer to a `screen` structure that contains cursor style and color information.
- **Control Flow**:
    - Checks if the `screen` structure is not NULL to set the cursor color based on the provided screen settings.
    - If the cursor mode indicates it should be off, it sets the cursor to invisible and returns the current mode.
    - Determines the cursor style based on the provided screen settings or defaults to the current tty settings.
    - If there are no changes in cursor mode or style, it returns the current mode without making any updates.
    - If changes are detected, it updates the cursor style and visibility based on the specified mode and style.
- **Output**:
    - Returns the updated cursor mode after applying the necessary changes.
- **Functions called**:
    - [`tty_force_cursor_colour`](#tty_force_cursor_colour)
    - [`tty_putcode`](#tty_putcode)
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
    - [`tty_putcode_i`](#tty_putcode_i)


---
### tty_update_mode<!-- {{#callable:tty_update_mode}} -->
Updates the terminal's mode and cursor settings based on the provided parameters.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal to be updated.
    - `mode`: An integer representing the new mode settings to apply to the terminal.
    - `s`: A pointer to a `struct screen` that may contain additional cursor style information.
- **Control Flow**:
    - Checks if the terminal is set to not display the cursor and modifies the mode accordingly.
    - Calls [`tty_update_cursor`](#tty_update_cursor) to update the cursor settings based on the new mode and screen information.
    - Calculates the difference between the new mode and the current mode to determine if any changes occurred.
    - Logs the current and new mode if logging is enabled and changes were detected.
    - If mouse mode settings have changed, sends appropriate escape sequences to the terminal to update mouse tracking settings.
    - Finally, updates the terminal's mode with the new settings.
- **Output**:
    - The function does not return a value; it modifies the state of the terminal directly.
- **Functions called**:
    - [`tty_update_cursor`](#tty_update_cursor)
    - [`log_get_level`](log.c.driver.md#log_get_level)
    - [`screen_mode_to_string`](screen.c.driver.md#screen_mode_to_string)
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
    - [`tty_puts`](#tty_puts)


---
### tty_emulate_repeat<!-- {{#callable:tty_emulate_repeat}} -->
The `tty_emulate_repeat` function sends terminal control codes to emulate a repeat action based on the terminal's capabilities.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `code`: An `enum tty_code_code` value representing the primary control code to be sent.
    - `code1`: An `enum tty_code_code` value representing the fallback control code to be sent if the primary code is not supported.
    - `n`: An unsigned integer representing the number of times to repeat the primary control code.
- **Control Flow**:
    - The function first checks if the terminal supports the primary control code using [`tty_term_has`](tty-term.c.driver.md#tty_term_has).
    - If the primary control code is supported, it sends the code with a repeat count using [`tty_putcode_i`](#tty_putcode_i).
    - If the primary control code is not supported, it enters a loop that decrements `n` and sends the fallback control code `code1` until `n` reaches zero.
- **Output**:
    - The function does not return a value; it directly sends control codes to the terminal based on the specified conditions.
- **Functions called**:
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
    - [`tty_putcode_i`](#tty_putcode_i)
    - [`tty_putcode`](#tty_putcode)


---
### tty_repeat_space<!-- {{#callable:tty_repeat_space}} -->
The `tty_repeat_space` function outputs a specified number of space characters to a terminal.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `n`: An unsigned integer representing the total number of space characters to output.
- **Control Flow**:
    - The function initializes a static character array `s` of size 500 to hold space characters.
    - If the first character of `s` is not a space, it fills the entire array `s` with spaces.
    - It enters a loop that continues as long as `n` is greater than the size of `s`, outputting `s` to the terminal using [`tty_putn`](#tty_putn) and decrementing `n` accordingly.
    - After the loop, if there are any remaining spaces to output (i.e., if `n` is not zero), it outputs the remaining spaces using [`tty_putn`](#tty_putn).
- **Output**:
    - The function does not return a value; it directly outputs space characters to the terminal.
- **Functions called**:
    - [`tty_putn`](#tty_putn)


---
### tty_window_bigger<!-- {{#callable:tty_window_bigger}} -->
Determines if the terminal window is smaller than the current window.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that contains information about the terminal, including its size.
- **Control Flow**:
    - Retrieve the `client` associated with the `tty` structure.
    - Access the `window` from the current `session` of the `client`.
    - Compare the width (`sx`) and height (`sy`) of the `tty` with the width and height of the `window`.
    - Return true if the terminal's width or height is less than the window's dimensions.
- **Output**:
    - Returns a non-zero value (true) if the terminal window is smaller than the current window, otherwise returns zero (false).
- **Functions called**:
    - [`status_line_size`](status.c.driver.md#status_line_size)


---
### tty_window_offset<!-- {{#callable:tty_window_offset}} -->
The `tty_window_offset` function retrieves the current window offsets and size for a given terminal.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal.
    - `ox`: A pointer to a `u_int` variable where the x-offset will be stored.
    - `oy`: A pointer to a `u_int` variable where the y-offset will be stored.
    - `sx`: A pointer to a `u_int` variable where the width of the window will be stored.
    - `sy`: A pointer to a `u_int` variable where the height of the window will be stored.
- **Control Flow**:
    - The function dereferences the pointers `ox`, `oy`, `sx`, and `sy` to assign them the values of `tty->oox`, `tty->ooy`, `tty->osx`, and `tty->osy`, respectively.
    - Finally, it returns the value of `tty->oflag`.
- **Output**:
    - The function returns an integer representing the current flags of the terminal, specifically `tty->oflag`.


---
### tty_window_offset1<!-- {{#callable:tty_window_offset1}} -->
The `tty_window_offset1` function calculates the offsets and sizes for a terminal window based on the current terminal and window dimensions.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal.
    - `ox`: A pointer to an unsigned integer where the x-offset will be stored.
    - `oy`: A pointer to an unsigned integer where the y-offset will be stored.
    - `sx`: A pointer to an unsigned integer where the width of the window will be stored.
    - `sy`: A pointer to an unsigned integer where the height of the window will be stored.
- **Control Flow**:
    - The function retrieves the current client and window associated with the terminal.
    - It calculates the number of lines occupied by the status line.
    - If the terminal size is greater than or equal to the window size, it sets offsets and sizes to zero and the window dimensions, returning 0.
    - If the terminal is in a 'pan' state, it adjusts the offsets based on the current pan offsets.
    - If the cursor mode is not active, it sets offsets to zero; otherwise, it calculates the offsets based on the cursor position.
- **Output**:
    - The function returns an integer indicating whether the offsets were set to zero (0) or adjusted based on the cursor position (1), and updates the provided pointers with the calculated offsets and sizes.
- **Functions called**:
    - [`server_client_get_pane`](server-client.c.driver.md#server_client_get_pane)
    - [`status_line_size`](status.c.driver.md#status_line_size)


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
    - The function does not return a value; it modifies the offsets of clients directly.
- **Functions called**:
    - [`tty_update_client_offset`](#tty_update_client_offset)


---
### tty_update_client_offset<!-- {{#callable:tty_update_client_offset}} -->
Updates the terminal offsets for a client and triggers a redraw if the offsets have changed.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client whose terminal offsets are to be updated.
- **Control Flow**:
    - Checks if the `CLIENT_TERMINAL` flag is set in the client's flags; if not, the function returns immediately.
    - Calls [`tty_window_offset1`](#tty_window_offset1) to retrieve the new offsets (ox, oy) and sizes (sx, sy) for the terminal window.
    - Compares the new offsets and sizes with the stored values in the client's `tty` structure.
    - If the offsets or sizes have changed, logs the change using `log_debug`.
    - Updates the stored offsets and sizes in the client's `tty` structure.
    - Sets the `CLIENT_REDRAWWINDOW` and `CLIENT_REDRAWSTATUS` flags in the client's flags to indicate that a redraw is needed.
- **Output**:
    - The function does not return a value; it modifies the state of the client and may trigger a redraw of the terminal window.
- **Functions called**:
    - [`tty_window_offset1`](#tty_window_offset1)


---
### tty_large_region<!-- {{#callable:tty_large_region}} -->
Determines if a specified region in a terminal context is large enough to warrant a redraw.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal.
    - `ctx`: A pointer to a `tty_ctx` structure containing context information about the terminal display.
- **Control Flow**:
    - Calculates the difference between `ctx->orlower` and `ctx->orupper` to determine the height of the region.
    - Compares the height of the region to half of the terminal's height (`ctx->sy / 2`).
    - Returns a boolean indicating whether the region is large enough for a redraw based on the comparison.
- **Output**:
    - Returns 1 (true) if the region is large enough to warrant a redraw, otherwise returns 0 (false).


---
### tty_fake_bce<!-- {{#callable:tty_fake_bce}} -->
The `tty_fake_bce` function determines if back color erase (BCE) emulation is needed based on terminal capabilities and color settings.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `gc`: A pointer to a `struct grid_cell` representing the grid cell's attributes.
    - `bg`: An unsigned integer representing the background color.
- **Control Flow**:
    - Check if the terminal supports back color erase (BCE) using [`tty_term_flag`](tty-term.c.driver.md#tty_term_flag).
    - If BCE is supported, return 0 indicating no emulation is needed.
    - Check if the background color (`bg`) or the grid cell's background color (`gc->bg`) is not the default color.
    - If either color is not default, return 1 indicating that BCE emulation is needed.
    - If both colors are default, return 0 indicating no emulation is needed.
- **Output**:
    - Returns 0 if BCE is supported or both colors are default; returns 1 if BCE emulation is needed.
- **Functions called**:
    - [`tty_term_flag`](tty-term.c.driver.md#tty_term_flag)


---
### tty_redraw_region<!-- {{#callable:tty_redraw_region}} -->
The `tty_redraw_region` function redraws a specified region of the terminal if it is not too large.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing the context for the redraw operation, including the region to be redrawn.
- **Control Flow**:
    - The function first checks if the region defined by `ctx` is large using the [`tty_large_region`](#tty_large_region) function.
    - If the region is large, it logs a debug message and calls the `redraw_cb` callback function provided in `ctx` to handle the redraw.
    - If the region is not large, it enters a loop to redraw each line in the specified region from `orupper` to `orlower` by calling [`tty_draw_pane`](#tty_draw_pane).
- **Output**:
    - The function does not return a value; it performs redraw operations directly on the terminal.
- **Functions called**:
    - [`tty_large_region`](#tty_large_region)
    - [`tty_draw_pane`](#tty_draw_pane)


---
### tty_is_visible<!-- {{#callable:tty_is_visible}} -->
The `tty_is_visible` function checks if a specified area is visible within a terminal's window.
- **Inputs**:
    - `tty`: A pointer to a `struct tty`, which represents the terminal.
    - `ctx`: A pointer to a `struct tty_ctx`, which contains context information about the terminal's drawing area.
    - `px`: The x-coordinate offset of the area to check.
    - `py`: The y-coordinate offset of the area to check.
    - `nx`: The width of the area to check.
    - `ny`: The height of the area to check.
- **Control Flow**:
    - The function first calculates the absolute x and y offsets by adding the context's offsets to the provided coordinates.
    - It checks if the `bigger` flag in the context is not set; if so, it returns 1, indicating the area is visible.
    - It then checks if the calculated area is outside the bounds of the terminal's window; if it is, it returns 0, indicating the area is not visible.
    - If none of the above conditions are met, it returns 1, indicating the area is visible.
- **Output**:
    - The function returns an integer: 1 if the area is visible, and 0 if it is not.


---
### tty_clamp_line<!-- {{#callable:tty_clamp_line}} -->
The `tty_clamp_line` function adjusts the position and size of a line within a terminal context to ensure it remains visible.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing the context information for the terminal.
    - `px`: The x-coordinate of the line to be clamped.
    - `py`: The y-coordinate of the line to be clamped.
    - `nx`: The width of the line to be clamped.
    - `i`: A pointer to a variable where the starting index of the visible part will be stored.
    - `x`: A pointer to a variable where the adjusted x-coordinate will be stored.
    - `rx`: A pointer to a variable where the width of the visible part will be stored.
    - `ry`: A pointer to a variable where the adjusted y-coordinate will be stored.
- **Control Flow**:
    - The function first calculates the offset `xoff` based on the context's right offset and the provided x-coordinate.
    - It checks if the line is visible using the [`tty_is_visible`](#tty_is_visible) function; if not, it returns 0.
    - The adjusted y-coordinate is calculated and stored in `*ry`.
    - The function then checks various conditions to determine how much of the line is visible and adjusts the indices accordingly:
    - 1. If the entire line is visible, it sets `*i`, `*x`, and `*rx` to indicate full visibility.
    - 2. If both ends of the line are outside the visible area, it sets `*i`, `*x`, and `*rx` to indicate that the line is completely off-screen.
    - 3. If only the left end is outside, it adjusts `*i`, `*x`, and `*rx` to reflect the visible portion on the right.
    - 4. If only the right end is outside, it adjusts `*i`, `*x`, and `*rx` to reflect the visible portion on the left.
    - Finally, it checks if the calculated visible width exceeds the original width and raises a fatal error if so.
- **Output**:
    - The function returns 1 if the line is visible and successfully clamped, or 0 if the line is not visible.
- **Functions called**:
    - [`tty_is_visible`](#tty_is_visible)


---
### tty_clear_line<!-- {{#callable:tty_clear_line}} -->
The `tty_clear_line` function clears a specified section of a line in a terminal, using escape sequences or spaces based on terminal capabilities.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `defaults`: A pointer to a `struct grid_cell` containing default cell attributes.
    - `py`: An unsigned integer representing the y-coordinate (row) of the line to clear.
    - `px`: An unsigned integer representing the x-coordinate (column) where the clearing starts.
    - `nx`: An unsigned integer indicating the number of cells to clear.
    - `bg`: An unsigned integer representing the background color to use when clearing.
- **Control Flow**:
    - Logs the function call with the client's name and parameters.
    - Checks if `nx` is zero; if so, returns immediately as there is nothing to clear.
    - Checks if genuine Background Color Erasure (BCE) is available; if not, proceeds to use escape sequences.
    - If the clearing area extends beyond the terminal width, uses the `EL` escape sequence to clear to the end of the line.
    - If clearing starts at the beginning of the line, uses the `EL1` escape sequence.
    - If clearing a section of the line, uses the `ECH` escape sequence to clear the specified number of cells.
    - If no escape sequences can be used, checks for overlays and clears the visible parts using spaces.
- **Output**:
    - The function does not return a value; it modifies the terminal display by clearing the specified line section.
- **Functions called**:
    - [`tty_fake_bce`](#tty_fake_bce)
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
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
    - `py`: The y-coordinate (row) of the line to be cleared.
    - `px`: The x-coordinate (column) from which to start clearing.
    - `nx`: The number of characters to clear from the line.
    - `bg`: The background color to use when clearing the line.
- **Control Flow**:
    - Logs the action of clearing the line with the specified parameters.
    - Calls [`tty_clamp_line`](#tty_clamp_line) to determine the visible portion of the line within the pane.
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
    - The function first calculates the absolute offsets of the area based on the provided coordinates and the context offsets.
    - It checks if the area is visible within the terminal pane using the [`tty_is_visible`](#tty_is_visible) function.
    - If the area is fully visible, it sets the output parameters accordingly.
    - If the area extends beyond the left and right bounds of the visible region, it adjusts the offsets and dimensions to fit within the visible area.
    - If only the left side is not visible, it adjusts the left offset and reduces the width accordingly.
    - If only the right side is not visible, it adjusts the width to fit within the visible area.
    - The same logic is applied for the vertical (y) coordinates to ensure the area fits within the top and bottom bounds of the visible region.
    - Finally, it checks if the adjusted height exceeds the original height and raises a fatal error if it does.
- **Output**:
    - The function returns 1 if the area was successfully clamped to fit within the visible region, or 0 if the area is not visible.
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
    - Logs the function call with parameters.
    - Checks if the area to clear is valid (non-zero dimensions).
    - If genuine Background Color Erasure (BCE) is available, attempts to use escape sequences to clear the area.
    - If the area is at the bottom of the terminal, uses the ED escape sequence to clear it.
    - If the terminal supports DECFRA and the background color is not default, uses a specific escape sequence.
    - If the area can be scrolled away, it uses scrolling commands to clear the area.
    - If none of the above methods are applicable, it loops through each line in the specified area and calls [`tty_clear_line`](#tty_clear_line) to clear it.
- **Output**:
    - The function does not return a value; it directly modifies the terminal display by clearing the specified area.
- **Functions called**:
    - [`tty_fake_bce`](#tty_fake_bce)
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
    - [`tty_cursor`](#tty_cursor)
    - [`tty_putcode`](#tty_putcode)
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)
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
The `tty_draw_pane` function draws a specified line of a pane on a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing the drawing context, including screen and pane dimensions.
    - `py`: An unsigned integer representing the y-coordinate of the line to be drawn.
- **Control Flow**:
    - Logs the function call with the client's name and the y-coordinate.
    - Checks if the pane context is not bigger; if so, it calls [`tty_draw_line`](#tty_draw_line) to draw the line directly.
    - If the pane context is bigger, it calls [`tty_clamp_line`](#tty_clamp_line) to determine the visible portion of the line and then draws it using [`tty_draw_line`](#tty_draw_line).
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
    - Copies the original `grid_cell` to a new static `grid_cell` for potential modifications.
    - Attempts to map the character to an ACS (Alternate Character Set) character using [`tty_acs_reverse_get`](tty-acs.c.driver.md#tty_acs_reverse_get).
    - If a valid ACS character is found, it updates the new `grid_cell` with this character and sets the `GRID_ATTR_CHARSET` attribute.
    - If no valid ACS character is found, it replaces the character data with underscores based on the width of the original character.
- **Output**:
    - Returns a pointer to a potentially modified `grid_cell`, which may represent the original character, an ACS character, or a series of underscores.
- **Functions called**:
    - [`tty_acs_reverse_get`](tty-acs.c.driver.md#tty_acs_reverse_get)
    - [`utf8_set`](utf8.c.driver.md#utf8_set)


---
### tty_check_overlay<!-- {{#callable:tty_check_overlay}} -->
Checks if a specific point on a terminal is obstructed by an overlay.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `px`: An unsigned integer representing the x-coordinate of the point to check.
    - `py`: An unsigned integer representing the y-coordinate of the point to check.
- **Control Flow**:
    - Calls [`tty_check_overlay_range`](#tty_check_overlay_range) to check the overlay ranges at the specified coordinates.
    - If the sum of the first two entries in the overlay ranges is zero, it indicates no overlay is present.
    - Returns 0 if no overlay is found, otherwise returns 1.
- **Output**:
    - Returns 0 if the specified point is not obstructed by an overlay, otherwise returns 1.
- **Functions called**:
    - [`tty_check_overlay_range`](#tty_check_overlay_range)


---
### tty_check_overlay_range<!-- {{#callable:tty_check_overlay_range}} -->
The `tty_check_overlay_range` function checks the visibility of a specified range of characters on a terminal overlay.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `px`: An unsigned integer representing the starting x-coordinate of the range to check.
    - `py`: An unsigned integer representing the y-coordinate of the line to check.
    - `nx`: An unsigned integer representing the width of the range to check.
    - `r`: A pointer to a `struct overlay_ranges` where the results of the visibility check will be stored.
- **Control Flow**:
    - The function retrieves the `client` associated with the `tty` structure.
    - It checks if the `overlay_check` function pointer of the `client` is NULL.
    - If `overlay_check` is NULL, it sets the first range in `r` to the provided coordinates and width, and the other ranges to zero.
    - If `overlay_check` is not NULL, it calls the `overlay_check` function with the provided parameters to fill the `overlay_ranges` structure.
- **Output**:
    - The function does not return a value but populates the `overlay_ranges` structure with the visible parts of the specified range.


---
### tty_draw_line<!-- {{#callable:tty_draw_line}} -->
The `tty_draw_line` function draws a line on a terminal screen based on specified parameters.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `s`: A pointer to a `struct screen` representing the screen to draw on.
    - `px`: The starting x-coordinate in the grid from which to draw.
    - `py`: The y-coordinate in the grid where the line will be drawn.
    - `nx`: The width of the line to draw.
    - `atx`: The x-coordinate on the terminal where the line will be drawn.
    - `aty`: The y-coordinate on the terminal where the line will be drawn.
    - `defaults`: A pointer to a `struct grid_cell` containing default attributes for the line.
    - `palette`: A pointer to a `struct colour_palette` for color management.
- **Control Flow**:
    - The function begins by logging the input parameters for debugging purposes.
    - It updates the terminal mode and disables the cursor temporarily.
    - The width of the line to be drawn is clamped to the screen size and cell size.
    - If the line is not wrapped and certain conditions are met, it clears the line using terminal escape codes.
    - The function iterates over the specified width, retrieving grid cells and checking their attributes.
    - It handles drawing characters, managing overlays, and checking for visibility based on the terminal's capabilities.
    - If the line drawing is completed, it ensures any remaining space is cleared if necessary.
    - Finally, it restores the cursor visibility and updates the terminal mode.
- **Output**:
    - The function does not return a value; it directly modifies the terminal display by drawing a line based on the provided parameters.
- **Functions called**:
    - [`tty_update_mode`](#tty_update_mode)
    - [`tty_region_off`](#tty_region_off)
    - [`tty_margin_off`](#tty_margin_off)
    - [`grid_get_line`](grid.c.driver.md#grid_get_line)
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
    - [`tty_fake_bce`](#tty_fake_bce)
    - [`tty_default_attributes`](#tty_default_attributes)
    - [`tty_cursor`](#tty_cursor)
    - [`tty_putcode`](#tty_putcode)
    - [`grid_view_get_cell`](grid-view.c.driver.md#grid_view_get_cell)
    - [`tty_check_codeset`](#tty_check_codeset)
    - [`tty_check_overlay`](#tty_check_overlay)
    - [`tty_attributes`](#tty_attributes)
    - [`tty_clear_line`](#tty_clear_line)
    - [`tty_putn`](#tty_putn)
    - [`screen_select_cell`](screen.c.driver.md#screen_select_cell)
    - [`tty_check_overlay_range`](#tty_check_overlay_range)
    - [`tty_repeat_space`](#tty_repeat_space)
    - [`tty_putc`](#tty_putc)


---
### tty_set_client_cb<!-- {{#callable:tty_set_client_cb}} -->
Sets the properties of a `tty_ctx` based on the current `client` and its associated `window_pane`.
- **Inputs**:
    - `ttyctx`: A pointer to a `tty_ctx` structure that holds the terminal context information.
    - `c`: A pointer to a `client` structure representing the current client.
- **Control Flow**:
    - Checks if the current client's window matches the window associated with the `tty_ctx`; if not, returns 0.
    - Checks if the layout cell of the window pane is NULL; if so, returns 0.
    - Calculates the terminal window offset and updates the `ttyctx` properties accordingly.
    - Adjusts the vertical offset in `ttyctx` based on the status line size if applicable.
    - Returns 1 to indicate successful property setting.
- **Output**:
    - Returns 1 if the properties were successfully set, otherwise returns 0.
- **Functions called**:
    - [`tty_window_offset`](#tty_window_offset)
    - [`status_at_line`](status.c.driver.md#status_at_line)
    - [`status_line_size`](status.c.driver.md#status_line_size)


---
### tty_draw_images<!-- {{#callable:tty_draw_images}} -->
The `tty_draw_images` function draws images on a terminal screen for a specified client and window pane.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client for which the images are being drawn.
    - `wp`: A pointer to a `struct window_pane` representing the window pane where the images will be drawn.
    - `s`: A pointer to a `struct screen` containing the images to be drawn.
- **Control Flow**:
    - Iterates over each image in the `screen`'s image list using `TAILQ_FOREACH`.
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
    - `tty`: A pointer to a `struct tty` representing the terminal interface that is being synchronized.
- **Control Flow**:
    - Checks if the `TTY_BLOCK` flag is set in the `tty` structure; if so, the function returns immediately without doing anything.
    - Checks if the `TTY_SYNCING` flag is already set; if so, the function returns immediately.
    - Sets the `TTY_SYNCING` flag in the `tty` structure to indicate that synchronization has started.
    - Checks if the terminal supports the `TTYC_SYNC` capability; if it does, it logs a debug message indicating the start of synchronization.
    - Sends a synchronization command to the terminal using [`tty_putcode_i`](#tty_putcode_i).
- **Output**:
    - The function does not return a value; it modifies the state of the `tty` structure and may send a synchronization command to the terminal if supported.
- **Functions called**:
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
    - [`tty_putcode_i`](#tty_putcode_i)


---
### tty_sync_end<!-- {{#callable:tty_sync_end}} -->
The `tty_sync_end` function ends a synchronization state for a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal interface.
- **Control Flow**:
    - Checks if the `TTY_BLOCK` flag is set in the `tty` structure; if so, it returns immediately.
    - Checks if the `TTY_SYNCING` flag is not set; if so, it returns immediately.
    - Clears the `TTY_SYNCING` flag from the `tty` structure.
    - Checks if the terminal supports the `TTYC_SYNC` capability.
    - If supported, logs a debug message indicating the end of synchronization and sends a synchronization end code to the terminal.
- **Output**:
    - The function does not return a value; it modifies the state of the `tty` structure and may send output to the terminal.
- **Functions called**:
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
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
    - Finally, it checks if the terminal is frozen (indicated by the `TTY_FREEZE` flag), returning 0 if it is not frozen.
    - If none of the above conditions are met, the function returns 1, indicating the client is ready.
- **Output**:
    - The function returns an integer: 0 if the client is not ready, and 1 if the client is ready for interaction.


---
### tty_write<!-- {{#callable:tty_write}} -->
The `tty_write` function sends commands to all ready clients based on the provided context.
- **Inputs**:
    - `cmdfn`: A function pointer to a command function that takes a `struct tty *` and a `struct tty_ctx *` as parameters.
    - `ctx`: A pointer to a `struct tty_ctx` that contains the context for the command execution.
- **Control Flow**:
    - Checks if the `set_client_cb` callback in the context is NULL; if so, it returns immediately.
    - Iterates over all clients in the `clients` list using `TAILQ_FOREACH`.
    - For each client, it checks if the client is ready using the [`tty_client_ready`](#tty_client_ready) function.
    - If the client is ready, it calls the `set_client_cb` callback with the context and client, storing the return value in `state`.
    - If `state` is -1, the loop breaks, indicating an error or stop condition.
    - If `state` is 0, it continues to the next client without executing the command.
    - If `state` is positive, it calls the command function `cmdfn` with the client's `tty` and the context.
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
    - Check if the `set_client_cb` callback in the `ctx` is NULL; if so, return immediately.
    - Invoke the `set_client_cb` callback with the context and client; if it returns 1, proceed to execute the command.
    - Call the command function `cmdfn` with the client's `tty` and the context.
- **Output**:
    - The function does not return a value; it performs an action based on the command function and the client context.


---
### tty_cmd_insertcharacter<!-- {{#callable:tty_cmd_insertcharacter}} -->
Inserts a character at the current cursor position in a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal interface.
    - `ctx`: A pointer to a `struct tty_ctx` containing context information such as cursor position and attributes.
- **Control Flow**:
    - Checks if the terminal context indicates a larger size or if the terminal is not full width.
    - Checks if background color emulation is needed and if the terminal supports the insert character operation.
    - If any of the above checks fail, it redraws the pane at the current cursor Y position and exits.
    - Sets the default attributes for the terminal based on the context provided.
    - Moves the cursor to the specified position in the pane.
    - Emulates the insertion of the character using the appropriate terminal codes.
- **Output**:
    - No return value; the function modifies the terminal display by inserting a character at the cursor position.
- **Functions called**:
    - [`tty_fake_bce`](#tty_fake_bce)
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
    - [`tty_draw_pane`](#tty_draw_pane)
    - [`tty_default_attributes`](#tty_default_attributes)
    - [`tty_cursor_pane`](#tty_cursor_pane)
    - [`tty_emulate_repeat`](#tty_emulate_repeat)


---
### tty_cmd_deletecharacter<!-- {{#callable:tty_cmd_deletecharacter}} -->
The `tty_cmd_deletecharacter` function deletes a character at the current cursor position in a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing the current context, including cursor position and attributes.
- **Control Flow**:
    - The function first retrieves the associated `client` from the `tty` structure.
    - It checks several conditions to determine if the character deletion can proceed, including whether the context is larger than the terminal, if the terminal supports the delete character operation, and if there are any overlays present.
    - If any of these conditions are not met, it calls [`tty_draw_pane`](#tty_draw_pane) to redraw the pane at the current cursor y-coordinate and exits.
    - If the conditions are satisfied, it sets the default attributes for the terminal using [`tty_default_attributes`](#tty_default_attributes).
    - The cursor position is updated using [`tty_cursor_pane`](#tty_cursor_pane) to reflect the current context.
    - Finally, it calls [`tty_emulate_repeat`](#tty_emulate_repeat) to perform the actual character deletion operation, using the appropriate terminal codes.
- **Output**:
    - The function does not return a value; it modifies the terminal display by deleting a character at the cursor position.
- **Functions called**:
    - [`tty_fake_bce`](#tty_fake_bce)
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
    - [`tty_draw_pane`](#tty_draw_pane)
    - [`tty_default_attributes`](#tty_default_attributes)
    - [`tty_cursor_pane`](#tty_cursor_pane)
    - [`tty_emulate_repeat`](#tty_emulate_repeat)


---
### tty_cmd_clearcharacter<!-- {{#callable:tty_cmd_clearcharacter}} -->
Clears a specified number of characters from the current line in a terminal.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing the context for the operation, including cursor position and attributes.
- **Control Flow**:
    - Calls [`tty_default_attributes`](#tty_default_attributes) to set the terminal's default attributes based on the provided context.
    - Invokes [`tty_clear_pane_line`](#tty_clear_pane_line) to clear a specified number of characters from the line at the cursor's position.
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
    - Checks if the terminal context indicates a larger size or if the terminal does not support full-width operations.
    - If any of the conditions are met, it calls [`tty_redraw_region`](#tty_redraw_region) to redraw the affected region and exits.
    - Sets the default attributes for the terminal using [`tty_default_attributes`](#tty_default_attributes).
    - Defines the region for the pane using [`tty_region_pane`](#tty_region_pane).
    - Disables margins with [`tty_margin_off`](#tty_margin_off).
    - Moves the cursor to the specified position using [`tty_cursor_pane`](#tty_cursor_pane).
    - Emulates the insertion of lines using [`tty_emulate_repeat`](#tty_emulate_repeat) with the appropriate terminal codes.
    - Resets the cursor position to an invalid state (UINT_MAX) to indicate it needs to be recalculated.
- **Output**:
    - The function does not return a value but modifies the terminal display by inserting lines and updating the cursor position.
- **Functions called**:
    - [`tty_fake_bce`](#tty_fake_bce)
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
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
    - Checks if the terminal context indicates a larger size or if the terminal does not support full width operations.
    - If any of the conditions are met, it calls [`tty_redraw_region`](#tty_redraw_region) to refresh the display and exits the function.
    - Sets the default attributes for the terminal using [`tty_default_attributes`](#tty_default_attributes).
    - Defines the region of the pane to be affected using [`tty_region_pane`](#tty_region_pane).
    - Disables margins with [`tty_margin_off`](#tty_margin_off).
    - Positions the cursor in the pane using [`tty_cursor_pane`](#tty_cursor_pane).
    - Emulates the deletion of a line using [`tty_emulate_repeat`](#tty_emulate_repeat) with the appropriate terminal codes.
    - Resets the cursor position to an invalid state.
- **Output**:
    - The function does not return a value but modifies the terminal display by deleting a line and updating the cursor position.
- **Functions called**:
    - [`tty_fake_bce`](#tty_fake_bce)
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
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
    - `tty`: A pointer to a `tty` structure representing the terminal.
    - `ctx`: A pointer to a `tty_ctx` structure containing context information such as cursor position and default attributes.
- **Control Flow**:
    - Calls [`tty_default_attributes`](#tty_default_attributes) to set the terminal's default attributes based on the provided context.
    - Invokes [`tty_clear_pane_line`](#tty_clear_pane_line) to clear the specified line in the terminal, using the context's cursor position and background color.
- **Output**:
    - The function does not return a value; it modifies the terminal display by clearing the current line.
- **Functions called**:
    - [`tty_default_attributes`](#tty_default_attributes)
    - [`tty_clear_pane_line`](#tty_clear_pane_line)


---
### tty_cmd_clearendofline<!-- {{#callable:tty_cmd_clearendofline}} -->
Clears the end of the current line in a terminal context.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal.
    - `ctx`: A pointer to a `struct tty_ctx` containing the context for the terminal operations.
- **Control Flow**:
    - Calculates the number of columns to clear from the current cursor position to the end of the line.
    - Calls [`tty_default_attributes`](#tty_default_attributes) to set the terminal attributes based on the context.
    - Calls [`tty_clear_pane_line`](#tty_clear_pane_line) to perform the actual clearing of the line from the cursor position to the end.
- **Output**:
    - The function does not return a value; it modifies the terminal display by clearing part of the line.
- **Functions called**:
    - [`tty_default_attributes`](#tty_default_attributes)
    - [`tty_clear_pane_line`](#tty_clear_pane_line)


---
### tty_cmd_clearstartofline<!-- {{#callable:tty_cmd_clearstartofline}} -->
The `tty_cmd_clearstartofline` function clears the start of the current line in a terminal context.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing the context for the terminal operations, including cursor position and default attributes.
- **Control Flow**:
    - The function first calls [`tty_default_attributes`](#tty_default_attributes) to set the terminal's attributes based on the provided context's defaults, palette, background color, and hyperlinks.
    - Next, it invokes [`tty_clear_pane_line`](#tty_clear_pane_line) to clear the line from the start up to the current cursor position, effectively removing any text from the start of the line.
- **Output**:
    - The function does not return a value; it modifies the terminal display by clearing part of the line as specified.
- **Functions called**:
    - [`tty_default_attributes`](#tty_default_attributes)
    - [`tty_clear_pane_line`](#tty_clear_pane_line)


---
### tty_cmd_reverseindex<!-- {{#callable:tty_cmd_reverseindex}} -->
The `tty_cmd_reverseindex` function handles the reverse index command for a terminal, adjusting the cursor position and redrawing the region if necessary.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing the current terminal context, including cursor position and region information.
- **Control Flow**:
    - Checks if the cursor's y-coordinate (`ocy`) matches the upper region (`orupper`); if not, the function returns immediately.
    - Evaluates several conditions to determine if the terminal can perform a reverse index operation; if any condition fails, it calls [`tty_redraw_region`](#tty_redraw_region) to redraw the current region and returns.
    - Sets the default attributes for the terminal using [`tty_default_attributes`](#tty_default_attributes) based on the provided context.
    - Defines the region for the pane using [`tty_region_pane`](#tty_region_pane) and sets the margin using [`tty_margin_pane`](#tty_margin_pane).
    - Moves the cursor to the upper region using [`tty_cursor_pane`](#tty_cursor_pane).
    - Finally, sends the appropriate escape code for reverse indexing using [`tty_putcode`](#tty_putcode) or [`tty_putcode_i`](#tty_putcode_i).
- **Output**:
    - The function does not return a value but modifies the terminal's state by potentially moving the cursor and redrawing the terminal region.
- **Functions called**:
    - [`tty_fake_bce`](#tty_fake_bce)
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
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
    - Checks if the cursor's y-coordinate (`ocy`) is equal to the lower region limit (`orlower`); if not, the function returns immediately.
    - Evaluates several conditions to determine if the terminal needs to redraw the region instead of performing a line feed.
    - If the conditions for redrawing are not met, it sets the default attributes for the terminal using [`tty_default_attributes`](#tty_default_attributes).
    - Defines the region for the pane using [`tty_region_pane`](#tty_region_pane) and sets the margins with [`tty_margin_pane`](#tty_margin_pane).
    - Adjusts the cursor position based on the current context and whether margins are used.
    - Finally, it outputs a newline character to the terminal using [`tty_putc`](#tty_putc).
- **Output**:
    - The function does not return a value; it modifies the terminal's state and outputs a newline character to the terminal.
- **Functions called**:
    - [`tty_fake_bce`](#tty_fake_bce)
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
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
    - If any of the conditions are met, it calls [`tty_redraw_region`](#tty_redraw_region) to redraw the region instead of scrolling.
    - Sets the default attributes for the terminal display using [`tty_default_attributes`](#tty_default_attributes).
    - Defines the scroll region using [`tty_region_pane`](#tty_region_pane) and [`tty_margin_pane`](#tty_margin_pane).
    - If the number of lines to scroll is 1 or the terminal does not support the `INDN` capability, it moves the cursor to the appropriate position and outputs new line characters.
    - If the terminal supports the `INDN` capability, it uses [`tty_putcode_i`](#tty_putcode_i) to scroll up the specified number of lines.
- **Output**:
    - The function does not return a value but modifies the terminal display by scrolling up the content and adjusting the cursor position accordingly.
- **Functions called**:
    - [`tty_fake_bce`](#tty_fake_bce)
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
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
    - Checks if the terminal context indicates a larger display or if certain terminal capabilities are not supported.
    - If any of the conditions are met, it calls [`tty_redraw_region`](#tty_redraw_region) to redraw the region instead of scrolling.
    - Sets the default attributes for the terminal using [`tty_default_attributes`](#tty_default_attributes).
    - Defines the scroll region using [`tty_region_pane`](#tty_region_pane) and sets the cursor position with [`tty_cursor_pane`](#tty_cursor_pane).
    - If the terminal supports the `RI` command, it uses [`tty_putcode_i`](#tty_putcode_i) to scroll down the specified number of lines.
    - If `RI` is not supported, it falls back to a loop that calls [`tty_putcode`](#tty_putcode) for each line to be scrolled down.
- **Output**:
    - The function does not return a value but modifies the terminal display by scrolling down the specified number of lines or redrawing the region if scrolling is not possible.
- **Functions called**:
    - [`tty_fake_bce`](#tty_fake_bce)
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
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
    - Calls [`tty_default_attributes`](#tty_default_attributes) to set the default attributes for the terminal based on the context.
    - Sets the region for the pane using [`tty_region_pane`](#tty_region_pane) to cover the entire height of the terminal.
    - Disables margins with [`tty_margin_off`](#tty_margin_off).
    - Calculates the starting position (px, py) and dimensions (nx, ny) for the area to be cleared.
    - Calls [`tty_clear_pane_area`](#tty_clear_pane_area) to clear the area from the cursor's current y position to the bottom of the screen.
    - Calculates the position and width for the line at the cursor's current y position and calls [`tty_clear_pane_line`](#tty_clear_pane_line) to clear that line.
- **Output**:
    - No return value; the function modifies the terminal display by clearing the specified areas.
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
    - `ctx`: A pointer to a `struct tty_ctx` containing context information for the terminal.
- **Control Flow**:
    - Calls [`tty_default_attributes`](#tty_default_attributes) to set the default attributes for the terminal based on the context.
    - Sets the region for the pane using [`tty_region_pane`](#tty_region_pane) to cover the entire height of the terminal.
    - Disables margins with [`tty_margin_off`](#tty_margin_off).
    - Defines the coordinates for the area to clear: from the top of the screen to the current cursor position.
    - Calls [`tty_clear_pane_area`](#tty_clear_pane_area) to clear the area from the top to the current cursor y-coordinate.
    - Defines the coordinates for the line to clear: from the start of the line to the current cursor x-coordinate plus one.
    - Calls [`tty_clear_pane_line`](#tty_clear_pane_line) to clear the line at the current cursor y-coordinate.
- **Output**:
    - The function does not return a value; it modifies the terminal display by clearing the specified area.
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
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing the context for the terminal operations, including dimensions and default attributes.
- **Control Flow**:
    - Calls [`tty_default_attributes`](#tty_default_attributes) to set the terminal's default attributes based on the provided context.
    - Sets the terminal region to cover the entire screen using [`tty_region_pane`](#tty_region_pane).
    - Disables any margins with [`tty_margin_off`](#tty_margin_off).
    - Defines the coordinates for the area to clear, setting `px` and `py` to 0, and `nx` and `ny` to the screen dimensions from the context.
    - Calls [`tty_clear_pane_area`](#tty_clear_pane_area) to clear the specified area of the terminal.
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
    - `ctx`: A pointer to a `struct tty_ctx` containing the context for the terminal operations, including size and attributes.
- **Control Flow**:
    - Checks if the `ctx->bigger` flag is set; if so, it calls the `redraw_cb` callback function and returns early.
    - Calls [`tty_attributes`](#tty_attributes) to set the terminal attributes based on the provided context.
    - Sets the drawing region for the entire height of the terminal using [`tty_region_pane`](#tty_region_pane).
    - Disables margins with [`tty_margin_off`](#tty_margin_off).
    - Iterates over each row of the terminal (from 0 to `ctx->sy - 1`), setting the cursor position to the start of each row using [`tty_cursor_pane`](#tty_cursor_pane).
    - Within each row, iterates over each column (from 0 to `ctx->sx - 1`), placing the character 'E' at each position using [`tty_putc`](#tty_putc).
- **Output**:
    - The function does not return a value but modifies the terminal display by filling it with the character 'E' across the specified dimensions.
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
    - Checks if the cell is visible and if it is obstructed by overlays; if not, the function returns early.
    - Handles partially obstructed wide characters by checking overlay ranges and drawing the visible part.
    - If the cursor position exceeds the terminal width and is at the lower edge, it adjusts the pane region.
    - Sets the margin off and positions the cursor unless it would wrap.
    - Draws the cell using the specified attributes and palette.
    - Invalidates the terminal if the number of cells to draw is one.
- **Output**:
    - The function does not return a value but updates the terminal display by rendering the specified cell at the calculated position.
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
The `tty_cmd_cells` function handles the drawing of a specified number of cells in a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing the context for drawing cells, including position and attributes.
- **Control Flow**:
    - Checks if the specified cells are visible in the terminal using [`tty_is_visible`](#tty_is_visible).
    - If the cells are not fully visible and the context indicates a larger area, it decides whether to redraw the pane or call a redraw callback.
    - Disables margins with [`tty_margin_off`](#tty_margin_off) and sets the cursor position unless wrapping is required.
    - Sets the attributes for the cells using [`tty_attributes`](#tty_attributes).
    - Calculates the terminal position for overlay checks and checks for any overlays that might obstruct the cells.
    - Iterates through the overlay ranges and draws the visible parts of the cells using [`tty_putn`](#tty_putn).
- **Output**:
    - The function does not return a value but modifies the terminal display by drawing the specified cells at the given position, taking into account visibility and overlays.
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
    - `tty`: A pointer to the `tty` structure representing the terminal.
    - `ctx`: A pointer to the `tty_ctx` structure containing selection parameters such as pointers and length.
- **Control Flow**:
    - Calls the [`tty_set_selection`](#tty_set_selection) function with the terminal pointer and selection parameters from the context.
    - The [`tty_set_selection`](#tty_set_selection) function encodes the selection data and sends it to the terminal.
- **Output**:
    - No return value; the function modifies the terminal's selection state.
- **Functions called**:
    - [`tty_set_selection`](#tty_set_selection)


---
### tty_set_selection<!-- {{#callable:tty_set_selection}} -->
Sets the selection in a terminal by encoding the provided buffer in base64 and sending it to the terminal.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `flags`: A string containing flags that modify the selection behavior.
    - `buf`: A pointer to the buffer containing the data to be selected.
    - `len`: The length of the data in the buffer.
- **Control Flow**:
    - Checks if the terminal has been started; if not, the function returns immediately.
    - Checks if the terminal supports mouse selection; if not, the function returns.
    - Calculates the size needed for the base64 encoded string based on the length of the input buffer.
    - Allocates memory for the encoded string.
    - Encodes the input buffer into base64 format using [`b64_ntop`](compat/base64.c.driver.md#b64_ntop).
    - Sets the terminal to non-blocking mode by updating the `tty` flags.
    - Sends the encoded selection to the terminal using [`tty_putcode_ss`](#tty_putcode_ss).
    - Frees the allocated memory for the encoded string.
- **Output**:
    - The function does not return a value; it modifies the terminal state and sends data to the terminal.
- **Functions called**:
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`b64_ntop`](compat/base64.c.driver.md#b64_ntop)
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
    - Check if the terminal supports sixel graphics; if not, set a fallback flag.
    - Check if the terminal's pixel dimensions are valid; if not, set the fallback flag.
    - Determine the size of the sixel image in cells.
    - Clamp the area where the image will be drawn to ensure it fits within the terminal's visible area.
    - If fallback is triggered, duplicate the fallback image data; otherwise, scale the sixel image to fit the terminal's pixel dimensions.
    - If the image data is valid, log the size and prepare the terminal for drawing by turning off regions and margins, and setting the cursor position.
    - Add the image data to the terminal's output buffer and invalidate the terminal to trigger a redraw.
    - Free any allocated resources for the new image if fallback was not used.
- **Output**:
    - The function does not return a value but modifies the terminal's output buffer to include the image data, which will be displayed in the terminal.
- **Functions called**:
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
    - [`sixel_size_in_cells`](image-sixel.c.driver.md#sixel_size_in_cells)
    - [`tty_clamp_area`](#tty_clamp_area)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`sixel_scale`](image-sixel.c.driver.md#sixel_scale)
    - [`sixel_print`](image-sixel.c.driver.md#sixel_print)
    - [`tty_region_off`](#tty_region_off)
    - [`tty_margin_off`](#tty_margin_off)
    - [`tty_cursor`](#tty_cursor)
    - [`tty_add`](#tty_add)
    - [`tty_invalidate`](#tty_invalidate)
    - [`sixel_free`](image-sixel.c.driver.md#sixel_free)


---
### tty_cmd_syncstart<!-- {{#callable:tty_cmd_syncstart}} -->
The `tty_cmd_syncstart` function initiates synchronized updates for a terminal based on the context provided.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing context information, including the command number.
- **Control Flow**:
    - The function first checks if the `num` field of `ctx` is equal to 0x11, indicating an overlay command.
    - If it is an overlay command, it calls `tty_sync_start(tty)` to begin synchronized updates.
    - If `ctx->num` is not 0x11, it checks if the `num` field is not masked by 0x10, indicating a pane command.
    - For pane commands, it checks if `ctx->num` is non-zero or if there is an overlay drawing request, and if so, it calls `tty_sync_start(tty)`.
- **Output**:
    - The function does not return a value but initiates synchronization for terminal updates based on the conditions checked.
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
    - The function first checks if the terminal is in a state where it should skip writing (e.g., if it's a 'stupid' terminal and the cursor is at the bottom right corner).
    - It then checks if the character is a padding character and returns early if so.
    - The function checks the output codeset and applies the necessary attributes to the character.
    - If the character is a single character, it writes it directly to the terminal using [`tty_putc`](#tty_putc), handling any special character cases.
    - If the character is a string (more than one character), it writes the data using [`tty_putn`](#tty_putn).
- **Output**:
    - The function does not return a value; it directly writes to the terminal output based on the provided character data and attributes.
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
    - Checks if the current cell attributes are different from the default cell attributes.
    - If the current cell has a link, it sends a highlight sequence to the terminal.
    - If the current cell uses a character set and alternate character set is needed, it sends a reset sequence.
    - Sends a reset graphics mode sequence to the terminal.
    - Copies the default cell attributes into the current cell.
    - Copies the default cell attributes into the last cell attributes.
- **Output**:
    - The function does not return a value; it modifies the state of the terminal by resetting its cell attributes.
- **Functions called**:
    - [`grid_cells_equal`](grid.c.driver.md#grid_cells_equal)
    - [`tty_putcode_ss`](#tty_putcode_ss)
    - [`tty_acs_needed`](tty-acs.c.driver.md#tty_acs_needed)
    - [`tty_putcode`](#tty_putcode)


---
### tty_invalidate<!-- {{#callable:tty_invalidate}} -->
The `tty_invalidate` function resets the terminal state and cursor position for a given `tty` structure.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal state to be invalidated.
- **Control Flow**:
    - The function begins by copying the default cell attributes from `grid_default_cell` to the `tty`'s current cell and last cell.
    - It sets the cursor position (`cx`, `cy`) and the region boundaries (`rupper`, `rleft`, `rlower`, `rright`) to `UINT_MAX`, indicating an invalid state.
    - If the `tty` has the `TTY_STARTED` flag set, it performs additional operations to reset the terminal state, including sending escape codes to the terminal.
    - The function updates the mode of the `tty` to `ALL_MODES` and calls [`tty_update_mode`](#tty_update_mode) to apply the cursor mode.
    - Finally, it moves the cursor to the top-left corner of the terminal and disables any active regions or margins.
- **Output**:
    - The function does not return a value; it modifies the state of the `tty` structure and the terminal directly.
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
    - Calls the [`tty_region`](#tty_region) function with parameters 0 and `tty->sy - 1` to define the region from the top to the bottom of the terminal.
- **Output**:
    - This function does not return a value; it modifies the terminal's region settings.
- **Functions called**:
    - [`tty_region`](#tty_region)


---
### tty_region_pane<!-- {{#callable:tty_region_pane}} -->
The `tty_region_pane` function sets the scrolling region for a specific pane in a terminal context.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing context information for the terminal display.
    - `rupper`: An unsigned integer representing the upper boundary of the scrolling region.
    - `rlower`: An unsigned integer representing the lower boundary of the scrolling region.
- **Control Flow**:
    - The function calculates the absolute positions for the upper and lower boundaries of the scrolling region based on the pane's offset and the context's window offset.
    - It then calls the [`tty_region`](#tty_region) function to set the scrolling region using the calculated positions.
- **Output**:
    - The function does not return a value; it modifies the terminal's scrolling region based on the provided parameters.
- **Functions called**:
    - [`tty_region`](#tty_region)


---
### tty_region<!-- {{#callable:tty_region}} -->
The `tty_region` function sets the scrolling region of a terminal, adjusting the cursor position and handling terminal-specific quirks.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `rupper`: An unsigned integer representing the upper boundary of the scrolling region.
    - `rlower`: An unsigned integer representing the lower boundary of the scrolling region.
- **Control Flow**:
    - Check if the current scrolling region matches the requested region; if so, return early.
    - Check if the terminal supports the `CS` (Change Scrolling Region) capability; if not, return.
    - Update the terminal's scrolling region boundaries with the new values.
    - If the cursor is currently beyond the last column, move it to the start of the line or to its current row.
    - Send the command to the terminal to set the new scrolling region using [`tty_putcode_ii`](#tty_putcode_ii).
    - Reset the cursor position to an invalid state.
- **Output**:
    - The function does not return a value; it modifies the terminal's scrolling region and cursor position directly.
- **Functions called**:
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
    - [`tty_cursor`](#tty_cursor)
    - [`tty_putcode_ii`](#tty_putcode_ii)


---
### tty_margin_off<!-- {{#callable:tty_margin_off}} -->
The `tty_margin_off` function disables the margin feature for a given terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal interface whose margin is to be disabled.
- **Control Flow**:
    - The function calls [`tty_margin`](#tty_margin) with the `tty` pointer, setting the left margin to 0 and the right margin to the maximum width of the terminal minus one.
    - This effectively disables any margins that were previously set for the terminal.
- **Output**:
    - The function does not return a value; it modifies the state of the terminal by disabling its margins.
- **Functions called**:
    - [`tty_margin`](#tty_margin)


---
### tty_margin_pane<!-- {{#callable:tty_margin_pane}} -->
The `tty_margin_pane` function sets the margin for a terminal pane based on the provided context.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `ctx`: A pointer to a `struct tty_ctx` containing the context information for the terminal pane.
- **Control Flow**:
    - The function calls [`tty_margin`](#tty_margin) with calculated left and right margin values based on the `ctx` structure.
    - The left margin is calculated as `ctx->xoff - ctx->wox`.
    - The right margin is calculated as `ctx->xoff + ctx->sx - 1 - ctx->wox`.
- **Output**:
    - The function does not return a value; it modifies the terminal's margin settings directly.
- **Functions called**:
    - [`tty_margin`](#tty_margin)


---
### tty_margin<!-- {{#callable:tty_margin}} -->
The `tty_margin` function sets the left and right margins for a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` that represents the terminal context.
    - `rleft`: An unsigned integer representing the new left margin position.
    - `rright`: An unsigned integer representing the new right margin position.
- **Control Flow**:
    - Checks if margins are supported for the terminal using `tty_use_margin(tty)`.
    - If margins are not supported, the function returns immediately.
    - If the current left and right margins are the same as the new values, the function returns.
    - Calls [`tty_putcode_ii`](#tty_putcode_ii) to set the cursor scroll region based on the current upper and lower margins.
    - Updates the `rleft` and `rright` fields of the `tty` structure with the new margin values.
    - If the new margins cover the entire width of the terminal, it sends a command to clear the margins using [`tty_putcode`](#tty_putcode).
    - Otherwise, it sends a command to set the new margins using [`tty_putcode_ii`](#tty_putcode_ii).
    - Finally, it resets the cursor position to an invalid state by setting `cx` and `cy` to `UINT_MAX`.
- **Output**:
    - The function does not return a value but modifies the state of the `tty` structure to reflect the new margin settings.
- **Functions called**:
    - [`tty_putcode_ii`](#tty_putcode_ii)
    - [`tty_putcode`](#tty_putcode)


---
### tty_cursor_pane_unless_wrap<!-- {{#callable:tty_cursor_pane_unless_wrap}} -->
Moves the cursor within a pane unless it would wrap to the next line.
- **Inputs**:
    - `tty`: A pointer to the `tty` structure representing the terminal.
    - `ctx`: A pointer to the `tty_ctx` structure containing context information for the terminal.
    - `cx`: The desired x-coordinate (column) for the cursor position.
    - `cy`: The desired y-coordinate (row) for the cursor position.
- **Control Flow**:
    - Checks if the cursor should wrap based on the `ctx->wrapped` flag.
    - Evaluates several conditions to determine if the cursor can be moved to the specified position without wrapping.
    - If wrapping is not needed, calls [`tty_cursor_pane`](#tty_cursor_pane) to move the cursor.
    - If wrapping is needed, logs a debug message indicating the cursor would wrap.
- **Output**:
    - The function does not return a value; it modifies the cursor position or logs a message based on the conditions evaluated.
- **Functions called**:
    - [`tty_cursor_pane`](#tty_cursor_pane)


---
### tty_cursor_pane<!-- {{#callable:tty_cursor_pane}} -->
Moves the cursor to a specified position within a terminal pane.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal.
    - `ctx`: A pointer to a `tty_ctx` structure containing context information for the terminal pane.
    - `cx`: An unsigned integer representing the x-coordinate (column) to move the cursor to.
    - `cy`: An unsigned integer representing the y-coordinate (row) to move the cursor to.
- **Control Flow**:
    - Calls the [`tty_cursor`](#tty_cursor) function with adjusted coordinates based on the context offsets.
    - Calculates the new cursor position by adding the offsets from the context to the provided coordinates.
- **Output**:
    - The function does not return a value; it directly modifies the cursor position in the terminal.
- **Functions called**:
    - [`tty_cursor`](#tty_cursor)


---
### tty_cursor<!-- {{#callable:tty_cursor}} -->
Moves the cursor to a specified position in a terminal.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal.
    - `cx`: An unsigned integer representing the target x-coordinate (column) for the cursor.
    - `cy`: An unsigned integer representing the target y-coordinate (row) for the cursor.
- **Control Flow**:
    - Checks if the terminal is in a blocked state; if so, it returns immediately.
    - Stores the current cursor position in `thisx` and `thisy`.
    - If the target position is the same as the current position and at the right edge, it returns.
    - If the target x-coordinate exceeds the terminal width, it adjusts it to the maximum width.
    - If the target position is the same as the current position, it returns.
    - If the cursor is at the end of the line, it jumps to absolute movement.
    - Handles special cases for moving to the home position or to the next line.
    - If moving only in the same row, it checks for left or right movements and uses appropriate escape codes.
    - If moving in the same column, it checks for up or down movements and uses appropriate escape codes.
    - If no special movement codes are applicable, it uses absolute movement to set the cursor position.
    - Finally, it updates the current cursor position in the `tty` structure.
- **Output**:
    - The function does not return a value but updates the cursor position in the terminal.
- **Functions called**:
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
    - [`tty_putcode`](#tty_putcode)
    - [`tty_putc`](#tty_putc)
    - [`tty_putcode_i`](#tty_putcode_i)
    - [`tty_putcode_ii`](#tty_putcode_ii)


---
### tty_hyperlink<!-- {{#callable:tty_hyperlink}} -->
The `tty_hyperlink` function updates the hyperlink information for a terminal cell.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `gc`: A pointer to a `struct grid_cell` containing the cell's attributes, including hyperlink information.
    - `hl`: A pointer to a `struct hyperlinks` that manages the hyperlink data.
- **Control Flow**:
    - The function first checks if the hyperlink in the current cell (`gc->link`) is the same as the one stored in the terminal's current cell (`tty->cell.link`). If they are the same, the function returns early.
    - If the hyperlink data (`hl`) is NULL, the function returns early.
    - If the hyperlink ID (`gc->link`) is 0 or if the hyperlink cannot be retrieved from the `hl` structure, the function sends a command to clear the hyperlink using [`tty_putcode_ss`](#tty_putcode_ss).
    - If the hyperlink is valid, the function sends a command to set the hyperlink using [`tty_putcode_ss`](#tty_putcode_ss), passing the hyperlink ID and URI.
- **Output**:
    - The function does not return a value; it modifies the terminal's state by sending commands to update the hyperlink display.
- **Functions called**:
    - [`hyperlinks_get`](hyperlinks.c.driver.md#hyperlinks_get)
    - [`tty_putcode_ss`](#tty_putcode_ss)


---
### tty_attributes<!-- {{#callable:tty_attributes}} -->
The `tty_attributes` function updates the terminal's display attributes based on the provided grid cell attributes.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `gc`: A pointer to a `struct grid_cell` containing the attributes to be applied.
    - `defaults`: A pointer to a `struct grid_cell` representing the default attributes.
    - `palette`: A pointer to a `struct colour_palette` used for color management.
    - `hl`: A pointer to a `struct hyperlinks` for managing hyperlink attributes.
- **Control Flow**:
    - The function begins by copying the attributes from the input `gc` to a local variable `gc2`.
    - It checks if the `GRID_FLAG_NOPALETTE` flag is not set and updates the foreground and background colors if they are set to default values.
    - If the new attributes are the same as the last attributes applied, the function returns early to avoid unnecessary updates.
    - It checks for terminal capabilities and adjusts the attributes accordingly, particularly for reverse colors.
    - The function then verifies and updates the foreground, background, and underline attributes using helper functions.
    - If any attributes are being cleared or the underline color is default, it resets the terminal attributes.
    - The new colors are set, and the function filters out already set attributes before applying the new ones.
    - Finally, it updates the hyperlink attributes if any are present and stores the last applied cell attributes.
- **Output**:
    - The function does not return a value but modifies the terminal's display attributes based on the provided grid cell attributes.
- **Functions called**:
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
    - [`tty_check_fg`](#tty_check_fg)
    - [`tty_check_bg`](#tty_check_bg)
    - [`tty_check_us`](#tty_check_us)
    - [`tty_reset`](#tty_reset)
    - [`tty_colours`](#tty_colours)
    - [`tty_putcode`](#tty_putcode)
    - [`tty_set_italics`](#tty_set_italics)
    - [`tty_putcode_i`](#tty_putcode_i)
    - [`tty_acs_needed`](tty-acs.c.driver.md#tty_acs_needed)
    - [`tty_hyperlink`](#tty_hyperlink)


---
### tty_colours<!-- {{#callable:tty_colours}} -->
The `tty_colours` function updates the foreground, background, and underscore colors of a terminal's current cell based on the provided grid cell attributes.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `gc`: A pointer to a `struct grid_cell` containing the desired color attributes (foreground, background, and underline style).
- **Control Flow**:
    - Checks if the new colors are the same as the current colors; if so, it returns early without making changes.
    - Handles special cases where either the foreground or background color is set to default, potentially resetting both colors to default if necessary.
    - Sets the foreground color if it is not default and differs from the current foreground color.
    - Sets the background color if it is not default and differs from the current background color.
    - Sets the underscore color if it differs from the current underscore color.
- **Output**:
    - The function does not return a value but modifies the terminal's current cell attributes to reflect the new colors.
- **Functions called**:
    - [`tty_term_flag`](tty-term.c.driver.md#tty_term_flag)
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
    - Checks if the `GRID_FLAG_NOPALETTE` flag is not set in the grid cell's flags.
    - If the foreground color is less than 8 and the bright attribute is set, it adjusts the color to a bright variant.
    - Retrieves the color from the palette if available.
    - Checks if the foreground color is a 24-bit RGB color and translates it to a 256-color palette if necessary.
    - Determines the number of colors supported by the terminal.
    - If the foreground color is a 256-color value, it checks if the terminal supports 256 colors and adjusts accordingly.
    - Handles special cases for aixterm colors based on terminal capabilities.
- **Output**:
    - The function modifies the foreground color of the provided grid cell (`gc`) based on the terminal's capabilities and the specified color palette.
- **Functions called**:
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
    - [`colour_palette_get`](colour.c.driver.md#colour_palette_get)
    - [`colour_split_rgb`](colour.c.driver.md#colour_split_rgb)
    - [`colour_find_rgb`](colour.c.driver.md#colour_find_rgb)
    - [`tty_term_number`](tty-term.c.driver.md#tty_term_number)
    - [`colour_256to16`](colour.c.driver.md#colour_256to16)


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
- **Functions called**:
    - [`colour_palette_get`](colour.c.driver.md#colour_palette_get)
    - [`colour_split_rgb`](colour.c.driver.md#colour_split_rgb)
    - [`colour_find_rgb`](colour.c.driver.md#colour_find_rgb)
    - [`tty_term_number`](tty-term.c.driver.md#tty_term_number)
    - [`colour_256to16`](colour.c.driver.md#colour_256to16)


---
### tty_check_us<!-- {{#callable:tty_check_us}} -->
The `tty_check_us` function updates the underline color in a `grid_cell` based on the terminal's capabilities and a provided color palette.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `palette`: A pointer to a `struct colour_palette` used for color substitutions.
    - `gc`: A pointer to a `struct grid_cell` that contains the current cell's attributes including underline color.
- **Control Flow**:
    - The function first checks if the `GRID_FLAG_NOPALETTE` flag is not set in the `gc` structure.
    - If the palette is available, it attempts to retrieve a new underline color using [`colour_palette_get`](colour.c.driver.md#colour_palette_get).
    - If a valid color is found (not -1), it updates the underline color in `gc`.
    - Next, it checks if the terminal supports setting underline colors with `TTYC_SETULC1`.
    - If not supported, it attempts to convert the underline color to RGB using [`colour_force_rgb`](colour.c.driver.md#colour_force_rgb).
    - If the conversion fails (returns -1), it defaults the underline color to 8.
- **Output**:
    - The function does not return a value; it modifies the `gc` structure in place to reflect the updated underline color.
- **Functions called**:
    - [`colour_palette_get`](colour.c.driver.md#colour_palette_get)
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
    - [`colour_force_rgb`](colour.c.driver.md#colour_force_rgb)


---
### tty_colours_fg<!-- {{#callable:tty_colours_fg}} -->
The `tty_colours_fg` function sets the foreground color of a terminal's text output based on the provided color specifications.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `gc`: A pointer to a `struct grid_cell` containing the new foreground color information.
- **Control Flow**:
    - Checks if the current foreground color is a bright aixterm color and resets if the new color is not bright.
    - Determines if the new color is a 24-bit or 256-color by checking specific flags.
    - If the color is 24-bit or 256-color, attempts to set it using [`tty_try_colour`](#tty_try_colour).
    - If the color is a bright aixterm color, it sets the color using terminal-specific codes.
    - If the color is neither bright nor 24-bit/256-color, it sets the foreground color using a standard terminal code.
    - Finally, it saves the new foreground color in the terminal's current cell.
- **Output**:
    - The function does not return a value but updates the terminal's foreground color and the current cell's foreground color.
- **Functions called**:
    - [`tty_reset`](#tty_reset)
    - [`tty_try_colour`](#tty_try_colour)
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)
    - [`tty_puts`](#tty_puts)
    - [`tty_putcode_i`](#tty_putcode_i)


---
### tty_colours_bg<!-- {{#callable:tty_colours_bg}} -->
Sets the background color of a terminal cell based on the provided grid cell attributes.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `gc`: A pointer to a `struct grid_cell` containing the color attributes to be set.
- **Control Flow**:
    - Checks if the background color is a 24-bit or 256-color value using bitwise operations.
    - If it is a 24-bit or 256-color, attempts to set the color using [`tty_try_colour`](#tty_try_colour) and saves the new value if successful.
    - If the background color is an aixterm bright color (90-97), it sets the color accordingly based on terminal capabilities.
    - For other colors, it directly sets the background color using [`tty_putcode_i`](#tty_putcode_i).
    - Finally, it saves the new background color in the terminal's current cell.
- **Output**:
    - The function does not return a value but updates the terminal's background color and saves it in the current cell.
- **Functions called**:
    - [`tty_try_colour`](#tty_try_colour)
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)
    - [`tty_puts`](#tty_puts)
    - [`tty_putcode_i`](#tty_putcode_i)


---
### tty_colours_us<!-- {{#callable:tty_colours_us}} -->
The `tty_colours_us` function sets the underline color for a terminal's text display.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `gc`: A pointer to a `struct grid_cell` containing the color information for the underline.
- **Control Flow**:
    - Checks if the underline color is set to default; if so, it sends a command to clear the underline color.
    - If the underline color is not an RGB color, it adjusts the color value and sends a command to set the underline color.
    - If the underline color is an RGB color, it splits the RGB values and calculates the color value, then sends the appropriate command to set the underline color.
    - Finally, it saves the new underline color value in the terminal's current cell.
- **Output**:
    - The function does not return a value but modifies the terminal's display by setting the underline color based on the provided `grid_cell` structure.
- **Functions called**:
    - [`tty_putcode`](#tty_putcode)
    - [`tty_putcode_i`](#tty_putcode_i)
    - [`colour_split_rgb`](colour.c.driver.md#colour_split_rgb)
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)


---
### tty_try_colour<!-- {{#callable:tty_try_colour}} -->
The `tty_try_colour` function attempts to set the terminal's foreground or background color based on the provided color value and type.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
    - `colour`: An integer representing the color value, which can indicate RGB or 256-color formats.
    - `type`: A string indicating the type of color setting, where '3' denotes foreground color.
- **Control Flow**:
    - Checks if the `colour` has the `COLOUR_FLAG_256` flag set.
    - If true, it checks if the `type` is '3' and if the terminal supports setting foreground color; if so, it sets the foreground color.
    - If the terminal does not support setting foreground color, it checks for background color support and sets it if available.
    - If the `colour` has the `COLOUR_FLAG_RGB` flag set, it splits the RGB components and attempts to set the foreground or background color based on the terminal's capabilities.
    - If neither color setting is successful, it returns -1.
- **Output**:
    - Returns 0 on success when a color is set, or -1 if the color setting is not supported.
- **Functions called**:
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
    - [`tty_putcode_i`](#tty_putcode_i)
    - [`colour_split_rgb`](colour.c.driver.md#colour_split_rgb)
    - [`tty_putcode_iii`](#tty_putcode_iii)


---
### tty_window_default_style<!-- {{#callable:tty_window_default_style}} -->
The `tty_window_default_style` function initializes a `grid_cell` structure with default styling based on the associated `window_pane`.
- **Inputs**:
    - `gc`: A pointer to a `grid_cell` structure that will be modified to reflect the default style.
    - `wp`: A pointer to a `window_pane` structure that contains the palette information used to set the foreground and background colors.
- **Control Flow**:
    - The function begins by copying the contents of `grid_default_cell` into the provided `grid_cell` structure `gc` using `memcpy`.
    - It then sets the foreground color of `gc` to the foreground color defined in the `palette` of the `window_pane` `wp`.
    - Similarly, it sets the background color of `gc` to the background color defined in the `palette` of `wp`.
- **Output**:
    - The function does not return a value; instead, it modifies the `grid_cell` structure pointed to by `gc` to reflect the default style, including the foreground and background colors from the `window_pane`'s palette.


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
    - The function modifies the `gc` structure in place to reflect the default colors for the terminal pane, based on the current state and options of the pane.
- **Functions called**:
    - [`format_create`](format.c.driver.md#format_create)
    - [`format_defaults`](format.c.driver.md#format_defaults)
    - [`tty_window_default_style`](#tty_window_default_style)
    - [`style_add`](style.c.driver.md#style_add)
    - [`format_free`](format.c.driver.md#format_free)


---
### tty_default_attributes<!-- {{#callable:tty_default_attributes}} -->
Sets the default attributes for a terminal's tty context.
- **Inputs**:
    - `tty`: A pointer to the `tty` structure representing the terminal.
    - `defaults`: A pointer to a `grid_cell` structure containing default cell attributes.
    - `palette`: A pointer to a `colour_palette` structure for color management.
    - `bg`: An unsigned integer representing the background color.
    - `hl`: A pointer to a `hyperlinks` structure for managing hyperlink attributes.
- **Control Flow**:
    - Creates a temporary `grid_cell` structure `gc` and initializes it with the default cell attributes.
    - Sets the background color of `gc` to the provided `bg` value.
    - Calls the [`tty_attributes`](#tty_attributes) function to apply the attributes defined in `gc`, along with the provided `defaults`, `palette`, and `hl`.
- **Output**:
    - The function does not return a value; it modifies the terminal's attributes based on the provided parameters.
- **Functions called**:
    - [`tty_attributes`](#tty_attributes)


---
### tty_clipboard_query_callback<!-- {{#callable:tty_clipboard_query_callback}} -->
The `tty_clipboard_query_callback` function handles the callback for clipboard query events in a terminal context.
- **Inputs**:
    - `fd`: An unused file descriptor, typically representing the terminal.
    - `events`: An unused short representing event types that triggered the callback.
    - `data`: A pointer to a `tty` structure containing terminal-related data.
- **Control Flow**:
    - The function retrieves the `tty` structure from the `data` pointer.
    - It accesses the associated `client` structure from the `tty` structure.
    - The `CLIENT_CLIPBOARDBUFFER` flag is cleared from the client's flags, indicating that the clipboard buffer is no longer active.
    - The clipboard panes associated with the client are freed and set to NULL.
    - The number of clipboard panes is reset to zero.
    - The `TTY_OSC52QUERY` flag is cleared from the `tty` structure, indicating that the clipboard query has been processed.
- **Output**:
    - The function does not return a value; it modifies the state of the `client` and `tty` structures to reflect the completion of a clipboard query.


---
### tty_clipboard_query<!-- {{#callable:tty_clipboard_query}} -->
The `tty_clipboard_query` function initiates a clipboard query for a terminal interface.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` representing the terminal context.
- **Control Flow**:
    - Check if the terminal has started and if a clipboard query is already in progress; if not, proceed.
    - Send a clipboard query command to the terminal using [`tty_putcode_ss`](#tty_putcode_ss).
    - Set the `TTY_OSC52QUERY` flag to indicate that a clipboard query is in progress.
    - Set a timer for the clipboard query callback using `evtimer_set` and `evtimer_add` with a timeout defined by `TTY_QUERY_TIMEOUT`.
- **Output**:
    - The function does not return a value but sets up a timer for handling the clipboard query response and modifies the state of the `tty` structure.
- **Functions called**:
    - [`tty_putcode_ss`](#tty_putcode_ss)


