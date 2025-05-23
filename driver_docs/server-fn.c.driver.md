# Purpose
This C source code file is part of the tmux terminal multiplexer project, specifically handling server-side operations related to session and window management. The file provides a collection of functions that manage the lifecycle and state of sessions, windows, and panes within the tmux server. Key functionalities include redrawing clients, sessions, and windows, managing session groups, locking clients and sessions, and handling the destruction and detachment of sessions and windows. The code is structured to ensure that changes in the state of one component (like a window or session) are appropriately reflected across all associated clients and sessions, maintaining consistency in the user interface.

The file is not an executable but rather a component of the tmux server, likely intended to be compiled and linked with other parts of the tmux codebase. It does not define public APIs or external interfaces directly but provides internal functions that are crucial for the server's operation. The functions utilize various data structures such as linked lists and red-black trees to manage collections of clients, sessions, and windows efficiently. The code also includes mechanisms for handling session groups, ensuring that operations affecting one session can be propagated to all sessions within a group. Overall, this file is integral to the server's ability to manage multiple user sessions and windows, providing the necessary backend functionality to support tmux's multiplexing capabilities.
# Imports and Dependencies

---
- `sys/types.h`
- `sys/wait.h`
- `sys/uio.h`
- `stdlib.h`
- `string.h`
- `time.h`
- `unistd.h`
- `tmux.h`


# Functions

---
### server_redraw_client<!-- {{#callable:server_redraw_client}} -->
The `server_redraw_client` function sets the redraw flags for a given client to trigger a full redraw.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client that needs to be redrawn.
- **Control Flow**:
    - The function takes a single argument, a pointer to a `client` structure.
    - It modifies the `flags` field of the `client` structure by performing a bitwise OR operation with `CLIENT_ALLREDRAWFLAGS`.
    - This operation sets the redraw flags for the client, indicating that a full redraw is required.
- **Output**:
    - The function does not return any value; it modifies the `flags` field of the provided `client` structure in place.


---
### server_status_client<!-- {{#callable:server_status_client}} -->
The `server_status_client` function sets a flag on a client to indicate that its status needs to be redrawn.
- **Inputs**:
    - `c`: A pointer to a `client` structure, representing the client whose status needs to be updated.
- **Control Flow**:
    - The function takes a single argument, a pointer to a `client` structure.
    - It modifies the `flags` field of the `client` structure by performing a bitwise OR operation with the `CLIENT_REDRAWSTATUS` flag.
    - This operation sets the `CLIENT_REDRAWSTATUS` flag in the `flags` field, indicating that the client's status should be redrawn.
- **Output**:
    - The function does not return any value; it modifies the `flags` field of the `client` structure in place.


---
### server_redraw_session<!-- {{#callable:server_redraw_session}} -->
The `server_redraw_session` function iterates over all clients and triggers a redraw for those associated with a specified session.
- **Inputs**:
    - `s`: A pointer to a `session` structure representing the session for which associated clients need to be redrawn.
- **Control Flow**:
    - Iterate over each client in the global `clients` list using the `TAILQ_FOREACH` macro.
    - For each client, check if the client's session matches the provided session `s`.
    - If a match is found, call [`server_redraw_client`](#server_redraw_client) for that client to trigger a redraw.
- **Output**:
    - The function does not return a value; it performs actions on clients associated with the given session.
- **Functions called**:
    - [`server_redraw_client`](#server_redraw_client)


---
### server_redraw_session_group<!-- {{#callable:server_redraw_session_group}} -->
The `server_redraw_session_group` function redraws all sessions in a session group or a single session if it is not part of a group.
- **Inputs**:
    - `s`: A pointer to a `struct session` which represents the session to be redrawn or checked for group membership.
- **Control Flow**:
    - Check if the session `s` is part of a session group using `session_group_contains(s)`.
    - If the session is not part of a group (`sg` is NULL), call `server_redraw_session(s)` to redraw the single session.
    - If the session is part of a group, iterate over all sessions in the group using `TAILQ_FOREACH` and call `server_redraw_session(s)` for each session in the group.
- **Output**:
    - The function does not return a value; it performs actions to redraw sessions.
- **Functions called**:
    - [`session_group_contains`](session.c.driver.md#session_group_contains)
    - [`server_redraw_session`](#server_redraw_session)


---
### server_status_session<!-- {{#callable:server_status_session}} -->
The `server_status_session` function iterates over all clients and updates the status of those associated with a given session.
- **Inputs**:
    - `s`: A pointer to a `struct session` representing the session for which the status needs to be updated.
- **Control Flow**:
    - Iterate over each client in the global `clients` list using the `TAILQ_FOREACH` macro.
    - For each client, check if the client's session matches the given session `s`.
    - If a match is found, call [`server_status_client`](#server_status_client) for that client to update its status.
- **Output**:
    - The function does not return a value; it performs actions on the clients associated with the given session.
- **Functions called**:
    - [`server_status_client`](#server_status_client)


---
### server_status_session_group<!-- {{#callable:server_status_session_group}} -->
The `server_status_session_group` function updates the status of a session or all sessions in a session group by invoking [`server_status_session`](#server_status_session) on each session.
- **Inputs**:
    - `s`: A pointer to a `struct session` representing the session whose status needs to be updated.
- **Control Flow**:
    - Check if the session `s` is part of a session group using [`session_group_contains`](session.c.driver.md#session_group_contains) function.
    - If `s` is not part of a session group (i.e., `sg` is NULL), call [`server_status_session`](#server_status_session) for the session `s`.
    - If `s` is part of a session group, iterate over all sessions in the group using `TAILQ_FOREACH` and call [`server_status_session`](#server_status_session) for each session.
- **Output**:
    - The function does not return a value; it performs status updates on sessions.
- **Functions called**:
    - [`session_group_contains`](session.c.driver.md#session_group_contains)
    - [`server_status_session`](#server_status_session)


---
### server_redraw_window<!-- {{#callable:server_redraw_window}} -->
The `server_redraw_window` function iterates over all clients and triggers a redraw for those whose current window matches the specified window.
- **Inputs**:
    - `w`: A pointer to a `window` structure that represents the window to be redrawn.
- **Control Flow**:
    - Iterate over each client in the global `clients` list using the `TAILQ_FOREACH` macro.
    - For each client, check if the client's session is not NULL and if the current window of the session matches the specified window `w`.
    - If both conditions are met, call [`server_redraw_client`](#server_redraw_client) to set the redraw flags for the client.
- **Output**:
    - The function does not return a value; it modifies the state of clients by setting redraw flags for those associated with the specified window.
- **Functions called**:
    - [`server_redraw_client`](#server_redraw_client)


---
### server_redraw_window_borders<!-- {{#callable:server_redraw_window_borders}} -->
The `server_redraw_window_borders` function sets a flag to redraw the borders of a window for all clients currently displaying that window.
- **Inputs**:
    - `w`: A pointer to a `struct window` representing the window whose borders need to be redrawn.
- **Control Flow**:
    - Iterate over each client in the global `clients` list using `TAILQ_FOREACH`.
    - For each client, check if the client's session is not NULL and if the current window of the session matches the given window `w`.
    - If both conditions are met, set the `CLIENT_REDRAWBORDERS` flag in the client's flags to indicate that the window borders need to be redrawn.
- **Output**:
    - The function does not return any value; it modifies the state of clients by setting a flag.


---
### server_status_window<!-- {{#callable:server_status_window}} -->
The `server_status_window` function updates the status line of all clients that contain a specified window.
- **Inputs**:
    - `w`: A pointer to a `window` structure representing the window whose status needs to be updated.
- **Control Flow**:
    - Iterate over all sessions using the `RB_FOREACH` macro.
    - For each session, check if the session contains the specified window using the [`session_has`](session.c.driver.md#session_has) function.
    - If the session contains the window, call [`server_status_session`](#server_status_session) to update the status line for that session.
- **Output**:
    - The function does not return a value; it performs an action by updating the status lines of relevant clients.
- **Functions called**:
    - [`session_has`](session.c.driver.md#session_has)
    - [`server_status_session`](#server_status_session)


---
### server_lock<!-- {{#callable:server_lock}} -->
The `server_lock` function iterates over all clients and locks those that have an active session.
- **Inputs**:
    - None
- **Control Flow**:
    - Iterate over each client in the global `clients` list using the `TAILQ_FOREACH` macro.
    - For each client, check if the `session` attribute is not `NULL`.
    - If the client has an active session, call [`server_lock_client`](#server_lock_client) to lock the client.
- **Output**:
    - The function does not return any value; it performs actions on the clients directly.
- **Functions called**:
    - [`server_lock_client`](#server_lock_client)


---
### server_lock_session<!-- {{#callable:server_lock_session}} -->
The `server_lock_session` function locks all clients associated with a given session.
- **Inputs**:
    - `s`: A pointer to a `struct session` representing the session whose clients are to be locked.
- **Control Flow**:
    - Iterates over all clients in the global `clients` list using the `TAILQ_FOREACH` macro.
    - For each client, checks if the client's session matches the provided session `s`.
    - If a match is found, calls [`server_lock_client`](#server_lock_client) on the client to lock it.
- **Output**:
    - The function does not return a value; it performs actions on the clients associated with the session.
- **Functions called**:
    - [`server_lock_client`](#server_lock_client)


---
### server_lock_client<!-- {{#callable:server_lock_client}} -->
The `server_lock_client` function locks a client session by executing a lock command and suspending the client if certain conditions are met.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client to be locked.
- **Control Flow**:
    - Check if the client has the `CLIENT_CONTROL` flag set; if so, return immediately without locking.
    - Check if the client has the `CLIENT_SUSPENDED` flag set; if so, return immediately without locking.
    - Retrieve the lock command from the client's session options using [`options_get_string`](options.c.driver.md#options_get_string).
    - Check if the lock command is empty or if its length exceeds the maximum allowed size; if so, return immediately without locking.
    - Stop the client's terminal using [`tty_stop_tty`](tty.c.driver.md#tty_stop_tty).
    - Send raw terminal commands to clear and prepare the terminal for locking using [`tty_raw`](tty.c.driver.md#tty_raw).
    - Set the `CLIENT_SUSPENDED` flag on the client to indicate it is now suspended.
    - Send the lock command to the client's peer process using [`proc_send`](proc.c.driver.md#proc_send).
- **Output**:
    - The function does not return a value; it modifies the state of the client by suspending it and sending a lock command to its peer process.
- **Functions called**:
    - [`options_get_string`](options.c.driver.md#options_get_string)
    - [`tty_stop_tty`](tty.c.driver.md#tty_stop_tty)
    - [`tty_raw`](tty.c.driver.md#tty_raw)
    - [`tty_term_string`](tty-term.c.driver.md#tty_term_string)
    - [`proc_send`](proc.c.driver.md#proc_send)


---
### server_kill_pane<!-- {{#callable:server_kill_pane}} -->
The `server_kill_pane` function removes a specified pane from its window, and if it's the only pane, it kills the entire window.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane to be removed or killed.
- **Control Flow**:
    - Retrieve the window associated with the given pane `wp`.
    - Check if the window has only one pane using `window_count_panes(w) == 1`.
    - If the window has only one pane, call `server_kill_window(w, 1)` to kill the window and then call `recalculate_sizes()` to adjust the layout.
    - If the window has more than one pane, call `server_unzoom_window(w)` to unzoom the window if it is zoomed.
    - Remove the pane from the client using `server_client_remove_pane(wp)`.
    - Close the layout of the pane using `layout_close_pane(wp)`.
    - Remove the pane from the window using `window_remove_pane(w, wp)`.
    - Redraw the window using `server_redraw_window(w)` to update the display.
- **Output**:
    - The function does not return a value; it performs operations to remove or kill a pane and potentially its window.
- **Functions called**:
    - [`window_count_panes`](window.c.driver.md#window_count_panes)
    - [`server_kill_window`](#server_kill_window)
    - [`recalculate_sizes`](resize.c.driver.md#recalculate_sizes)
    - [`server_unzoom_window`](#server_unzoom_window)
    - [`server_client_remove_pane`](server-client.c.driver.md#server_client_remove_pane)
    - [`layout_close_pane`](layout.c.driver.md#layout_close_pane)
    - [`window_remove_pane`](window.c.driver.md#window_remove_pane)
    - [`server_redraw_window`](#server_redraw_window)


---
### server_kill_window<!-- {{#callable:server_kill_window}} -->
The `server_kill_window` function detaches and destroys a specified window from all sessions that contain it, optionally renumbering the sessions' windows.
- **Inputs**:
    - `w`: A pointer to the `window` structure that is to be killed.
    - `renumber`: An integer flag indicating whether to renumber the windows in the sessions after the window is killed (non-zero to renumber, zero otherwise).
- **Control Flow**:
    - Iterate over all sessions using `RB_FOREACH_SAFE` to safely traverse and modify the session list.
    - For each session, check if it contains the specified window using [`session_has`](session.c.driver.md#session_has).
    - If the session contains the window, call [`server_unzoom_window`](#server_unzoom_window) to unzoom the window if necessary.
    - Use a loop to find and detach all winlinks associated with the window in the session using [`winlink_find_by_window`](window.c.driver.md#winlink_find_by_window).
    - If detaching a winlink results in an empty session, call [`server_destroy_session_group`](#server_destroy_session_group) to destroy the session group and break out of the loop.
    - If the session is not destroyed, call [`server_redraw_session_group`](#server_redraw_session_group) to redraw the session group.
    - If the `renumber` flag is set, call [`server_renumber_session`](#server_renumber_session) to renumber the session's windows.
    - After processing all sessions, call [`recalculate_sizes`](resize.c.driver.md#recalculate_sizes) to update the sizes of all windows.
- **Output**:
    - The function does not return a value; it performs operations on the session and window structures to detach and destroy the specified window.
- **Functions called**:
    - [`session_has`](session.c.driver.md#session_has)
    - [`server_unzoom_window`](#server_unzoom_window)
    - [`winlink_find_by_window`](window.c.driver.md#winlink_find_by_window)
    - [`session_detach`](session.c.driver.md#session_detach)
    - [`server_destroy_session_group`](#server_destroy_session_group)
    - [`server_redraw_session_group`](#server_redraw_session_group)
    - [`server_renumber_session`](#server_renumber_session)
    - [`recalculate_sizes`](resize.c.driver.md#recalculate_sizes)


---
### server_renumber_session<!-- {{#callable:server_renumber_session}} -->
The `server_renumber_session` function renumbers the windows in a session or session group if the 'renumber-windows' option is enabled.
- **Inputs**:
    - `s`: A pointer to a `struct session` representing the session to be renumbered.
- **Control Flow**:
    - Check if the 'renumber-windows' option is enabled for the session `s`.
    - If the session is part of a session group, iterate over all sessions in the group and renumber their windows.
    - If the session is not part of a session group, renumber the windows of the session `s`.
- **Output**:
    - The function does not return a value; it performs operations on the session or session group to renumber windows.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`session_group_contains`](session.c.driver.md#session_group_contains)
    - [`session_renumber_windows`](session.c.driver.md#session_renumber_windows)


---
### server_renumber_all<!-- {{#callable:server_renumber_all}} -->
The `server_renumber_all` function iterates over all sessions and renumbers the windows within each session if the renumbering option is enabled.
- **Inputs**:
    - None
- **Control Flow**:
    - The function uses the `RB_FOREACH` macro to iterate over all sessions in the `sessions` red-black tree.
    - For each session, it calls the [`server_renumber_session`](#server_renumber_session) function to potentially renumber the windows within that session.
- **Output**:
    - The function does not return any value; it performs its operations directly on the session data structures.
- **Functions called**:
    - [`server_renumber_session`](#server_renumber_session)


---
### server_link_window<!-- {{#callable:server_link_window}} -->
The `server_link_window` function links a window from a source session to a destination session, handling potential conflicts and options for killing or selecting windows.
- **Inputs**:
    - `src`: The source session from which the window is being linked.
    - `srcwl`: The winlink structure representing the window in the source session.
    - `dst`: The destination session to which the window is being linked.
    - `dstidx`: The index in the destination session where the window should be linked, or -1 to use the base index.
    - `killflag`: A flag indicating whether to kill the existing window at the destination index if it exists.
    - `selectflag`: A flag indicating whether to select the newly linked window in the destination session.
    - `cause`: A pointer to a string that will be set with an error message if the function fails.
- **Control Flow**:
    - Check if the source and destination sessions are in the same session group and return an error if they are.
    - Attempt to find a winlink at the specified destination index in the destination session.
    - If a winlink is found and it points to the same window as the source, return an error.
    - If the killflag is set and a winlink is found, unlink the window from the destination session, handling special cases if it is the current window.
    - If no destination index is specified, calculate it using the base index option of the destination session.
    - Attach the source window to the destination session at the specified index, returning an error if this fails.
    - If the selectflag is set, select the newly linked window in the destination session.
    - Redraw the session group of the destination session to reflect the changes.
- **Output**:
    - Returns 0 on success, or -1 on failure, with an error message set in the 'cause' parameter if provided.
- **Functions called**:
    - [`session_group_contains`](session.c.driver.md#session_group_contains)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`winlink_find_by_index`](window.c.driver.md#winlink_find_by_index)
    - [`notify_session_window`](notify.c.driver.md#notify_session_window)
    - [`winlink_stack_remove`](window.c.driver.md#winlink_stack_remove)
    - [`winlink_remove`](window.c.driver.md#winlink_remove)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`session_attach`](session.c.driver.md#session_attach)
    - [`session_select`](session.c.driver.md#session_select)
    - [`server_redraw_session_group`](#server_redraw_session_group)


---
### server_unlink_window<!-- {{#callable:server_unlink_window}} -->
The `server_unlink_window` function detaches a window from a session and either destroys the session group or redraws it based on the result of the detachment.
- **Inputs**:
    - `s`: A pointer to the session from which the window is to be unlinked.
    - `wl`: A pointer to the winlink structure representing the window to be unlinked from the session.
- **Control Flow**:
    - Call [`session_detach`](session.c.driver.md#session_detach) with the session `s` and winlink `wl` to attempt detaching the window from the session.
    - If [`session_detach`](session.c.driver.md#session_detach) returns true, call [`server_destroy_session_group`](#server_destroy_session_group) with the session `s` to destroy the session group.
    - If [`session_detach`](session.c.driver.md#session_detach) returns false, call [`server_redraw_session_group`](#server_redraw_session_group) with the session `s` to redraw the session group.
- **Output**:
    - The function does not return a value; it performs actions based on the success or failure of detaching a window from a session.
- **Functions called**:
    - [`session_detach`](session.c.driver.md#session_detach)
    - [`server_destroy_session_group`](#server_destroy_session_group)
    - [`server_redraw_session_group`](#server_redraw_session_group)


---
### server_destroy_pane<!-- {{#callable:server_destroy_pane}} -->
The `server_destroy_pane` function handles the destruction of a window pane, managing its resources and updating the window state accordingly.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane to be destroyed.
    - `notify`: An integer flag indicating whether to notify about the pane's destruction.
- **Control Flow**:
    - Check if the pane's file descriptor is valid; if so, remove any associated records, free the event buffer, and close the file descriptor.
    - Retrieve the 'remain-on-exit' option value and determine if the pane should remain on exit based on its status and flags.
    - If the pane should remain on exit, check its status and potentially draw a status message on the pane's screen, then mark it for redraw.
    - If the pane is not set to remain on exit, notify about the pane's exit if required, and proceed to remove the pane from its window.
    - Unzoom the window if necessary, remove the pane from the client, close its layout, and remove it from the window.
    - If the window has no more panes, kill the window; otherwise, redraw the window.
- **Output**:
    - The function does not return a value; it performs operations to clean up and remove a window pane.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`notify_pane`](notify.c.driver.md#notify_pane)
    - [`options_get_string`](options.c.driver.md#options_get_string)
    - [`screen_write_start_pane`](screen-write.c.driver.md#screen_write_start_pane)
    - [`screen_write_scrollregion`](screen-write.c.driver.md#screen_write_scrollregion)
    - [`screen_write_cursormove`](screen-write.c.driver.md#screen_write_cursormove)
    - [`screen_write_linefeed`](screen-write.c.driver.md#screen_write_linefeed)
    - [`format_single`](format.c.driver.md#format_single)
    - [`format_draw`](format-draw.c.driver.md#format_draw)
    - [`screen_write_stop`](screen-write.c.driver.md#screen_write_stop)
    - [`server_unzoom_window`](#server_unzoom_window)
    - [`server_client_remove_pane`](server-client.c.driver.md#server_client_remove_pane)
    - [`layout_close_pane`](layout.c.driver.md#layout_close_pane)
    - [`window_remove_pane`](window.c.driver.md#window_remove_pane)
    - [`server_kill_window`](#server_kill_window)
    - [`server_redraw_window`](#server_redraw_window)


---
### server_destroy_session_group<!-- {{#callable:server_destroy_session_group}} -->
The `server_destroy_session_group` function destroys a session and all sessions in its group if it belongs to one.
- **Inputs**:
    - `s`: A pointer to the session to be destroyed.
- **Control Flow**:
    - Check if the session `s` is part of a session group using `session_group_contains(s)`.
    - If `s` is not part of a session group, call `server_destroy_session(s)` and `session_destroy(s, 1, __func__)` to destroy the session.
    - If `s` is part of a session group, iterate over all sessions in the group using `TAILQ_FOREACH_SAFE`.
    - For each session in the group, call `server_destroy_session(s)` and `session_destroy(s, 1, __func__)` to destroy them.
- **Output**:
    - The function does not return a value; it performs operations to destroy sessions.
- **Functions called**:
    - [`session_group_contains`](session.c.driver.md#session_group_contains)
    - [`server_destroy_session`](#server_destroy_session)
    - [`session_destroy`](session.c.driver.md#session_destroy)


---
### server_find_session<!-- {{#callable:server_find_session}} -->
The `server_find_session` function iterates through a collection of sessions to find and return a session that satisfies a given comparison function, excluding a specified session.
- **Inputs**:
    - `s`: A pointer to a session that should be excluded from the search.
    - `f`: A pointer to a comparison function that takes two session pointers and returns an integer, used to determine the preferred session.
- **Control Flow**:
    - Initialize two session pointers, `s_loop` for iteration and `s_out` for storing the result, to NULL.
    - Iterate over all sessions in the `sessions` collection using the `RB_FOREACH` macro.
    - For each session `s_loop`, check if it is not the same as the input session `s` and if `s_out` is NULL or the comparison function `f` returns true when comparing `s_loop` and `s_out`.
    - If the conditions are met, assign `s_loop` to `s_out`.
    - After the loop, return `s_out` as the result.
- **Output**:
    - Returns a pointer to the session that satisfies the comparison function and is not the same as the input session `s`, or NULL if no such session is found.


---
### server_newer_session<!-- {{#callable:server_newer_session}} -->
The `server_newer_session` function compares the activity times of two sessions and returns true if the first session is newer than the second.
- **Inputs**:
    - `s_loop`: A pointer to the first session structure whose activity time is to be compared.
    - `s_out`: A pointer to the second session structure whose activity time is to be compared.
- **Control Flow**:
    - The function uses the `timercmp` macro to compare the `activity_time` of `s_loop` and `s_out`.
    - The comparison checks if the `activity_time` of `s_loop` is greater than that of `s_out`.
- **Output**:
    - The function returns a non-zero value if `s_loop` has a more recent `activity_time` than `s_out`, otherwise it returns zero.


---
### server_newer_detached_session<!-- {{#callable:server_newer_detached_session}} -->
The function `server_newer_detached_session` checks if a session is detached and, if so, compares it to another session to determine which is newer based on activity time.
- **Inputs**:
    - `s_loop`: A pointer to a session structure that is being checked to see if it is detached and potentially newer.
    - `s_out`: A pointer to a session structure that is used as a reference for comparison to determine if `s_loop` is newer.
- **Control Flow**:
    - Check if the session `s_loop` is attached.
    - If `s_loop` is attached, return 0, indicating it is not a newer detached session.
    - If `s_loop` is not attached, call [`server_newer_session`](#server_newer_session) to compare `s_loop` and `s_out` based on their activity times.
- **Output**:
    - Returns 0 if `s_loop` is attached, otherwise returns the result of [`server_newer_session`](#server_newer_session), which is a comparison of the activity times of `s_loop` and `s_out`.
- **Functions called**:
    - [`server_newer_session`](#server_newer_session)


---
### server_destroy_session<!-- {{#callable:server_destroy_session}} -->
The `server_destroy_session` function handles the destruction of a session, reassigning clients to new sessions based on specific options and detaching them if necessary.
- **Inputs**:
    - `s`: A pointer to the session structure that is to be destroyed.
- **Control Flow**:
    - Retrieve the 'detach-on-destroy' option from the session's options to determine the behavior for client reassignment.
    - Based on the 'detach-on-destroy' value, find a suitable new session for clients using helper functions like [`server_find_session`](#server_find_session), [`session_previous_session`](session.c.driver.md#session_previous_session), or [`session_next_session`](session.c.driver.md#session_next_session).
    - If no suitable session is found and the 'detach-on-destroy' value is 1 or 2, attempt to find any newer session as a fallback.
    - Iterate over all clients, checking if they are associated with the session to be destroyed.
    - For each client associated with the session, attempt to reassign them to the new session found earlier, or set them to exit if no session is available.
    - Call [`recalculate_sizes`](resize.c.driver.md#recalculate_sizes) to adjust any necessary sizes after session destruction.
- **Output**:
    - The function does not return a value; it modifies client-session associations and potentially marks clients for exit.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`server_find_session`](#server_find_session)
    - [`session_previous_session`](session.c.driver.md#session_previous_session)
    - [`session_next_session`](session.c.driver.md#session_next_session)
    - [`server_client_set_session`](server-client.c.driver.md#server_client_set_session)
    - [`recalculate_sizes`](resize.c.driver.md#recalculate_sizes)


---
### server_check_unattached<!-- {{#callable:server_check_unattached}} -->
The `server_check_unattached` function iterates over all sessions and destroys those that are unattached and have specific 'destroy-unattached' options set.
- **Inputs**:
    - None
- **Control Flow**:
    - Iterate over all sessions using `RB_FOREACH`.
    - For each session, check if it is attached (`s->attached != 0`).
    - If the session is attached, continue to the next session.
    - Retrieve the 'destroy-unattached' option for the session using [`options_get_number`](options.c.driver.md#options_get_number).
    - Based on the value of 'destroy-unattached', decide whether to destroy the session:
    - - If 0, do nothing and continue.
    - - If 1, destroy the session.
    - - If 2, check if the session is in a group and if the group has more than one session; if not, continue.
    - - If 3, check if the session is in a group and if the group has only one session; if so, continue.
    - Call [`session_destroy`](session.c.driver.md#session_destroy) to destroy the session if conditions are met.
- **Output**:
    - The function does not return any value; it performs operations on sessions, potentially destroying them based on their 'destroy-unattached' settings.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`session_group_contains`](session.c.driver.md#session_group_contains)
    - [`session_group_count`](session.c.driver.md#session_group_count)
    - [`session_destroy`](session.c.driver.md#session_destroy)


---
### server_unzoom_window<!-- {{#callable:server_unzoom_window}} -->
The `server_unzoom_window` function attempts to unzoom a given window and redraws it if successful.
- **Inputs**:
    - `w`: A pointer to a `struct window` that represents the window to be unzoomed.
- **Control Flow**:
    - Call the [`window_unzoom`](window.c.driver.md#window_unzoom) function with the window `w` and a flag value of `1`.
    - Check if the return value of [`window_unzoom`](window.c.driver.md#window_unzoom) is `0`, indicating success.
    - If successful, call [`server_redraw_window`](#server_redraw_window) with the window `w` to redraw it.
- **Output**:
    - The function does not return a value; it performs actions on the window `w`.
- **Functions called**:
    - [`window_unzoom`](window.c.driver.md#window_unzoom)
    - [`server_redraw_window`](#server_redraw_window)


