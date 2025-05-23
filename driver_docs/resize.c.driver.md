# Purpose
This C source code file is part of a larger application, likely a terminal multiplexer like `tmux`, which manages window resizing and client interactions. The primary functionality of this file is to handle the resizing of windows within a session, taking into account various constraints and client-specific settings. The code includes functions to resize windows ([`resize_window`](#resize_window)), calculate the appropriate size for windows based on client preferences ([`clients_calculate_size`](#clients_calculate_size)), and adjust window sizes dynamically ([`recalculate_size`](#recalculate_size) and [`recalculate_sizes_now`](#recalculate_sizes_now)). It ensures that windows adhere to minimum and maximum size constraints and considers the state of clients, such as whether they are in control mode or have specific size flags.

The file is not a standalone executable but rather a component of a larger system, likely intended to be compiled and linked with other parts of the application. It does not define public APIs or external interfaces directly but provides internal functionality crucial for managing window layouts and client interactions. The code is structured to handle multiple clients and sessions, ensuring that window sizes are adjusted according to the current state of the application and user preferences. The use of logging and debug statements throughout the code suggests that it is designed to facilitate troubleshooting and ensure that window resizing operations are performed correctly.
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
    - `w`: A pointer to the `window` structure representing the window to be resized.
    - `sx`: An unsigned integer representing the desired width of the window.
    - `sy`: An unsigned integer representing the desired height of the window.
    - `xpixel`: An integer representing the pixel width for the window, used for high-resolution displays.
    - `ypixel`: An integer representing the pixel height for the window, used for high-resolution displays.
- **Control Flow**:
    - Check if the desired width (`sx`) and height (`sy`) are within the defined minimum and maximum limits, adjusting them if necessary.
    - Determine if the window is currently zoomed and unzoom it if so.
    - Resize the window's layout to the new dimensions (`sx`, `sy`).
    - Ensure the window's size is not smaller than its layout's root size, adjusting `sx` and `sy` if needed.
    - Call `window_resize` to apply the new size to the window, including pixel dimensions.
    - Log the resizing operation for debugging purposes.
    - If the window was previously zoomed, restore its zoom state.
    - Update the window's offset for the terminal display.
    - Redraw the window on the server to reflect the new size.
    - Notify any listeners that the window's layout and size have changed.
    - Clear the `WINDOW_RESIZE` flag from the window's flags.
- **Output**:
    - The function does not return a value; it modifies the state of the `window` structure and related components.


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
    - If none of the above conditions are met, return 0 to indicate the client's size should not be ignored.
- **Output**:
    - Returns an integer value: 1 if the client's size should be ignored, or 0 if it should not be ignored.


---
### clients_with_window<!-- {{#callable:clients_with_window}} -->
The `clients_with_window` function counts the number of clients that are associated with a given window and are not ignored due to size constraints.
- **Inputs**:
    - `w`: A pointer to a `struct window` representing the window for which the client count is being determined.
- **Control Flow**:
    - Initialize a counter `n` to zero to keep track of the number of clients associated with the window.
    - Iterate over each client in the global `clients` list using the `TAILQ_FOREACH` macro.
    - For each client, check if it should be ignored using the [`ignore_client_size`](#ignore_client_size) function or if its session does not have the specified window using `session_has`. If either condition is true, skip to the next client.
    - Increment the counter `n` for each client that is not ignored and is associated with the window.
    - If the counter `n` exceeds 1, break out of the loop early.
    - Return the counter `n` as the result.
- **Output**:
    - The function returns an unsigned integer representing the number of clients associated with the specified window that are not ignored due to size constraints.
- **Functions called**:
    - [`ignore_client_size`](#ignore_client_size)


---
### clients_calculate_size<!-- {{#callable:clients_calculate_size}} -->
The `clients_calculate_size` function determines the optimal size for a window based on various criteria and client configurations.
- **Inputs**:
    - `type`: An integer representing the type of size calculation to perform, such as WINDOW_SIZE_LARGEST, WINDOW_SIZE_MANUAL, or WINDOW_SIZE_LATEST.
    - `current`: An integer flag indicating whether to consider only the current window for size calculation.
    - `c`: A pointer to the current client structure, which may be excluded from size calculations.
    - `s`: A pointer to the current session structure.
    - `w`: A pointer to the window structure for which the size is being calculated.
    - `skip_client`: A function pointer used to determine if a client should be skipped during size calculations.
    - `sx`: A pointer to an unsigned integer where the calculated width of the window will be stored.
    - `sy`: A pointer to an unsigned integer where the calculated height of the window will be stored.
    - `xpixel`: A pointer to an unsigned integer where the calculated x-pixel size will be stored.
    - `ypixel`: A pointer to an unsigned integer where the calculated y-pixel size will be stored.
- **Control Flow**:
    - Initialize size variables based on the type of calculation (largest, manual, or smallest/latest).
    - If the type is WINDOW_SIZE_LATEST, count the number of clients with the window to determine if there are multiple clients.
    - If the type is WINDOW_SIZE_MANUAL, skip the size calculation loop and use the manual size directly.
    - Iterate over all clients, skipping those that should be ignored or skipped based on the provided function pointer.
    - For each client, determine the size based on per-window size or default client size, adjusting the calculated size if it is larger or smaller than the current best size.
    - Clamp the calculated size to not exceed any per-client window size if applicable.
    - Log the calculated size and return whether a suitable size was found based on the type of calculation.
- **Output**:
    - Returns an integer indicating whether a suitable size was found, with specific conditions for each type of size calculation.
- **Functions called**:
    - [`clients_with_window`](#clients_with_window)
    - [`ignore_client_size`](#ignore_client_size)


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
    - Returns an integer value: 1 if the client should be skipped, otherwise 0.


---
### default_window_size<!-- {{#callable:default_window_size}} -->
The `default_window_size` function determines and sets the default size for a window based on client, session, and window parameters, while ensuring the size adheres to predefined limits.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the window size is being determined.
    - `s`: A pointer to a `session` structure representing the session associated with the window.
    - `w`: A pointer to a `window` structure representing the window whose size is being set.
    - `sx`: A pointer to an unsigned integer where the calculated width of the window will be stored.
    - `sy`: A pointer to an unsigned integer where the calculated height of the window will be stored.
    - `xpixel`: A pointer to an unsigned integer where the calculated x-pixel size of the window will be stored.
    - `ypixel`: A pointer to an unsigned integer where the calculated y-pixel size of the window will be stored.
    - `type`: An integer representing the type of window size calculation to perform, with specific values indicating different sizing strategies.
- **Control Flow**:
    - If `type` is -1, retrieve the window size type from global options.
    - If `type` is `WINDOW_SIZE_LATEST` and the client `c` is valid and not ignored, set the window size based on the client's terminal size and adjust for the status line, then log and exit.
    - If the client `c` is a control client, set `c` to NULL to ignore it for size calculation.
    - Call [`clients_calculate_size`](#clients_calculate_size) to determine the window size based on available clients, session, and window, using a skip function to filter clients.
    - If no suitable size is found, retrieve the default size from session options or use a fallback size of 80x24, then log the chosen size.
    - Ensure the calculated size is within predefined minimum and maximum limits, adjusting if necessary, and log the final size.
- **Output**:
    - The function outputs the calculated window size by modifying the values pointed to by `sx`, `sy`, `xpixel`, and `ypixel`.
- **Functions called**:
    - [`ignore_client_size`](#ignore_client_size)
    - [`clients_calculate_size`](#clients_calculate_size)


---
### recalculate_size_skip_client<!-- {{#callable:recalculate_size_skip_client}} -->
The `recalculate_size_skip_client` function determines whether a client should be skipped during window size recalculation based on whether the window is the current window or part of the session.
- **Inputs**:
    - `loop`: A pointer to a `client` structure representing the client being evaluated.
    - `type`: An unused integer parameter, typically used for type specification in similar functions.
    - `current`: An integer flag indicating whether the current window should be considered (aggressive-resize).
    - `s`: An unused pointer to a `session` structure, typically representing the session context.
    - `w`: A pointer to a `window` structure representing the window being checked.
- **Control Flow**:
    - Check if the current window in the client's session is NULL; if so, return 1 to skip the client.
    - If the `current` flag is set, check if the current window in the client's session is not the window `w`; if true, return 1 to skip the client.
    - If the `current` flag is not set, check if the session does not contain the window `w` using `session_has`; if true, return 1 to skip the client.
    - Return 0 if none of the conditions for skipping the client are met.
- **Output**:
    - Returns an integer value, 1 if the client should be skipped, otherwise 0.


---
### recalculate_size<!-- {{#callable:recalculate_size}} -->
The `recalculate_size` function adjusts the size of a window based on client configurations and window options, either immediately or by scheduling a resize.
- **Inputs**:
    - `w`: A pointer to the `window` structure that represents the window to be resized.
    - `now`: An integer flag indicating whether the resize should occur immediately (non-zero) or be scheduled for later (zero).
- **Control Flow**:
    - Check if the window has an active pane; if not, exit the function as the window is likely being destroyed.
    - Log the current size of the window for debugging purposes.
    - Retrieve the window size type and aggressive-resize option from the window's options.
    - Calculate the new size of the window by calling [`clients_calculate_size`](#clients_calculate_size), which considers client configurations and window options.
    - Determine if the size has changed by comparing the new size with the current or scheduled size, depending on whether a resize is already scheduled.
    - If the size hasn't changed, update the window offset and exit the function.
    - If the `now` flag is set or the window size type is manual, resize the window immediately using [`resize_window`](#resize_window).
    - If not resizing immediately, store the new size in the window structure and set a flag to indicate a resize is scheduled, then update the window offset.
- **Output**:
    - The function does not return a value but modifies the `window` structure to reflect the new size or schedule a resize.
- **Functions called**:
    - [`clients_calculate_size`](#clients_calculate_size)
    - [`resize_window`](#resize_window)


---
### recalculate_sizes<!-- {{#callable:recalculate_sizes}} -->
The `recalculate_sizes` function triggers the recalculation of window sizes by calling [`recalculate_sizes_now`](#recalculate_sizes_now) with a parameter of 0.
- **Inputs**:
    - None
- **Control Flow**:
    - The function `recalculate_sizes` is called without any parameters.
    - Inside `recalculate_sizes`, the function [`recalculate_sizes_now`](#recalculate_sizes_now) is invoked with the argument `0`.
- **Output**:
    - The function does not return any value; it initiates a process to recalculate window sizes.
- **Functions called**:
    - [`recalculate_sizes_now`](#recalculate_sizes_now)


---
### recalculate_sizes_now<!-- {{#callable:recalculate_sizes_now}} -->
The `recalculate_sizes_now` function updates the attached client count for each session, adjusts the status line visibility for each client, and recalculates the size of each window based on the current state.
- **Inputs**:
    - `now`: An integer flag indicating whether the size recalculation should be applied immediately (non-zero) or deferred (zero).
- **Control Flow**:
    - Iterate over all sessions using `RB_FOREACH` to reset the attached client count to zero and update the status line cache for each session.
    - Iterate over all clients using `TAILQ_FOREACH` to increment the attached count for each session if the client is attached and not flagged as unattached.
    - For each client, check if the client size should be ignored; if not, adjust the client's status line visibility based on the terminal size and client flags.
    - Iterate over all windows using `RB_FOREACH` and call [`recalculate_size`](#recalculate_size) for each window, passing the `now` parameter to determine immediate or deferred resizing.
- **Output**:
    - The function does not return a value; it performs updates on session, client, and window structures in place.
- **Functions called**:
    - [`ignore_client_size`](#ignore_client_size)
    - [`recalculate_size`](#recalculate_size)


