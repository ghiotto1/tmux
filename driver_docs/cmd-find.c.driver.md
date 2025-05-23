# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file provides a set of functions for finding and managing sessions, windows, and panes within tmux. It includes functionality to determine the "best" session or client based on certain criteria, such as activity time or attachment status. The code also includes functions to parse and resolve target strings into specific tmux entities, such as sessions, windows, or panes, and to handle various target formats, including those specified by mouse events or marked panes.

The file defines several static functions and data structures that are used internally to manage the state of tmux sessions and their components. It includes functions to log the current state, clear or copy states, and validate the integrity of a state. The code is designed to be integrated into the larger tmux codebase, providing essential functionality for session and window management. It does not define a public API or external interfaces, as it is intended for internal use within the tmux application. The file is structured to handle various scenarios and edge cases, ensuring robust management of tmux's complex session and window hierarchy.
# Imports and Dependencies

---
- `sys/types.h`
- `fnmatch.h`
- `limits.h`
- `stdlib.h`
- `string.h`
- `unistd.h`
- `tmux.h`


# Global Variables

---
### cmd_find_best_session
- **Type**: `static struct session *`
- **Description**: The `cmd_find_best_session` function is a static function that returns a pointer to a `struct session`. It is designed to find the best session from a list of sessions or from all available sessions if the list is NULL. The function uses certain criteria, such as session activity time and attachment status, to determine the 'best' session.
- **Use**: This function is used to select the most appropriate session based on specific criteria, which can be used in various parts of the program to manage session-related operations.


---
### cmd_find_map_table
- **Type**: `function`
- **Description**: `cmd_find_map_table` is a static function that takes a two-dimensional array of strings and a single string as arguments. It searches the array for a match with the given string and returns the corresponding mapped value from the array.
- **Use**: This function is used to map a given string to its corresponding value in a predefined table of string pairs.


---
### cmd_find_session_table
- **Type**: `const char *[2]`
- **Description**: The `cmd_find_session_table` is a static constant array of string pairs, where each pair consists of two `const char *` elements. It is initialized with a single pair of `NULL` values, indicating an empty or placeholder entry.
- **Use**: This variable is used to map session-related strings to their corresponding values, although it is currently initialized with no mappings.


---
### cmd_find_window_table
- **Type**: `const char *[6][2]`
- **Description**: The `cmd_find_window_table` is a static constant array of string pairs, where each pair consists of a window command and its corresponding symbol. It is used to map specific window-related commands to their symbolic representations.
- **Use**: This variable is used to translate window command strings into their symbolic equivalents for easier processing and interpretation within the program.


---
### cmd_find_pane_table
- **Type**: `const char *[2]`
- **Description**: The `cmd_find_pane_table` is a static constant array of string pairs, where each pair consists of a pane identifier and its corresponding command or description. This table is used to map specific pane-related commands or identifiers to their respective actions or descriptions within the tmux application.
- **Use**: This variable is used to map pane identifiers to their corresponding commands or descriptions for pane-related operations in tmux.


# Functions

---
### cmd_find_inside_pane<!-- {{#callable:cmd_find_inside_pane}} -->
The `cmd_find_inside_pane` function locates a window pane associated with a given client, if any, by matching the client's TTY or using an environment variable.
- **Inputs**:
    - `c`: A pointer to a `struct client`, representing the client for which the associated window pane is to be found.
- **Control Flow**:
    - Check if the client `c` is NULL; if so, return NULL immediately.
    - Iterate over all window panes using `RB_FOREACH` to find a pane with a matching TTY name and a valid file descriptor; if found, break the loop.
    - If no matching pane is found, look for an environment variable `TMUX_PANE` in the client's environment and attempt to find a pane by its ID string from this variable.
    - If a pane is found, log a debug message with the pane's ID and TTY.
    - Return the found window pane or NULL if none was found.
- **Output**:
    - Returns a pointer to a `struct window_pane` that is associated with the given client, or NULL if no such pane is found.


---
### cmd_find_client_better<!-- {{#callable:cmd_find_client_better}} -->
The `cmd_find_client_better` function determines if one client is 'better' than another based on their activity time.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client to be compared.
    - `than`: A pointer to a `client` structure representing the client to compare against, or NULL if no comparison client is provided.
- **Control Flow**:
    - Check if the `than` client is NULL; if so, return 1 indicating that `c` is better by default.
    - Use the `timercmp` macro to compare the `activity_time` of `c` and `than`, returning the result of the comparison.
- **Output**:
    - Returns 1 if `c` is considered better than `than` (either because `than` is NULL or `c` has a more recent `activity_time`), otherwise returns 0.


---
### cmd_find_best_client<!-- {{#callable:cmd_find_best_client}} -->
The `cmd_find_best_client` function identifies the most suitable client associated with a given session, or the best client overall if no session is specified.
- **Inputs**:
    - `s`: A pointer to a `struct session` representing the session for which the best client is to be found.
- **Control Flow**:
    - Check if the session `s` is attached; if not, set `s` to NULL.
    - Initialize a client pointer `c` to NULL to store the best client found.
    - Iterate over all clients in the `clients` list using `TAILQ_FOREACH`.
    - For each client `c_loop`, skip if the client is not associated with any session.
    - If `s` is not NULL, skip clients whose session does not match `s`.
    - Use [`cmd_find_client_better`](#cmd_find_client_better) to determine if `c_loop` is a better client than the current best `c`; if so, update `c` to `c_loop`.
    - Return the best client found, stored in `c`.
- **Output**:
    - A pointer to the best `struct client` found, or NULL if no suitable client is found.
- **Functions called**:
    - [`cmd_find_client_better`](#cmd_find_client_better)


---
### cmd_find_session_better<!-- {{#callable:cmd_find_session_better}} -->
The `cmd_find_session_better` function determines if one session is 'better' than another based on attachment status and activity time.
- **Inputs**:
    - `s`: A pointer to the session being evaluated.
    - `than`: A pointer to the session to compare against, which can be NULL.
    - `flags`: An integer representing flags that modify the comparison behavior, such as CMD_FIND_PREFER_UNATTACHED.
- **Control Flow**:
    - Check if the 'than' session is NULL; if so, return 1 indicating 's' is better.
    - If the CMD_FIND_PREFER_UNATTACHED flag is set, compare the attachment status of 's' and 'than':
    - - If 'than' is attached and 's' is not, return 1 indicating 's' is better.
    - - If 'than' is not attached and 's' is, return 0 indicating 'than' is better.
    - If the CMD_FIND_PREFER_UNATTACHED flag is not set or the attachment status does not determine the result, compare the activity times of 's' and 'than' using the timercmp macro.
    - Return the result of the timercmp comparison, which checks if 's' has a more recent activity time than 'than'.
- **Output**:
    - Returns 1 if the session 's' is considered better than 'than', otherwise returns 0.


---
### cmd_find_best_session<!-- {{#callable:cmd_find_best_session}} -->
The `cmd_find_best_session` function identifies the most suitable session from a given list or from all available sessions based on specified criteria.
- **Inputs**:
    - `slist`: A pointer to an array of session pointers, representing the list of sessions to evaluate; can be NULL to evaluate all sessions.
    - `ssize`: An unsigned integer representing the size of the session list array.
    - `flags`: An integer representing flags that influence the selection criteria for the best session.
- **Control Flow**:
    - Initialize a session pointer `s` to NULL.
    - Log the number of sessions to try using the `log_debug` function.
    - If `slist` is not NULL, iterate over the session list array using a for loop.
    - For each session in the list, call [`cmd_find_session_better`](#cmd_find_session_better) to determine if the current session is better than the previously selected best session, updating `s` if it is.
    - If `slist` is NULL, iterate over all sessions using the `RB_FOREACH` macro.
    - For each session, call [`cmd_find_session_better`](#cmd_find_session_better) to determine if the current session is better than the previously selected best session, updating `s` if it is.
    - Return the best session found, or NULL if no suitable session is found.
- **Output**:
    - Returns a pointer to the best session found based on the criteria, or NULL if no suitable session is found.
- **Functions called**:
    - [`cmd_find_session_better`](#cmd_find_session_better)


---
### cmd_find_best_session_with_window<!-- {{#callable:cmd_find_best_session_with_window}} -->
The function `cmd_find_best_session_with_window` identifies the best session containing a specified window and then attempts to find the best winlink for that window.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that contains the state information for finding sessions and windows, including the target window `w` and flags for the search.
- **Control Flow**:
    - Initialize a session list `slist` to NULL and a session size `ssize` to 0.
    - Log the window ID from the `cmd_find_state` structure.
    - Iterate over all sessions using `RB_FOREACH` to check if each session contains the specified window `fs->w`.
    - If a session contains the window, reallocate the session list to accommodate the new session and add it to the list, incrementing `ssize`.
    - If no sessions contain the window (`ssize` is 0), go to the `fail` label.
    - Call [`cmd_find_best_session`](#cmd_find_best_session) with the session list, its size, and the flags from `fs` to find the best session and assign it to `fs->s`.
    - If no best session is found (`fs->s` is NULL), go to the `fail` label.
    - Free the session list `slist`.
    - Return the result of `cmd_find_best_winlink_with_window(fs)`.
    - At the `fail` label, free the session list `slist` and return -1.
- **Output**:
    - The function returns an integer: 0 if a suitable session and winlink are found, or -1 if no session contains the window or if no best session or winlink can be determined.
- **Functions called**:
    - [`cmd_find_best_session`](#cmd_find_best_session)
    - [`cmd_find_best_winlink_with_window`](#cmd_find_best_winlink_with_window)


---
### cmd_find_best_winlink_with_window<!-- {{#callable:cmd_find_best_winlink_with_window}} -->
The function `cmd_find_best_winlink_with_window` identifies the best winlink associated with a specified window within a session.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that contains the current state of the command find operation, including the session and window information.
- **Control Flow**:
    - Log the window ID from the `cmd_find_state` structure for debugging purposes.
    - Initialize a `winlink` pointer `wl` to `NULL`.
    - Check if the current winlink in the session (`fs->s->curw`) is associated with the specified window (`fs->w`).
    - If the current winlink is associated with the window, set `wl` to the current winlink.
    - If not, iterate over all winlinks in the session's windows to find a winlink associated with the specified window.
    - If a matching winlink is found during iteration, set `wl` to this winlink and break the loop.
    - If no matching winlink is found, return -1 indicating failure.
    - If a matching winlink is found, update the `cmd_find_state` structure with this winlink and its index, and return 0 indicating success.
- **Output**:
    - Returns 0 if a matching winlink is found and updates the `cmd_find_state` structure; returns -1 if no matching winlink is found.


---
### cmd_find_map_table<!-- {{#callable:cmd_find_map_table}} -->
The `cmd_find_map_table` function searches a two-dimensional array for a string match and returns the corresponding mapped value.
- **Inputs**:
    - `table`: A two-dimensional array of strings, where each sub-array contains two elements: a key and its corresponding mapped value.
    - `s`: A string to search for in the first element of each sub-array in the table.
- **Control Flow**:
    - Initialize an unsigned integer `i` to iterate over the table.
    - Enter a for loop that continues as long as the first element of the current sub-array (`table[i][0]`) is not NULL.
    - Within the loop, compare the input string `s` with the first element of the current sub-array using `strcmp`.
    - If a match is found (i.e., `strcmp` returns 0), return the second element of the current sub-array (`table[i][1]`).
    - If no match is found after iterating through the table, return the input string `s`.
- **Output**:
    - Returns the mapped value corresponding to the input string `s` if found in the table; otherwise, returns the input string `s` itself.


---
### cmd_find_get_session<!-- {{#callable:cmd_find_get_session}} -->
The `cmd_find_get_session` function attempts to locate a session based on a given session name or identifier and updates the provided `cmd_find_state` structure with the found session.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that will be updated with the found session.
    - `session`: A string representing the session name or identifier to search for.
- **Control Flow**:
    - Log the function call with the session name for debugging purposes.
    - Check if the session name starts with '$', indicating a session ID, and attempt to find the session by ID using `session_find_by_id_str`. If found, update `fs->s` and return 0; otherwise, return -1.
    - Attempt to find the session by name using `session_find`. If found, update `fs->s` and return 0.
    - Attempt to find a client with the given session name using [`cmd_find_client`](#cmd_find_client). If a client is found and it has an associated session, update `fs->s` with the client's session and return 0.
    - If the `CMD_FIND_EXACT_SESSION` flag is set in `fs->flags`, return -1 to stop further searching.
    - Iterate over all sessions to find a session whose name starts with the given session name. If exactly one match is found, update `fs->s` and return 0; if multiple matches are found, return -1.
    - Iterate over all sessions to find a session whose name matches the given session name as a pattern using `fnmatch`. If exactly one match is found, update `fs->s` and return 0; if multiple matches are found, return -1.
    - If no session is found, return -1.
- **Output**:
    - Returns 0 if a session is found and updates `fs->s` with the session; returns -1 if no session is found.
- **Functions called**:
    - [`cmd_find_client`](#cmd_find_client)


---
### cmd_find_get_window<!-- {{#callable:cmd_find_get_window}} -->
The `cmd_find_get_window` function attempts to locate a window within a session based on a given window identifier or name, updating the provided `cmd_find_state` structure with the found window and session details.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that will be updated with the found window and session information.
    - `window`: A string representing the window identifier or name to be searched for.
    - `only`: An integer flag indicating whether to restrict the search to the current session only.
- **Control Flow**:
    - Log the function call and the window identifier being searched.
    - Check if the window identifier starts with '@', indicating a window ID; if so, attempt to find the window by ID and update the state with the best session containing this window.
    - If the window identifier is not an ID, use the current session from the `cmd_find_state` structure.
    - Attempt to find the window within the current session using [`cmd_find_get_window_with_session`](#cmd_find_get_window_with_session); if successful, return 0.
    - If the window is not found and `only` is false, attempt to find the window as a session using [`cmd_find_get_session`](#cmd_find_get_session); if successful, update the state with the current window and index, then return 0.
    - If none of the above conditions are met, return -1 indicating failure to find the window.
- **Output**:
    - Returns 0 if the window is successfully found and the state is updated, or -1 if the window cannot be found.
- **Functions called**:
    - [`cmd_find_best_session_with_window`](#cmd_find_best_session_with_window)
    - [`cmd_find_get_window_with_session`](#cmd_find_get_window_with_session)
    - [`cmd_find_get_session`](#cmd_find_get_session)


---
### cmd_find_get_window_with_session<!-- {{#callable:cmd_find_get_window_with_session}} -->
The function `cmd_find_get_window_with_session` attempts to locate a window within a given session based on a string identifier, updating the state with the found window and winlink if successful.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that holds the current state of the search, including session, window, and winlink information.
    - `window`: A string representing the window identifier, which can be a window ID, an offset, a special character, or a window name.
- **Control Flow**:
    - Log the function call with the window identifier.
    - Check if the window identifier is an exact match flag and set the default window to the current window in the session.
    - If the window identifier starts with '@', attempt to find the window by ID and verify it belongs to the session; if found, update the state and return success.
    - If the window identifier is an offset ('+' or '-'), calculate the new index or find the next/previous winlink based on the offset and update the state if successful.
    - If the window identifier is a special character ('!', '^', '$'), find the corresponding winlink in the session and update the state if successful.
    - If the window identifier is a numeric index, attempt to find the winlink by index in the session and update the state if successful.
    - If the window identifier is a name, search for an exact match among the windows in the session; if found, update the state and return success.
    - If the exact match flag is set and no match is found, return failure.
    - If not exact, search for a window name starting with the identifier or matching a pattern; if found, update the state and return success.
    - Return failure if no window is found.
- **Output**:
    - Returns 0 on success, updating the `cmd_find_state` with the found window and winlink, or -1 on failure if no matching window is found.
- **Functions called**:
    - [`cmd_find_best_winlink_with_window`](#cmd_find_best_winlink_with_window)


---
### cmd_find_get_pane<!-- {{#callable:cmd_find_get_pane}} -->
The `cmd_find_get_pane` function attempts to locate a specific pane within a tmux session, window, or pane context based on a given identifier string.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that holds the current state of the session, window, and pane being searched.
    - `pane`: A string representing the identifier of the pane to be found, which can be a pane ID or a name.
    - `only`: An integer flag indicating whether to restrict the search to only the current window or session.
- **Control Flow**:
    - The function logs the function name and the pane identifier for debugging purposes.
    - It checks if the pane identifier starts with '%', indicating a pane ID, and attempts to find the pane by ID using `window_pane_find_by_id_str`.
    - If the pane is found by ID, it sets the window in the `cmd_find_state` and calls [`cmd_find_best_session_with_window`](#cmd_find_best_session_with_window) to find the best session containing the window.
    - If the pane is not a pane ID, it defaults to the current session and window stored in `fs->current`.
    - It then attempts to find the pane within the current window using [`cmd_find_get_pane_with_window`](#cmd_find_get_pane_with_window).
    - If the pane is found within the current window, it returns success (0).
    - If the pane is not found and `only` is not set, it attempts to find the pane as a window using [`cmd_find_get_window`](#cmd_find_get_window).
    - If the pane is found as a window, it sets the active pane in the window and returns success (0).
    - If none of the above conditions are met, it returns failure (-1).
- **Output**:
    - Returns 0 on success if the pane is found, or -1 if the pane cannot be located.
- **Functions called**:
    - [`cmd_find_best_session_with_window`](#cmd_find_best_session_with_window)
    - [`cmd_find_get_pane_with_window`](#cmd_find_get_pane_with_window)
    - [`cmd_find_get_window`](#cmd_find_get_window)


---
### cmd_find_get_pane_with_session<!-- {{#callable:cmd_find_get_pane_with_session}} -->
The function `cmd_find_get_pane_with_session` attempts to locate a specific pane within a given session, updating the `cmd_find_state` structure accordingly.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that holds the current state of the command find operation.
    - `pane`: A string representing the pane identifier, which may start with '%' to indicate a pane ID.
- **Control Flow**:
    - Log the function call and the pane identifier for debugging purposes.
    - Check if the pane identifier starts with '%', indicating a pane ID.
    - If it is a pane ID, attempt to find the pane using `window_pane_find_by_id_str`.
    - If the pane is found, update the `cmd_find_state` structure with the pane and its associated window, then find the best winlink with the window using [`cmd_find_best_winlink_with_window`](#cmd_find_best_winlink_with_window).
    - If the pane is not a pane ID, use the current window from the session stored in `fs->s`.
    - Set the current window and index in the `cmd_find_state` structure.
    - Attempt to find the pane within the current window using [`cmd_find_get_pane_with_window`](#cmd_find_get_pane_with_window).
- **Output**:
    - Returns 0 on success, indicating the pane was found and the state was updated, or -1 if the pane could not be found.
- **Functions called**:
    - [`cmd_find_best_winlink_with_window`](#cmd_find_best_winlink_with_window)
    - [`cmd_find_get_pane_with_window`](#cmd_find_get_pane_with_window)


---
### cmd_find_get_pane_with_window<!-- {{#callable:cmd_find_get_pane_with_window}} -->
The `cmd_find_get_pane_with_window` function locates a specific pane within a given window based on a string identifier and updates the `cmd_find_state` structure accordingly.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that holds the current state of the command find operation, including session, window, and pane information.
    - `pane`: A string representing the identifier of the pane to be found, which can be a pane ID, special character, offset, index, or description.
- **Control Flow**:
    - Log the function call with the provided pane identifier.
    - Check if the pane identifier starts with '%', indicating a pane ID, and attempt to find the pane by ID; return -1 if not found or if the pane's window does not match the current window.
    - Handle special character identifiers ('!', '{up-of}', '{down-of}', '{left-of}', '{right-of}') by finding the corresponding pane relative to the current active pane; return -1 if not found.
    - If the pane identifier is an offset ('+' or '-'), calculate the offset and find the next or previous pane by number; return 0 if found.
    - Attempt to interpret the pane identifier as an index and find the pane at that index; return 0 if found.
    - Finally, try to find the pane by treating the identifier as a description string; return 0 if found.
    - Return -1 if no pane is found by any method.
- **Output**:
    - Returns 0 if the pane is successfully found and updated in the `cmd_find_state` structure, otherwise returns -1 if the pane cannot be found.


---
### cmd_find_clear_state<!-- {{#callable:cmd_find_clear_state}} -->
The `cmd_find_clear_state` function initializes a `cmd_find_state` structure by zeroing its memory, setting its flags, and initializing its index to -1.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that will be cleared and initialized.
    - `flags`: An integer representing the flags to be set in the `cmd_find_state` structure.
- **Control Flow**:
    - The function begins by zeroing out the memory of the `cmd_find_state` structure pointed to by `fs` using `memset`.
    - The `flags` field of the `cmd_find_state` structure is set to the value of the `flags` parameter.
    - The `idx` field of the `cmd_find_state` structure is set to -1.
- **Output**:
    - The function does not return a value; it modifies the `cmd_find_state` structure pointed to by `fs` in place.


---
### cmd_find_empty_state<!-- {{#callable:cmd_find_empty_state}} -->
The function `cmd_find_empty_state` checks if all fields in a `cmd_find_state` structure are NULL, indicating an empty state.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure, which contains fields representing a session, winlink, window, and window pane.
- **Control Flow**:
    - The function checks if the `s`, `wl`, `w`, and `wp` fields of the `cmd_find_state` structure pointed to by `fs` are all NULL.
    - If all fields are NULL, the function returns 1, indicating the state is empty.
    - If any field is not NULL, the function returns 0, indicating the state is not empty.
- **Output**:
    - The function returns an integer: 1 if the state is empty (all fields are NULL), or 0 if the state is not empty (any field is not NULL).


---
### cmd_find_valid_state<!-- {{#callable:cmd_find_valid_state}} -->
The `cmd_find_valid_state` function checks if a given command find state is valid by verifying the presence and consistency of its session, winlink, window, and pane components.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that contains the session, winlink, window, and pane to be validated.
- **Control Flow**:
    - Check if any of the components (session, winlink, window, pane) in the `cmd_find_state` structure are NULL; if so, return 0 indicating an invalid state.
    - Verify if the session is alive using `session_alive`; if not, return 0 indicating an invalid state.
    - Iterate over the winlinks in the session's windows to find a match for the window and winlink; if no match is found, return 0 indicating an invalid state.
    - Check if the window in the `cmd_find_state` matches the window in the winlink; if not, return 0 indicating an invalid state.
    - Return the result of `window_has_pane` to check if the window contains the specified pane, indicating a valid state if true.
- **Output**:
    - Returns an integer, 1 if the state is valid and 0 if it is invalid.


---
### cmd_find_copy_state<!-- {{#callable:cmd_find_copy_state}} -->
The `cmd_find_copy_state` function copies the state from one `cmd_find_state` structure to another.
- **Inputs**:
    - `dst`: A pointer to the destination `cmd_find_state` structure where the state will be copied to.
    - `src`: A pointer to the source `cmd_find_state` structure from which the state will be copied.
- **Control Flow**:
    - The function assigns the `s` field of the `src` structure to the `s` field of the `dst` structure.
    - The function assigns the `wl` field of the `src` structure to the `wl` field of the `dst` structure.
    - The function assigns the `idx` field of the `src` structure to the `idx` field of the `dst` structure.
    - The function assigns the `w` field of the `src` structure to the `w` field of the `dst` structure.
    - The function assigns the `wp` field of the `src` structure to the `wp` field of the `dst` structure.
- **Output**:
    - The function does not return any value; it performs the copy operation directly on the provided `cmd_find_state` structures.


---
### cmd_find_log_state<!-- {{#callable:cmd_find_log_state}} -->
The `cmd_find_log_state` function logs the current state of a `cmd_find_state` structure, including session, winlink, window pane, and index information, using a specified prefix for the log messages.
- **Inputs**:
    - `prefix`: A constant character pointer representing the prefix to be used in the log messages.
    - `fs`: A pointer to a `cmd_find_state` structure containing the current state information to be logged.
- **Control Flow**:
    - Check if the session (`fs->s`) is not NULL; if so, log the session ID and name, otherwise log 's=none'.
    - Check if the winlink (`fs->wl`) is not NULL; if so, log the winlink index, whether the winlink's window matches the current window, and the window ID and name, otherwise log 'wl=none'.
    - Check if the window pane (`fs->wp`) is not NULL; if so, log the window pane ID, otherwise log 'wp=none'.
    - Check if the index (`fs->idx`) is not -1; if so, log the index, otherwise log 'idx=none'.
- **Output**:
    - The function does not return any value; it performs logging as a side effect.


---
### cmd_find_from_session<!-- {{#callable:cmd_find_from_session}} -->
The `cmd_find_from_session` function initializes a `cmd_find_state` structure with the current session, window, and pane information from a given session.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that will be initialized with session, window, and pane information.
    - `s`: A pointer to a `session` structure from which the current window and pane information will be extracted.
    - `flags`: An integer representing flags that may modify the behavior of the state initialization.
- **Control Flow**:
    - Call [`cmd_find_clear_state`](#cmd_find_clear_state) to reset the `cmd_find_state` structure `fs` with the provided `flags`.
    - Assign the session `s` to the session field `fs->s` of the `cmd_find_state` structure.
    - Set the current window link `fs->wl` to the current window of the session `s`.
    - Set the window `fs->w` to the window of the current window link `fs->wl`.
    - Set the active pane `fs->wp` to the active pane of the window `fs->w`.
    - Log the state of `fs` using [`cmd_find_log_state`](#cmd_find_log_state).
- **Output**:
    - The function does not return a value; it modifies the `cmd_find_state` structure pointed to by `fs`.
- **Functions called**:
    - [`cmd_find_clear_state`](#cmd_find_clear_state)
    - [`cmd_find_log_state`](#cmd_find_log_state)


---
### cmd_find_from_winlink<!-- {{#callable:cmd_find_from_winlink}} -->
The `cmd_find_from_winlink` function initializes a `cmd_find_state` structure using a given `winlink` and logs the state.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that will be initialized.
    - `wl`: A pointer to a `winlink` structure from which the state will be derived.
    - `flags`: An integer representing flags that may modify the behavior of the state initialization.
- **Control Flow**:
    - Call [`cmd_find_clear_state`](#cmd_find_clear_state) to reset the `cmd_find_state` structure `fs` with the provided `flags`.
    - Set the session (`s`), winlink (`wl`), window (`w`), and active window pane (`wp`) in `fs` using the `wl` structure.
    - Log the state of `fs` using [`cmd_find_log_state`](#cmd_find_log_state).
- **Output**:
    - The function does not return a value; it modifies the `cmd_find_state` structure pointed to by `fs`.
- **Functions called**:
    - [`cmd_find_clear_state`](#cmd_find_clear_state)
    - [`cmd_find_log_state`](#cmd_find_log_state)


---
### cmd_find_from_session_window<!-- {{#callable:cmd_find_from_session_window}} -->
The `cmd_find_from_session_window` function initializes a `cmd_find_state` structure with a given session and window, and attempts to find the best winlink for the window within the session.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that will be initialized and populated with session and window information.
    - `s`: A pointer to a `session` structure representing the session to be used in the state.
    - `w`: A pointer to a `window` structure representing the window to be used in the state.
    - `flags`: An integer representing flags that control the behavior of the function, such as clearing the state.
- **Control Flow**:
    - The function begins by clearing the state of the `cmd_find_state` structure using [`cmd_find_clear_state`](#cmd_find_clear_state) with the provided flags.
    - It assigns the provided session `s` and window `w` to the `cmd_find_state` structure `fs`.
    - The function then calls [`cmd_find_best_winlink_with_window`](#cmd_find_best_winlink_with_window) to find the best winlink for the given window within the session.
    - If [`cmd_find_best_winlink_with_window`](#cmd_find_best_winlink_with_window) returns a non-zero value, indicating failure, the state is cleared again and the function returns -1.
    - If successful, the active pane of the window is assigned to `fs->wp`.
    - The function logs the state using [`cmd_find_log_state`](#cmd_find_log_state) and returns 0 to indicate success.
- **Output**:
    - The function returns 0 on success, indicating that the state was successfully populated, or -1 on failure, indicating that the best winlink could not be found.
- **Functions called**:
    - [`cmd_find_clear_state`](#cmd_find_clear_state)
    - [`cmd_find_best_winlink_with_window`](#cmd_find_best_winlink_with_window)
    - [`cmd_find_log_state`](#cmd_find_log_state)


---
### cmd_find_from_window<!-- {{#callable:cmd_find_from_window}} -->
The `cmd_find_from_window` function attempts to find the best session and winlink associated with a given window and updates the command find state accordingly.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that will be updated with the found session, winlink, and window pane.
    - `w`: A pointer to the `window` structure for which the best session and winlink are to be found.
    - `flags`: An integer representing flags that may modify the behavior of the function.
- **Control Flow**:
    - The function begins by clearing the current state in `fs` using [`cmd_find_clear_state`](#cmd_find_clear_state) with the provided flags.
    - It sets the window `w` in the `fs` structure.
    - The function then calls [`cmd_find_best_session_with_window`](#cmd_find_best_session_with_window) to find the best session associated with the window `w`.
    - If no suitable session is found, the state is cleared again and the function returns -1.
    - If a session is found, it proceeds to call [`cmd_find_best_winlink_with_window`](#cmd_find_best_winlink_with_window) to find the best winlink for the window.
    - If no suitable winlink is found, the state is cleared again and the function returns -1.
    - If both a session and winlink are found, it sets the active window pane in `fs` to the active pane of the window `w`.
    - Finally, it logs the state using [`cmd_find_log_state`](#cmd_find_log_state) and returns 0 to indicate success.
- **Output**:
    - Returns 0 on success if a session and winlink are found, otherwise returns -1 if either cannot be found.
- **Functions called**:
    - [`cmd_find_clear_state`](#cmd_find_clear_state)
    - [`cmd_find_best_session_with_window`](#cmd_find_best_session_with_window)
    - [`cmd_find_best_winlink_with_window`](#cmd_find_best_winlink_with_window)
    - [`cmd_find_log_state`](#cmd_find_log_state)


---
### cmd_find_from_winlink_pane<!-- {{#callable:cmd_find_from_winlink_pane}} -->
The `cmd_find_from_winlink_pane` function initializes a `cmd_find_state` structure using a specified winlink and window pane, and logs the state.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that will be initialized and populated with session, winlink, window, and pane information.
    - `wl`: A pointer to a `winlink` structure representing the winlink to be used for initializing the state.
    - `wp`: A pointer to a `window_pane` structure representing the window pane to be used for initializing the state.
    - `flags`: An integer representing flags that may modify the behavior of the state initialization.
- **Control Flow**:
    - Call [`cmd_find_clear_state`](#cmd_find_clear_state) to reset the `cmd_find_state` structure `fs` with the provided `flags`.
    - Set the session (`fs->s`) in the state to the session associated with the provided winlink (`wl->session`).
    - Set the winlink (`fs->wl`) in the state to the provided winlink (`wl`).
    - Set the index (`fs->idx`) in the state to the index of the provided winlink (`wl->idx`).
    - Set the window (`fs->w`) in the state to the window associated with the provided winlink (`wl->window`).
    - Set the window pane (`fs->wp`) in the state to the provided window pane (`wp`).
    - Call [`cmd_find_log_state`](#cmd_find_log_state) to log the current state of the `cmd_find_state` structure.
- **Output**:
    - The function does not return a value; it modifies the `cmd_find_state` structure pointed to by `fs`.
- **Functions called**:
    - [`cmd_find_clear_state`](#cmd_find_clear_state)
    - [`cmd_find_log_state`](#cmd_find_log_state)


---
### cmd_find_from_pane<!-- {{#callable:cmd_find_from_pane}} -->
The `cmd_find_from_pane` function initializes a `cmd_find_state` structure from a given `window_pane` and logs the state.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that will be initialized.
    - `wp`: A pointer to a `window_pane` structure from which the state will be derived.
    - `flags`: An integer representing flags that modify the behavior of the function.
- **Control Flow**:
    - Call [`cmd_find_from_window`](#cmd_find_from_window) with the `window` of the given `window_pane` and the provided flags.
    - If [`cmd_find_from_window`](#cmd_find_from_window) returns a non-zero value, return -1 indicating failure.
    - Set the `wp` field of the `cmd_find_state` structure to the provided `window_pane`.
    - Log the state using [`cmd_find_log_state`](#cmd_find_log_state).
    - Return 0 indicating success.
- **Output**:
    - Returns 0 on success, or -1 if the state could not be initialized from the window.
- **Functions called**:
    - [`cmd_find_from_window`](#cmd_find_from_window)
    - [`cmd_find_log_state`](#cmd_find_log_state)


---
### cmd_find_from_nothing<!-- {{#callable:cmd_find_from_nothing}} -->
The `cmd_find_from_nothing` function initializes a `cmd_find_state` structure by finding the best available session and setting the current window, index, and active pane within that session.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that will be initialized and populated with session, window, and pane information.
    - `flags`: An integer representing flags that modify the behavior of the function, such as preferences for session selection.
- **Control Flow**:
    - The function begins by clearing the state of the `cmd_find_state` structure using [`cmd_find_clear_state`](#cmd_find_clear_state) with the provided flags.
    - It attempts to find the best session by calling [`cmd_find_best_session`](#cmd_find_best_session) with `NULL` for the session list, `0` for the size, and the provided flags.
    - If no session is found (`fs->s` is `NULL`), it clears the state again and returns `-1` to indicate failure.
    - If a session is found, it sets the current window link (`fs->wl`), index (`fs->idx`), window (`fs->w`), and active pane (`fs->wp`) based on the session's current window.
    - The function logs the state using [`cmd_find_log_state`](#cmd_find_log_state) and returns `0` to indicate success.
- **Output**:
    - The function returns `0` on success, indicating that a session was found and the state was populated, or `-1` if no session could be found.
- **Functions called**:
    - [`cmd_find_clear_state`](#cmd_find_clear_state)
    - [`cmd_find_best_session`](#cmd_find_best_session)
    - [`cmd_find_log_state`](#cmd_find_log_state)


---
### cmd_find_from_mouse<!-- {{#callable:cmd_find_from_mouse}} -->
The `cmd_find_from_mouse` function attempts to find and set the state of a command based on a mouse event within a terminal multiplexer environment.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that will be populated with the state information based on the mouse event.
    - `m`: A pointer to a `mouse_event` structure that contains information about the mouse event, including its validity.
    - `flags`: An integer representing flags that may modify the behavior of the state finding process.
- **Control Flow**:
    - The function begins by clearing the current state in the `cmd_find_state` structure using [`cmd_find_clear_state`](#cmd_find_clear_state) with the provided flags.
    - It checks if the mouse event is valid by evaluating `m->valid`; if not valid, it returns -1 indicating failure.
    - If the mouse event is valid, it attempts to find the window pane associated with the mouse event using `cmd_mouse_pane`, updating the `cmd_find_state` structure with the session and winlink if successful.
    - If no window pane is found, it clears the state again and returns -1 indicating failure.
    - If a window pane is found, it sets the window in the `cmd_find_state` structure and logs the state using [`cmd_find_log_state`](#cmd_find_log_state).
    - Finally, it returns 0 indicating success.
- **Output**:
    - The function returns 0 on success, indicating that the state was successfully set based on the mouse event, or -1 on failure, indicating that the mouse event was invalid or no associated window pane was found.
- **Functions called**:
    - [`cmd_find_clear_state`](#cmd_find_clear_state)
    - [`cmd_find_log_state`](#cmd_find_log_state)


---
### cmd_find_from_client<!-- {{#callable:cmd_find_from_client}} -->
The `cmd_find_from_client` function determines the current state of a client in terms of its associated session, window, and pane, and updates the `cmd_find_state` structure accordingly.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that will be updated with the current state of the client.
    - `c`: A pointer to a `client` structure representing the client whose state is to be determined.
    - `flags`: An integer representing flags that modify the behavior of the function.
- **Control Flow**:
    - Check if the client `c` is NULL; if so, call [`cmd_find_from_nothing`](#cmd_find_from_nothing) to handle the state as if there is no client.
    - If the client is attached to a session, clear the current state and update `fs` with the session, window, and pane information from the client.
    - If the client is unattached but running in a pane, attempt to find the pane and limit the session list to those containing the pane.
    - If the pane is found, update `fs` with the best session and window information; otherwise, handle the state as if there is no pane.
    - Log the state using [`cmd_find_log_state`](#cmd_find_log_state) and return 0 if successful, or handle the state as if there is no client if the pane is unknown.
- **Output**:
    - Returns 0 on success, indicating that the state was successfully determined and updated, or calls [`cmd_find_from_nothing`](#cmd_find_from_nothing) if the client or pane is unknown.
- **Functions called**:
    - [`cmd_find_from_nothing`](#cmd_find_from_nothing)
    - [`cmd_find_clear_state`](#cmd_find_clear_state)
    - [`cmd_find_from_session`](#cmd_find_from_session)
    - [`cmd_find_log_state`](#cmd_find_log_state)
    - [`cmd_find_inside_pane`](#cmd_find_inside_pane)
    - [`cmd_find_best_session_with_window`](#cmd_find_best_session_with_window)


---
### cmd_find_target<!-- {{#callable:cmd_find_target}} -->
The `cmd_find_target` function resolves a target string into a specific session, window, or pane within a tmux environment, based on the provided type and flags.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure where the resolved target state will be stored.
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item, used for context and error reporting.
    - `target`: A string representing the target to be resolved, which can be a session, window, or pane identifier.
    - `type`: An enum `cmd_find_type` indicating the type of target to resolve (session, window, or pane).
    - `flags`: An integer representing various flags that modify the behavior of the target resolution process.
- **Control Flow**:
    - If the `CMD_FIND_CANFAIL` flag is set, the `CMD_FIND_QUIET` flag is also set.
    - The function logs the input parameters for debugging purposes.
    - The current state is cleared using [`cmd_find_clear_state`](#cmd_find_clear_state).
    - The function attempts to determine the current state based on marked panes, the command queue, or the client, logging errors if no valid state is found.
    - If the target is empty or NULL, the current state is used as the target.
    - If the target is a mouse event or marked pane, it resolves the target accordingly and logs errors if no valid target is found.
    - The target string is parsed to separate session, window, and pane components, and exact match flags are set if necessary.
    - The function attempts to resolve the session, window, and pane components using various helper functions, logging errors if any component cannot be resolved.
    - If a valid target is found, the function logs the resolved state and returns 0; otherwise, it logs an error and returns -1.
- **Output**:
    - Returns 0 on successful resolution of the target, or -1 if an error occurs and the `CMD_FIND_CANFAIL` flag is not set.
- **Functions called**:
    - [`cmd_find_clear_state`](#cmd_find_clear_state)
    - [`cmd_find_valid_state`](#cmd_find_valid_state)
    - [`cmd_find_from_client`](#cmd_find_from_client)
    - [`cmd_find_copy_state`](#cmd_find_copy_state)
    - [`cmd_find_map_table`](#cmd_find_map_table)
    - [`cmd_find_get_session`](#cmd_find_get_session)
    - [`cmd_find_get_window_with_session`](#cmd_find_get_window_with_session)
    - [`cmd_find_get_pane_with_session`](#cmd_find_get_pane_with_session)
    - [`cmd_find_get_pane_with_window`](#cmd_find_get_pane_with_window)
    - [`cmd_find_get_window`](#cmd_find_get_window)
    - [`cmd_find_get_pane`](#cmd_find_get_pane)
    - [`cmd_find_log_state`](#cmd_find_log_state)


---
### cmd_find_current_client<!-- {{#callable:cmd_find_current_client}} -->
The `cmd_find_current_client` function attempts to find and return the current client associated with a given command queue item, or the best available client if none is directly associated.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, representing the command queue item from which to derive the current client.
    - `quiet`: An integer flag indicating whether to suppress error messages if no client is found (non-zero to suppress).
- **Control Flow**:
    - Initialize a `client` pointer `c` to NULL and attempt to retrieve the client from the `item` using `cmdq_get_client` if `item` is not NULL.
    - If `c` is not NULL and has an associated session, return `c` immediately as the current client.
    - If `c` is not NULL, attempt to find a window pane inside the client using [`cmd_find_inside_pane`](#cmd_find_inside_pane).
    - If a window pane is found, clear the `cmd_find_state` structure and set its window to the found pane's window.
    - Attempt to find the best session containing the window and then find the best client for that session.
    - If no client is found, attempt to find the best session overall and then the best client for that session.
    - If no client is found and `quiet` is false, log an error message indicating no current client was found.
    - Log a debug message with the result and return the found client, or NULL if none was found.
- **Output**:
    - Returns a pointer to the `client` structure representing the current or best available client, or NULL if no suitable client is found.
- **Functions called**:
    - [`cmd_find_inside_pane`](#cmd_find_inside_pane)
    - [`cmd_find_clear_state`](#cmd_find_clear_state)
    - [`cmd_find_best_session_with_window`](#cmd_find_best_session_with_window)
    - [`cmd_find_best_client`](#cmd_find_best_client)
    - [`cmd_find_best_session`](#cmd_find_best_session)


---
### cmd_find_client<!-- {{#callable:cmd_find_client}} -->
The `cmd_find_client` function searches for a client based on a given target string, returning the client if found or reporting an error if not.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, representing the command queue item associated with the client search.
    - `target`: A constant character pointer representing the target client name or identifier to search for; if NULL, the current client is used.
    - `quiet`: An integer flag indicating whether to suppress error messages if the client is not found (non-zero to suppress).
- **Control Flow**:
    - If the `target` is NULL, the function calls [`cmd_find_current_client`](#cmd_find_current_client) to find the current client and returns it.
    - The `target` string is duplicated using `xstrdup` to create a modifiable copy.
    - If the copied target string ends with a colon, the colon is removed by setting the last character to a null terminator.
    - The function iterates over the list of clients using `TAILQ_FOREACH`, checking each client's session, name, and ttyname against the target string.
    - If a matching client is found, the loop breaks and the client is returned.
    - If no client is found and `quiet` is false, an error message is reported using `cmdq_error`.
    - The copied target string is freed before the function returns.
    - The function logs the result using `log_debug` and returns the found client or NULL if not found.
- **Output**:
    - Returns a pointer to the `client` structure if a matching client is found, or NULL if no client matches the target.
- **Functions called**:
    - [`cmd_find_current_client`](#cmd_find_current_client)


# Function Declarations (Public API)

---
### cmd_find_session_better<!-- {{#callable_declaration:cmd_find_session_better}} -->
Determine if one session is better than another based on specified criteria.
- **Description**: This function is used to compare two sessions to determine which one is considered 'better' based on certain criteria. It is typically used in scenarios where a decision needs to be made between two sessions, such as selecting the most appropriate session for a given task. The function considers whether the sessions are attached or not and their activity times. It is important to note that if the 'than' session is NULL, the function will always consider the 's' session as better. The function also takes into account a flag that can influence the comparison by preferring unattached sessions.
- **Inputs**:
    - `s`: A pointer to the session to be compared. Must not be null. Represents the session that is being evaluated as potentially better.
    - `than`: A pointer to the session to compare against. Can be null, in which case 's' is automatically considered better.
    - `flags`: An integer representing comparison flags. The CMD_FIND_PREFER_UNATTACHED flag can be used to prefer unattached sessions over attached ones.
- **Output**: Returns an integer: 1 if the 's' session is considered better, 0 otherwise.
- **See also**: [`cmd_find_session_better`](#cmd_find_session_better)  (Implementation)


---
### cmd_find_best_session<!-- {{#callable_declaration:cmd_find_best_session}} -->
Finds the best session from a list or all available sessions.
- **Description**: This function is used to identify the most suitable session from a provided list of sessions or from all available sessions if no list is provided. It evaluates each session based on certain criteria, which are influenced by the flags parameter, to determine the best session. This function is useful when you need to select a session that meets specific conditions, such as being unattached or having recent activity. It is important to ensure that the session list, if provided, is valid and that the flags are set according to the desired criteria for session selection.
- **Inputs**:
    - `slist`: A pointer to an array of session pointers. This array contains the sessions to be evaluated. If this parameter is NULL, the function will consider all available sessions. The caller retains ownership and must ensure the array is valid.
    - `ssize`: The number of sessions in the slist array. This parameter is ignored if slist is NULL. It must be a non-negative integer.
    - `flags`: An integer representing criteria flags that influence the selection of the best session. The specific flags and their effects are not detailed in the header, but they typically modify the selection criteria.
- **Output**: Returns a pointer to the best session found based on the criteria. If no suitable session is found, it returns NULL.
- **See also**: [`cmd_find_best_session`](#cmd_find_best_session)  (Implementation)


---
### cmd_find_best_session_with_window<!-- {{#callable_declaration:cmd_find_best_session_with_window}} -->
Finds the best session and winlink for a given window.
- **Description**: This function is used to identify the most suitable session and winlink that contain a specified window, as indicated by the `cmd_find_state` structure. It should be called when you need to determine the best session for a window, typically in scenarios where multiple sessions may contain the same window. The function updates the `cmd_find_state` structure with the best session and winlink found. It is important to ensure that the `cmd_find_state` structure is properly initialized and that the window field is set before calling this function. If no suitable session is found, the function returns an error code.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that must be initialized with a valid window. The structure will be updated with the best session and winlink if found. Must not be null.
- **Output**: Returns 0 on success if a suitable session and winlink are found, or -1 if no suitable session is found.
- **See also**: [`cmd_find_best_session_with_window`](#cmd_find_best_session_with_window)  (Implementation)


---
### cmd_find_best_winlink_with_window<!-- {{#callable_declaration:cmd_find_best_winlink_with_window}} -->
Finds the best winlink for a specified window within a session.
- **Description**: This function is used to locate the most suitable winlink for a given window within a session, prioritizing the current winlink if it contains the window. It should be called when a specific window's winlink needs to be identified within a session. The function requires a valid `cmd_find_state` structure with initialized session and window fields. If the current winlink in the session contains the window, it is selected; otherwise, the function searches through all winlinks in the session. If no winlink is found, the function returns an error.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that must have its session (`s`) and window (`w`) fields initialized. The structure is updated with the found winlink and its index. Must not be null.
- **Output**: Returns 0 on success, with the `cmd_find_state` structure updated to include the best winlink and its index. Returns -1 if no suitable winlink is found.
- **See also**: [`cmd_find_best_winlink_with_window`](#cmd_find_best_winlink_with_window)  (Implementation)


---
### cmd_find_map_table<!-- {{#callable_declaration:cmd_find_map_table}} -->
Maps a string to its corresponding value in a table.
- **Description**: Use this function to find a mapped value for a given string from a specified table of string pairs. It is useful when you have a predefined set of string mappings and need to retrieve the corresponding value for a given key. The function returns the mapped value if the key is found; otherwise, it returns the original string. This function assumes that the table is terminated with a NULL entry and that both the table and the string are valid and non-null.
- **Inputs**:
    - `table`: A two-dimensional array of string pairs, where each pair consists of a key and its corresponding value. The array must be terminated with a NULL entry to indicate the end of the table. The caller retains ownership of the table.
    - `s`: A string to be mapped. It must not be null, and the caller retains ownership of the string.
- **Output**: Returns the mapped value if the string is found in the table; otherwise, returns the original string.
- **See also**: [`cmd_find_map_table`](#cmd_find_map_table)  (Implementation)


---
### cmd_find_log_state<!-- {{#callable_declaration:cmd_find_log_state}} -->
Logs the state of a command find operation.
- **Description**: Use this function to log the current state of a command find operation, which includes details about the session, window link, window pane, and index. This function is useful for debugging purposes to understand the current state of the command find operation. It requires a valid command find state structure and a prefix string for log messages. Ensure that the command find state structure is properly initialized before calling this function.
- **Inputs**:
    - `prefix`: A string used as a prefix in log messages. It must not be null, and the caller retains ownership.
    - `fs`: A pointer to a cmd_find_state structure containing the current state of the command find operation. It must not be null, and the structure should be properly initialized before calling this function.
- **Output**: None
- **See also**: [`cmd_find_log_state`](#cmd_find_log_state)  (Implementation)


---
### cmd_find_get_session<!-- {{#callable_declaration:cmd_find_get_session}} -->
Finds a session based on a given session name or pattern.
- **Description**: This function attempts to locate a session using a specified session name or pattern and updates the provided `cmd_find_state` structure with the found session. It first checks if the session name is an exact match, then looks for a client with the session name, and finally attempts to match the session name as a prefix or pattern. If the session is found, the function returns 0; otherwise, it returns -1. This function should be used when you need to resolve a session name to a session object, and it requires a valid `cmd_find_state` structure to store the result.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure where the found session will be stored. Must not be null.
    - `session`: A string representing the session name or pattern to search for. Must not be null.
- **Output**: Returns 0 if a session is found and -1 if no session matches the given name or pattern.
- **See also**: [`cmd_find_get_session`](#cmd_find_get_session)  (Implementation)


---
### cmd_find_get_window<!-- {{#callable_declaration:cmd_find_get_window}} -->
Finds a window based on a given identifier or name.
- **Description**: This function is used to locate a window within a tmux session based on a specified identifier or name. It updates the provided `cmd_find_state` structure with the session, window, and winlink information if the window is found. The function can handle both window IDs and names, and it can also attempt to find a session if the window is not directly found. It is important to ensure that the `cmd_find_state` structure is properly initialized before calling this function. The function returns an error if the window cannot be found.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that will be updated with the found window's session, winlink, and window information. Must not be null.
    - `window`: A string representing the window identifier or name to search for. It can start with '@' to indicate a window ID.
    - `only`: An integer flag indicating whether to restrict the search to the current session only. A non-zero value restricts the search.
- **Output**: Returns 0 if the window is found and the state is updated successfully, or -1 if the window cannot be found.
- **See also**: [`cmd_find_get_window`](#cmd_find_get_window)  (Implementation)


---
### cmd_find_get_window_with_session<!-- {{#callable_declaration:cmd_find_get_window_with_session}} -->
Finds a window within a session based on a given identifier.
- **Description**: This function is used to locate a specific window within a given session using a string identifier. It supports various formats for the identifier, including window IDs, offsets, special characters, and patterns. The function updates the provided `cmd_find_state` structure with the located window and its index. It is essential to ensure that the session within the `cmd_find_state` is valid before calling this function. The function returns an error if the window cannot be found or if multiple matches are found when an exact match is required.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that must be pre-initialized with a valid session. The function updates this structure with the found window and its index.
    - `window`: A string representing the window identifier. It can be a window ID (starting with '@'), an offset (starting with '+' or '-'), a special character ('!', '^', '$'), or a pattern. The string must not be null.
- **Output**: Returns 0 on success, with the `cmd_find_state` structure updated to reflect the found window and its index. Returns -1 if the window cannot be found or if there are multiple matches when an exact match is required.
- **See also**: [`cmd_find_get_window_with_session`](#cmd_find_get_window_with_session)  (Implementation)


---
### cmd_find_get_pane<!-- {{#callable_declaration:cmd_find_get_pane}} -->
Finds a pane based on a given identifier or name.
- **Description**: This function is used to locate a specific pane within a tmux session based on a provided identifier or name. It should be called when you need to resolve a pane reference to its corresponding pane object. The function first checks if the identifier is a pane ID, and if not, it attempts to find the pane in the current session and window. If the pane is not found, it may also try to interpret the identifier as a window or session name, unless restricted by the 'only' parameter. The function modifies the provided cmd_find_state structure to reflect the found pane or returns an error if the pane cannot be found.
- **Inputs**:
    - `fs`: A pointer to a cmd_find_state structure that will be updated with the found pane's details. Must not be null.
    - `pane`: A string representing the pane identifier or name. It can start with '%' to indicate a pane ID. Must not be null.
    - `only`: An integer flag that, if non-zero, restricts the search to only panes, preventing fallback to window or session searches.
- **Output**: Returns 0 on success if the pane is found and updates the cmd_find_state structure. Returns -1 if the pane cannot be found.
- **See also**: [`cmd_find_get_pane`](#cmd_find_get_pane)  (Implementation)


---
### cmd_find_get_pane_with_session<!-- {{#callable_declaration:cmd_find_get_pane_with_session}} -->
Finds a pane within a session based on a given identifier.
- **Description**: This function is used to locate a specific pane within a session using a string identifier. It should be called when you need to resolve a pane reference within a session context. The function first checks if the identifier is a pane ID, indicated by a leading '%'. If so, it attempts to find the pane by its ID. If the pane is not found, the function returns -1. If the identifier is not a pane ID, the function defaults to using the current window of the session and attempts to find the pane within that window. The function modifies the provided cmd_find_state structure to reflect the found pane and associated window or returns an error if the pane cannot be found.
- **Inputs**:
    - `fs`: A pointer to a cmd_find_state structure that will be updated with the found pane and window information. Must not be null.
    - `pane`: A string representing the pane identifier. It can be a pane ID starting with '%' or a name to be resolved within the current window. Must not be null.
- **Output**: Returns 0 on success, indicating the pane was found and the state was updated. Returns -1 if the pane could not be found.
- **See also**: [`cmd_find_get_pane_with_session`](#cmd_find_get_pane_with_session)  (Implementation)


---
### cmd_find_get_pane_with_window<!-- {{#callable_declaration:cmd_find_get_pane_with_window}} -->
Finds a pane within a specified window based on a given identifier.
- **Description**: This function is used to locate a specific pane within a given window, as specified by the `cmd_find_state` structure. It supports various forms of pane identification, including pane IDs, special character identifiers, offsets, and descriptions. The function must be called with a valid `cmd_find_state` structure that has the window (`w`) field already set. It returns 0 on success, indicating that the pane was found and the `wp` field in `cmd_find_state` is updated. If the pane cannot be found, it returns -1. This function is useful in scenarios where precise pane identification is required within a window context.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that must have the `w` field (window) set. The function updates the `wp` field (window pane) if a pane is found.
    - `pane`: A string representing the pane identifier. It can be a pane ID (starting with '%'), a special character (e.g., '!', '{up-of}'), an offset (e.g., '+1', '-1'), or a description. The function handles invalid identifiers by returning -1.
- **Output**: Returns 0 if the pane is found and updates the `wp` field in `cmd_find_state`; returns -1 if the pane cannot be found.
- **See also**: [`cmd_find_get_pane_with_window`](#cmd_find_get_pane_with_window)  (Implementation)


