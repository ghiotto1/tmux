# Purpose
The provided C source code is part of the `tmux` terminal multiplexer, specifically handling server-client interactions. This file is responsible for managing client connections, handling input events, and maintaining the state of client sessions. It includes functions for processing mouse and keyboard events, managing client overlays, and handling client-specific commands and messages. The code also deals with client lifecycle events such as creation, suspension, and detachment, and it manages the communication between the client and the server through message dispatching.

Key components of this code include the handling of mouse events, which are translated into key codes for further processing, and the management of client overlays, which are used to display temporary UI elements over the terminal. The code also includes functions for setting client-specific attributes like the terminal title and path, and for managing client-specific key tables, which determine how key events are interpreted. Additionally, the code handles the dispatching of commands and messages from clients, ensuring that they are processed in the correct context and with the appropriate permissions. Overall, this file is crucial for the interactive functionality of `tmux`, enabling it to respond to user inputs and manage multiple client sessions effectively.
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
    - `SCROLLBAR_UP`: Indicates the mouse is over the up button of a scrollbar.
    - `SCROLLBAR_SLIDER`: Indicates the mouse is over the slider of a scrollbar.
    - `SCROLLBAR_DOWN`: Indicates the mouse is over the down button of a scrollbar.
- **Description**: The `mouse_where` enumeration defines various locations within a user interface where a mouse event can occur. It is used to determine the specific area of the interface that the mouse is interacting with, such as panes, status lines, borders, or scrollbars. This enumeration helps in handling mouse events by providing a clear distinction of where the mouse is located, allowing for appropriate actions to be taken based on the mouse's position.


# Functions

---
### server_client_window_cmp<!-- {{#callable:server_client_window_cmp}} -->
Compares two `client_window` structures based on their `window` field.
- **Inputs**:
    - `cw1`: Pointer to the first `client_window` structure to compare.
    - `cw2`: Pointer to the second `client_window` structure to compare.
- **Control Flow**:
    - The function first checks if the `window` field of `cw1` is less than that of `cw2`, returning -1 if true.
    - If the `window` field of `cw1` is greater than that of `cw2`, it returns 1.
    - If both `window` fields are equal, it returns 0.
- **Output**:
    - Returns an integer: -1 if `cw1`'s window is less than `cw2`'s, 1 if greater, and 0 if they are equal.


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
    - After iterating through all clients, return the count `n`.
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
    - The function calls `server_client_clear_overlay(data)` to clear the overlay associated with the client.
    - No conditional logic or loops are present, as the function directly invokes another function.
- **Output**:
    - The function does not return a value; it modifies the state of the client by clearing its overlay.
- **Functions called**:
    - [`server_client_clear_overlay`](#server_client_clear_overlay)


---
### server_client_set_overlay<!-- {{#callable:server_client_set_overlay}} -->
Sets an overlay for a client with specified callbacks and a delay.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client to set the overlay for.
    - `delay`: An unsigned integer representing the delay in milliseconds before the overlay is activated.
    - `checkcb`: A callback function of type `overlay_check_cb` to check conditions for the overlay.
    - `modecb`: A callback function of type `overlay_mode_cb` to set the overlay mode.
    - `drawcb`: A callback function of type `overlay_draw_cb` to handle drawing the overlay.
    - `keycb`: A callback function of type `overlay_key_cb` to handle key events for the overlay.
    - `freecb`: A callback function of type `overlay_free_cb` to free resources associated with the overlay.
    - `resizecb`: A callback function of type `overlay_resize_cb` to handle resizing the overlay.
    - `data`: A pointer to user-defined data that can be passed to the overlay callbacks.
- **Control Flow**:
    - If the `overlay_draw` field of the client is not NULL, the existing overlay is cleared by calling [`server_client_clear_overlay`](#server_client_clear_overlay).
    - The delay is converted from milliseconds to a `struct timeval` format for use with the event timer.
    - If the overlay timer is already initialized, it is deleted using `evtimer_del`.
    - The overlay timer is set with `evtimer_set`, pointing to the `server_client_overlay_timer` function and passing the client pointer.
    - If the delay is not zero, the timer is added with `evtimer_add` using the previously set `struct timeval`.
    - The overlay callback functions are assigned to the respective fields in the client structure.
    - If the `overlay_check` callback is NULL, the TTY flags are updated to freeze the client display.
    - If the `overlay_mode` callback is NULL, the TTY flags are updated to hide the cursor.
    - The window focus is updated and the client is redrawn by calling [`server_redraw_client`](server-fn.c.driver.md#server_redraw_client).
- **Output**:
    - The function does not return a value; it modifies the state of the client by setting up an overlay with specified callbacks and a delay.
- **Functions called**:
    - [`server_client_clear_overlay`](#server_client_clear_overlay)
    - [`window_update_focus`](window.c.driver.md#window_update_focus)
    - [`server_redraw_client`](server-fn.c.driver.md#server_redraw_client)


---
### server_client_clear_overlay<!-- {{#callable:server_client_clear_overlay}} -->
Clears the overlay state for a given client.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client whose overlay is to be cleared.
- **Control Flow**:
    - Check if the `overlay_draw` pointer in the client structure is NULL; if it is, exit the function immediately.
    - If the `overlay_timer` event is initialized, delete it to stop any pending overlay actions.
    - If the `overlay_free` callback is set, invoke it to free any resources associated with the overlay.
    - Set all overlay-related pointers in the client structure to NULL to clear the overlay state.
    - Clear specific flags in the `tty` structure of the client to unfreeze the terminal and show the cursor.
    - If the client has an active session, update the focus of the current window.
    - Call [`server_redraw_client`](server-fn.c.driver.md#server_redraw_client) to refresh the client display.
- **Output**:
    - This function does not return a value; it modifies the state of the client by clearing its overlay.
- **Functions called**:
    - [`window_update_focus`](window.c.driver.md#window_update_focus)
    - [`server_redraw_client`](server-fn.c.driver.md#server_redraw_client)


---
### server_client_overlay_range<!-- {{#callable:server_client_overlay_range}} -->
The `server_client_overlay_range` function calculates the visible ranges of a specified overlay based on its position and dimensions.
- **Inputs**:
    - `x`: The x-coordinate of the overlay's top-left corner.
    - `y`: The y-coordinate of the overlay's top-left corner.
    - `sx`: The width of the overlay.
    - `sy`: The height of the overlay.
    - `px`: The x-coordinate of the point to check for visibility.
    - `py`: The y-coordinate of the point to check for visibility.
    - `nx`: The width of the range to check for visibility.
    - `r`: A pointer to a struct `overlay_ranges` where the results will be stored.
- **Control Flow**:
    - Initialize the output ranges in the `overlay_ranges` structure to zero.
    - Check if the y-coordinate of the point is outside the vertical bounds of the overlay; if so, set the output ranges to the input point and return.
    - If the point is to the left of the overlay, calculate the visible range to the left and store it in the output structure.
    - If the point is to the right of the overlay, calculate the visible range to the right and store it in the output structure.
- **Output**:
    - The function populates the `overlay_ranges` structure with up to two visible ranges based on the input coordinates and dimensions.


---
### server_client_check_nested<!-- {{#callable:server_client_check_nested}} -->
Checks if a client is nested within a tmux session.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client to check.
- **Control Flow**:
    - The function first attempts to find the environment variable 'TMUX' in the client's environment.
    - If the 'TMUX' variable is not found or is empty, the function returns 0, indicating the client is not nested.
    - If the 'TMUX' variable is found, the function iterates over all window panes in the `window_pane_tree`.
    - For each window pane, it compares the `tty` of the pane with the `ttyname` of the client.
    - If a match is found, the function returns 1, indicating the client is nested.
    - If no matches are found after checking all panes, the function returns 0.
- **Output**:
    - Returns 1 if the client is nested within a tmux session, otherwise returns 0.
- **Functions called**:
    - [`environ_find`](environ.c.driver.md#environ_find)


---
### server_client_set_key_table<!-- {{#callable:server_client_set_key_table}} -->
Sets the key table for a client, updating it based on the provided name or the default if none is given.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client whose key table is to be set.
    - `name`: A string representing the name of the key table to set; if NULL, the default key table for the client is used.
- **Control Flow**:
    - Checks if the `name` parameter is NULL; if so, retrieves the default key table name for the client using [`server_client_get_key_table`](#server_client_get_key_table).
    - Calls [`key_bindings_unref_table`](key-bindings.c.driver.md#key_bindings_unref_table) to release the current key table associated with the client.
    - Retrieves the new key table using [`key_bindings_get_table`](key-bindings.c.driver.md#key_bindings_get_table) with the specified name and increments its reference count.
    - Updates the `activity_time` of the key table to the current time using `gettimeofday`, and if it fails, calls `fatal` to terminate the program.
- **Output**:
    - The function does not return a value; it modifies the state of the `client` structure by updating its key table and setting the activity time.
- **Functions called**:
    - [`server_client_get_key_table`](#server_client_get_key_table)
    - [`key_bindings_unref_table`](key-bindings.c.driver.md#key_bindings_unref_table)
    - [`key_bindings_get_table`](key-bindings.c.driver.md#key_bindings_get_table)


---
### server_client_key_table_activity_diff<!-- {{#callable:server_client_key_table_activity_diff}} -->
Calculates the difference in milliseconds between a client's last activity time and the key table's last activity time.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client whose activity time is being compared.
- **Control Flow**:
    - The function uses `timersub` to compute the difference between the `activity_time` of the client and the `activity_time` of the client's key table.
    - The result is stored in a `struct timeval` variable named `diff`.
    - The function then converts the time difference from seconds and microseconds to milliseconds.
- **Output**:
    - Returns a `uint64_t` value representing the time difference in milliseconds.


---
### server_client_get_key_table<!-- {{#callable:server_client_get_key_table}} -->
Retrieves the key table name associated with a given client.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client whose key table is being queried.
- **Control Flow**:
    - The function first retrieves the `session` associated with the client `c`.
    - If the session is `NULL`, it returns the string 'root'.
    - It then attempts to get the key table name from the session's options using [`options_get_string`](options.c.driver.md#options_get_string).
    - If the retrieved name is an empty string, it returns 'root'.
    - Otherwise, it returns the retrieved key table name.
- **Output**:
    - Returns a pointer to a string containing the name of the key table, or 'root' if no valid key table is found.
- **Functions called**:
    - [`options_get_string`](options.c.driver.md#options_get_string)


---
### server_client_is_default_key_table<!-- {{#callable:server_client_is_default_key_table}} -->
The `server_client_is_default_key_table` function checks if a given `key_table` is the default key table for a specified client.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client whose key table is being checked.
    - `table`: A pointer to a `key_table` structure that is being compared to the client's default key table.
- **Control Flow**:
    - The function uses `strcmp` to compare the name of the provided `key_table` with the name of the key table returned by `server_client_get_key_table(c)`.
    - It returns the result of the comparison, which is either true (1) if they match or false (0) if they do not.
- **Output**:
    - The function outputs an integer value: 1 if the provided `key_table` is the default for the client, and 0 otherwise.
- **Functions called**:
    - [`server_client_get_key_table`](#server_client_get_key_table)


---
### server_client_create<!-- {{#callable:server_client_create}} -->
Creates a new client structure and initializes its properties.
- **Inputs**:
    - `fd`: The file descriptor for the client connection.
- **Control Flow**:
    - Sets the file descriptor to non-blocking mode using [`setblocking`](tmux.c.driver.md#setblocking).
    - Allocates memory for a new `client` structure using [`xcalloc`](xmalloc.c.driver.md#xcalloc).
    - Initializes the `client` structure's properties, including references, peer connection, and timestamps.
    - Creates a new environment for the client using [`environ_create`](environ.c.driver.md#environ_create).
    - Initializes the command queue and red-black trees for windows and files.
    - Sets default terminal dimensions and theme.
    - Initializes the client status and key bindings.
    - Sets up timers for repeat and click events.
    - Inserts the new client into the global clients list and logs the creation.
- **Output**:
    - Returns a pointer to the newly created `client` structure.
- **Functions called**:
    - [`setblocking`](tmux.c.driver.md#setblocking)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`proc_add_peer`](proc.c.driver.md#proc_add_peer)
    - [`environ_create`](environ.c.driver.md#environ_create)
    - [`cmdq_new`](cmd-queue.c.driver.md#cmdq_new)
    - [`status_init`](status.c.driver.md#status_init)
    - [`key_bindings_get_table`](key-bindings.c.driver.md#key_bindings_get_table)


---
### server_client_open<!-- {{#callable:server_client_open}} -->
The `server_client_open` function initializes a client terminal connection if certain conditions are met.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client whose terminal is to be opened.
    - `cause`: A pointer to a string that will hold an error message if the function fails.
- **Control Flow**:
    - The function first checks if the client is a control client by evaluating the `CLIENT_CONTROL` flag; if true, it returns 0 immediately.
    - It then compares the client's `ttyname` with the default terminal path and checks if the terminal is a valid tty for standard input, output, or error; if any match, it sets an error message in `cause` and returns -1.
    - Next, it checks if the `CLIENT_TERMINAL` flag is set; if not, it sets an error message indicating that the client is not a terminal and returns -1.
    - If the previous checks pass, it attempts to open the terminal associated with the client by calling [`tty_open`](tty.c.driver.md#tty_open); if this fails, it returns -1.
    - If all checks are successful, the function returns 0, indicating that the terminal was opened successfully.
- **Output**:
    - The function returns 0 on success, -1 on failure, and sets an error message in `cause` if applicable.
- **Functions called**:
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`tty_open`](tty.c.driver.md#tty_open)


---
### server_client_attached_lost<!-- {{#callable:server_client_attached_lost}} -->
The `server_client_attached_lost` function handles the scenario when a client that was attached to a session is lost.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the lost client.
- **Control Flow**:
    - Logs the loss of the attached client using `log_debug`.
    - Iterates over all windows to find any window where the lost client was the latest attached client.
    - For each window found, it iterates over all clients to find the one with the most recent activity time that is not the lost client.
    - If a suitable client is found, it updates the latest client for that window using [`server_client_update_latest`](#server_client_update_latest).
- **Output**:
    - The function does not return a value; it updates the state of the server by potentially changing which client is considered the latest for any windows that had the lost client attached.
- **Functions called**:
    - [`server_client_update_latest`](#server_client_update_latest)


---
### server_client_set_session<!-- {{#callable:server_client_set_session}} -->
Sets the session for a client and updates various related states.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client whose session is being set.
    - `s`: A pointer to the `session` structure that is to be assigned to the client.
- **Control Flow**:
    - Store the current session of the client in `old`.
    - If the new session `s` is not NULL and the current session is different from `s`, update `last_session` to the current session.
    - If `s` is NULL, set `last_session` to NULL.
    - Set the client's session to `s` and mark the client as focused.
    - If there was an old session and it has a current window, update the focus of that window.
    - If the new session `s` is not NULL, perform several updates including recalculating sizes, updating focus, and notifying the client.
    - Finally, check for unattached clients and update the socket.
- **Output**:
    - The function does not return a value but modifies the state of the client and its session, updating various flags and notifying the client of changes.
- **Functions called**:
    - [`window_update_focus`](window.c.driver.md#window_update_focus)
    - [`recalculate_sizes`](resize.c.driver.md#recalculate_sizes)
    - [`session_update_activity`](session.c.driver.md#session_update_activity)
    - [`session_theme_changed`](session.c.driver.md#session_theme_changed)
    - [`alerts_check_session`](alerts.c.driver.md#alerts_check_session)
    - [`tty_update_client_offset`](tty.c.driver.md#tty_update_client_offset)
    - [`status_timer_start`](status.c.driver.md#status_timer_start)
    - [`notify_client`](notify.c.driver.md#notify_client)
    - [`server_redraw_client`](server-fn.c.driver.md#server_redraw_client)
    - [`server_check_unattached`](server-fn.c.driver.md#server_check_unattached)
    - [`server_update_socket`](server.c.driver.md#server_update_socket)


---
### server_client_lost<!-- {{#callable:server_client_lost}} -->
The `server_client_lost` function handles the cleanup and disconnection process for a client that has lost connection to the server.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client that has lost connection.
- **Control Flow**:
    - Sets the `CLIENT_DEAD` flag for the client to indicate it is no longer active.
    - Clears any overlays, prompts, and messages associated with the client.
    - Iterates over the client's files and marks them as interrupted, firing done events for each.
    - Iterates over the client's windows, removing them from the client's window list and freeing their memory.
    - Removes the client from the global list of clients and logs the disconnection.
    - If the client was attached, it handles the loss of the attached client and notifies the system.
    - Cleans up various resources associated with the client, including terminal settings, clipboard, and environment variables.
    - Closes any open file descriptors associated with the client.
    - Calls functions to update the server state and check for unattached clients.
- **Output**:
    - The function does not return a value; it performs cleanup operations and modifies the state of the server and client structures.
- **Functions called**:
    - [`server_client_clear_overlay`](#server_client_clear_overlay)
    - [`status_prompt_clear`](status.c.driver.md#status_prompt_clear)
    - [`status_message_clear`](status.c.driver.md#status_message_clear)
    - [`file_fire_done`](file.c.driver.md#file_fire_done)
    - [`server_client_attached_lost`](#server_client_attached_lost)
    - [`notify_client`](notify.c.driver.md#notify_client)
    - [`control_stop`](control.c.driver.md#control_stop)
    - [`tty_free`](tty.c.driver.md#tty_free)
    - [`tty_term_free_list`](tty-term.c.driver.md#tty_term_free_list)
    - [`status_free`](status.c.driver.md#status_free)
    - [`key_bindings_unref_table`](key-bindings.c.driver.md#key_bindings_unref_table)
    - [`format_lost_client`](format.c.driver.md#format_lost_client)
    - [`environ_free`](environ.c.driver.md#environ_free)
    - [`proc_remove_peer`](proc.c.driver.md#proc_remove_peer)
    - [`server_client_unref`](#server_client_unref)
    - [`server_add_accept`](server.c.driver.md#server_add_accept)
    - [`recalculate_sizes`](resize.c.driver.md#recalculate_sizes)
    - [`server_check_unattached`](server-fn.c.driver.md#server_check_unattached)
    - [`server_update_socket`](server.c.driver.md#server_update_socket)


---
### server_client_unref<!-- {{#callable:server_client_unref}} -->
The `server_client_unref` function decrements the reference count of a client and schedules its memory to be freed if the reference count reaches zero.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client whose reference count is to be decremented.
- **Control Flow**:
    - Logs the current reference count of the client.
    - Decrements the `references` field of the client structure.
    - Checks if the reference count has reached zero.
    - If the reference count is zero, schedules the `server_client_free` function to be called once the event loop is ready.
- **Output**:
    - The function does not return a value; it modifies the state of the client and may schedule its deallocation.


---
### server_client_free<!-- {{#callable:server_client_free}} -->
The `server_client_free` function frees the resources associated with a client when it is no longer referenced.
- **Inputs**:
    - `fd`: An unused file descriptor, typically representing the client connection.
    - `events`: An unused event mask, indicating the types of events to listen for.
    - `arg`: A pointer to the client structure that needs to be freed.
- **Control Flow**:
    - Logs the debug message indicating the client being freed and its reference count.
    - Calls [`cmdq_free`](cmd-queue.c.driver.md#cmdq_free) to free the command queue associated with the client.
    - Checks if the reference count of the client is zero.
    - If the reference count is zero, it frees the client's name and the client structure itself.
- **Output**:
    - The function does not return a value; it performs cleanup by freeing memory associated with the client.
- **Functions called**:
    - [`cmdq_free`](cmd-queue.c.driver.md#cmdq_free)


---
### server_client_suspend<!-- {{#callable:server_client_suspend}} -->
Suspends a client by stopping its terminal and marking it as suspended.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client to be suspended.
- **Control Flow**:
    - Checks if the `session` associated with the client is NULL or if the client has unattached flags set.
    - If either condition is true, the function returns immediately without performing any actions.
    - Calls [`tty_stop_tty`](tty.c.driver.md#tty_stop_tty) to stop the terminal associated with the client.
    - Sets the `CLIENT_SUSPENDED` flag in the client's flags.
    - Sends a suspend message to the peer of the client using [`proc_send`](proc.c.driver.md#proc_send).
- **Output**:
    - The function does not return a value; it modifies the state of the client and sends a message to its peer.
- **Functions called**:
    - [`tty_stop_tty`](tty.c.driver.md#tty_stop_tty)
    - [`proc_send`](proc.c.driver.md#proc_send)


---
### server_client_detach<!-- {{#callable:server_client_detach}} -->
Detaches a client from its session.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client to be detached.
    - `msgtype`: An enumeration value of type `msgtype` indicating the type of message to send upon detachment.
- **Control Flow**:
    - Checks if the session pointer in the client structure is NULL or if the client has flags set that prevent detachment.
    - If the checks pass, it sets the CLIENT_EXIT flag on the client.
    - Sets the exit type to CLIENT_EXIT_DETACH.
    - Stores the session name in the exit_session field of the client.
- **Output**:
    - The function does not return a value; it modifies the state of the client by setting its exit type and message.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### server_client_exec<!-- {{#callable:server_client_exec}} -->
Executes a command in the context of a client session.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client executing the command.
    - `cmd`: A string containing the command to be executed.
- **Control Flow**:
    - Checks if the command string is empty; if so, it returns immediately.
    - Calculates the size of the command and retrieves the default shell from the session options or global options.
    - Validates the shell path using [`checkshell`](tmux.c.driver.md#checkshell); if invalid, it defaults to `_PATH_BSHELL`.
    - Allocates memory for a message that combines the command and shell path.
    - Copies the command and shell path into the allocated message buffer.
    - Sends the message to the client's peer using [`proc_send`](proc.c.driver.md#proc_send) with the message type `MSG_EXEC`.
    - Frees the allocated message buffer after sending.
- **Output**:
    - The function does not return a value; it sends a message to execute the command in the context of the client.
- **Functions called**:
    - [`options_get_string`](options.c.driver.md#options_get_string)
    - [`checkshell`](tmux.c.driver.md#checkshell)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`proc_send`](proc.c.driver.md#proc_send)


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
    - Check if the mouse coordinates are within the pane or scrollbar area.
    - If within the scrollbar area, determine if the mouse is above, on, or below the scrollbar slider.
    - If not within the scrollbar, check if the mouse is over the pane borders if the window is not zoomed.
    - Return the appropriate enum value indicating the mouse's position relative to the pane or scrollbar.
- **Output**:
    - Returns an enum value of type `mouse_where` indicating the mouse's position: whether it is in the pane, on the scrollbar, or outside of it.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`window_pane_show_scrollbar`](window.c.driver.md#window_pane_show_scrollbar)


---
### server_client_check_mouse<!-- {{#callable:server_client_check_mouse}} -->
The `server_client_check_mouse` function processes mouse events for a client in a terminal multiplexer.
- **Inputs**:
    - `struct client *c`: A pointer to the `client` structure representing the client that is sending the mouse event.
    - `struct key_event *event`: A pointer to the `key_event` structure containing details about the mouse event.
- **Control Flow**:
    - The function begins by logging the mouse event details.
    - It determines the type of mouse event (e.g., MOVE, DOWN, UP, DRAG, WHEEL, DOUBLE, TRIPLE) based on the event data.
    - If the event is a double-click, it sets the type to DOUBLE and logs the event.
    - For drag events, it checks if the drag is ongoing and updates the position accordingly.
    - If the event is a mouse wheel event, it sets the type to WHEEL.
    - For mouse release events, it sets the type to UP and logs the position.
    - If no specific type is identified, it defaults to DOWN and logs the event.
    - The function checks if the mouse event occurred on the status line and updates the mouse event structure accordingly.
    - If the event is not on the status line, it checks if the mouse is within a pane, border, or scrollbar.
    - Finally, it converts the mouse event into a key code and applies any modifiers before returning the key code.
- **Output**:
    - The function returns a `key_code` representing the processed mouse event, or `KEYC_UNKNOWN` if the event could not be processed.
- **Functions called**:
    - [`status_at_line`](status.c.driver.md#status_at_line)
    - [`status_line_size`](status.c.driver.md#status_line_size)
    - [`status_get_range`](status.c.driver.md#status_get_range)
    - [`window_pane_find_by_id`](window.c.driver.md#window_pane_find_by_id)
    - [`winlink_find_by_index`](window.c.driver.md#winlink_find_by_index)
    - [`session_find_by_id`](session.c.driver.md#session_find_by_id)
    - [`tty_window_offset`](tty.c.driver.md#tty_window_offset)
    - [`window_get_active_at`](window.c.driver.md#window_get_active_at)
    - [`server_client_check_mouse_in_pane`](#server_client_check_mouse_in_pane)
    - [`log_get_level`](log.c.driver.md#log_get_level)
    - [`key_string_lookup_key`](key-string.c.driver.md#key_string_lookup_key)


---
### server_client_is_bracket_paste<!-- {{#callable:server_client_is_bracket_paste}} -->
The `server_client_is_bracket_paste` function manages the bracket pasting state for a client based on key events.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client whose bracket pasting state is being managed.
    - `key`: A `key_code` representing the key event that is being processed to determine if it is a paste start or end.
- **Control Flow**:
    - The function first checks if the `key` corresponds to the `KEYC_PASTE_START` constant, indicating the start of a bracket paste operation.
    - If it matches, the function sets the `CLIENT_BRACKETPASTING` flag in the client's `flags` and logs the action.
    - Next, it checks if the `key` corresponds to the `KEYC_PASTE_END` constant, indicating the end of a bracket paste operation.
    - If it matches, the function clears the `CLIENT_BRACKETPASTING` flag from the client's `flags` and logs the action.
    - If neither condition is met, the function returns the current state of the `CLIENT_BRACKETPASTING` flag as a boolean value.
- **Output**:
    - The function returns 0 if a paste start or end was detected, or a non-zero value indicating the current state of bracket pasting (1 if active, 0 if not).


---
### server_client_is_assume_paste<!-- {{#callable:server_client_is_assume_paste}} -->
Determines if the client is likely pasting text based on the time between recent activity.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client whose activity is being checked.
- **Control Flow**:
    - Checks if the `CLIENT_BRACKETPASTING` flag is set on the client; if so, returns 0 immediately.
    - Retrieves the `assume-paste-time` option from the client's session options; if it is 0, returns 0.
    - Calculates the time difference between the current activity time and the last activity time.
    - If the time difference is less than the configured `assume-paste-time`, it checks if the `CLIENT_ASSUMEPASTING` flag is set.
    - If the `CLIENT_ASSUMEPASTING` flag is not set, it sets this flag and logs that pasting is assumed.
    - If the time difference exceeds the configured time, it clears the `CLIENT_ASSUMEPASTING` flag if it was set and logs that pasting is off.
- **Output**:
    - Returns 1 if the client is assumed to be pasting, 0 otherwise.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)


---
### server_client_update_latest<!-- {{#callable:server_client_update_latest}} -->
Updates the latest client associated with the current window.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client to be updated.
- **Control Flow**:
    - Checks if the `session` of the client `c` is NULL; if so, the function returns immediately.
    - Retrieves the current `window` associated with the client's session.
    - If the current window's `latest` client is already `c`, the function returns without making changes.
    - Updates the `latest` client of the window to `c`.
    - If the window's option for `window-size` is set to `WINDOW_SIZE_LATEST`, it calls [`recalculate_size`](resize.c.driver.md#recalculate_size) to adjust the window size.
    - Finally, it sends a notification to the client indicating that it is now the active client.
- **Output**:
    - The function does not return a value; it modifies the state of the window and client directly.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`recalculate_size`](resize.c.driver.md#recalculate_size)
    - [`notify_client`](notify.c.driver.md#notify_client)


---
### server_client_repeat_time<!-- {{#callable:server_client_repeat_time}} -->
Calculates the repeat time for a key binding based on client and session options.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client requesting the repeat time.
    - `bd`: A pointer to the `key_binding` structure representing the key binding for which the repeat time is being calculated.
- **Control Flow**:
    - Checks if the `KEY_BINDING_REPEAT` flag is set in the key binding; if not, returns 0.
    - Retrieves the repeat time from the session options; if it is 0, returns 0.
    - Checks if the client is not in repeat mode or if the last key pressed is different from the current key; if true, retrieves the initial repeat time.
    - If the initial repeat time is not 0, it sets the repeat time to the initial value.
    - Returns the calculated repeat time.
- **Output**:
    - Returns the repeat time as an unsigned integer, which can be 0 if repeat is not allowed or if no repeat time is configured.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)


---
### server_client_key_callback<!-- {{#callable:server_client_key_callback}} -->
Handles key input events from a client, processing them according to the current session and key bindings.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item associated with the key event.
    - `data`: A pointer to a `key_event` structure containing the details of the key event.
- **Control Flow**:
    - Checks if the client is attached and able to accept input; if not, it exits early.
    - Updates the client's last activity time and checks for mouse events.
    - Processes theme reporting keys and determines the affected pane based on the key event.
    - Handles mouse key forwarding and bracket pasting conditions.
    - Determines the current key table based on the pane's mode and checks for prefix keys.
    - Attempts to find a key binding in the current key table and executes it if found.
    - If no binding is found, it checks for the ANY key binding and tries again.
    - If still no match, it forwards the key to the appropriate window pane or handles paste events.
    - Finally, updates the client's latest activity if applicable and frees the event data.
- **Output**:
    - Returns a command return value indicating the result of processing the key event, typically `CMD_RETURN_NORMAL`.
- **Functions called**:
    - [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client)
    - [`session_update_activity`](session.c.driver.md#session_update_activity)
    - [`server_client_check_mouse`](#server_client_check_mouse)
    - [`server_client_report_theme`](#server_client_report_theme)
    - [`cmd_find_from_mouse`](cmd-find.c.driver.md#cmd_find_from_mouse)
    - [`cmd_find_from_client`](cmd-find.c.driver.md#cmd_find_from_client)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`server_client_is_bracket_paste`](#server_client_is_bracket_paste)
    - [`server_client_is_assume_paste`](#server_client_is_assume_paste)
    - [`server_client_is_default_key_table`](#server_client_is_default_key_table)
    - [`key_bindings_get_table`](key-bindings.c.driver.md#key_bindings_get_table)
    - [`server_client_set_key_table`](#server_client_set_key_table)
    - [`server_status_client`](server-fn.c.driver.md#server_status_client)
    - [`key_bindings_get`](key-bindings.c.driver.md#key_bindings_get)
    - [`server_client_key_table_activity_diff`](#server_client_key_table_activity_diff)
    - [`server_client_repeat_time`](#server_client_repeat_time)
    - [`key_bindings_dispatch`](key-bindings.c.driver.md#key_bindings_dispatch)
    - [`key_bindings_unref_table`](key-bindings.c.driver.md#key_bindings_unref_table)
    - [`window_pane_key`](window.c.driver.md#window_pane_key)
    - [`window_pane_paste`](window.c.driver.md#window_pane_paste)
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
    - [`status_message_clear`](status.c.driver.md#status_message_clear)
    - [`server_client_clear_overlay`](#server_client_clear_overlay)
    - [`status_prompt_key`](status.c.driver.md#status_prompt_key)
    - [`cmdq_append`](cmd-queue.c.driver.md#cmdq_append)


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
    - [`check_window_name`](names.c.driver.md#check_window_name)
    - [`window_pane_send_theme_update`](window.c.driver.md#window_pane_send_theme_update)


---
### server_client_check_window_resize<!-- {{#callable:server_client_check_window_resize}} -->
Checks if a window needs to be resized and performs the resize if necessary.
- **Inputs**:
    - `w`: A pointer to a `struct window` representing the window to check for resizing.
- **Control Flow**:
    - Checks if the `WINDOW_RESIZE` flag is set for the window `w`.
    - If the flag is not set, the function returns immediately.
    - Iterates through the linked list of `winlink` structures associated with the window `w`.
    - Finds the first `winlink` that has an attached session and is the current window for that session.
    - If no such `winlink` is found, the function returns.
    - Logs a debug message indicating that the window is being resized.
    - Calls the [`resize_window`](resize.c.driver.md#resize_window) function to perform the actual resizing of the window with new dimensions.
- **Output**:
    - The function does not return a value; it modifies the window's size if conditions are met.
- **Functions called**:
    - [`resize_window`](resize.c.driver.md#resize_window)


---
### server_client_resize_timer<!-- {{#callable:server_client_resize_timer}} -->
The `server_client_resize_timer` function handles the expiration of a resize timer for a window pane.
- **Inputs**:
    - `fd`: An unused file descriptor, typically for event handling.
    - `events`: An unused short representing event types.
    - `data`: A pointer to the `window_pane` structure associated with the timer.
- **Control Flow**:
    - The function begins by logging a debug message indicating that the resize timer has expired for the specific window pane identified by its ID.
    - It then calls `evtimer_del` to remove the associated resize timer from the event loop, ensuring that it will not trigger again.
- **Output**:
    - The function does not return a value; it performs actions related to logging and timer management.


---
### server_client_check_pane_resize<!-- {{#callable:server_client_check_pane_resize}} -->
The `server_client_check_pane_resize` function manages the resizing of a window pane based on queued resize requests.
- **Inputs**:
    - `wp`: A pointer to a `window_pane` structure representing the pane that may need resizing.
- **Control Flow**:
    - Checks if the resize queue for the pane is empty; if so, it returns immediately.
    - Initializes a resize timer if it hasn't been set and checks if it's already pending.
    - Logs the need for resizing and iterates through the resize queue to log queued resize requests.
    - Handles three cases based on the number of resize requests in the queue: applying a single resize, discarding previous resizes if the final size differs, or applying the first resize if the final size is the same as the initial size.
    - Updates the timer for the next resize check.
- **Output**:
    - The function does not return a value; it modifies the state of the window pane by sending resize commands and managing the resize queue.
- **Functions called**:
    - [`window_pane_send_resize`](window.c.driver.md#window_pane_send_resize)


---
### server_client_check_pane_buffer<!-- {{#callable:server_client_check_pane_buffer}} -->
The `server_client_check_pane_buffer` function manages the input buffer of a window pane, adjusting its size based on the data consumption by attached clients.
- **Inputs**:
    - `wp`: A pointer to a `struct window_pane` representing the pane whose buffer is being checked.
- **Control Flow**:
    - Calculate the minimum size of data that can be removed from the buffer based on the pane's offset and the offsets of attached clients.
    - Iterate through all attached clients to determine their data usage and adjust the minimum size accordingly.
    - If no clients are attached, set a flag to indicate that the pane is off.
    - Drain the buffer by removing the calculated minimum size of data.
    - Adjust the base offset of the pane, ensuring it does not overflow.
    - Enable or disable reading from the event based on whether clients can consume remaining data.
- **Output**:
    - The function does not return a value but modifies the state of the pane's buffer and its reading capabilities based on the presence of attached clients.
- **Functions called**:
    - [`control_pane_offset`](control.c.driver.md#control_pane_offset)
    - [`window_pane_get_new_data`](window.c.driver.md#window_pane_get_new_data)


---
### server_client_reset_state<!-- {{#callable:server_client_reset_state}} -->
Resets the state of a client in the server, updating cursor position, terminal modes, and screen attributes.
- **Inputs**:
    - `struct client *c`: A pointer to the `client` structure representing the client whose state is to be reset.
- **Control Flow**:
    - Checks if the client is in control or suspended mode; if so, it returns early without making changes.
    - Disables the block flag on the terminal associated with the client.
    - Determines the current screen mode based on whether an overlay is active or if a prompt string is present.
    - Resets the terminal's region and margin settings.
    - Calculates the cursor position based on the current pane or prompt string.
    - Updates the terminal mode based on the calculated mode and resets terminal attributes.
    - Sends a synchronization end signal to the terminal.
- **Output**:
    - The function does not return a value; it modifies the state of the client and its associated terminal directly.
- **Functions called**:
    - [`server_client_get_pane`](#server_client_get_pane)
    - [`log_get_level`](log.c.driver.md#log_get_level)
    - [`screen_mode_to_string`](screen.c.driver.md#screen_mode_to_string)
    - [`tty_region_off`](tty.c.driver.md#tty_region_off)
    - [`tty_margin_off`](tty.c.driver.md#tty_margin_off)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`status_line_size`](status.c.driver.md#status_line_size)
    - [`tty_window_offset`](tty.c.driver.md#tty_window_offset)
    - [`status_at_line`](status.c.driver.md#status_at_line)
    - [`tty_cursor`](tty.c.driver.md#tty_cursor)
    - [`tty_update_mode`](tty.c.driver.md#tty_update_mode)
    - [`tty_reset`](tty.c.driver.md#tty_reset)
    - [`tty_sync_end`](tty.c.driver.md#tty_sync_end)


---
### server_client_repeat_timer<!-- {{#callable:server_client_repeat_timer}} -->
The `server_client_repeat_timer` function handles the expiration of a repeat timer for a client, resetting the client's key table and flags.
- **Inputs**:
    - `fd`: An unused file descriptor, typically for event handling.
    - `events`: An unused short representing event types.
    - `data`: A pointer to the `client` structure associated with the timer.
- **Control Flow**:
    - The function checks if the `CLIENT_REPEAT` flag is set for the client.
    - If the flag is set, it calls [`server_client_set_key_table`](#server_client_set_key_table) to reset the key table to NULL.
    - The `CLIENT_REPEAT` flag is then cleared from the client's flags.
    - Finally, it calls [`server_status_client`](server-fn.c.driver.md#server_status_client) to update the client's status.
- **Output**:
    - The function does not return a value; it modifies the state of the client by resetting its key table and updating its status.
- **Functions called**:
    - [`server_client_set_key_table`](#server_client_set_key_table)
    - [`server_status_client`](server-fn.c.driver.md#server_status_client)


---
### server_client_click_timer<!-- {{#callable:server_client_click_timer}} -->
The `server_client_click_timer` function handles the expiration of a click timer for a client, determining if a double-click event should be processed.
- **Inputs**:
    - `fd`: An unused file descriptor, typically for event handling.
    - `events`: An unused event mask, indicating the types of events that triggered the callback.
    - `data`: A pointer to the `client` structure associated with the client whose click timer has expired.
- **Control Flow**:
    - Logs a debug message indicating that the click timer has expired.
    - Checks if the client has the `CLIENT_TRIPLECLICK` flag set, indicating that a third click is expected.
    - If the `CLIENT_TRIPLECLICK` flag is set, it allocates memory for a new `key_event` structure and sets it to represent a double-click event.
    - Copies the last click event data into the new event structure.
    - Calls [`server_client_handle_key`](#server_client_handle_key) to process the double-click event, and if it fails, frees the allocated memory.
    - Clears the `CLIENT_DOUBLECLICK` and `CLIENT_TRIPLECLICK` flags from the client's flags.
- **Output**:
    - The function does not return a value; it modifies the state of the client and may trigger further processing of a double-click event.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`server_client_handle_key`](#server_client_handle_key)


---
### server_client_check_exit<!-- {{#callable:server_client_check_exit}} -->
The `server_client_check_exit` function checks if a client should exit and handles the exit process accordingly.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client that is being checked for exit conditions.
- **Control Flow**:
    - The function first checks if the client is already dead or has exited, returning early if so.
    - It checks if the client is a control client and discards control messages if necessary.
    - It iterates through the client's files to ensure there are no pending buffers; if any exist, it returns early.
    - If the client is ready to exit, it sets the `CLIENT_EXITED` flag and processes the exit type.
    - Depending on the exit type, it sends appropriate messages to the peer, including exit return values or shutdown messages.
    - Finally, it frees any allocated exit session and message strings.
- **Output**:
    - The function does not return a value but sends messages to the client's peer based on the exit type and cleans up resources associated with the client.
- **Functions called**:
    - [`control_discard`](control.c.driver.md#control_discard)
    - [`control_all_done`](control.c.driver.md#control_all_done)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`proc_send`](proc.c.driver.md#proc_send)


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
    - `c`: A pointer to a `client` structure representing the client whose modes are to be checked.
- **Control Flow**:
    - The function first checks if the client's flags indicate that it is either a control client or suspended; if so, it returns immediately.
    - Next, it checks if the `CLIENT_REDRAWSTATUS` flag is not set; if it is not set, the function returns.
    - The function then iterates over each pane in the current window of the client using `TAILQ_FOREACH`.
    - For each pane, it retrieves the first mode entry and checks if it is not NULL and if it has an update function.
    - If both conditions are met, it calls the update function for that mode entry.
- **Output**:
    - The function does not return a value; it performs updates on the modes of the panes directly.


---
### server_client_check_redraw<!-- {{#callable:server_client_check_redraw}} -->
The `server_client_check_redraw` function manages the redraw process for a client in a terminal multiplexer, determining what needs to be redrawn based on the client's state and the state of its associated panes.
- **Inputs**:
    - `struct client *c`: A pointer to the `client` structure representing the client that is being checked for redraw.
- **Control Flow**:
    - The function first checks if the client is in a state that allows for redrawing (not suspended or a control client).
    - It logs the redraw flags if any are set for the client.
    - It checks if any panes need to be redrawn or if the entire window needs to be redrawn based on the flags set in the client.
    - If there is data left in the output buffer, it defers the redraw by setting a timer.
    - If no data is left, it proceeds to redraw the necessary panes or the entire window based on the flags.
    - Finally, it updates the terminal mode and resets the flags related to redrawing.
- **Output**:
    - The function does not return a value; instead, it modifies the state of the client and its associated panes, triggering redraws as necessary and logging the actions taken.
- **Functions called**:
    - [`screen_redraw_pane`](screen-redraw.c.driver.md#screen_redraw_pane)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`server_client_set_title`](#server_client_set_title)
    - [`server_client_set_path`](#server_client_set_path)
    - [`screen_redraw_screen`](screen-redraw.c.driver.md#screen_redraw_screen)
    - [`tty_update_mode`](tty.c.driver.md#tty_update_mode)


---
### server_client_set_title<!-- {{#callable:server_client_set_title}} -->
Sets the title of a client based on a specified template.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client whose title is to be set.
- **Control Flow**:
    - Retrieve the title template string from the session options using [`options_get_string`](options.c.driver.md#options_get_string).
    - Create a new `format_tree` using [`format_create`](format.c.driver.md#format_create) and set default values with [`format_defaults`](format.c.driver.md#format_defaults).
    - Expand the title template into a string using [`format_expand_time`](format.c.driver.md#format_expand_time).
    - Check if the new title differs from the current title; if so, free the old title and set the new title.
    - Update the terminal title using [`tty_set_title`](tty.c.driver.md#tty_set_title).
    - Free the allocated title string and the format tree.
- **Output**:
    - The function does not return a value; it modifies the client's title and updates the terminal display accordingly.
- **Functions called**:
    - [`options_get_string`](options.c.driver.md#options_get_string)
    - [`format_create`](format.c.driver.md#format_create)
    - [`format_defaults`](format.c.driver.md#format_defaults)
    - [`format_expand_time`](format.c.driver.md#format_expand_time)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`tty_set_title`](tty.c.driver.md#tty_set_title)
    - [`format_free`](format.c.driver.md#format_free)


---
### server_client_set_path<!-- {{#callable:server_client_set_path}} -->
Sets the path for a client based on the current session's active window.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client whose path is to be set.
- **Control Flow**:
    - Checks if the current window (`curw`) of the session associated with the client is NULL; if so, the function returns immediately.
    - Retrieves the path from the active window's base path; if it is NULL, an empty string is assigned to `path`.
    - Compares the current path of the client with the retrieved path; if they differ, the client's path is updated.
    - Frees the existing path memory and duplicates the new path string into the client's path.
    - Calls [`tty_set_path`](tty.c.driver.md#tty_set_path) to update the terminal's path with the new path.
- **Output**:
    - The function does not return a value; it modifies the client's path and updates the terminal's path accordingly.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`tty_set_path`](tty.c.driver.md#tty_set_path)


---
### server_client_dispatch<!-- {{#callable:server_client_dispatch}} -->
The `server_client_dispatch` function processes incoming messages from a client and dispatches them to the appropriate handler based on the message type.
- **Inputs**:
    - `imsg`: A pointer to an `imsg` structure containing the message data received from the client.
    - `arg`: A pointer to a `client` structure representing the client that sent the message.
- **Control Flow**:
    - The function first checks if the client is marked as dead; if so, it returns immediately.
    - If the `imsg` pointer is NULL, it calls [`server_client_lost`](#server_client_lost) to handle the lost client scenario.
    - It calculates the length of the data payload in the message by subtracting the header size from the total length.
    - A switch statement is used to determine the type of message based on `imsg->hdr.type`, and the corresponding handler function is called.
    - For specific message types like `MSG_IDENTIFY_*`, it calls [`server_client_dispatch_identify`](#server_client_dispatch_identify) to handle identification messages.
    - For `MSG_COMMAND`, it calls [`server_client_dispatch_command`](#server_client_dispatch_command) to process command messages.
    - For `MSG_RESIZE`, it checks the data length and handles resizing the client's terminal if applicable.
    - For `MSG_EXITING`, it processes the exit request and cleans up the session.
    - For `MSG_WAKEUP` and `MSG_UNLOCK`, it checks if the client is suspended and resumes activity if necessary.
    - For `MSG_SHELL`, it dispatches to [`server_client_dispatch_shell`](#server_client_dispatch_shell) to handle shell requests.
    - For `MSG_WRITE_READY`, `MSG_READ`, and `MSG_READ_DONE`, it calls respective file handling functions.
- **Output**:
    - The function does not return a value; instead, it modifies the state of the client and may send responses back to the client based on the processed message.
- **Functions called**:
    - [`server_client_lost`](#server_client_lost)
    - [`server_client_dispatch_identify`](#server_client_dispatch_identify)
    - [`server_client_dispatch_command`](#server_client_dispatch_command)
    - [`server_client_update_latest`](#server_client_update_latest)
    - [`tty_resize`](tty.c.driver.md#tty_resize)
    - [`tty_repeat_requests`](tty.c.driver.md#tty_repeat_requests)
    - [`recalculate_sizes`](resize.c.driver.md#recalculate_sizes)
    - [`server_client_clear_overlay`](#server_client_clear_overlay)
    - [`server_redraw_client`](server-fn.c.driver.md#server_redraw_client)
    - [`notify_client`](notify.c.driver.md#notify_client)
    - [`server_client_set_session`](#server_client_set_session)
    - [`tty_close`](tty.c.driver.md#tty_close)
    - [`proc_send`](proc.c.driver.md#proc_send)
    - [`tty_start_tty`](tty.c.driver.md#tty_start_tty)
    - [`session_update_activity`](session.c.driver.md#session_update_activity)
    - [`server_client_dispatch_shell`](#server_client_dispatch_shell)
    - [`file_write_ready`](file.c.driver.md#file_write_ready)
    - [`file_read_data`](file.c.driver.md#file_read_data)
    - [`file_read_done`](file.c.driver.md#file_read_done)


---
### server_client_read_only<!-- {{#callable:server_client_read_only}} -->
The `server_client_read_only` function handles the scenario where a command is attempted on a read-only client.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item that is being processed.
    - `data`: An unused pointer to additional data, which is not utilized in this function.
- **Control Flow**:
    - The function calls `cmdq_error` to log an error message indicating that the client is read-only.
    - It then returns a value indicating an error condition (CMD_RETURN_ERROR).
- **Output**:
    - The function outputs an error message and returns a command return value indicating an error.


---
### server_client_command_done<!-- {{#callable:server_client_command_done}} -->
The `server_client_command_done` function processes the completion of a command for a client, handling client exit conditions and sending requests.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item being processed.
    - `data`: An unused pointer to additional data, which is not utilized in this function.
- **Control Flow**:
    - Retrieves the client associated with the command queue item using [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client).
    - Checks if the client is not attached; if so, sets the `CLIENT_EXIT` flag.
    - If the client is not in the exit state, checks if it is in control mode and calls [`control_ready`](control.c.driver.md#control_ready) if true.
    - Sends requests to the client's terminal using [`tty_send_requests`](tty.c.driver.md#tty_send_requests).
    - Returns `CMD_RETURN_NORMAL` to indicate successful command processing.
- **Output**:
    - Returns `CMD_RETURN_NORMAL`, indicating that the command has been processed successfully.
- **Functions called**:
    - [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client)
    - [`control_ready`](control.c.driver.md#control_ready)
    - [`tty_send_requests`](tty.c.driver.md#tty_send_requests)


---
### server_client_dispatch_command<!-- {{#callable:server_client_dispatch_command}} -->
The `server_client_dispatch_command` function processes a command message received from a client.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client that sent the command.
    - `imsg`: A pointer to the `imsg` structure containing the command message.
- **Control Flow**:
    - The function first checks if the client is marked for exit and returns early if so.
    - It verifies the size of the incoming message and copies the command data into a local structure.
    - The function extracts the command arguments from the message and checks for errors in unpacking them.
    - If no arguments are provided, it retrieves a default command list; otherwise, it parses the command arguments.
    - Based on the parsing result, it either prepares a command for execution or handles errors.
    - Finally, it appends the command to the client's command queue and handles any errors that occurred during processing.
- **Output**:
    - The function does not return a value but modifies the client's command queue and may append error messages if any issues arise during command processing.
- **Functions called**:
    - [`cmd_unpack_argv`](cmd.c.driver.md#cmd_unpack_argv)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`cmd_list_copy`](cmd.c.driver.md#cmd_list_copy)
    - [`options_get_command`](options.c.driver.md#options_get_command)
    - [`args_from_vector`](arguments.c.driver.md#args_from_vector)
    - [`args_free_values`](arguments.c.driver.md#args_free_values)
    - [`cmd_free_argv`](cmd.c.driver.md#cmd_free_argv)
    - [`cmd_list_all_have`](cmd.c.driver.md#cmd_list_all_have)
    - [`cmdq_get_command`](cmd-queue.c.driver.md#cmdq_get_command)
    - [`cmdq_append`](cmd-queue.c.driver.md#cmdq_append)
    - [`cmd_list_free`](cmd.c.driver.md#cmd_list_free)
    - [`cmdq_get_error`](cmd-queue.c.driver.md#cmdq_get_error)


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
    - If the message type is `MSG_IDENTIFY_DONE`, it marks the client as identified and sets its name based on the terminal name or process ID.
    - If the client is a control client, it starts control operations; otherwise, it initializes the terminal and handles configuration loading for the first client.
- **Output**:
    - The function does not return a value but modifies the state of the `client` structure based on the identification messages received.
- **Functions called**:
    - [`tty_get_features`](tty-features.c.driver.md#tty_get_features)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`find_home`](tmux.c.driver.md#find_home)
    - [`imsg_get_fd`](compat/imsg.c.driver.md#imsg_get_fd)
    - [`environ_put`](environ.c.driver.md#environ_put)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`control_start`](control.c.driver.md#control_start)
    - [`tty_init`](tty.c.driver.md#tty_init)
    - [`tty_resize`](tty.c.driver.md#tty_resize)
    - [`start_cfg`](cfg.c.driver.md#start_cfg)


---
### server_client_dispatch_shell<!-- {{#callable:server_client_dispatch_shell}} -->
The `server_client_dispatch_shell` function sends the default shell command to a client and terminates the client's peer connection.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client to which the shell command will be sent.
- **Control Flow**:
    - Retrieve the default shell from global options using [`options_get_string`](options.c.driver.md#options_get_string).
    - Check if the retrieved shell is valid using [`checkshell`](tmux.c.driver.md#checkshell); if not, fallback to a predefined shell path.
    - Send the shell command to the client using [`proc_send`](proc.c.driver.md#proc_send).
    - Terminate the peer connection for the client using [`proc_kill_peer`](proc.c.driver.md#proc_kill_peer).
- **Output**:
    - The function does not return a value; it performs actions that affect the state of the client and the server, specifically sending a shell command and closing the connection.
- **Functions called**:
    - [`options_get_string`](options.c.driver.md#options_get_string)
    - [`checkshell`](tmux.c.driver.md#checkshell)
    - [`proc_send`](proc.c.driver.md#proc_send)
    - [`proc_kill_peer`](proc.c.driver.md#proc_kill_peer)


---
### server_client_get_cwd<!-- {{#callable:server_client_get_cwd}} -->
Retrieves the current working directory for a client or session.
- **Inputs**:
    - `struct client *c`: Pointer to the `client` structure representing the client whose current working directory is being requested.
    - `struct session *s`: Pointer to the `session` structure representing the session associated with the client.
- **Control Flow**:
    - Checks if the configuration is finished and if a global client configuration is set, returning its current working directory if so.
    - If the client `c` is not NULL and has no associated session but has a current working directory, it returns that directory.
    - If the session `s` is not NULL and has a current working directory, it returns that directory.
    - If the client `c` is not NULL and has an associated session, it checks the session's current working directory and returns it if available.
    - If none of the above conditions are met, it attempts to find the home directory using `find_home()` and returns it if found.
    - If all checks fail, it returns the root directory '/' as a fallback.
- **Output**:
    - Returns a pointer to a string representing the current working directory, which may be the client's directory, the session's directory, the home directory, or the root directory.
- **Functions called**:
    - [`find_home`](tmux.c.driver.md#find_home)


---
### server_client_control_flags<!-- {{#callable:server_client_control_flags}} -->
The `server_client_control_flags` function processes control flag commands for a client.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client whose control flags are being set.
    - `next`: A string representing the control command to be processed.
- **Control Flow**:
    - The function first checks if the `next` string is equal to 'pause-after' and sets the client's `pause_age` to 0, returning the `CLIENT_CONTROL_PAUSEAFTER` flag.
    - If `next` matches the format 'pause-after=<value>', it parses the value, multiplies it by 1000, and returns the `CLIENT_CONTROL_PAUSEAFTER` flag.
    - If `next` is 'no-output', it returns the `CLIENT_CONTROL_NOOUTPUT` flag.
    - If `next` is 'wait-exit', it returns the `CLIENT_CONTROL_WAITEXIT` flag.
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
    - Duplicates the input `flags` string for processing.
    - Iterates over each flag in the `flags` string, splitting by commas.
    - Checks if the flag should be negated (if it starts with '!').
    - Determines the corresponding flag value based on the flag name.
    - Logs the action of setting or unsetting the flag.
    - Updates the client's flags accordingly, either setting or clearing them.
    - Resets control offsets if a specific flag is set.
    - Sends the updated flags to the client's peer.
- **Output**:
    - The function does not return a value but modifies the `flags` field of the `client` structure and sends a message to the client peer with the updated flags.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`strsep`](compat/strsep.c.driver.md#strsep)
    - [`server_client_control_flags`](#server_client_control_flags)
    - [`control_reset_offsets`](control.c.driver.md#control_reset_offsets)
    - [`proc_send`](proc.c.driver.md#proc_send)


---
### server_client_get_flags<!-- {{#callable:server_client_get_flags}} -->
The `server_client_get_flags` function retrieves a string representation of the flags set for a given client.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client whose flags are to be retrieved.
- **Control Flow**:
    - Initializes a static character array `s` to store the flags string.
    - Checks each flag in the `client` structure using bitwise AND operations.
    - If a flag is set, appends the corresponding string representation to `s` using [`strlcat`](compat/strlcat.c.driver.md#strlcat).
    - If the resulting string is not empty, removes the trailing comma.
    - Returns the constructed string of flags.
- **Output**:
    - Returns a pointer to a string containing the flags set for the client, formatted as a comma-separated list.
- **Functions called**:
    - [`strlcat`](compat/strlcat.c.driver.md#strlcat)
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)


---
### server_client_get_client_window<!-- {{#callable:server_client_get_client_window}} -->
Retrieves a `client_window` structure associated with a given client and window ID.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client whose window is being queried.
    - `id`: An unsigned integer representing the ID of the window to retrieve.
- **Control Flow**:
    - A `client_window` structure is initialized with the provided window ID.
    - The function uses `RB_FIND` to search for the `client_window` in the red-black tree of windows associated with the client.
    - If a matching `client_window` is found, it is returned; otherwise, NULL is returned.
- **Output**:
    - Returns a pointer to the `client_window` structure if found, or NULL if no matching window exists.


---
### server_client_add_client_window<!-- {{#callable:server_client_add_client_window}} -->
The `server_client_add_client_window` function adds a new client window to a specified client if it does not already exist.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client to which the window is being added.
    - `id`: An unsigned integer representing the identifier of the window to be added.
- **Control Flow**:
    - The function first attempts to retrieve an existing `client_window` associated with the given client and window ID using [`server_client_get_client_window`](#server_client_get_client_window).
    - If no existing window is found (i.e., `cw` is NULL), it allocates memory for a new `client_window` structure using [`xcalloc`](xmalloc.c.driver.md#xcalloc).
    - The new window's ID is set to the provided `id`, and it is then inserted into the client's window list using `RB_INSERT`.
- **Output**:
    - The function returns a pointer to the `client_window` structure, which may be either the newly created window or an existing one if it was found.
- **Functions called**:
    - [`server_client_get_client_window`](#server_client_get_client_window)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)


---
### server_client_get_pane<!-- {{#callable:server_client_get_pane}} -->
The `server_client_get_pane` function retrieves the active pane associated with a given client.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client whose active pane is to be retrieved.
- **Control Flow**:
    - The function first checks if the session associated with the client is NULL; if so, it returns NULL.
    - It then checks if the `CLIENT_ACTIVEPANE` flag is not set; if true, it returns the active pane of the current window in the session.
    - Next, it attempts to retrieve the client window associated with the current window's ID using [`server_client_get_client_window`](#server_client_get_client_window).
    - If the client window is NULL, it again returns the active pane of the current window.
    - Finally, if the client window is found, it returns the pane associated with that client window.
- **Output**:
    - Returns a pointer to the `window_pane` structure representing the active pane for the client, or NULL if no valid pane can be determined.
- **Functions called**:
    - [`server_client_get_client_window`](#server_client_get_client_window)


---
### server_client_set_pane<!-- {{#callable:server_client_set_pane}} -->
Sets the active pane for a given client in a tmux session.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client whose pane is being set.
    - `wp`: A pointer to the `window_pane` structure representing the pane to be set as active.
- **Control Flow**:
    - Checks if the session associated with the client is NULL; if so, the function returns immediately.
    - Calls [`server_client_add_client_window`](#server_client_add_client_window) to retrieve or create a `client_window` structure for the current window of the client.
    - Sets the `pane` field of the `client_window` structure to the provided `window_pane`.
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
    - Iterates through the list of all clients using `TAILQ_FOREACH`.
    - For each client, retrieves the associated `client_window` using [`server_client_get_client_window`](#server_client_get_client_window).
    - Checks if the retrieved `client_window` is not NULL and if its `pane` matches the provided `window_pane`.
    - If a match is found, removes the `client_window` from the client's list of windows using `RB_REMOVE` and frees the memory allocated for it.
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
    - The function first retrieves the data and size from the `evbuffer`.
    - If `parse` is false, it sanitizes the data using [`utf8_stravisx`](utf8.c.driver.md#utf8_stravisx) to create a message string.
    - If `parse` is true and the buffer is empty, it sets the message to an empty character.
    - The function logs the message for debugging purposes.
    - If the client pointer `c` is NULL, it exits early.
    - It checks if the client session is NULL or if the client is in control mode, and handles the output accordingly.
    - If the client is not in control mode, it retrieves the associated window pane and sets the mode to `window_view_mode` if necessary.
    - If `parse` is true, it reads lines from the `evbuffer` and adds them to the window pane, otherwise it adds the message directly.
- **Output**:
    - The function does not return a value; instead, it modifies the state of the client by sending formatted output to it, either as sanitized text or as raw data, depending on the `parse` flag.
- **Functions called**:
    - [`utf8_stravisx`](utf8.c.driver.md#utf8_stravisx)
    - [`utf8_sanitize`](utf8.c.driver.md#utf8_sanitize)
    - [`server_client_get_pane`](#server_client_get_pane)
    - [`window_pane_set_mode`](window.c.driver.md#window_pane_set_mode)


---
### server_client_report_theme<!-- {{#callable:server_client_report_theme}} -->
Reports the theme preference of a client and requests color information.
- **Inputs**:
    - `c`: A pointer to the `client` structure representing the client whose theme is being reported.
    - `theme`: An enumeration value of type `client_theme` indicating the selected theme (light or dark).
- **Control Flow**:
    - Checks if the `theme` is `THEME_LIGHT` and sets the client's theme accordingly.
    - Notifies the client of the selected theme using [`notify_client`](notify.c.driver.md#notify_client).
    - If the theme is not light, it defaults to `THEME_DARK` and notifies the client.
    - Sends escape sequences to request the current foreground and background colors from the terminal.
- **Output**:
    - The function does not return a value but updates the client's theme and requests color information.
- **Functions called**:
    - [`notify_client`](notify.c.driver.md#notify_client)
    - [`tty_puts`](tty.c.driver.md#tty_puts)


