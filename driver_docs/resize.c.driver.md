# Purpose
This C source code file is part of a larger application, likely a terminal multiplexer like `tmux`, which manages window resizing and client interactions. The primary functionality of this file is to handle the resizing of windows based on various criteria, such as the size of attached clients, user-defined options, and specific window states. The code includes functions to resize individual windows ([`resize_window`](#resize_window)), calculate the appropriate size for windows based on client data ([`clients_calculate_size`](#clients_calculate_size)), and adjust window sizes across sessions ([`recalculate_sizes_now`](#recalculate_sizes_now)). It also manages window zoom states and ensures that size constraints are respected, such as minimum and maximum window dimensions.

The file contains several static helper functions that determine whether a client's size should be considered ([`ignore_client_size`](#ignore_client_size)) and calculate the number of clients associated with a window ([`clients_with_window`](#clients_with_window)). The code is structured to support different resizing strategies, such as using the largest, smallest, or latest client size, and it includes mechanisms to handle manual size settings. The functions are designed to be integrated into a larger system, as indicated by the use of external structures and functions like `layout_resize`, `window_resize`, and `log_debug`. This file does not define a public API but rather provides internal functionality for managing window sizes within the application.
# Imports and Dependencies

---
- `sys/types.h`
- `string.h`
- `tmux.h`


# Functions

---
### resize_window<!-- {{#callable:resize_window}} -->
The `resize_window` function adjusts the size of a given window, ensuring it adheres to specified minimum and maximum constraints, and updates the window's layout and state accordingly.
- **Inputs**:
    - `w`: A pointer to the `window` structure that represents the window to be resized.
    - `sx`: An unsigned integer representing the desired width of the window.
    - `sy`: An unsigned integer representing the desired height of the window.
    - `xpixel`: An integer representing the pixel width for the window, used for high-resolution displays.
    - `ypixel`: An integer representing the pixel height for the window, used for high-resolution displays.
- **Control Flow**:
    - Check if the desired width (`sx`) and height (`sy`) are within the defined minimum and maximum limits, adjusting them if necessary.
    - Determine if the window is currently zoomed and unzoom it if so.
    - Resize the window's layout to the specified dimensions (`sx`, `sy`).
    - Ensure the window's size is not smaller than its layout's root size, adjusting `sx` and `sy` if needed.
    - Call [`window_resize`](window.c.driver.md#window_resize) to apply the new size to the window, including pixel dimensions.
    - Log the resizing operation for debugging purposes.
    - If the window was previously zoomed, restore its zoom state.
    - Update the window's offset and redraw the window on the server.
    - Notify any listeners that the window's layout and size have changed.
    - Clear the `WINDOW_RESIZE` flag from the window's flags.
- **Output**:
    - The function does not return a value; it modifies the state of the `window` structure pointed to by `w`.
- **Functions called**:
    - [`window_unzoom`](window.c.driver.md#window_unzoom)
    - [`layout_resize`](layout.c.driver.md#layout_resize)
    - [`window_resize`](window.c.driver.md#window_resize)
    - [`window_zoom`](window.c.driver.md#window_zoom)
    - [`tty_update_window_offset`](tty.c.driver.md#tty_update_window_offset)
    - [`server_redraw_window`](server-fn.c.driver.md#server_redraw_window)
    - [`notify_window`](notify.c.driver.md#notify_window)


---
### ignore_client_size<!-- {{#callable:ignore_client_size}} -->
The `ignore_client_size` function determines whether a client's size should be ignored based on various conditions related to the client's session and flags.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client whose size is being evaluated.
- **Control Flow**:
    - Check if the client's session is NULL; if so, return 1 to ignore the size.
    - Check if the client's flags include `CLIENT_NOSIZEFLAGS`; if so, return 1 to ignore the size.
    - If the client's flags include `CLIENT_IGNORESIZE`, iterate over all clients to check if any attached client does not have the `CLIENT_IGNORESIZE` flag; if found, return 1 to ignore the size.
    - Check if the client's flags include `CLIENT_CONTROL` and do not include `CLIENT_SIZECHANGED` or `CLIENT_WINDOWSIZECHANGED`; if so, return 1 to ignore the size.
    - If none of the above conditions are met, return 0 to not ignore the size.
- **Output**:
    - Returns an integer value: 1 if the client's size should be ignored, otherwise 0.


---
### clients_with_window<!-- {{#callable:clients_with_window}} -->
The `clients_with_window` function counts the number of clients that are associated with a given window and are not ignored due to size constraints.
- **Inputs**:
    - `w`: A pointer to a `struct window` representing the window for which the function checks client associations.
- **Control Flow**:
    - Initialize a counter `n` to zero.
    - Iterate over each client in the global `clients` list using `TAILQ_FOREACH`.
    - For each client, check if it should be ignored using [`ignore_client_size`](#ignore_client_size) or if its session does not have the window `w` using [`session_has`](session.c.driver.md#session_has).
    - If the client is not ignored and its session has the window, increment the counter `n`.
    - If the counter `n` exceeds 1, break out of the loop early.
    - Return the counter `n`.
- **Output**:
    - The function returns an unsigned integer representing the number of clients associated with the given window that are not ignored due to size constraints.
- **Functions called**:
    - [`ignore_client_size`](#ignore_client_size)
    - [`session_has`](session.c.driver.md#session_has)


---
### clients_calculate_size<!-- {{#callable:clients_calculate_size}} -->
The `clients_calculate_size` function determines the optimal size for a window based on various criteria and client configurations.
- **Inputs**:
    - `type`: An integer representing the type of size calculation to perform, such as WINDOW_SIZE_LARGEST, WINDOW_SIZE_MANUAL, or WINDOW_SIZE_LATEST.
    - `current`: An integer flag indicating whether to consider only the current window for size calculation.
    - `c`: A pointer to the current client structure, which may be excluded from certain calculations.
    - `s`: A pointer to the session structure associated with the window.
    - `w`: A pointer to the window structure for which the size is being calculated.
    - `skip_client`: A function pointer used to determine if a client should be skipped during size calculation.
    - `sx`: A pointer to an unsigned integer where the calculated width of the window will be stored.
    - `sy`: A pointer to an unsigned integer where the calculated height of the window will be stored.
    - `xpixel`: A pointer to an unsigned integer where the calculated x pixel size will be stored.
    - `ypixel`: A pointer to an unsigned integer where the calculated y pixel size will be stored.
- **Control Flow**:
    - Initialize size variables based on the type of size calculation requested (largest, manual, or smallest/latest).
    - If the type is WINDOW_SIZE_LATEST, count the number of clients with the window to determine if there are multiple clients attached.
    - Skip size calculation if the type is WINDOW_SIZE_MANUAL.
    - Iterate over all clients, skipping those that should be ignored or skipped based on the provided function pointer.
    - For each client, determine the size based on per-window size or the client's terminal size, adjusting for the status line.
    - Update the size variables if the current client's size is larger or smaller than the best size found so far, depending on the type of calculation.
    - Clamp the size to not exceed any per-client window size if one exists.
    - Log the calculated size and return whether a suitable size was found based on the type of calculation.
- **Output**:
    - Returns an integer indicating whether a suitable size was found, with specific conditions for each type of size calculation.
- **Functions called**:
    - [`clients_with_window`](#clients_with_window)
    - [`ignore_client_size`](#ignore_client_size)
    - [`server_client_get_client_window`](server-client.c.driver.md#server_client_get_client_window)
    - [`status_line_size`](status.c.driver.md#status_line_size)


---
### default_window_size_skip_client<!-- {{#callable:default_window_size_skip_client}} -->
The `default_window_size_skip_client` function determines whether a client should be skipped based on its session and window association.
- **Inputs**:
    - `loop`: A pointer to a `client` structure representing the client being evaluated.
    - `type`: An unused integer parameter, typically representing the type of size calculation.
    - `current`: An unused integer parameter, typically representing the current state or flag.
    - `s`: A pointer to a `session` structure representing the session to compare against.
    - `w`: A pointer to a `window` structure representing the window to check for association with the client's session.
- **Control Flow**:
    - Check if the window `w` is not NULL and if the client's session does not have the window `w`, return 1 to indicate the client should be skipped.
    - If the window `w` is NULL and the client's session is not the same as the session `s`, return 1 to indicate the client should be skipped.
    - If neither condition is met, return 0 to indicate the client should not be skipped.
- **Output**:
    - Returns an integer value: 1 if the client should be skipped, 0 otherwise.
- **Functions called**:
    - [`session_has`](session.c.driver.md#session_has)


---
### default_window_size<!-- {{#callable:default_window_size}} -->
The `default_window_size` function determines and sets the default size for a window based on client, session, and window parameters, while ensuring the size adheres to predefined limits.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the window size is being determined.
    - `s`: A pointer to a `session` structure representing the session associated with the window.
    - `w`: A pointer to a `window` structure representing the window whose size is being set.
    - `sx`: A pointer to an unsigned integer where the calculated width of the window will be stored.
    - `sy`: A pointer to an unsigned integer where the calculated height of the window will be stored.
    - `xpixel`: A pointer to an unsigned integer where the calculated pixel width of the window will be stored.
    - `ypixel`: A pointer to an unsigned integer where the calculated pixel height of the window will be stored.
    - `type`: An integer representing the type of window size calculation to perform, with specific values indicating different sizing strategies.
- **Control Flow**:
    - If the `type` is -1, it retrieves the window size type from global options.
    - If the `type` is `WINDOW_SIZE_LATEST` and the client is valid, it uses the client's terminal size, adjusting for the status line, and logs the size.
    - If the client is a control client, it is ignored by setting it to NULL.
    - The function attempts to calculate the size based on available clients using [`clients_calculate_size`](#clients_calculate_size). If unsuccessful, it defaults to a predefined size from session options or a hardcoded default of 80x24.
    - The function ensures the calculated size does not exceed or fall below predefined minimum and maximum limits.
    - Logs the resulting size after adjustments.
- **Output**:
    - The function outputs the calculated window size by modifying the values pointed to by `sx`, `sy`, `xpixel`, and `ypixel`.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`ignore_client_size`](#ignore_client_size)
    - [`status_line_size`](status.c.driver.md#status_line_size)
    - [`clients_calculate_size`](#clients_calculate_size)
    - [`options_get_string`](options.c.driver.md#options_get_string)


---
### recalculate_size_skip_client<!-- {{#callable:recalculate_size_skip_client}} -->
The `recalculate_size_skip_client` function determines whether a client should be skipped during window size recalculation based on the current window and session conditions.
- **Inputs**:
    - `loop`: A pointer to a `client` structure representing the client being evaluated.
    - `type`: An unused integer parameter, typically used for specifying the type of operation or condition.
    - `current`: An integer flag indicating whether the current window should be considered (used for aggressive-resize).
    - `s`: An unused pointer to a `session` structure, representing the session context.
    - `w`: A pointer to a `window` structure representing the window being evaluated.
- **Control Flow**:
    - Check if the current window in the client's session is NULL; if so, return 1 to skip the client.
    - If the `current` flag is set, check if the client's current window is not the specified window `w`; if true, return 1 to skip the client.
    - If the `current` flag is not set, check if the session does not contain the window `w` using [`session_has`](session.c.driver.md#session_has); if true, return 1 to skip the client.
    - If none of the above conditions are met, return 0 to indicate the client should not be skipped.
- **Output**:
    - Returns an integer value (1 or 0) indicating whether the client should be skipped (1) or not (0) during window size recalculation.
- **Functions called**:
    - [`session_has`](session.c.driver.md#session_has)


---
### recalculate_size<!-- {{#callable:recalculate_size}} -->
The `recalculate_size` function adjusts the size of a window based on client preferences and current conditions, either immediately or by scheduling a resize.
- **Inputs**:
    - `w`: A pointer to the `window` structure that represents the window to be resized.
    - `now`: An integer flag indicating whether the resize should occur immediately (non-zero) or be scheduled for later (zero).
- **Control Flow**:
    - Check if the window has an active pane; if not, exit the function as the window is likely being destroyed.
    - Log the current size of the window for debugging purposes.
    - Retrieve the window size type and aggressive-resize option from the window's options.
    - Calculate the new size of the window by calling [`clients_calculate_size`](#clients_calculate_size), which considers client preferences and conditions.
    - Determine if the size has actually changed by comparing the new size with the current or scheduled size, depending on the `now` flag and existing resize flags.
    - If the size hasn't changed, update the window offset and exit the function.
    - If the `now` flag is set or the window size type is manual, resize the window immediately using [`resize_window`](#resize_window).
    - If resizing is not immediate, store the new size in the window's structure and set a flag to indicate a resize is scheduled, then update the window offset.
- **Output**:
    - The function does not return a value; it modifies the window's size and state directly.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`clients_calculate_size`](#clients_calculate_size)
    - [`tty_update_window_offset`](tty.c.driver.md#tty_update_window_offset)
    - [`resize_window`](#resize_window)


---
### recalculate_sizes<!-- {{#callable:recalculate_sizes}} -->
The `recalculate_sizes` function triggers a recalculation of window sizes by calling [`recalculate_sizes_now`](#recalculate_sizes_now) with a parameter of 0.
- **Inputs**:
    - None
- **Control Flow**:
    - The function `recalculate_sizes` is called.
    - It calls another function [`recalculate_sizes_now`](#recalculate_sizes_now) with an argument of 0, indicating that the recalculation should not be immediate.
- **Output**:
    - This function does not return any value; it initiates a process to recalculate window sizes.
- **Functions called**:
    - [`recalculate_sizes_now`](#recalculate_sizes_now)


---
### recalculate_sizes_now<!-- {{#callable:recalculate_sizes_now}} -->
The `recalculate_sizes_now` function updates the attached client count for each session, adjusts the status line visibility for each client, and recalculates the size of each window based on the current state.
- **Inputs**:
    - `now`: An integer flag indicating whether the size adjustment should be applied immediately (non-zero) or deferred (zero).
- **Control Flow**:
    - Iterate over all sessions using `RB_FOREACH` to reset the attached client count and update the status line cache for each session.
    - Iterate over all clients using `TAILQ_FOREACH` to increment the attached count for each session and adjust the status line visibility based on client size and flags.
    - Iterate over all windows using `RB_FOREACH` to call [`recalculate_size`](#recalculate_size) for each window, passing the `now` parameter to determine immediate or deferred resizing.
- **Output**:
    - The function does not return a value; it performs updates on session, client, and window structures in place.
- **Functions called**:
    - [`status_update_cache`](status.c.driver.md#status_update_cache)
    - [`ignore_client_size`](#ignore_client_size)
    - [`recalculate_size`](#recalculate_size)


