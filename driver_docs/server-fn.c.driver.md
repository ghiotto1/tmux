# Purpose
This C source code file is part of the tmux terminal multiplexer project, providing server-side functionality for managing sessions, windows, and panes. The code defines a set of functions that handle various operations such as redrawing clients, sessions, and windows, managing session groups, locking clients, and handling the lifecycle of windows and panes. The functions are designed to interact with the tmux server's internal data structures, such as clients, sessions, session groups, windows, and panes, to maintain the state and behavior of the tmux environment.

The file includes functions for redrawing and updating the status of clients and sessions, locking clients, and managing the destruction and renumbering of sessions and windows. It also provides mechanisms for linking and unlinking windows between sessions, handling pane exits, and ensuring that sessions are properly destroyed when they become unattached. The code is structured to support the dynamic and interactive nature of tmux, allowing users to manage multiple terminal sessions efficiently. The functions are primarily intended for internal use within the tmux server, and they manipulate the state of the server based on user actions and configuration options.
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
    - The function accesses the `flags` field of the `client` structure pointed to by `c`.
    - It uses the bitwise OR operator to set the `CLIENT_ALLREDRAWFLAGS` on the `flags` field, ensuring that all redraw-related flags are activated for the client.
- **Output**:
    - The function does not return any value; it modifies the `flags` field of the `client` structure in place.


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
    - `s`: A pointer to a `struct session` representing the session for which associated clients need to be redrawn.
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
    - `s`: A pointer to a `struct session` representing the session to be redrawn or checked for group membership.
- **Control Flow**:
    - Check if the session `s` is part of a session group using `session_group_contains(s)`.
    - If `s` is not part of a session group (i.e., `sg` is NULL), call `server_redraw_session(s)` to redraw the single session.
    - If `s` is part of a session group, iterate over all sessions in the group using `TAILQ_FOREACH` and call `server_redraw_session(s)` for each session in the group.
- **Output**:
    - The function does not return a value; it performs actions to redraw sessions.
- **Functions called**:
    - [`server_redraw_session`](#server_redraw_session)


---
### server_status_session<!-- {{#callable:server_status_session}} -->
The `server_status_session` function updates the status of all clients associated with a given session.
- **Inputs**:
    - `s`: A pointer to a `session` structure representing the session whose clients' statuses need to be updated.
- **Control Flow**:
    - Iterate over each client in the global `clients` list using the `TAILQ_FOREACH` macro.
    - For each client, check if the client's session matches the given session `s`.
    - If a match is found, call [`server_status_client`](#server_status_client) on the client to update its status.
- **Output**:
    - The function does not return a value; it performs operations on the clients associated with the given session.
- **Functions called**:
    - [`server_status_client`](#server_status_client)


---
### server_status_session_group<!-- {{#callable:server_status_session_group}} -->
The `server_status_session_group` function updates the status of a session or all sessions in a session group.
- **Inputs**:
    - `s`: A pointer to a `session` structure representing the session to be checked and potentially updated.
- **Control Flow**:
    - Check if the session `s` is part of a session group using `session_group_contains` function.
    - If `s` is not part of a session group (i.e., `sg` is NULL), call [`server_status_session`](#server_status_session) for the session `s`.
    - If `s` is part of a session group, iterate over all sessions in the group using `TAILQ_FOREACH` and call [`server_status_session`](#server_status_session) for each session.
- **Output**:
    - The function does not return a value; it performs status updates on sessions.
- **Functions called**:
    - [`server_status_session`](#server_status_session)


---
### server_redraw_window<!-- {{#callable:server_redraw_window}} -->
The `server_redraw_window` function iterates through all clients and triggers a redraw for those whose current window matches the specified window.
- **Inputs**:
    - `w`: A pointer to a `window` structure that represents the window to be redrawn.
- **Control Flow**:
    - Iterate over each client in the global `clients` list using `TAILQ_FOREACH`.
    - For each client, check if the client's session is not NULL and if the current window of the session matches the specified window `w`.
    - If both conditions are met, call [`server_redraw_client`](#server_redraw_client) to redraw the client.
- **Output**:
    - The function does not return a value; it performs actions on clients whose current window matches the specified window.
- **Functions called**:
    - [`server_redraw_client`](#server_redraw_client)


---
### server_redraw_window_borders<!-- {{#callable:server_redraw_window_borders}} -->
The `server_redraw_window_borders` function sets a flag to redraw the borders of a window for all clients currently displaying that window.
- **Inputs**:
    - `w`: A pointer to a `window` structure representing the window whose borders need to be redrawn.
- **Control Flow**:
    - Iterates over all clients in the global `clients` list using the `TAILQ_FOREACH` macro.
    - For each client, checks if the client's session is not NULL and if the current window of the session matches the given window `w`.
    - If both conditions are met, sets the `CLIENT_REDRAWBORDERS` flag in the client's flags to indicate that the window borders need to be redrawn.
- **Output**:
    - The function does not return a value; it modifies the state of client structures by setting a flag.


---
### server_status_window<!-- {{#callable:server_status_window}} -->
The `server_status_window` function updates the status line of all clients that contain a specified window.
- **Inputs**:
    - `w`: A pointer to a `window` structure representing the window whose status needs to be updated.
- **Control Flow**:
    - Iterate over all sessions using the `RB_FOREACH` macro.
    - For each session, check if the session contains the specified window using the `session_has` function.
    - If the session contains the window, call [`server_status_session`](#server_status_session) to update the status line for that session.
- **Output**:
    - The function does not return a value; it performs an action by updating the status lines of relevant clients.
- **Functions called**:
    - [`server_status_session`](#server_status_session)


---
### server_lock<!-- {{#callable:server_lock}} -->
The `server_lock` function iterates over all clients and locks those that have an active session.
- **Inputs**:
    - None
- **Control Flow**:
    - Iterate over each client in the global `clients` list using the `TAILQ_FOREACH` macro.
    - For each client, check if the client has an active session (`c->session != NULL`).
    - If the client has an active session, call `server_lock_client(c)` to lock the client.
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
    - The function does not return a value; it performs an action by locking clients associated with the specified session.
- **Functions called**:
    - [`server_lock_client`](#server_lock_client)


---
### server_lock_client<!-- {{#callable:server_lock_client}} -->
The `server_lock_client` function locks a client session by executing a lock command and suspending the client if certain conditions are met.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client to be locked.
- **Control Flow**:
    - Check if the client has the CLIENT_CONTROL flag set; if so, return immediately without locking.
    - Check if the client is already suspended (CLIENT_SUSPENDED flag); if so, return immediately without locking.
    - Retrieve the lock command from the client's session options using `options_get_string`.
    - Check if the lock command is empty or if its length exceeds the maximum allowed size; if so, return immediately without locking.
    - Stop the client's terminal (TTY) using `tty_stop_tty`.
    - Send raw terminal commands to clear and prepare the terminal for locking using `tty_raw`.
    - Set the CLIENT_SUSPENDED flag on the client to mark it as suspended.
    - Send the lock command to the client's peer process using `proc_send`.
- **Output**:
    - The function does not return a value; it performs actions to lock the client session and suspend the client.


---
### server_kill_pane<!-- {{#callable:server_kill_pane}} -->
The `server_kill_pane` function removes a specified pane from its window, and if it's the only pane, it kills the entire window.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane to be killed.
- **Control Flow**:
    - Retrieve the window associated with the pane `wp`.
    - Check if the window has only one pane using `window_count_panes(w)`.
    - If the window has only one pane, call `server_kill_window(w, 1)` to kill the window and then recalculate sizes.
    - If the window has more than one pane, unzoom the window using `server_unzoom_window(w)`.
    - Remove the pane from the client using `server_client_remove_pane(wp)`.
    - Close the layout of the pane using `layout_close_pane(wp)`.
    - Remove the pane from the window using `window_remove_pane(w, wp)`.
    - Redraw the window using `server_redraw_window(w)` to reflect the changes.
- **Output**:
    - The function does not return a value; it performs operations to remove a pane and potentially a window, updating the display as necessary.
- **Functions called**:
    - [`server_kill_window`](#server_kill_window)
    - [`server_unzoom_window`](#server_unzoom_window)
    - [`server_redraw_window`](#server_redraw_window)


---
### server_kill_window<!-- {{#callable:server_kill_window}} -->
The `server_kill_window` function detaches and potentially destroys a window from all sessions that contain it, optionally renumbering the sessions' windows.
- **Inputs**:
    - `w`: A pointer to the `window` structure that is to be killed.
    - `renumber`: An integer flag indicating whether to renumber the windows in the sessions after detaching the window.
- **Control Flow**:
    - Iterate over all sessions using `RB_FOREACH_SAFE` to safely traverse and modify the session list.
    - For each session, check if it contains the window `w` using `session_has`.
    - If the session contains the window, call [`server_unzoom_window`](#server_unzoom_window) to unzoom the window if necessary.
    - Find and detach the window from the session using `winlink_find_by_window` and `session_detach`.
    - If detaching the window results in an empty session, call [`server_destroy_session_group`](#server_destroy_session_group) to destroy the session group.
    - If the session is not destroyed, call [`server_redraw_session_group`](#server_redraw_session_group) to redraw the session group.
    - If the `renumber` flag is set, call [`server_renumber_session`](#server_renumber_session) to renumber the windows in the session.
    - After processing all sessions, call `recalculate_sizes` to update the sizes of all windows.
- **Output**:
    - The function does not return a value; it performs operations on the session and window structures to detach and potentially destroy the specified window.
- **Functions called**:
    - [`server_unzoom_window`](#server_unzoom_window)
    - [`server_destroy_session_group`](#server_destroy_session_group)
    - [`server_redraw_session_group`](#server_redraw_session_group)
    - [`server_renumber_session`](#server_renumber_session)


---
### server_renumber_session<!-- {{#callable:server_renumber_session}} -->
The `server_renumber_session` function renumbers the windows in a session or session group if the 'renumber-windows' option is enabled.
- **Inputs**:
    - `s`: A pointer to a `struct session` object representing the session to be renumbered.
- **Control Flow**:
    - Check if the 'renumber-windows' option is enabled for the session `s` using `options_get_number`.
    - If the session is part of a session group, iterate over all sessions in the group and call `session_renumber_windows` for each session.
    - If the session is not part of a session group, call `session_renumber_windows` for the session `s`.
- **Output**:
    - The function does not return a value; it performs operations on the session or session group to renumber windows.


---
### server_renumber_all<!-- {{#callable:server_renumber_all}} -->
The `server_renumber_all` function iterates over all sessions and renumbers the windows within each session if the renumbering option is enabled.
- **Inputs**:
    - None
- **Control Flow**:
    - The function uses the `RB_FOREACH` macro to iterate over each session in the global `sessions` tree.
    - For each session, it calls the [`server_renumber_session`](#server_renumber_session) function to potentially renumber the windows in that session.
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
    - `killflag`: A flag indicating whether to remove an existing window at the destination index if it exists.
    - `selectflag`: A flag indicating whether the newly linked window should be selected in the destination session.
    - `cause`: A pointer to a string that will be set with a cause message if the function fails.
- **Control Flow**:
    - Check if the source and destination sessions are in the same session group and return an error if they are.
    - Attempt to find a winlink at the destination index in the destination session.
    - If a winlink is found and it points to the same window as the source, return an error.
    - If the killflag is set and a winlink is found, unlink the window from the destination session, handling current window selection if necessary.
    - If no destination index is specified, calculate it using the base index option from the destination session.
    - Attach the source window to the destination session at the specified index, returning an error if this fails.
    - If the selectflag is set, select the newly linked window in the destination session.
    - Redraw the session group for the destination session.
- **Output**:
    - Returns 0 on success, or -1 on failure with a cause message set.
- **Functions called**:
    - [`server_redraw_session_group`](#server_redraw_session_group)


---
### server_unlink_window<!-- {{#callable:server_unlink_window}} -->
The `server_unlink_window` function detaches a window from a session and either destroys the session group or redraws it based on the result of the detachment.
- **Inputs**:
    - `s`: A pointer to the session from which the window is to be unlinked.
    - `wl`: A pointer to the winlink structure representing the window to be unlinked from the session.
- **Control Flow**:
    - Call `session_detach` with the session `s` and winlink `wl` to attempt detaching the window from the session.
    - If `session_detach` returns true, call [`server_destroy_session_group`](#server_destroy_session_group) to destroy the session group associated with the session.
    - If `session_detach` returns false, call [`server_redraw_session_group`](#server_redraw_session_group) to redraw the session group associated with the session.
- **Output**:
    - The function does not return a value; it performs actions based on the success of detaching a window from a session.
- **Functions called**:
    - [`server_destroy_session_group`](#server_destroy_session_group)
    - [`server_redraw_session_group`](#server_redraw_session_group)


---
### server_destroy_pane<!-- {{#callable:server_destroy_pane}} -->
The `server_destroy_pane` function handles the destruction of a window pane, managing its resources and updating the window state accordingly.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane to be destroyed.
    - `notify`: An integer flag indicating whether to notify about the pane's destruction.
- **Control Flow**:
    - Check if the file descriptor `fd` of the pane is valid; if so, remove any associated utempter record, free the bufferevent, and close the file descriptor.
    - Retrieve the 'remain-on-exit' option value and check if the pane should remain on exit; if so, handle the pane's status and potentially notify about its death.
    - If the pane is not set to remain on exit, notify about its exit if required, and proceed to unzoom the window, remove the pane from clients, close its layout, and remove it from the window.
    - Check if the window has no more panes; if so, kill the window, otherwise, redraw the window.
- **Output**:
    - The function does not return a value; it performs operations to clean up and manage the state of the window pane and its associated window.
- **Functions called**:
    - [`server_unzoom_window`](#server_unzoom_window)
    - [`server_kill_window`](#server_kill_window)
    - [`server_redraw_window`](#server_redraw_window)


---
### server_destroy_session_group<!-- {{#callable:server_destroy_session_group}} -->
The `server_destroy_session_group` function destroys a session and its associated group if it exists, otherwise it destroys the session individually.
- **Inputs**:
    - `s`: A pointer to the session to be destroyed.
- **Control Flow**:
    - Check if the session `s` is part of a session group using `session_group_contains(s)`.
    - If the session is not part of a group (i.e., `sg` is NULL), call `server_destroy_session(s)` and `session_destroy(s, 1, __func__)` to destroy the session individually.
    - If the session is part of a group, iterate over all sessions in the group using `TAILQ_FOREACH_SAFE`, and for each session, call `server_destroy_session(s)` and `session_destroy(s, 1, __func__)` to destroy them.
- **Output**:
    - The function does not return a value; it performs operations to destroy sessions and session groups.
- **Functions called**:
    - [`server_destroy_session`](#server_destroy_session)


---
### server_find_session<!-- {{#callable:server_find_session}} -->
The `server_find_session` function searches through a collection of sessions to find and return a session that meets a specified condition, as determined by a provided comparison function.
- **Inputs**:
    - `s`: A pointer to the session that should be excluded from the search.
    - `f`: A pointer to a comparison function that takes two session pointers and returns an integer, used to determine the preferred session.
- **Control Flow**:
    - Initialize two session pointers, `s_loop` for iteration and `s_out` for storing the result, to NULL.
    - Iterate over all sessions in the `sessions` collection using the `RB_FOREACH` macro.
    - For each session `s_loop`, check if it is not the same as the input session `s` and if `s_out` is NULL or the comparison function `f` returns true when comparing `s_loop` and `s_out`.
    - If the conditions are met, assign `s_loop` to `s_out`.
    - After the loop, return `s_out`, which is the session that meets the criteria specified by the comparison function.
- **Output**:
    - Returns a pointer to the session that meets the criteria specified by the comparison function, or NULL if no such session is found.


---
### server_newer_session<!-- {{#callable:server_newer_session}} -->
The `server_newer_session` function compares the activity times of two sessions and returns whether the first session is newer than the second.
- **Inputs**:
    - `s_loop`: A pointer to the first session structure whose activity time is to be compared.
    - `s_out`: A pointer to the second session structure whose activity time is to be compared.
- **Control Flow**:
    - The function uses the `timercmp` macro to compare the `activity_time` of `s_loop` and `s_out`.
    - The comparison checks if the `activity_time` of `s_loop` is greater than that of `s_out`.
    - The result of the comparison is returned as the function's output.
- **Output**:
    - The function returns a non-zero value if `s_loop` has a more recent `activity_time` than `s_out`, otherwise it returns zero.


---
### server_newer_detached_session<!-- {{#callable:server_newer_detached_session}} -->
The function `server_newer_detached_session` checks if a session is detached and, if so, compares it with another session to determine which is newer based on activity time.
- **Inputs**:
    - `s_loop`: A pointer to a session structure that is being checked to see if it is detached and potentially newer.
    - `s_out`: A pointer to a session structure that is used as a reference for comparison to determine if `s_loop` is newer.
- **Control Flow**:
    - Check if the session `s_loop` is attached.
    - If `s_loop` is attached, return 0, indicating it is not a newer detached session.
    - If `s_loop` is not attached, call [`server_newer_session`](#server_newer_session) to compare `s_loop` and `s_out` based on their activity times.
- **Output**:
    - Returns 0 if `s_loop` is attached, otherwise returns the result of [`server_newer_session`](#server_newer_session), which indicates if `s_loop` is newer than `s_out` based on activity time.
- **Functions called**:
    - [`server_newer_session`](#server_newer_session)


---
### server_destroy_session<!-- {{#callable:server_destroy_session}} -->
The `server_destroy_session` function handles the destruction of a session, reassigning clients to a new session if possible, and marking clients for exit if no suitable session is found.
- **Inputs**:
    - `s`: A pointer to the session structure that is to be destroyed.
- **Control Flow**:
    - Retrieve the 'detach-on-destroy' option value from the session's options.
    - Based on the 'detach-on-destroy' value, attempt to find a suitable new session for clients using helper functions like [`server_find_session`](#server_find_session), `session_previous_session`, or `session_next_session`.
    - If no suitable session is found and the 'detach-on-destroy' value is 1 or 2, attempt to find any newer session as a fallback.
    - Iterate over all clients, checking if their current session matches the session to be destroyed.
    - For each client associated with the session, attempt to assign them to the new session found earlier, or mark them for exit if no session is available.
    - Call `recalculate_sizes` to update any necessary size calculations after session destruction.
- **Output**:
    - The function does not return a value; it modifies client session assignments and potentially marks clients for exit.
- **Functions called**:
    - [`server_find_session`](#server_find_session)


---
### server_check_unattached<!-- {{#callable:server_check_unattached}} -->
The `server_check_unattached` function iterates over all sessions and destroys those that are unattached and meet specific criteria based on the 'destroy-unattached' option.
- **Inputs**:
    - None
- **Control Flow**:
    - Iterate over all sessions using `RB_FOREACH` macro.
    - For each session, check if it is attached by evaluating `s->attached`.
    - If the session is attached, continue to the next session.
    - Retrieve the 'destroy-unattached' option value for the session using `options_get_number`.
    - Based on the option value, decide whether to destroy the session:
    - - If the value is 0, skip the session.
    - - If the value is 1, proceed to destroy the session.
    - - If the value is 2, check if the session is part of a group and if the group has more than one session; if not, skip the session.
    - - If the value is 3, check if the session is part of a group and if the group has only one session; if so, skip the session.
    - If the session meets the criteria for destruction, call `session_destroy` with the session.
- **Output**:
    - The function does not return any value; it performs operations on sessions, potentially destroying them based on their attachment status and configuration options.


---
### server_unzoom_window<!-- {{#callable:server_unzoom_window}} -->
The `server_unzoom_window` function attempts to unzoom a given window and redraws it if successful.
- **Inputs**:
    - `w`: A pointer to a `struct window` that represents the window to be unzoomed.
- **Control Flow**:
    - Call the `window_unzoom` function with the window `w` and a flag value of `1`.
    - Check if the return value of `window_unzoom` is `0`, indicating success.
    - If successful, call [`server_redraw_window`](#server_redraw_window) with the window `w` to redraw it.
- **Output**:
    - The function does not return a value; it performs actions on the window `w`.
- **Functions called**:
    - [`server_redraw_window`](#server_redraw_window)


# Function Declarations (Public API)

---
### server_destroy_session_group<!-- {{#callable_declaration:server_destroy_session_group}} -->
Destroys a session or all sessions in a session group.
- **Description**: This function is used to destroy a given session or all sessions within the same session group if the session is part of a group. It should be called when a session or session group needs to be terminated. The function handles both individual sessions and session groups, ensuring that all associated resources are properly released. It is important to ensure that the session passed to this function is valid and that any necessary cleanup or notifications are handled prior to calling this function.
- **Inputs**:
    - `s`: A pointer to the session to be destroyed. Must not be null. The session should be valid and properly initialized before calling this function. If the session is part of a session group, all sessions in the group will be destroyed.
- **Output**: None
- **See also**: [`server_destroy_session_group`](#server_destroy_session_group)  (Implementation)


