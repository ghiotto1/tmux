# Purpose
This C source file is part of the tmux terminal multiplexer, specifically handling server-client interactions. It provides a broad range of functionalities related to managing client sessions, handling input events, and maintaining the state of the terminal interface. The file includes functions for managing client connections, processing input events (such as key presses and mouse actions), and updating the display based on changes in the terminal state or user interactions.

Key technical components include functions for handling mouse events, managing client overlays, and processing key bindings. The file defines several static functions for internal operations, such as resizing panes, checking for client exits, and updating client modes. It also includes functions for setting client-specific attributes like titles and paths, and for managing client-specific data structures like client windows and panes. The file is designed to be part of the tmux server, interacting with clients to provide a responsive and dynamic terminal multiplexing environment. It does not define public APIs or external interfaces directly, but rather implements the internal logic necessary for tmux's server-client communication.
# Imports and Dependencies

---
- `sys/types.h`
- `sys/ioctl.h`
- `sys/uio.h`
- `errno.h`
- `fcntl.h`
- `stdlib.h`
- `string.h`
- `time.h`
- `unistd.h`
- `tmux.h`


# Data Structures

---
### mouse_where
- **Type**: `enum`
- **Members**:
    - `NOWHERE`: Represents a location where the mouse is not over any specific UI element.
    - `PANE`: Indicates the mouse is over a pane.
    - `STATUS`: Indicates the mouse is over the status line.
    - `STATUS_LEFT`: Indicates the mouse is over the left part of the status line.
    - `STATUS_RIGHT`: Indicates the mouse is over the right part of the status line.
    - `STATUS_DEFAULT`: Indicates the mouse is over the default part of the status line.
    - `BORDER`: Indicates the mouse is over a border.
    - `SCROLLBAR_UP`: Indicates the mouse is over the up arrow of a scrollbar.
    - `SCROLLBAR_SLIDER`: Indicates the mouse is over the slider of a scrollbar.
    - `SCROLLBAR_DOWN`: Indicates the mouse is over the down arrow of a scrollbar.
- **Description**: The `mouse_where` enumeration defines various locations within a user interface where a mouse event can occur. It is used to determine the specific area of the interface that the mouse is interacting with, such as panes, status lines, borders, or scrollbars. This enumeration helps in handling mouse events by providing a clear distinction of the mouse's position relative to different UI components.


# Functions

---
### server_client_window_cmp<!-- {{#callable:server_client_window_cmp}} -->
Compares two `client_window` structures based on their `window` field.
- **Inputs**:
    - `cw1`: A pointer to the first `client_window` structure to compare.
    - `cw2`: A pointer to the second `client_window` structure to compare.
- **Control Flow**:
    - The function first checks if the `window` field of `cw1` is less than that of `cw2`, returning -1 if true.
    - If the `window` field of `cw1` is greater than that of `cw2`, it returns 1.
    - If both `window` fields are equal, it returns 0.
- **Output**:
    - Returns an integer: -1 if `cw1` is less than `cw2`, 1 if greater, and 0 if they are equal.


---
### server_client_how_many<!-- {{#callable:server_client_how_many}} -->
The `server_client_how_many` function counts the number of attached clients with active sessions.
- **Inputs**:
    - None
- **Control Flow**:
    - Initialize a counter `n` to zero.
    - Iterate over the list of clients using `TAILQ_FOREACH`.
    - For each client, check if its session is not NULL and if it does not have the `CLIENT_UNATTACHEDFLAGS` set.
    - If both conditions are met, increment the counter `n`.
    - After the loop, return the count `n`.
- **Output**:
    - Returns an unsigned integer representing the number of clients that are currently attached and have active sessions.


---
### server_client_overlay_timer<!-- {{#callable:server_client_overlay_timer}} -->
The `server_client_overlay_timer` function clears the overlay for a client when the timer expires.
- **Inputs**:
    - `fd`: An unused file descriptor, typically for event handling.
    - `events`: An unused short representing event types.
    - `data`: A pointer to the client data associated with the timer.
- **Control Flow**:
    - The function calls [`server_client_clear_overlay`](#server_client_clear_overlay) with the `data` argument, which is expected to be a pointer to a `client` structure.
    - No conditional statements or loops are present, as the function directly invokes another function.
- **Output**:
    - The function does not return a value; it performs an action that modifies the state of the client by clearing its overlay.
- **Functions called**:
    - [`server_client_clear_overlay`](#server_client_clear_overlay)


---
### server_client_set_overlay<!-- {{#callable:server_client_set_overlay}} -->
Sets an overlay for a client with specified callbacks and delay.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client to set the overlay for.
    - `delay`: An unsigned integer representing the delay in milliseconds before the overlay is activated.
    - `checkcb`: A callback function of type `overlay_check_cb` to check conditions for the overlay.
    - `modecb`: A callback function of type `overlay_mode_cb` to set the overlay mode.
    - `drawcb`: A callback function of type `overlay_draw_cb` to handle drawing the overlay.
    - `keycb`: A callback function of type `overlay_key_cb` to handle key events for the overlay.
    - `freecb`: A callback function of type `overlay_free_cb` to free resources associated with the overlay.
    - `resizecb`: A callback function of type `overlay_resize_cb` to handle resizing the overlay.
    - `data`: A pointer to user-defined data that can be passed to the callback functions.
- **Control Flow**:
    - If the `overlay_draw` field of the client is not NULL, the existing overlay is cleared by calling [`server_client_clear_overlay`](#server_client_clear_overlay).
    - The delay is converted from milliseconds to a `struct timeval` format for use with the event timer.
    - If the overlay timer is already initialized, it is deleted using `evtimer_del`.
    - The overlay timer is set with `evtimer_set`, pointing to the `server_client_overlay_timer` function and passing the client pointer.
    - If the delay is not zero, the timer is added with `evtimer_add` using the previously set `struct timeval`.
    - The overlay callback functions are assigned to the respective fields in the client structure.
    - If `overlay_check` is NULL, the `TTY_FREEZE` flag is set in the client's `tty.flags`.
    - If `overlay_mode` is NULL, the `TTY_NOCURSOR` flag is set in the client's `tty.flags`.
    - The focus of the current window is updated with `window_update_focus`.
    - The client is redrawn by calling `server_redraw_client`.
- **Output**:
    - The function does not return a value; it modifies the state of the client by setting up an overlay with the provided callbacks and delay.
- **Functions called**:
    - [`server_client_clear_overlay`](#server_client_clear_overlay)


---
### server_client_clear_overlay<!-- {{#callable:server_client_clear_overlay}} -->
Clears the overlay state for a given client.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client whose overlay is to be cleared.
- **Control Flow**:
    - Check if the `overlay_draw` function pointer in the client is NULL; if so, exit the function early.
    - If the `overlay_timer` event is initialized, delete it to stop any pending overlay actions.
    - If the `overlay_free` function pointer is not NULL, call it to free any resources associated with the overlay.
    - Set all overlay-related pointers in the client structure to NULL to clear the overlay state.
    - Clear specific flags in the client's `tty` structure to unfreeze the terminal and show the cursor.
    - If the client has an active session, update the focus of the current window.
    - Call `server_redraw_client` to refresh the client's display.
- **Output**:
    - The function does not return a value; it modifies the state of the client by clearing its overlay and updating the display.


---
### server_client_overlay_range<!-- {{#callable:server_client_overlay_range}} -->
Calculates the visible overlay ranges for a client based on given coordinates and dimensions.
- **Inputs**:
    - `x`: The x-coordinate of the overlay's top-left corner.
    - `y`: The y-coordinate of the overlay's top-left corner.
    - `sx`: The width of the overlay.
    - `sy`: The height of the overlay.
    - `px`: The x-coordinate of the popup.
    - `py`: The y-coordinate of the popup.
    - `nx`: The width of the area to the right of the popup.
    - `r`: A pointer to a struct `overlay_ranges` where the results will be stored.
- **Control Flow**:
    - Initializes the output ranges in the `overlay_ranges` structure to zero.
    - Checks if there is no overlap in the y-direction; if so, sets the output ranges to the input popup coordinates.
    - Calculates the visible area to the left of the popup if the popup is to the right of the overlay.
    - Calculates the visible area to the right of the popup if the popup is to the left of the overlay.
- **Output**:
    - The function populates the `overlay_ranges` structure with up to two visible ranges based on the input coordinates and dimensions.


---
### server_client_check_nested<!-- {{#callable:server_client_check_nested}} -->
Checks if a client is nested within a tmux session.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client to check.
- **Control Flow**:
    - The function first attempts to find the environment variable 'TMUX' in the client's environment.
    - If the 'TMUX' variable is not found or its value is empty, the function returns 0, indicating the client is not nested.
    - If the 'TMUX' variable is found and has a non-empty value, the function iterates over all window panes.
    - During the iteration, if a window pane's tty matches the client's tty name, the function returns 1, indicating the client is nested.
    - If no matching tty is found after the iteration, the function returns 0.
- **Output**:
    - Returns 1 if the client is nested within a tmux session, otherwise returns 0.


---
### server_client_set_key_table<!-- {{#callable:server_client_set_key_table}} -->
Sets the key table for a client, updating it based on the provided name or the default.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client whose key table is to be set.
    - `name`: A string representing the name of the key table to set; if NULL, the default key table is retrieved.
- **Control Flow**:
    - Checks if the `name` parameter is NULL; if so, retrieves the current key table name for the client using [`server_client_get_key_table`](#server_client_get_key_table).
    - Calls `key_bindings_unref_table` to release the current key table associated with the client.
    - Retrieves the new key table using `key_bindings_get_table` with the specified name and increments its reference count.
    - Updates the `activity_time` of the key table to the current time using `gettimeofday`, and if it fails, calls `fatal` to terminate the program.
- **Output**:
    - The function does not return a value; it modifies the client's key table and updates its activity time.
- **Functions called**:
    - [`server_client_get_key_table`](#server_client_get_key_table)


---
### server_client_key_table_activity_diff<!-- {{#callable:server_client_key_table_activity_diff}} -->
Calculates the difference in milliseconds between a client's last activity time and the key table's last activity time.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client whose activity time is being compared.
- **Control Flow**:
    - The function uses `timersub` to compute the difference between the `activity_time` of the client and the `activity_time` of the client's key table.
    - The result is stored in a `struct timeval` variable named `diff`.
    - The function then converts the time difference from seconds and microseconds to milliseconds and returns it as a `uint64_t` value.
- **Output**:
    - Returns the time difference in milliseconds as a `uint64_t` value.


---
### server_client_get_key_table<!-- {{#callable:server_client_get_key_table}} -->
Retrieves the key table name associated with a given client.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client whose key table is being queried.
- **Control Flow**:
    - Check if the `session` associated with the client is NULL; if so, return 'root'.
    - Retrieve the key table name from the session's options using `options_get_string`.
    - If the retrieved name is an empty string, return 'root'.
    - Return the retrieved key table name.
- **Output**:
    - Returns a pointer to a string containing the name of the key table, or 'root' if no valid key table is found.


---
### server_client_is_default_key_table<!-- {{#callable:server_client_is_default_key_table}} -->
The `server_client_is_default_key_table` function checks if a given `key_table` is the default key table for a specified `client`.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client whose key table is being checked.
    - `table`: A pointer to a `key_table` structure representing the key table to be compared against the client's default key table.
- **Control Flow**:
    - The function uses `strcmp` to compare the name of the provided `key_table` with the name of the key table returned by `server_client_get_key_table(c)`.
    - It returns the result of the comparison, which is either 0 (if they are equal) or a non-zero value (if they are not equal).
- **Output**:
    - The function returns an integer value: 0 if the provided `key_table` is the default key table for the client, and a non-zero value otherwise.
- **Functions called**:
    - [`server_client_get_key_table`](#server_client_get_key_table)


---
### server_client_create<!-- {{#callable:server_client_create}} -->
Creates a new `client` structure and initializes it with the provided file descriptor.
- **Inputs**:
    - `fd`: An integer representing the file descriptor for the client connection.
- **Control Flow**:
    - Sets the file descriptor to non-blocking mode using `setblocking(fd, 0)`.
    - Allocates memory for a new `client` structure using `xcalloc`.
    - Initializes the `client` structure fields, including setting up peer communication and environment.
    - Records the creation time of the client using `gettimeofday`.
    - Initializes the command queue and red-black trees for windows and files.
    - Sets default terminal dimensions and theme.
    - Initializes the status and key bindings for the client.
    - Sets up timers for repeat and click events.
    - Inserts the new client into the global clients list and logs the creation.
- **Output**:
    - Returns a pointer to the newly created `client` structure.


---
### server_client_open<!-- {{#callable:server_client_open}} -->
The `server_client_open` function initializes a client terminal connection if certain conditions are met.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client whose terminal is to be opened.
    - `cause`: A pointer to a string that will hold an error message if the function fails.
- **Control Flow**:
    - Checks if the client is a control client; if so, it returns 0 immediately without opening a terminal.
    - Compares the client's `ttyname` with the default terminal path and checks if it matches any of the standard input/output/error file descriptors.
    - If a match is found, it sets an error message in `cause` and returns -1.
    - Checks if the client is marked as a terminal; if not, it sets an error message in `cause` and returns -1.
    - Attempts to open the terminal associated with the client using `tty_open`; if this fails, it returns -1.
    - If all checks pass and the terminal is successfully opened, it returns 0.
- **Output**:
    - Returns 0 on success, -1 on failure with an error message set in `cause`.


---
### server_client_attached_lost<!-- {{#callable:server_client_attached_lost}} -->
The `server_client_attached_lost` function handles the scenario when a client is detached from the server, updating the latest client associated with any windows that were using the lost client.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client that has been lost.
- **Control Flow**:
    - Logs the loss of the attached client using `log_debug`.
    - Iterates over all windows to find any that had the lost client as the latest.
    - For each window that had the lost client as the latest, it searches through all clients to find the one with the most recent activity time that is not the lost client.
    - If a suitable client is found, it updates the latest client for that window using [`server_client_update_latest`](#server_client_update_latest).
- **Output**:
    - The function does not return a value but updates the state of the server by potentially changing which client is considered the latest for any affected windows.
- **Functions called**:
    - [`server_client_update_latest`](#server_client_update_latest)


---
### server_client_set_session<!-- {{#callable:server_client_set_session}} -->
Sets the session for a client and updates various related states.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client whose session is being set.
    - `s`: A pointer to the `session` structure that is to be assigned to the client.
- **Control Flow**:
    - Store the current session of the client in `old` for later reference.
    - Check if the new session `s` is not NULL and different from the current session; if so, update `last_session` to the current session.
    - If `s` is NULL, set `last_session` to NULL.
    - Assign the new session `s` to the client's session.
    - Set the `CLIENT_FOCUSED` flag for the client.
    - If there was an old session and it has a current window, update the focus of that window.
    - If the new session `s` is not NULL, perform several updates including recalculating sizes, updating focus, and notifying the client of the session change.
    - Finally, check for unattached clients and update the socket.
- **Output**:
    - The function does not return a value; it modifies the state of the client and its session, updating various flags and notifying the client of changes.


---
### server_client_lost<!-- {{#callable:server_client_lost}} -->
The `server_client_lost` function handles the cleanup and disconnection of a client from the server.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client that has been lost.
- **Control Flow**:
    - Sets the `CLIENT_DEAD` flag for the client.
    - Clears any overlays, prompts, and messages associated with the client.
    - Iterates over the client's files and marks them as done with an error.
    - Removes all windows associated with the client and frees their memory.
    - Removes the client from the global list of clients.
    - Logs the disconnection of the client.
    - If the client was attached, handles the loss of the attached client.
    - Handles cleanup based on client flags (e.g., control, terminal).
    - Frees various resources associated with the client.
    - Removes the client's peer connection.
    - Closes any open file descriptors associated with the client.
    - Calls functions to update the server state and check for unattached clients.
- **Output**:
    - The function does not return a value; it performs cleanup operations and modifies the state of the server and client structures.
- **Functions called**:
    - [`server_client_clear_overlay`](#server_client_clear_overlay)
    - [`server_client_attached_lost`](#server_client_attached_lost)
    - [`server_client_unref`](#server_client_unref)


---
### server_client_unref<!-- {{#callable:server_client_unref}} -->
The `server_client_unref` function decrements the reference count of a client and schedules its memory to be freed if the count reaches zero.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client whose reference count is to be decremented.
- **Control Flow**:
    - Logs the current reference count of the client.
    - Decrements the reference count of the client.
    - Checks if the reference count has reached zero.
    - If zero, schedules the `server_client_free` function to be called once the event loop is ready.
- **Output**:
    - The function does not return a value; it modifies the state of the client and may schedule its deallocation.


---
### server_client_free<!-- {{#callable:server_client_free}} -->
Frees the resources associated with a `client` structure.
- **Inputs**:
    - `fd`: An unused file descriptor, typically not used in this function.
    - `events`: An unused event mask, typically not used in this function.
    - `arg`: A pointer to the `client` structure that needs to be freed.
- **Control Flow**:
    - Logs the debug information about the client being freed, including its reference count.
    - Calls `cmdq_free` to free the command queue associated with the client.
    - Checks if the reference count of the client is zero.
    - If the reference count is zero, it frees the client's name and the client structure itself.
- **Output**:
    - No output is returned; the function performs cleanup and deallocates memory.


---
### server_client_suspend<!-- {{#callable:server_client_suspend}} -->
Suspends a client by stopping its terminal and sending a suspend message.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client to be suspended.
- **Control Flow**:
    - Checks if the `session` associated with the client is NULL or if the client has unattached flags set.
    - If either condition is true, the function returns immediately without performing any actions.
    - Calls `tty_stop_tty` to stop the terminal associated with the client.
    - Sets the `CLIENT_SUSPENDED` flag in the client's flags.
    - Sends a suspend message to the peer of the client using `proc_send`.
- **Output**:
    - The function does not return a value; it modifies the state of the client and sends a message to its peer.


---
### server_client_detach<!-- {{#callable:server_client_detach}} -->
Detaches a client from its session.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client to be detached.
    - `msgtype`: An enumeration value of type `msgtype` indicating the type of message to send upon detachment.
- **Control Flow**:
    - Checks if the session pointer in the client is NULL or if the client has flags indicating it cannot be detached.
    - If the checks pass, sets the client's exit flag to indicate it is exiting.
    - Sets the exit type to indicate a detachment and assigns the exit message type.
    - Duplicates the session name and assigns it to the client's exit session field.
- **Output**:
    - The function does not return a value; it modifies the state of the client by setting its exit type, message type, and session name for detachment.


---
### server_client_exec<!-- {{#callable:server_client_exec}} -->
Executes a command in the context of a client session.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client executing the command.
    - `cmd`: A string containing the command to be executed.
- **Control Flow**:
    - Checks if the command string is empty; if so, it returns immediately.
    - Calculates the size of the command and retrieves the default shell from the session options or global options.
    - Validates the shell path using `checkshell`; if invalid, it defaults to `_PATH_BSHELL`.
    - Allocates memory for a message that combines the command and shell path.
    - Copies the command and shell path into the allocated message buffer.
    - Sends the message to the client's peer using `proc_send` with the message type `MSG_EXEC`.
    - Frees the allocated message buffer after sending.
- **Output**:
    - The function does not return a value; it sends a message to execute the command in the specified shell.


---
### server_client_check_mouse_in_pane<!-- {{#callable:server_client_check_mouse_in_pane}} -->
Checks the mouse position relative to a pane and its scrollbar.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane to check.
    - `px`: An unsigned integer representing the x-coordinate of the mouse pointer.
    - `py`: An unsigned integer representing the y-coordinate of the mouse pointer.
    - `sl_mpos`: A pointer to an unsigned integer that will store the mouse position within the scrollbar if applicable.
- **Control Flow**:
    - Retrieve scrollbar options and pane status from the window options.
    - Determine the visibility and dimensions of the scrollbar based on the pane's settings.
    - Check if the mouse coordinates are within the bounds of the pane or scrollbar.
    - If within the scrollbar, determine if the mouse is above, on, or below the scrollbar slider.
    - If not within the scrollbar, check if the mouse is over the pane borders if the window is not zoomed.
    - Return the appropriate enum value indicating the mouse's position relative to the pane or scrollbar.
- **Output**:
    - Returns an enum value of type `mouse_where` indicating the mouse's position: whether it is in the pane, on the scrollbar, or outside of it.


---
### server_client_check_mouse<!-- {{#callable:server_client_check_mouse}} -->
The `server_client_check_mouse` function processes mouse events for a client in a terminal multiplexer.
- **Inputs**:
    - `struct client *c`: A pointer to the `client` structure representing the client that is sending the mouse event.
    - `struct key_event *event`: A pointer to the `key_event` structure containing details about the mouse event.
- **Control Flow**:
    - The function begins by logging the mouse event details.
    - It determines the type of mouse event (e.g., MOVE, DOWN, UP, DRAG, WHEEL, DOUBLE, TRIPLE) based on the event data.
    - If the event is a double-click, it sets the type to DOUBLE and logs the position.
    - For drag events, it checks if the drag is ongoing and updates the position accordingly.
    - If the event is a mouse wheel event, it sets the type to WHEEL and logs the position.
    - For mouse release events, it sets the type to UP and logs the position.
    - If no specific type is matched, it defaults to DOWN and logs the position.
    - The function checks if the mouse event occurred on the status line and updates the mouse event structure accordingly.
    - If the event is not on the status line, it checks if the mouse is within the bounds of a pane or scrollbar.
    - Finally, it converts the mouse event into a key binding and returns the corresponding key code.
- **Output**:
    - The function returns a `key_code` representing the processed mouse event, or `KEYC_UNKNOWN` if the event could not be processed.
- **Functions called**:
    - [`server_client_check_mouse_in_pane`](#server_client_check_mouse_in_pane)


---
### server_client_is_bracket_paste<!-- {{#callable:server_client_is_bracket_paste}} -->
The `server_client_is_bracket_paste` function manages the bracket paste state for a client based on key events.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client whose paste state is being managed.
    - `key`: A `key_code` representing the key event that is being processed to determine if it is a paste start or end.
- **Control Flow**:
    - The function first checks if the `key` corresponds to the `KEYC_PASTE_START` constant, indicating the start of a bracket paste operation.
    - If it matches, the function sets the `CLIENT_BRACKETPASTING` flag in the client's `flags` and logs the action.
    - Next, it checks if the `key` corresponds to the `KEYC_PASTE_END` constant, indicating the end of a bracket paste operation.
    - If it matches, the function clears the `CLIENT_BRACKETPASTING` flag and logs the action.
    - If neither condition is met, the function returns the current state of the `CLIENT_BRACKETPASTING` flag.
- **Output**:
    - The function returns 0 when a paste start or end is detected, or a non-zero value indicating the current bracket paste state.


---
### server_client_is_assume_paste<!-- {{#callable:server_client_is_assume_paste}} -->
The `server_client_is_assume_paste` function determines if the client is likely pasting text based on the time since the last activity.
- **Inputs**:
    - `struct client *c`: A pointer to the `client` structure representing the client whose paste assumption is being checked.
- **Control Flow**:
    - Checks if the `CLIENT_BRACKETPASTING` flag is set; if so, returns 0 (not assuming paste).
    - Retrieves the `assume-paste-time` option from the client's session options; if it is 0, returns 0.
    - Calculates the time difference between the current activity time and the last activity time.
    - If the time difference is less than the configured `assume-paste-time`, it checks if the `CLIENT_ASSUMEPASTING` flag is set.
    - If `CLIENT_ASSUMEPASTING` is not set, it sets this flag and logs that paste is assumed.
    - If the time difference exceeds the threshold, it clears the `CLIENT_ASSUMEPASTING` flag if it was set and logs that paste is no longer assumed.
- **Output**:
    - Returns 1 if the client is assumed to be pasting, 0 otherwise.


---
### server_client_update_latest<!-- {{#callable:server_client_update_latest}} -->
Updates the latest client associated with the current window.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client to be updated.
- **Control Flow**:
    - Checks if the `session` of the client `c` is NULL; if so, the function returns immediately.
    - Retrieves the current window associated with the client's session.
    - Checks if the current window's latest client is already the client `c`; if so, the function returns.
    - Updates the window's latest client to `c`.
    - If the window's option for `window-size` is set to `WINDOW_SIZE_LATEST`, it calls `recalculate_size` to adjust the window size.
    - Finally, it sends a notification to the client indicating that it is now the active client.
- **Output**:
    - The function does not return a value; it modifies the state of the window and sends a notification to the client.


---
### server_client_repeat_time<!-- {{#callable:server_client_repeat_time}} -->
The `server_client_repeat_time` function calculates the repeat time for key bindings based on client and session options.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client requesting the repeat time.
    - `bd`: A pointer to the `key_binding` structure representing the key binding for which the repeat time is being calculated.
- **Control Flow**:
    - Checks if the `KEY_BINDING_REPEAT` flag is set in the key binding; if not, returns 0.
    - Retrieves the `repeat-time` option from the session's options; if it is 0, returns 0.
    - Checks if the client is not in repeat mode or if the last key pressed is different from the current key; if true, retrieves the `initial-repeat-time` option.
    - If `initial-repeat-time` is not 0, sets `repeat` to `initial`.
    - Returns the calculated repeat time.
- **Output**:
    - Returns the repeat time as an unsigned integer, which can be 0 if repeat is not enabled or if conditions are not met.


---
### server_client_key_callback<!-- {{#callable:server_client_key_callback}} -->
Handles key input events from a client, processing them according to the current session and key bindings.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item associated with the key event.
    - `data`: A pointer to a `key_event` structure containing details about the key event, including the key code and mouse event data.
- **Control Flow**:
    - Checks if the client is attached and able to accept input; if not, it exits early.
    - Updates the client's last activity time and checks for mouse events.
    - Processes mouse events if applicable, including mouse drag updates.
    - Handles specific key events for reporting themes (light or dark).
    - Determines the affected window pane based on the key event.
    - Checks if mouse keys should be forwarded or if bracket pasting is active.
    - Identifies the current key table and checks for prefix keys to switch tables.
    - Attempts to find a key binding in the current key table and executes it if found.
    - If no binding is found, it checks for a generic 'ANY' key binding.
    - If still no match, it forwards the key to the window pane or handles paste events.
    - Updates the client's latest activity if the session is valid.
- **Output**:
    - Returns a command return value indicating the result of processing the key event, typically `CMD_RETURN_NORMAL`.
- **Functions called**:
    - [`server_client_check_mouse`](#server_client_check_mouse)
    - [`server_client_report_theme`](#server_client_report_theme)
    - [`server_client_is_bracket_paste`](#server_client_is_bracket_paste)
    - [`server_client_is_assume_paste`](#server_client_is_assume_paste)
    - [`server_client_is_default_key_table`](#server_client_is_default_key_table)
    - [`server_client_set_key_table`](#server_client_set_key_table)
    - [`server_client_key_table_activity_diff`](#server_client_key_table_activity_diff)
    - [`server_client_repeat_time`](#server_client_repeat_time)
    - [`server_client_update_latest`](#server_client_update_latest)


---
### server_client_handle_key<!-- {{#callable:server_client_handle_key}} -->
Handles key input events for a client in a server environment.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client handling the key event.
    - `event`: A pointer to the `key_event` structure containing details about the key event.
- **Control Flow**:
    - Checks if the client is valid and can accept input; if not, returns 0.
    - Handles special cases for key presses in overlay mode and command prompts, processing them immediately if necessary.
    - Clears any existing status messages or overlays if applicable.
    - Adds the key event to a command queue for processing after any previously queued commands.
- **Output**:
    - Returns 1 if the key event was successfully queued for processing, or 0 if it was not processed.
- **Functions called**:
    - [`server_client_clear_overlay`](#server_client_clear_overlay)


---
### server_client_loop<!-- {{#callable:server_client_loop}} -->
The `server_client_loop` function manages the main event loop for handling client interactions and window updates in a server environment.
- **Inputs**:
    - None
- **Control Flow**:
    - Iterates over all windows to check for any necessary resizing before redrawing.
    - Iterates over all clients to check for exit conditions, session modes, redraw requests, and resets their state.
    - Iterates over all windows and their panes to check for pane resizing and buffer updates, clearing redraw flags afterward.
    - Sends theme updates to all panes in each window.
- **Output**:
    - The function does not return a value but updates the state of clients and windows based on the checks performed.
- **Functions called**:
    - [`server_client_check_window_resize`](#server_client_check_window_resize)
    - [`server_client_check_exit`](#server_client_check_exit)
    - [`server_client_check_modes`](#server_client_check_modes)
    - [`server_client_check_redraw`](#server_client_check_redraw)
    - [`server_client_reset_state`](#server_client_reset_state)
    - [`server_client_check_pane_resize`](#server_client_check_pane_resize)
    - [`server_client_check_pane_buffer`](#server_client_check_pane_buffer)


---
### server_client_check_window_resize<!-- {{#callable:server_client_check_window_resize}} -->
The `server_client_check_window_resize` function checks if a window needs to be resized and performs the resize if necessary.
- **Inputs**:
    - `struct window *w`: A pointer to the `window` structure that represents the window to be checked for resizing.
- **Control Flow**:
    - The function first checks if the `WINDOW_RESIZE` flag is set for the window; if not, it returns immediately.
    - It iterates through the `winlinks` of the window to find an attached session that is currently active.
    - If no active session is found, the function returns without resizing.
    - If an active session is found, it logs a debug message indicating the window is being resized.
    - Finally, it calls the `resize_window` function to perform the actual resizing of the window with the new dimensions.
- **Output**:
    - The function does not return a value; it performs an action (resizing the window) based on the conditions checked.


---
### server_client_resize_timer<!-- {{#callable:server_client_resize_timer}} -->
The `server_client_resize_timer` function handles the expiration of a resize timer for a window pane.
- **Inputs**:
    - `fd`: An unused file descriptor, typically not used in this function.
    - `events`: An unused short representing event types, typically not used in this function.
    - `data`: A pointer to the `window_pane` structure associated with the timer.
- **Control Flow**:
    - The function begins by casting the `data` pointer to a `window_pane` structure.
    - It logs a debug message indicating that the resize timer for the specific window pane has expired.
    - The function then deletes the associated resize timer from the event loop.
- **Output**:
    - The function does not return a value; it performs actions related to logging and timer management.


---
### server_client_check_pane_resize<!-- {{#callable:server_client_check_pane_resize}} -->
The `server_client_check_pane_resize` function manages the resizing of a window pane based on queued resize requests.
- **Inputs**:
    - `wp`: A pointer to a `window_pane` structure representing the pane that may need to be resized.
- **Control Flow**:
    - Checks if the resize queue for the pane is empty; if so, it returns immediately.
    - Initializes a resize timer if it hasn't been set and checks if it's already pending.
    - Logs the need for resizing and iterates through the resize queue to log queued resize requests.
    - Handles three cases for resizing based on the number of requests and their dimensions:
    - 1. If there is only one resize request, it applies it directly.
    - 2. If there are multiple requests and the last one results in a different size, it applies the last request and discards the others.
    - 3. If multiple requests result in the same size, it applies the first request and keeps the last one for the next check.
    - Sets a timer for the next resize check with a reduced interval if multiple requests resulted in the same size.
- **Output**:
    - The function does not return a value but modifies the state of the pane by sending resize commands and managing the resize queue.


---
### server_client_check_pane_buffer<!-- {{#callable:server_client_check_pane_buffer}} -->
The `server_client_check_pane_buffer` function manages the input buffer of a window pane, adjusting its size based on the data usage and the number of attached clients.
- **Inputs**:
    - `wp`: A pointer to a `struct window_pane` representing the pane whose buffer is being checked.
- **Control Flow**:
    - Calculate the minimum size of data that can be removed from the buffer based on the pane's offset and the attached clients' data usage.
    - Iterate through all attached clients to determine their data usage and adjust the minimum size accordingly.
    - If there are no attached clients, set a flag to indicate that the pane is off.
    - Drain the buffer by removing the calculated minimum size of data.
    - Adjust the base offset of the pane, ensuring it does not roll over.
    - Enable or disable reading from the event based on whether there are clients able to consume remaining data.
- **Output**:
    - The function does not return a value but modifies the state of the input pane's buffer and its associated clients, enabling or disabling read events as necessary.


---
### server_client_reset_state<!-- {{#callable:server_client_reset_state}} -->
Resets the state of a client in the server, including cursor position, terminal modes, and screen settings.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client whose state is to be reset.
- **Control Flow**:
    - Checks if the client is in control mode or suspended; if so, it returns immediately without making changes.
    - Disables the block flag on the terminal associated with the client.
    - Determines the current screen mode based on whether an overlay is active or if a prompt string is present.
    - Resets the terminal region and margin settings.
    - Calculates the new cursor position based on the current pane and any prompt settings.
    - Updates the terminal mode and resets attributes based on the calculated mode.
    - Sends a synchronization end signal to the terminal.
- **Output**:
    - The function does not return a value; it modifies the state of the client and its associated terminal directly.
- **Functions called**:
    - [`server_client_get_pane`](#server_client_get_pane)


---
### server_client_repeat_timer<!-- {{#callable:server_client_repeat_timer}} -->
The `server_client_repeat_timer` function handles the expiration of a repeat timer for a client, resetting the client's key table and status.
- **Inputs**:
    - `fd`: An unused file descriptor, typically for event handling.
    - `events`: An unused short representing event types.
    - `data`: A pointer to the `client` structure associated with the timer.
- **Control Flow**:
    - The function begins by casting the `data` pointer to a `client` structure.
    - It checks if the `CLIENT_REPEAT` flag is set in the client's flags.
    - If the flag is set, it calls [`server_client_set_key_table`](#server_client_set_key_table) to reset the key table to NULL.
    - The `CLIENT_REPEAT` flag is then cleared from the client's flags.
    - Finally, it calls `server_status_client` to update the client's status.
- **Output**:
    - The function does not return a value; it modifies the state of the client by resetting its key table and updating its status.
- **Functions called**:
    - [`server_client_set_key_table`](#server_client_set_key_table)


---
### server_client_click_timer<!-- {{#callable:server_client_click_timer}} -->
The `server_client_click_timer` function handles the expiration of a click timer for a client, determining if a double-click event should be processed.
- **Inputs**:
    - `fd`: An unused file descriptor, typically for event handling.
    - `events`: An unused short representing event types.
    - `data`: A pointer to the `client` structure associated with the timer.
- **Control Flow**:
    - Logs a debug message indicating the click timer has expired.
    - Checks if the client has the `CLIENT_TRIPLECLICK` flag set.
    - If the flag is set, it allocates memory for a new `key_event` structure and sets it to represent a double-click event.
    - Copies the mouse event data from the client to the new event structure.
    - Calls [`server_client_handle_key`](#server_client_handle_key) to process the double-click event.
    - If the event handling fails, it frees the allocated memory for the event.
    - Clears the `CLIENT_DOUBLECLICK` and `CLIENT_TRIPLECLICK` flags from the client.
- **Output**:
    - The function does not return a value; it modifies the state of the client and may trigger further processing of a double-click event.
- **Functions called**:
    - [`server_client_handle_key`](#server_client_handle_key)


---
### server_client_check_exit<!-- {{#callable:server_client_check_exit}} -->
The `server_client_check_exit` function checks if a client should exit and handles the exit process accordingly.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client that is being checked for exit conditions.
- **Control Flow**:
    - The function first checks if the client is already dead or has exited, returning early if so.
    - It checks if the client is a control client and discards control messages if necessary.
    - It iterates through the client's files to ensure there is no data left to process.
    - If the client is ready to exit, it sets the `CLIENT_EXITED` flag.
    - Based on the `exit_type`, it sends the appropriate exit message to the peer, which can be a return value, shutdown message, or detach message.
    - Finally, it frees the exit session and exit message resources.
- **Output**:
    - The function does not return a value but sends exit messages to the client's peer and modifies the client's state.


---
### server_client_redraw_timer<!-- {{#callable:server_client_redraw_timer}} -->
The `server_client_redraw_timer` function logs a debug message when the redraw timer is triggered.
- **Inputs**:
    - None
- **Control Flow**:
    - The function does not have any conditional statements or loops.
    - It directly logs a debug message indicating that the redraw timer has fired.
- **Output**:
    - The function does not return any value; it simply logs a message to the debug output.


---
### server_client_check_modes<!-- {{#callable:server_client_check_modes}} -->
The `server_client_check_modes` function updates the modes of all panes in the current window of a client if certain conditions are met.
- **Inputs**:
    - `struct client *c`: A pointer to the `client` structure representing the client whose modes are to be checked.
- **Control Flow**:
    - The function first checks if the client is in control mode or suspended; if so, it returns immediately.
    - Next, it checks if the `CLIENT_REDRAWSTATUS` flag is not set; if it is not set, the function returns.
    - The function then iterates over each pane in the current window of the client.
    - For each pane, it retrieves the first mode entry and checks if it has an update function.
    - If an update function exists, it calls this function to update the mode.
- **Output**:
    - The function does not return a value; it performs updates on the modes of the panes directly.


---
### server_client_check_redraw<!-- {{#callable:server_client_check_redraw}} -->
The `server_client_check_redraw` function manages the redraw process for a client in a terminal multiplexer, determining what needs to be redrawn based on the client's state and the state of its associated panes.
- **Inputs**:
    - `struct client *c`: A pointer to the `client` structure representing the client that is being checked for redraw requirements.
- **Control Flow**:
    - The function first checks if the client is in a state that allows for redrawing (not suspended or a control client).
    - It logs the redraw flags if any are set for the client.
    - It determines if a redraw is needed based on the flags set in the client and the state of its panes.
    - If there is data left in the output buffer, it defers the redraw by setting a timer.
    - If no data is left, it proceeds to redraw the necessary components (panes, scrollbars, etc.) based on the flags.
    - Finally, it updates the terminal mode and resets the flags related to redrawing.
- **Output**:
    - The function does not return a value but modifies the state of the client and its associated panes, potentially resulting in a redraw of the terminal output based on the flags set.
- **Functions called**:
    - [`server_client_set_title`](#server_client_set_title)
    - [`server_client_set_path`](#server_client_set_path)


---
### server_client_set_title<!-- {{#callable:server_client_set_title}} -->
Sets the title of a client based on a template string.
- **Inputs**:
    - `c`: A pointer to the `client` structure whose title is to be set.
- **Control Flow**:
    - Retrieve the title template string from the session options using `options_get_string`.
    - Create a `format_tree` using `format_create` to handle the title formatting.
    - Set default values for the format tree with `format_defaults`.
    - Expand the title using `format_expand_time` with the retrieved template.
    - Check if the new title differs from the current title; if so, update the title.
    - Free the old title and allocate memory for the new title using `xstrdup`.
    - Set the terminal title using `tty_set_title`.
    - Free the expanded title and the format tree.
- **Output**:
    - The function does not return a value; it modifies the title of the client directly.


---
### server_client_set_path<!-- {{#callable:server_client_set_path}} -->
Sets the path for a client based on the current session's active window.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client whose path is to be set.
- **Control Flow**:
    - Checks if the current session's window is valid by verifying if `s->curw` is not NULL.
    - If the active window's path is NULL, it sets the path to an empty string.
    - If the client's current path is NULL or different from the active window's path, it updates the client's path.
    - Frees the old path memory and duplicates the new path string.
    - Calls `tty_set_path` to update the terminal's path.
- **Output**:
    - The function does not return a value; it modifies the client's path and updates the terminal's path accordingly.


---
### server_client_dispatch<!-- {{#callable:server_client_dispatch}} -->
The `server_client_dispatch` function processes incoming messages from a client and dispatches them to the appropriate handler.
- **Inputs**:
    - `imsg`: Pointer to an `imsg` structure containing the message data from the client.
    - `arg`: Pointer to a `client` structure representing the client associated with the message.
- **Control Flow**:
    - Checks if the client is marked as dead; if so, it returns immediately.
    - If the incoming message (`imsg`) is NULL, it calls [`server_client_lost`](#server_client_lost) to handle the lost client.
    - Calculates the length of the data in the message by subtracting the header size from the total length.
    - Uses a switch statement to determine the type of message and calls the corresponding handler function based on the message type.
    - Handles various message types such as client identification, command execution, resizing, exiting, and others, delegating to specific functions for each type.
- **Output**:
    - The function does not return a value; instead, it performs actions based on the message type, such as updating client state, resizing the terminal, or executing commands.
- **Functions called**:
    - [`server_client_lost`](#server_client_lost)
    - [`server_client_dispatch_identify`](#server_client_dispatch_identify)
    - [`server_client_dispatch_command`](#server_client_dispatch_command)
    - [`server_client_update_latest`](#server_client_update_latest)
    - [`server_client_clear_overlay`](#server_client_clear_overlay)
    - [`server_client_set_session`](#server_client_set_session)
    - [`server_client_dispatch_shell`](#server_client_dispatch_shell)


---
### server_client_read_only<!-- {{#callable:server_client_read_only}} -->
The `server_client_read_only` function handles the scenario where a command is attempted on a read-only client.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item that is being processed.
    - `data`: An unused pointer to additional data, which is not utilized in this function.
- **Control Flow**:
    - The function calls `cmdq_error` to log an error message indicating that the client is read-only.
    - It then returns a value of `CMD_RETURN_ERROR` to indicate that the command could not be executed.
- **Output**:
    - The function outputs an error message and returns a command return value indicating failure.


---
### server_client_command_done<!-- {{#callable:server_client_command_done}} -->
The `server_client_command_done` function processes the completion of a command for a client, handling client exit conditions and sending requests.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item being processed.
    - `data`: An unused pointer to additional data, which is not utilized in this function.
- **Control Flow**:
    - Retrieves the client associated with the command queue item using `cmdq_get_client`.
    - Checks if the client is not attached; if so, sets the `CLIENT_EXIT` flag.
    - If the client is not in the exit state, checks if it is in control mode and calls `control_ready` if true.
    - Sends requests to the client's terminal using `tty_send_requests`.
- **Output**:
    - Returns `CMD_RETURN_NORMAL`, indicating the command was processed successfully.


---
### server_client_dispatch_command<!-- {{#callable:server_client_dispatch_command}} -->
The `server_client_dispatch_command` function processes a command message received from a client.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client that sent the command.
    - `imsg`: A pointer to the `imsg` structure containing the command message.
- **Control Flow**:
    - The function first checks if the client is marked for exit using the `CLIENT_EXIT` flag; if so, it returns immediately.
    - It verifies the size of the incoming message and copies the command data into a local structure.
    - The function extracts the command arguments from the message and checks if they are valid.
    - If no arguments are provided, it retrieves a default command list; otherwise, it parses the command arguments.
    - Based on the parsing result, it either prepares a command queue item or handles errors appropriately.
    - Finally, it appends the command item to the client's command queue and handles the completion callback.
- **Output**:
    - The function does not return a value but modifies the client's command queue based on the processed command, potentially resulting in command execution or error handling.


---
### server_client_dispatch_identify<!-- {{#callable:server_client_dispatch_identify}} -->
The `server_client_dispatch_identify` function processes identification messages from a client.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client sending the message.
    - `imsg`: A pointer to the `imsg` structure containing the message data.
- **Control Flow**:
    - Checks if the client has already been identified and raises an error if so.
    - Extracts the message type and data length from the `imsg` structure.
    - Processes the message based on its type, handling various identification messages such as features, flags, terminal name, and current working directory.
    - For each message type, it validates the data length, updates the client structure accordingly, and logs the action.
    - If the message type is `MSG_IDENTIFY_DONE`, it marks the client as identified and sets the client's name.
    - If the client is a control client, it starts control mode; otherwise, it initializes the terminal and resizes it if necessary.
    - If this is the first client, it loads configuration files.
- **Output**:
    - The function does not return a value but modifies the state of the `client` structure based on the identification messages received.


---
### server_client_dispatch_shell<!-- {{#callable:server_client_dispatch_shell}} -->
The `server_client_dispatch_shell` function sends the default shell command to a client and terminates the client's peer connection.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client to which the shell command will be sent.
- **Control Flow**:
    - Retrieve the default shell from global options using `options_get_string`.
    - Check if the retrieved shell is valid using `checkshell`.
    - If the shell is invalid, set it to a predefined default shell path `_PATH_BSHELL`.
    - Send the shell command to the client's peer using `proc_send`.
    - Terminate the peer connection for the client using `proc_kill_peer`.
- **Output**:
    - The function does not return a value; it performs actions that affect the state of the client and its connection.


---
### server_client_get_cwd<!-- {{#callable:server_client_get_cwd}} -->
Retrieves the current working directory for a client.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client whose current working directory is being requested.
    - `s`: A pointer to a `session` structure that may contain a current working directory.
- **Control Flow**:
    - Checks if the configuration is finished and if a global client configuration is set, returning its current working directory if so.
    - If the `client` pointer `c` is not NULL and its session is NULL, it returns the `cwd` of the client if it is set.
    - If the `session` pointer `s` is not NULL and its `cwd` is set, it returns that value.
    - If the `client` pointer `c` is not NULL and its session is not NULL, it retrieves the `cwd` from the session.
    - If none of the above conditions are met, it attempts to find the home directory using the `find_home()` function and returns it if found.
    - If all checks fail, it returns the root directory '/' as a fallback.
- **Output**:
    - Returns a pointer to a string representing the current working directory, which may be the client's cwd, session's cwd, home directory, or the root directory.


---
### server_client_control_flags<!-- {{#callable:server_client_control_flags}} -->
The `server_client_control_flags` function processes control flag commands for a client.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client whose control flags are being set.
    - `next`: A string representing the control command to be processed.
- **Control Flow**:
    - The function first checks if the `next` string is equal to 'pause-after' and sets the client's `pause_age` to 0, returning the `CLIENT_CONTROL_PAUSEAFTER` flag.
    - If the `next` string matches the format 'pause-after=<value>', it parses the value, multiplies it by 1000, and returns the `CLIENT_CONTROL_PAUSEAFTER` flag.
    - If the `next` string is 'no-output', it returns the `CLIENT_CONTROL_NOOUTPUT` flag.
    - If the `next` string is 'wait-exit', it returns the `CLIENT_CONTROL_WAITEXIT` flag.
    - If none of the conditions are met, it returns 0.
- **Output**:
    - The function returns a 64-bit unsigned integer representing the control flag corresponding to the command processed, or 0 if no valid command was found.


---
### server_client_set_flags<!-- {{#callable:server_client_set_flags}} -->
Sets flags for a client based on a comma-separated string of flag names.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client whose flags are to be set.
    - `flags`: A string containing comma-separated flag names to be set or unset for the client.
- **Control Flow**:
    - Duplicates the input `flags` string to avoid modifying the original.
    - Splits the duplicated string into individual flag names using `strsep`.
    - Checks if each flag name starts with '!', indicating it should be unset.
    - Determines the corresponding flag value for each flag name, including special handling for control flags.
    - Logs the action of setting or unsetting the flag for the client.
    - Updates the client's flags based on whether the flag is to be set or unset.
    - Resets control offsets if the `CLIENT_CONTROL_NOOUTPUT` flag is set.
    - Sends the updated flags to the client's peer.
- **Output**:
    - The function does not return a value but modifies the flags of the client and sends the updated flags to the peer.
- **Functions called**:
    - [`server_client_control_flags`](#server_client_control_flags)


---
### server_client_get_flags<!-- {{#callable:server_client_get_flags}} -->
The `server_client_get_flags` function retrieves a string representation of the flags set for a given client.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client whose flags are to be retrieved.
- **Control Flow**:
    - Initializes a static character array `s` to store the flags string.
    - Checks each flag in the `client` structure using bitwise AND operations.
    - If a flag is set, appends the corresponding string representation to `s` using `strlcat`.
    - If the resulting string is not empty, removes the trailing comma.
    - Returns the constructed string of flags.
- **Output**:
    - Returns a pointer to a string containing the flags of the client, formatted as a comma-separated list.


---
### server_client_get_client_window<!-- {{#callable:server_client_get_client_window}} -->
Retrieves a `client_window` structure associated with a given client and window ID.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client whose window is being queried.
    - `id`: An unsigned integer representing the ID of the window to be retrieved.
- **Control Flow**:
    - A `client_window` structure is initialized with the provided window ID.
    - The function uses `RB_FIND` to search for the `client_window` in the red-black tree of windows associated with the client.
    - If a matching `client_window` is found, it is returned; otherwise, NULL is returned.
- **Output**:
    - Returns a pointer to the `client_window` structure if found, or NULL if no matching window exists.


---
### server_client_add_client_window<!-- {{#callable:server_client_add_client_window}} -->
Adds a client window to a specified client if it does not already exist.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client to which the window is being added.
    - `id`: An unsigned integer representing the identifier of the window to be added.
- **Control Flow**:
    - Calls [`server_client_get_client_window`](#server_client_get_client_window) to check if a client window with the specified ID already exists.
    - If the client window does not exist (i.e., `cw` is NULL), it allocates memory for a new `client_window` structure.
    - Sets the `window` field of the new `client_window` to the provided ID.
    - Inserts the new `client_window` into the client's window tree using `RB_INSERT`.
- **Output**:
    - Returns a pointer to the `client_window` structure representing the added or existing client window.
- **Functions called**:
    - [`server_client_get_client_window`](#server_client_get_client_window)


---
### server_client_get_pane<!-- {{#callable:server_client_get_pane}} -->
Retrieves the active pane associated with a given client.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client requesting the pane.
- **Control Flow**:
    - Checks if the session associated with the client is NULL; if so, returns NULL.
    - Checks if the `CLIENT_ACTIVEPANE` flag is not set; if so, returns the active pane of the current window.
    - Retrieves the client window associated with the client and the current window's ID.
    - If the client window is NULL, returns the active pane of the current window.
    - Returns the pane associated with the retrieved client window.
- **Output**:
    - Returns a pointer to the `window_pane` structure representing the active pane for the client, or NULL if no valid pane is found.
- **Functions called**:
    - [`server_client_get_client_window`](#server_client_get_client_window)


---
### server_client_set_pane<!-- {{#callable:server_client_set_pane}} -->
Sets the active pane for a given client in a tmux session.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client whose active pane is being set.
    - `wp`: A pointer to the `window_pane` structure representing the pane to be set as active for the client.
- **Control Flow**:
    - Checks if the session associated with the client is NULL; if so, the function returns immediately.
    - Calls [`server_client_add_client_window`](#server_client_add_client_window) to retrieve or create a `client_window` structure for the current window of the session.
    - Sets the `pane` field of the `client_window` structure to the provided pane.
    - Logs a debug message indicating the pane has been set for the client.
- **Output**:
    - The function does not return a value; it modifies the state of the client by setting its active pane.
- **Functions called**:
    - [`server_client_add_client_window`](#server_client_add_client_window)


---
### server_client_remove_pane<!-- {{#callable:server_client_remove_pane}} -->
Removes a specified `window_pane` from all associated `client_window` structures.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure that needs to be removed from clients.
- **Control Flow**:
    - Iterates over all clients in the `clients` list using `TAILQ_FOREACH`.
    - For each client, retrieves the associated `client_window` using [`server_client_get_client_window`](#server_client_get_client_window).
    - If the `client_window` exists and its `pane` matches the provided `window_pane`, it removes the `client_window` from the client's `windows` red-black tree using `RB_REMOVE`.
    - Frees the memory allocated for the `client_window` after removal.
- **Output**:
    - The function does not return a value; it modifies the state of the clients by removing the specified pane from their associated windows.
- **Functions called**:
    - [`server_client_get_client_window`](#server_client_get_client_window)


---
### server_client_print<!-- {{#callable:server_client_print}} -->
The `server_client_print` function sends formatted output to a specified client, either as sanitized text or raw data.
- **Inputs**:
    - `struct client *c`: A pointer to the `client` structure representing the client to which the output will be sent.
    - `int parse`: An integer flag indicating whether the output should be parsed (1) or not (0).
    - `struct evbuffer *evb`: A pointer to an `evbuffer` structure containing the data to be sent to the client.
- **Control Flow**:
    - If `parse` is false, the function sanitizes the data in `evb` and prepares it for output.
    - If `parse` is true and `evb` is empty, it sets `msg` to a pointer to an empty character.
    - If the client `c` is NULL, the function exits early.
    - If the client session is NULL or the client is a control client, it sanitizes the message and sends it to the client or prints it to a file.
    - If the client is attached to a session, it retrieves the associated window pane and sets the mode to `window_view_mode` if necessary.
    - If `parse` is true, it reads lines from the `evbuffer` and adds them to the window pane, otherwise it adds the message directly.
- **Output**:
    - The function does not return a value; instead, it modifies the state of the client by sending output to it or updating the window pane.
- **Functions called**:
    - [`server_client_get_pane`](#server_client_get_pane)


---
### server_client_report_theme<!-- {{#callable:server_client_report_theme}} -->
Reports the theme preference of a client and requests color information.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client whose theme is being reported.
    - `theme`: An enumeration value of type `client_theme` indicating the selected theme (light or dark).
- **Control Flow**:
    - Checks if the `theme` is `THEME_LIGHT` and sets the client's theme accordingly.
    - Notifies the client of the selected theme using `notify_client`.
    - If the theme is not light, it defaults to `THEME_DARK` and notifies the client of the dark theme.
    - Sends escape sequences to the client to request the current foreground and background colors.
- **Output**:
    - The function does not return a value but updates the client's theme and requests color information.


# Function Declarations (Public API)

---
### server_client_free<!-- {{#callable_declaration:server_client_free}} -->
Frees a client structure if it has no remaining references.
- **Description**: This function is used to release resources associated with a client structure when it is no longer needed. It should be called when a client is no longer in use, and it will free the client's command queue and, if the reference count is zero, the client's name and the client structure itself. This function is typically used in the context of managing client lifecycles in a server application.
- **Inputs**:
    - `fd`: This parameter is unused and can be ignored.
    - `events`: This parameter is unused and can be ignored.
    - `arg`: A pointer to the client structure to be freed. This must not be null and should point to a valid client structure.
- **Output**: None
- **See also**: [`server_client_free`](#server_client_free)  (Implementation)


---
### server_client_check_pane_resize<!-- {{#callable_declaration:server_client_check_pane_resize}} -->
Checks and processes pending resize requests for a window pane.
- **Description**: This function is used to handle pending resize requests for a given window pane. It should be called when there is a need to process any queued resize operations for a pane. The function ensures that resize operations are applied correctly based on the number of pending requests and their characteristics. It manages a timer to delay subsequent resize checks, ensuring efficient processing of resize events.
- **Inputs**:
    - `wp`: A pointer to a `struct window_pane` representing the pane to be checked for resize requests. Must not be null. The function assumes ownership of managing the resize queue within the pane.
- **Output**: None
- **See also**: [`server_client_check_pane_resize`](#server_client_check_pane_resize)  (Implementation)


---
### server_client_check_pane_buffer<!-- {{#callable_declaration:server_client_check_pane_buffer}} -->
Checks and manages the buffer of a window pane.
- **Description**: This function is used to manage the buffer of a specified window pane, ensuring that the buffer does not exceed its limits and adjusting the buffer offsets as necessary. It should be called when there is a need to check the buffer size and manage data flow for a window pane. The function will disable reading from the buffer if no clients can consume the data, and it will enable reading if clients are available. It is important to ensure that the window pane is properly initialized and that the function is called in a context where buffer management is required.
- **Inputs**:
    - `wp`: A pointer to a 'struct window_pane' representing the window pane whose buffer is to be checked and managed. Must not be null, and the window pane should be properly initialized before calling this function.
- **Output**: None
- **See also**: [`server_client_check_pane_buffer`](#server_client_check_pane_buffer)  (Implementation)


---
### server_client_check_window_resize<!-- {{#callable_declaration:server_client_check_window_resize}} -->
Checks if a window needs resizing and performs the resize if necessary.
- **Description**: This function is used to verify if a given window requires resizing based on its flags and the state of its associated sessions. It should be called periodically to ensure that windows are resized when needed. The function checks if the window has the WINDOW_RESIZE flag set and if it is currently attached to a session. If these conditions are met, it proceeds to resize the window to its new dimensions. This function does not return any value and does not modify the input parameters directly.
- **Inputs**:
    - `w`: A pointer to a struct window representing the window to be checked for resizing. The pointer must not be null, and the window should be part of a session to be resized. If the window does not have the WINDOW_RESIZE flag set, the function will return immediately without performing any action.
- **Output**: None
- **See also**: [`server_client_check_window_resize`](#server_client_check_window_resize)  (Implementation)


---
### server_client_check_mouse<!-- {{#callable_declaration:server_client_check_mouse}} -->
Processes mouse events for a client and returns a corresponding key code.
- **Description**: This function is used to handle mouse events for a client within a session, determining the type of mouse event (such as click, drag, or wheel) and its location (such as on a pane, status line, or scrollbar). It should be called when a mouse event is detected to translate it into a key code that can be used for further processing. The function requires a valid client and key event structure, and it assumes that the client is part of an active session. It returns a key code that represents the mouse event, or a special code if the event is not recognized or relevant.
- **Inputs**:
    - `c`: A pointer to a client structure representing the client for which the mouse event is being processed. Must not be null and should be part of an active session.
    - `event`: A pointer to a key_event structure containing the mouse event details. Must not be null and should contain valid mouse event data.
- **Output**: Returns a key_code representing the mouse event, or KEYC_UNKNOWN if the event is not recognized or relevant.
- **See also**: [`server_client_check_mouse`](#server_client_check_mouse)  (Implementation)


---
### server_client_repeat_timer<!-- {{#callable_declaration:server_client_repeat_timer}} -->
Handles the repeat timer for a client.
- **Description**: This function is used to handle the repeat timer event for a client. It should be called when the repeat timer expires. If the client has the CLIENT_REPEAT flag set, the function will reset the client's key table to the default and clear the CLIENT_REPEAT flag. This function is typically used in the context of managing client input repeat behavior in a server-client architecture.
- **Inputs**:
    - `fd`: This parameter is unused and should be ignored.
    - `events`: This parameter is unused and should be ignored.
    - `data`: A pointer to a client structure. Must not be null, as it is used to access the client's flags and key table.
- **Output**: None
- **See also**: [`server_client_repeat_timer`](#server_client_repeat_timer)  (Implementation)


---
### server_client_click_timer<!-- {{#callable_declaration:server_client_click_timer}} -->
Handles a click timer expiration for a client.
- **Description**: This function is used to handle the expiration of a click timer for a client, specifically when waiting for a third click that does not occur, thus confirming a double-click event. It should be called when the click timer expires, and it processes the double-click event if the client was in a state of expecting a triple-click. The function clears the double-click and triple-click flags from the client after processing.
- **Inputs**:
    - `fd`: This parameter is unused and can be ignored.
    - `events`: This parameter is unused and can be ignored.
    - `data`: A pointer to a client structure. Must not be null, as it is used to access the client's state and flags.
- **Output**: None
- **See also**: [`server_client_click_timer`](#server_client_click_timer)  (Implementation)


---
### server_client_check_exit<!-- {{#callable_declaration:server_client_check_exit}} -->
Checks if a client should exit and performs necessary actions.
- **Description**: This function is used to determine if a client should exit based on its current state and flags. It should be called periodically to ensure that clients are properly terminated when they meet the exit conditions. The function checks various flags and conditions, such as whether the client is marked for exit or if it is in control mode, and performs the appropriate actions, such as sending exit messages or freeing resources. It is important to ensure that the client is not already marked as dead or exited before calling this function.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client to check. Must not be null. The function expects the client structure to be properly initialized and managed by the caller.
- **Output**: None
- **See also**: [`server_client_check_exit`](#server_client_check_exit)  (Implementation)


---
### server_client_check_redraw<!-- {{#callable_declaration:server_client_check_redraw}} -->
Checks if a client requires a redraw and schedules it if necessary.
- **Description**: This function determines whether a client needs to be redrawn based on its current flags and the state of its associated panes. It should be called periodically to ensure the client display is up-to-date. If a redraw is needed but the output buffer is not empty, the redraw is deferred and a timer is set to retry later. This function should not be called for clients that are in control mode or suspended.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client to check. Must not be null. The function will check the client's flags and associated session and window state to determine if a redraw is needed.
- **Output**: None
- **See also**: [`server_client_check_redraw`](#server_client_check_redraw)  (Implementation)


---
### server_client_check_modes<!-- {{#callable_declaration:server_client_check_modes}} -->
Checks and updates modes for the client's current window.
- **Description**: Use this function to ensure that the modes for the panes in the client's current window are up-to-date. It should be called when the status line is redrawn, as it only updates modes in the current window. The function does nothing if the client is in control mode, suspended, or if the redraw status flag is not set.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client whose window modes need to be checked. Must not be null. The function will return immediately if the client is in control mode, suspended, or if the redraw status flag is not set.
- **Output**: None
- **See also**: [`server_client_check_modes`](#server_client_check_modes)  (Implementation)


---
### server_client_set_title<!-- {{#callable_declaration:server_client_set_title}} -->
Sets the title of a client based on a session's title template.
- **Description**: This function updates the title of a client using a template string defined in the client's session options. It should be called when the client's title needs to be refreshed, such as after a session change or when the title template is modified. The function ensures that the title is only updated if it differs from the current title, minimizing unnecessary operations. It requires a valid client structure with an associated session and assumes that the session's options are properly configured.
- **Inputs**:
    - `c`: A pointer to a client structure. This must not be null and must have an associated session with valid options. The function will update the title field of this client if necessary.
- **Output**: None
- **See also**: [`server_client_set_title`](#server_client_set_title)  (Implementation)


---
### server_client_set_path<!-- {{#callable_declaration:server_client_set_path}} -->
Sets the path for a client based on the active window pane.
- **Description**: This function updates the path associated with a client to match the path of the active window pane in the client's current session. It should be called when the active window pane changes or when the client's path needs to be synchronized with the active pane. The function ensures that the client's path is updated only if it differs from the current path, minimizing unnecessary operations. It must be called with a valid client structure that is part of an active session.
- **Inputs**:
    - `c`: A pointer to a client structure. The client must be part of an active session, and the pointer must not be null. If the session's current window is null, the function returns immediately without making changes.
- **Output**: None
- **See also**: [`server_client_set_path`](#server_client_set_path)  (Implementation)


---
### server_client_reset_state<!-- {{#callable_declaration:server_client_reset_state}} -->
Resets the state of a client.
- **Description**: This function is used to reset the state of a client, typically as part of a redraw or update cycle. It should be called when the client is not in control or suspended mode. The function updates the terminal mode, cursor position, and other display settings based on the current screen or overlay mode. It also handles mouse mode settings and ensures that the terminal attributes are reset. This function is essential for maintaining the correct display state of the client.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client whose state is to be reset. This parameter must not be null, and the client should be in a valid state for display updates.
- **Output**: None
- **See also**: [`server_client_reset_state`](#server_client_reset_state)  (Implementation)


---
### server_client_update_latest<!-- {{#callable_declaration:server_client_update_latest}} -->
Updates the latest active client for a window.
- **Description**: This function should be called to update the latest active client associated with a window in a session. It is typically used when a client becomes active or when the window's state changes. The function requires that the client is associated with a valid session. If the client is already the latest for the window, no changes are made. Additionally, if the window size option is set to 'WINDOW_SIZE_LATEST', the window size is recalculated. A notification is sent to indicate the client is active.
- **Inputs**:
    - `c`: A pointer to a 'struct client' representing the client to be updated. Must not be null and must be associated with a valid session. If the session is null, the function returns immediately without making changes.
- **Output**: None
- **See also**: [`server_client_update_latest`](#server_client_update_latest)  (Implementation)


---
### server_client_dispatch<!-- {{#callable_declaration:server_client_dispatch}} -->
Dispatches messages from a client to the appropriate handler.
- **Description**: This function is responsible for processing incoming messages from a client and dispatching them to the appropriate handler based on the message type. It should be called whenever a message is received from a client. The function handles various message types, such as client identification, command execution, resizing, and more. It is important to ensure that the client is not marked as dead before processing the message. If the message is null, it indicates that the client has been lost, and appropriate cleanup should be performed.
- **Inputs**:
    - `imsg`: A pointer to the incoming message structure. It must not be null unless indicating a lost client.
    - `arg`: A pointer to the client structure associated with the message. The client must not be marked as dead.
- **Output**: None
- **See also**: [`server_client_dispatch`](#server_client_dispatch)  (Implementation)


---
### server_client_dispatch_command<!-- {{#callable_declaration:server_client_dispatch_command}} -->
Dispatches a command from a client based on the received message.
- **Description**: This function processes a command message received from a client and executes the corresponding command. It should be called when a command message is received from a client. The function checks if the client is marked for exit and returns immediately if so. It validates the message size and ensures the command string is null-terminated. If the command arguments are valid, it parses them and executes the command. If the client is read-only and the command is not allowed, it triggers a read-only error callback. In case of errors during argument unpacking or parsing, it appends an error message to the client's command queue and marks the client for exit.
- **Inputs**:
    - `c`: A pointer to a client structure representing the client sending the command. Must not be null. The function checks the client's flags to determine if it should proceed or return immediately.
    - `imsg`: A pointer to an imsg structure containing the command message. Must not be null. The function validates the message size and checks for a null-terminated command string.
- **Output**: None
- **See also**: [`server_client_dispatch_command`](#server_client_dispatch_command)  (Implementation)


---
### server_client_dispatch_identify<!-- {{#callable_declaration:server_client_dispatch_identify}} -->
Handles client identification messages.
- **Description**: This function processes identification messages from a client, updating the client's state based on the message type. It should be called when a client sends an identification message to ensure the client's attributes are correctly set. The function expects the client to not be already identified, and it will set various client attributes such as terminal features, flags, terminal name, and environment variables. It also handles the completion of the identification process by marking the client as identified and performing additional setup if necessary.
- **Inputs**:
    - `c`: A pointer to a client structure. The client must not be null and should not already be identified. The function updates this structure with information from the identification message.
    - `imsg`: A pointer to an imsg structure containing the identification message. The message must be valid and contain the expected data for the specified message type. Invalid message sizes or formats will result in a fatal error.
- **Output**: None
- **See also**: [`server_client_dispatch_identify`](#server_client_dispatch_identify)  (Implementation)


---
### server_client_dispatch_shell<!-- {{#callable_declaration:server_client_dispatch_shell}} -->
Dispatches a shell command to a client.
- **Description**: This function is used to send a shell command to a specified client. It retrieves the default shell from the global options and checks its validity. If the shell is not valid, it defaults to a standard shell. The function then sends the shell command to the client's peer and terminates the peer connection. This function should be called when a shell command needs to be executed on a client, ensuring that the client is ready to receive commands.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client to which the shell command will be dispatched. This parameter must not be null, and the client should be properly initialized and connected.
- **Output**: None
- **See also**: [`server_client_dispatch_shell`](#server_client_dispatch_shell)  (Implementation)


---
### server_client_report_theme<!-- {{#callable_declaration:server_client_report_theme}} -->
Reports the client's theme and requests color updates.
- **Description**: Use this function to update the client's theme to either light or dark mode and notify the client of the change. It also requests the current foreground and background colors from the terminal. This function should be called whenever the client's theme needs to be reported or updated. Ensure that the client structure is properly initialized before calling this function.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client whose theme is being reported. Must not be null.
    - `theme`: An `enum client_theme` value indicating the theme to be set, either `THEME_LIGHT` or `THEME_DARK`. Invalid values are not handled.
- **Output**: None
- **See also**: [`server_client_report_theme`](#server_client_report_theme)  (Implementation)


