# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file provides a set of functions for finding and managing sessions, windows, and panes within tmux. It includes functionality to determine the "best" session or client based on certain criteria, such as activity time or attachment status. The code also includes functions to resolve targets specified by users, which can be sessions, windows, or panes, and to handle various target specifications, including those involving mouse events or marked panes.

The file defines several static functions and data structures that are used internally to manage the state of sessions, windows, and panes. It includes functions to clear, validate, and copy the state of these entities, as well as to log their current state for debugging purposes. The code also provides mechanisms to map user-friendly identifiers to internal representations, allowing users to specify targets using intuitive strings. Overall, this file is a crucial component of tmux's ability to manage and navigate between different terminal sessions, providing a robust API for session management within the application.
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
- **Type**: ``struct session *``
- **Description**: The `cmd_find_best_session` is a static function that returns a pointer to a `struct session`. It is designed to find the best session from a list of sessions or from all available sessions if the list is NULL. The function evaluates sessions based on certain criteria, such as activity time and attachment status, to determine the 'best' session.
- **Use**: This function is used to select the most appropriate session from a given list or from all sessions, based on specific criteria.


---
### cmd_find_map_table
- **Type**: `function pointer`
- **Description**: `cmd_find_map_table` is a static function pointer that takes a two-dimensional array of constant character pointers and a constant character pointer as arguments, and returns a constant character pointer. It is defined as a static function, meaning it is only accessible within the file it is declared in.
- **Use**: This function is used to map a given string to another string based on a predefined table of string pairs.


---
### cmd_find_session_table
- **Type**: `const char *[2]`
- **Description**: `cmd_find_session_table` is a static constant array of string pairs, where each pair consists of two `const char *` elements. It is initialized with a single pair of `NULL` values, indicating an empty or uninitialized state.
- **Use**: This variable is used as a lookup table for mapping session-related strings, although it is currently initialized with no mappings.


---
### cmd_find_window_table
- **Type**: `const char *[6][2]`
- **Description**: The `cmd_find_window_table` is a static constant array of string pairs, where each pair consists of a window command and its corresponding symbolic representation. It is used to map specific window-related commands to their symbolic equivalents, such as "{start}" to "^" and "{end}" to "$".
- **Use**: This variable is used to translate window command strings into their symbolic representations for processing within the application.


---
### cmd_find_pane_table
- **Type**: `const char *[2]`
- **Description**: The `cmd_find_pane_table` is a static constant array of string pairs, where each pair consists of a pane identifier and its corresponding command or description. This array is used to map specific pane-related commands or identifiers to their respective string representations.
- **Use**: This variable is used to facilitate the mapping of pane-related commands to their string representations within the program.


# Functions

---
### cmd_find_inside_pane<!-- {{#callable:cmd_find_inside_pane}} -->
The `cmd_find_inside_pane` function locates a window pane associated with a given client, if any, by matching the client's TTY or environment variable.
- **Inputs**:
    - `c`: A pointer to a `struct client`, representing the client for which the associated window pane is to be found.
- **Control Flow**:
    - Check if the client `c` is NULL; if so, return NULL immediately.
    - Iterate over all window panes using `RB_FOREACH` to find a pane with a matching TTY name and a valid file descriptor.
    - If a matching pane is found, break out of the loop.
    - If no matching pane is found, look for an environment variable `TMUX_PANE` in the client's environment.
    - If the environment variable is found, attempt to find the pane by its ID string using [`window_pane_find_by_id_str`](window.c.driver.md#window_pane_find_by_id_str).
    - If a pane is found, log a debug message with the pane's ID and TTY.
    - Return the found window pane, or NULL if none was found.
- **Output**:
    - Returns a pointer to the `struct window_pane` that is associated with the client, or NULL if no such pane is found.
- **Functions called**:
    - [`environ_find`](environ.c.driver.md#environ_find)
    - [`window_pane_find_by_id_str`](window.c.driver.md#window_pane_find_by_id_str)


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
    - Returns 1 if `c` is better than `than` (either because `than` is NULL or `c` has a more recent `activity_time`), otherwise returns 0.


---
### cmd_find_best_client<!-- {{#callable:cmd_find_best_client}} -->
The `cmd_find_best_client` function identifies the most suitable client associated with a given session, or the best client overall if no session is specified.
- **Inputs**:
    - `s`: A pointer to a `struct session` representing the session for which the best client is to be found; if the session is not attached, it is set to NULL.
- **Control Flow**:
    - Check if the session `s` is attached; if not, set `s` to NULL.
    - Initialize a client pointer `c` to NULL to keep track of the best client found.
    - Iterate over all clients in the global `clients` list using `TAILQ_FOREACH`.
    - For each client `c_loop`, skip if the client is not associated with any session.
    - If a session `s` is specified, skip clients not associated with this session.
    - Use the helper function [`cmd_find_client_better`](#cmd_find_client_better) to determine if `c_loop` is a better client than the current best `c`; if so, update `c` to `c_loop`.
    - Return the best client found, or NULL if no suitable client was found.
- **Output**:
    - Returns a pointer to the best `struct client` found, or NULL if no suitable client is found.
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
    - If `than` is NULL, the function immediately returns 1, indicating that `s` is better.
    - If the CMD_FIND_PREFER_UNATTACHED flag is set, the function checks the attachment status of both sessions.
    - If `than` is attached and `s` is not, the function returns 1, indicating `s` is better.
    - If `than` is not attached and `s` is attached, the function returns 0, indicating `than` is better.
    - If neither of the above conditions are met, the function compares the activity times of the two sessions using `timercmp` and returns the result of this comparison.
- **Output**:
    - The function returns an integer: 1 if session `s` is considered better, or 0 otherwise.


---
### cmd_find_best_session<!-- {{#callable:cmd_find_best_session}} -->
The `cmd_find_best_session` function identifies the most suitable session from a given list or from all available sessions based on specified criteria.
- **Inputs**:
    - `slist`: A pointer to an array of session pointers, representing the list of sessions to evaluate; can be NULL to consider all sessions.
    - `ssize`: An unsigned integer representing the size of the session list array.
    - `flags`: An integer representing flags that influence the selection criteria for the best session.
- **Control Flow**:
    - Initialize a session pointer `s` to NULL to store the best session found.
    - Log the number of sessions to be evaluated for debugging purposes.
    - If `slist` is not NULL, iterate over the session list array using a for loop.
    - For each session in the list, use [`cmd_find_session_better`](#cmd_find_session_better) to determine if it is better than the current best session `s` based on the provided flags.
    - If a better session is found, update `s` to point to this session.
    - If `slist` is NULL, iterate over all sessions using the `RB_FOREACH` macro.
    - For each session in the global session tree, use [`cmd_find_session_better`](#cmd_find_session_better) to determine if it is better than the current best session `s` based on the provided flags.
    - If a better session is found, update `s` to point to this session.
    - Return the best session found, which is stored in `s`.
- **Output**:
    - Returns a pointer to the best session found, or NULL if no suitable session is identified.
- **Functions called**:
    - [`cmd_find_session_better`](#cmd_find_session_better)


---
### cmd_find_best_session_with_window<!-- {{#callable:cmd_find_best_session_with_window}} -->
The function `cmd_find_best_session_with_window` identifies the best session containing a specified window and then attempts to find the best winlink for that window.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that contains the state information for finding sessions and windows, including the target window `w` and flags.
- **Control Flow**:
    - Initialize a session list `slist` to NULL and a session size `ssize` to 0.
    - Log the window ID from the `cmd_find_state` structure.
    - Iterate over all sessions using `RB_FOREACH` to find sessions that contain the specified window `fs->w`.
    - For each session that contains the window, reallocate the `slist` array to include the session and increment `ssize`.
    - If no sessions contain the window (`ssize` is 0), go to the `fail` label.
    - Call [`cmd_find_best_session`](#cmd_find_best_session) to find the best session from the `slist` and store it in `fs->s`.
    - If no best session is found (`fs->s` is NULL), go to the `fail` label.
    - Free the `slist` array.
    - Return the result of `cmd_find_best_winlink_with_window(fs)`.
    - At the `fail` label, free the `slist` array and return -1.
- **Output**:
    - The function returns an integer, which is the result of `cmd_find_best_winlink_with_window(fs)` if successful, or -1 if it fails to find a suitable session or winlink.
- **Functions called**:
    - [`session_has`](session.c.driver.md#session_has)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`cmd_find_best_session`](#cmd_find_best_session)
    - [`cmd_find_best_winlink_with_window`](#cmd_find_best_winlink_with_window)


---
### cmd_find_best_winlink_with_window<!-- {{#callable:cmd_find_best_winlink_with_window}} -->
The function `cmd_find_best_winlink_with_window` identifies the best winlink associated with a specified window within a session.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that contains the current state, including the session and window to be matched.
- **Control Flow**:
    - Log the window ID for debugging purposes.
    - Initialize a `winlink` pointer `wl` to NULL.
    - Check if the current winlink in the session (`fs->s->curw`) matches the specified window (`fs->w`).
    - If a match is found, assign the current winlink to `wl`.
    - If no match is found, iterate over all winlinks in the session's window list to find a winlink that matches the specified window.
    - If a matching winlink is found during iteration, assign it to `wl` and break the loop.
    - If no matching winlink is found, return -1 to indicate failure.
    - If a matching winlink is found, update the `fs` structure with the winlink and its index, then return 0 to indicate success.
- **Output**:
    - Returns 0 if a matching winlink is found and updates the `cmd_find_state` structure; returns -1 if no matching winlink is found.


---
### cmd_find_map_table<!-- {{#callable:cmd_find_map_table}} -->
The `cmd_find_map_table` function searches a 2D array for a string match and returns the corresponding mapped value or the input string if no match is found.
- **Inputs**:
    - `table`: A 2D array of strings where each element is a pair of strings, representing a mapping from one string to another.
    - `s`: A string to be searched for in the first element of each pair in the table.
- **Control Flow**:
    - Initialize an unsigned integer `i` to iterate over the table.
    - Loop through the table until a NULL entry is found in the first element of a pair.
    - For each pair, compare the input string `s` with the first element of the pair using `strcmp`.
    - If a match is found, return the second element of the pair.
    - If no match is found after the loop, return the input string `s`.
- **Output**:
    - Returns the mapped string from the table if a match is found, otherwise returns the input string `s`.


---
### cmd_find_get_session<!-- {{#callable:cmd_find_get_session}} -->
The `cmd_find_get_session` function attempts to locate a session based on a given session name or identifier and updates the provided `cmd_find_state` structure with the found session.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that will be updated with the found session.
    - `session`: A string representing the session name or identifier to search for.
- **Control Flow**:
    - Logs the function call with the session name for debugging purposes.
    - Checks if the session identifier starts with '$', indicating a session ID, and attempts to find the session by ID.
    - If the session is found by ID, updates `fs->s` and returns 0; otherwise, returns -1.
    - Attempts to find the session by name using [`session_find`](session.c.driver.md#session_find).
    - If the session is found by name, updates `fs->s` and returns 0.
    - Attempts to find a client with the given session name and checks if it has an associated session.
    - If a client with a session is found, updates `fs->s` with the client's session and returns 0.
    - If the `CMD_FIND_EXACT_SESSION` flag is set in `fs->flags`, returns -1 to stop further searching.
    - Iterates over all sessions to find a session whose name starts with the given session name as a prefix.
    - If a unique session is found with a matching prefix, updates `fs->s` and returns 0; if multiple matches are found, returns -1.
    - Iterates over all sessions to find a session whose name matches the given session name as a pattern using `fnmatch`.
    - If a unique session is found with a matching pattern, updates `fs->s` and returns 0; if multiple matches are found, returns -1.
    - If no session is found, returns -1.
- **Output**:
    - Returns 0 if a session is successfully found and updates `fs->s` with the session; returns -1 if no session is found or if multiple matches are found when searching by prefix or pattern.
- **Functions called**:
    - [`session_find_by_id_str`](session.c.driver.md#session_find_by_id_str)
    - [`session_find`](session.c.driver.md#session_find)
    - [`cmd_find_client`](#cmd_find_client)


---
### cmd_find_get_window<!-- {{#callable:cmd_find_get_window}} -->
The `cmd_find_get_window` function attempts to locate a window based on a given string identifier and updates the `cmd_find_state` structure with the found window and session information.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that will be updated with the found window and session information.
    - `window`: A string representing the window identifier, which can be a window ID starting with '@' or a window name.
    - `only`: An integer flag indicating whether to restrict the search to the current session only.
- **Control Flow**:
    - The function logs the window identifier for debugging purposes.
    - It checks if the window identifier starts with '@', indicating a window ID, and attempts to find the window by ID using [`window_find_by_id_str`](window.c.driver.md#window_find_by_id_str).
    - If a window is found by ID, it calls [`cmd_find_best_session_with_window`](#cmd_find_best_session_with_window) to find the best session containing the window and returns the result.
    - If the window identifier is not an ID, it uses the current session from `fs->current->s`.
    - It attempts to find the window within the current session using [`cmd_find_get_window_with_session`](#cmd_find_get_window_with_session).
    - If the window is found within the session, it returns success (0).
    - If the window is not found and `only` is false, it tries to interpret the window identifier as a session name using [`cmd_find_get_session`](#cmd_find_get_session).
    - If a session is found, it updates the `cmd_find_state` with the current window and index from the session and returns success (0).
    - If none of the above conditions are met, it returns failure (-1).
- **Output**:
    - The function returns 0 on success if the window is found and -1 on failure if the window cannot be located.
- **Functions called**:
    - [`window_find_by_id_str`](window.c.driver.md#window_find_by_id_str)
    - [`cmd_find_best_session_with_window`](#cmd_find_best_session_with_window)
    - [`cmd_find_get_window_with_session`](#cmd_find_get_window_with_session)
    - [`cmd_find_get_session`](#cmd_find_get_session)


---
### cmd_find_get_window_with_session<!-- {{#callable:cmd_find_get_window_with_session}} -->
The function `cmd_find_get_window_with_session` attempts to locate a window within a given session based on a string identifier, handling various formats and conditions.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that holds the current state of the command find operation, including session, window, and winlink information.
    - `window`: A string representing the window identifier, which can be a window ID, an offset, a special character, or a window name.
- **Control Flow**:
    - Log the function call with the window identifier.
    - Set the default window to the current window in the session.
    - Check if the window identifier starts with '@' and attempt to find the window by ID.
    - If the window identifier is an offset ('+' or '-'), calculate the new index or find the next/previous winlink.
    - Handle special characters ('!', '^', '$') to find the last, first, or last window in the session respectively.
    - Attempt to interpret the window identifier as a numeric index within the session.
    - Search for an exact match of the window name within the session's windows, returning an error if multiple matches are found.
    - If not in exact mode, search for windows starting with the given name or matching a pattern, returning an error if multiple matches are found.
    - Return -1 if no window is found or if multiple matches occur where only one is expected.
- **Output**:
    - Returns 0 if a window is successfully found and set in the `cmd_find_state`, or -1 if no suitable window is found or if there are multiple ambiguous matches.
- **Functions called**:
    - [`window_find_by_id_str`](window.c.driver.md#window_find_by_id_str)
    - [`session_has`](session.c.driver.md#session_has)
    - [`cmd_find_best_winlink_with_window`](#cmd_find_best_winlink_with_window)
    - [`strtonum`](compat/strtonum.c.driver.md#strtonum)
    - [`winlink_next_by_number`](window.c.driver.md#winlink_next_by_number)
    - [`winlink_previous_by_number`](window.c.driver.md#winlink_previous_by_number)
    - [`winlink_find_by_index`](window.c.driver.md#winlink_find_by_index)


---
### cmd_find_get_pane<!-- {{#callable:cmd_find_get_pane}} -->
The `cmd_find_get_pane` function attempts to locate a specific pane within a tmux session based on a given pane identifier or name.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that holds the current state of the command find operation.
    - `pane`: A string representing the pane identifier or name to be located.
    - `only`: An integer flag indicating whether to restrict the search to the current session and window only.
- **Control Flow**:
    - Log the function call and the pane identifier using `log_debug`.
    - Check if the pane identifier starts with '%', indicating a pane ID, and attempt to find the pane using [`window_pane_find_by_id_str`](window.c.driver.md#window_pane_find_by_id_str).
    - If the pane is found by ID, set the window in the `cmd_find_state` and find the best session with the window using [`cmd_find_best_session_with_window`](#cmd_find_best_session_with_window).
    - If the pane is not a pane ID, set the current session, window, and index in the `cmd_find_state`.
    - Attempt to find the pane within the current window using [`cmd_find_get_pane_with_window`](#cmd_find_get_pane_with_window).
    - If the pane is not found and `only` is not set, attempt to find the pane as a window using [`cmd_find_get_window`](#cmd_find_get_window), and set the active pane if successful.
    - Return -1 if the pane cannot be found.
- **Output**:
    - Returns 0 if the pane is successfully found and -1 if it cannot be located.
- **Functions called**:
    - [`window_pane_find_by_id_str`](window.c.driver.md#window_pane_find_by_id_str)
    - [`cmd_find_best_session_with_window`](#cmd_find_best_session_with_window)
    - [`cmd_find_get_pane_with_window`](#cmd_find_get_pane_with_window)
    - [`cmd_find_get_window`](#cmd_find_get_window)


---
### cmd_find_get_pane_with_session<!-- {{#callable:cmd_find_get_pane_with_session}} -->
The function `cmd_find_get_pane_with_session` attempts to find a specific pane within a given session, updating the `cmd_find_state` structure accordingly.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that holds the current state of the command find operation, including session, window, and pane information.
    - `pane`: A string representing the pane identifier, which can be a pane ID starting with '%' or a pane name.
- **Control Flow**:
    - The function logs the function name and the pane identifier for debugging purposes.
    - It checks if the pane identifier starts with '%', indicating a pane ID.
    - If it is a pane ID, it attempts to find the pane using [`window_pane_find_by_id_str`](window.c.driver.md#window_pane_find_by_id_str).
    - If the pane is found, it updates the `cmd_find_state` structure with the pane and its associated window, then calls [`cmd_find_best_winlink_with_window`](#cmd_find_best_winlink_with_window) to find the best winlink for the window.
    - If the pane is not found, it returns -1 indicating failure.
    - If the pane identifier does not start with '%', it uses the current window from the session stored in `fs->s`.
    - It updates the `cmd_find_state` structure with the current window and its index.
    - Finally, it calls [`cmd_find_get_pane_with_window`](#cmd_find_get_pane_with_window) to find the pane within the current window.
- **Output**:
    - The function returns 0 on success, indicating that the pane was found and the state was updated, or -1 on failure, indicating that the pane could not be found.
- **Functions called**:
    - [`window_pane_find_by_id_str`](window.c.driver.md#window_pane_find_by_id_str)
    - [`cmd_find_best_winlink_with_window`](#cmd_find_best_winlink_with_window)
    - [`cmd_find_get_pane_with_window`](#cmd_find_get_pane_with_window)


---
### cmd_find_get_pane_with_window<!-- {{#callable:cmd_find_get_pane_with_window}} -->
The function `cmd_find_get_pane_with_window` attempts to locate a specific pane within a given window based on a string identifier.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that holds the current state of the command find operation, including the window and pane being searched.
    - `pane`: A string representing the identifier of the pane to be found, which can be a pane ID, a special character, an offset, an index, or a description.
- **Control Flow**:
    - Log the function call with the provided pane identifier.
    - Check if the pane identifier starts with '%', indicating a pane ID, and attempt to find the pane by ID.
    - If the pane is found and belongs to the current window, return success (0); otherwise, return failure (-1).
    - Check for special character identifiers ('!', '{up-of}', '{down-of}', '{left-of}', '{right-of}') and attempt to find the corresponding pane relative to the active pane.
    - If a special character pane is found, return success (0); otherwise, return failure (-1).
    - If the pane identifier is an offset ('+' or '-'), calculate the offset and attempt to find the next or previous pane by number.
    - If a pane is found using the offset, return success (0).
    - Attempt to interpret the pane identifier as an index and find the pane at that index within the window.
    - If a pane is found at the index, return success (0).
    - Finally, attempt to find the pane using a description string.
    - If a pane is found using the description, return success (0).
    - If none of the methods succeed, return failure (-1).
- **Output**:
    - Returns 0 if the pane is successfully found and updated in the `cmd_find_state` structure, otherwise returns -1 if the pane cannot be found.
- **Functions called**:
    - [`window_pane_find_by_id_str`](window.c.driver.md#window_pane_find_by_id_str)
    - [`window_pane_find_up`](window.c.driver.md#window_pane_find_up)
    - [`window_pane_find_down`](window.c.driver.md#window_pane_find_down)
    - [`window_pane_find_left`](window.c.driver.md#window_pane_find_left)
    - [`window_pane_find_right`](window.c.driver.md#window_pane_find_right)
    - [`strtonum`](compat/strtonum.c.driver.md#strtonum)
    - [`window_pane_next_by_number`](window.c.driver.md#window_pane_next_by_number)
    - [`window_pane_previous_by_number`](window.c.driver.md#window_pane_previous_by_number)
    - [`window_pane_at_index`](window.c.driver.md#window_pane_at_index)
    - [`window_find_string`](window.c.driver.md#window_find_string)


---
### cmd_find_clear_state<!-- {{#callable:cmd_find_clear_state}} -->
The `cmd_find_clear_state` function initializes a `cmd_find_state` structure by setting all its fields to zero and then assigns specific values to the `flags` and `idx` fields.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that will be cleared and initialized.
    - `flags`: An integer representing the flags to be set in the `cmd_find_state` structure.
- **Control Flow**:
    - The function begins by using `memset` to set all bytes of the `cmd_find_state` structure pointed to by `fs` to zero, effectively clearing it.
    - The `flags` field of the `cmd_find_state` structure is then set to the value of the `flags` parameter.
    - The `idx` field of the `cmd_find_state` structure is set to -1, indicating an uninitialized or invalid index.
- **Output**:
    - The function does not return a value; it modifies the `cmd_find_state` structure pointed to by `fs` in place.


---
### cmd_find_empty_state<!-- {{#callable:cmd_find_empty_state}} -->
The `cmd_find_empty_state` function checks if all components of a `cmd_find_state` structure are NULL, indicating an empty state.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure, which contains pointers to session, winlink, window, and window pane.
- **Control Flow**:
    - The function checks if all the pointers (`s`, `wl`, `w`, `wp`) in the `cmd_find_state` structure are NULL.
    - If all pointers are NULL, the function returns 1, indicating the state is empty.
    - If any pointer is not NULL, the function returns 0, indicating the state is not empty.
- **Output**:
    - The function returns an integer: 1 if the state is empty (all pointers are NULL), or 0 if the state is not empty (any pointer is not NULL).


---
### cmd_find_valid_state<!-- {{#callable:cmd_find_valid_state}} -->
The `cmd_find_valid_state` function checks if a given `cmd_find_state` structure represents a valid state by verifying the presence and consistency of its session, winlink, window, and pane components.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that contains the session, winlink, window, and pane to be validated.
- **Control Flow**:
    - Check if any of the components (session, winlink, window, pane) in the `cmd_find_state` structure are NULL; if so, return 0 indicating an invalid state.
    - Verify if the session is alive using the [`session_alive`](session.c.driver.md#session_alive) function; if not, return 0 indicating an invalid state.
    - Iterate over the winlinks in the session's windows to find a winlink that matches both the window and the winlink in the `cmd_find_state`; if not found, return 0 indicating an invalid state.
    - Check if the window in the `cmd_find_state` matches the window in the winlink; if not, return 0 indicating an invalid state.
    - Return the result of [`window_has_pane`](window.c.driver.md#window_has_pane) to check if the window contains the specified pane, indicating a valid state if true.
- **Output**:
    - Returns an integer, 1 if the state is valid and 0 if it is invalid.
- **Functions called**:
    - [`session_alive`](session.c.driver.md#session_alive)
    - [`window_has_pane`](window.c.driver.md#window_has_pane)


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
    - The function does not return a value; it modifies the `dst` structure in place.


---
### cmd_find_log_state<!-- {{#callable:cmd_find_log_state}} -->
The `cmd_find_log_state` function logs the current state of a `cmd_find_state` structure, including session, winlink, window pane, and index information, using a specified prefix.
- **Inputs**:
    - `prefix`: A constant character pointer representing the prefix to be used in the log messages.
    - `fs`: A pointer to a `cmd_find_state` structure containing the state information to be logged.
- **Control Flow**:
    - Check if the session (`fs->s`) is not NULL; if so, log the session ID and name, otherwise log 's=none'.
    - Check if the winlink (`fs->wl`) is not NULL; if so, log the winlink index, whether the winlink's window matches the current window, and the window ID and name, otherwise log 'wl=none'.
    - Check if the window pane (`fs->wp`) is not NULL; if so, log the window pane ID, otherwise log 'wp=none'.
    - Check if the index (`fs->idx`) is not -1; if so, log the index, otherwise log 'idx=none'.
- **Output**:
    - The function does not return any value; it performs logging operations.


---
### cmd_find_from_session<!-- {{#callable:cmd_find_from_session}} -->
The `cmd_find_from_session` function initializes a `cmd_find_state` structure with the current session, window, and active pane from a given session.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that will be initialized with the session, window, and pane information.
    - `s`: A pointer to a `session` structure from which the current window and active pane will be extracted.
    - `flags`: An integer representing flags that may modify the behavior of the state initialization.
- **Control Flow**:
    - Call [`cmd_find_clear_state`](#cmd_find_clear_state) to reset the `cmd_find_state` structure `fs` with the provided `flags`.
    - Assign the session `s` to the session field `fs->s` of the `cmd_find_state` structure.
    - Set the current window link `fs->wl` to the current window of the session `s`.
    - Set the window `fs->w` to the window of the current window link `fs->wl`.
    - Set the active pane `fs->wp` to the active pane of the window `fs->w`.
    - Log the state using [`cmd_find_log_state`](#cmd_find_log_state) with the function name and the `cmd_find_state` structure `fs`.
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
    - Set the session (`s`) in `fs` to the session of the provided `winlink` (`wl`).
    - Set the winlink (`wl`) in `fs` to the provided `winlink` (`wl`).
    - Set the window (`w`) in `fs` to the window of the provided `winlink` (`wl`).
    - Set the active window pane (`wp`) in `fs` to the active pane of the window in the provided `winlink` (`wl`).
    - Call [`cmd_find_log_state`](#cmd_find_log_state) to log the current state of `fs`.
- **Output**:
    - The function does not return a value; it modifies the `cmd_find_state` structure pointed to by `fs`.
- **Functions called**:
    - [`cmd_find_clear_state`](#cmd_find_clear_state)
    - [`cmd_find_log_state`](#cmd_find_log_state)


---
### cmd_find_from_session_window<!-- {{#callable:cmd_find_from_session_window}} -->
The function `cmd_find_from_session_window` initializes a `cmd_find_state` structure with a given session and window, and attempts to find the best winlink for the window within the session.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that will be initialized and populated with session and window information.
    - `s`: A pointer to a `session` structure representing the session to be used in the state.
    - `w`: A pointer to a `window` structure representing the window to be used in the state.
    - `flags`: An integer representing flags that control the behavior of the state initialization and search process.
- **Control Flow**:
    - The function begins by clearing the state of the `cmd_find_state` structure `fs` using [`cmd_find_clear_state`](#cmd_find_clear_state) with the provided flags.
    - The session `s` and window `w` are assigned to the `s` and `w` fields of the `cmd_find_state` structure `fs`, respectively.
    - The function attempts to find the best winlink for the window within the session by calling [`cmd_find_best_winlink_with_window`](#cmd_find_best_winlink_with_window).
    - If [`cmd_find_best_winlink_with_window`](#cmd_find_best_winlink_with_window) returns a non-zero value, indicating failure, the state is cleared again and the function returns -1.
    - If successful, the active pane of the window is assigned to the `wp` field of the `cmd_find_state` structure `fs`.
    - The function logs the state using [`cmd_find_log_state`](#cmd_find_log_state) and returns 0 to indicate success.
- **Output**:
    - The function returns 0 on success, indicating that the state was successfully initialized and the best winlink was found, or -1 on failure, indicating that the best winlink could not be found.
- **Functions called**:
    - [`cmd_find_clear_state`](#cmd_find_clear_state)
    - [`cmd_find_best_winlink_with_window`](#cmd_find_best_winlink_with_window)
    - [`cmd_find_log_state`](#cmd_find_log_state)


---
### cmd_find_from_window<!-- {{#callable:cmd_find_from_window}} -->
The `cmd_find_from_window` function attempts to find the best session and winlink associated with a given window and updates the command find state accordingly.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that will be updated with the session, winlink, and pane information.
    - `w`: A pointer to the `window` structure for which the best session and winlink are to be found.
    - `flags`: An integer representing flags that may modify the behavior of the function.
- **Control Flow**:
    - The function begins by clearing the current state of `fs` using [`cmd_find_clear_state`](#cmd_find_clear_state) with the provided flags.
    - It assigns the provided window `w` to the `w` field of `fs`.
    - The function then calls [`cmd_find_best_session_with_window`](#cmd_find_best_session_with_window) to find the best session associated with the window `w`.
    - If [`cmd_find_best_session_with_window`](#cmd_find_best_session_with_window) returns a non-zero value, indicating failure, the state is cleared again and the function returns -1.
    - If the session is found successfully, it proceeds to call [`cmd_find_best_winlink_with_window`](#cmd_find_best_winlink_with_window) to find the best winlink for the window.
    - If [`cmd_find_best_winlink_with_window`](#cmd_find_best_winlink_with_window) also returns a non-zero value, the state is cleared again and the function returns -1.
    - If both session and winlink are found successfully, it sets the active pane of the window as the active pane in `fs`.
    - Finally, it logs the state using [`cmd_find_log_state`](#cmd_find_log_state) and returns 0 to indicate success.
- **Output**:
    - The function returns 0 on success, indicating that the best session and winlink were found and the state was updated; it returns -1 on failure, indicating that no suitable session or winlink could be found.
- **Functions called**:
    - [`cmd_find_clear_state`](#cmd_find_clear_state)
    - [`cmd_find_best_session_with_window`](#cmd_find_best_session_with_window)
    - [`cmd_find_best_winlink_with_window`](#cmd_find_best_winlink_with_window)
    - [`cmd_find_log_state`](#cmd_find_log_state)


---
### cmd_find_from_winlink_pane<!-- {{#callable:cmd_find_from_winlink_pane}} -->
The `cmd_find_from_winlink_pane` function initializes a `cmd_find_state` structure with information from a given winlink and window pane, and logs the state.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that will be initialized with the winlink and pane information.
    - `wl`: A pointer to a `winlink` structure representing the winlink to be used for initializing the state.
    - `wp`: A pointer to a `window_pane` structure representing the window pane to be used for initializing the state.
    - `flags`: An integer representing flags that may modify the behavior of the state initialization.
- **Control Flow**:
    - Call [`cmd_find_clear_state`](#cmd_find_clear_state) to reset the `cmd_find_state` structure `fs` with the provided `flags`.
    - Set the session (`s`) in `fs` to the session of the provided `winlink` (`wl`).
    - Set the winlink (`wl`) in `fs` to the provided `winlink` (`wl`).
    - Set the index (`idx`) in `fs` to the index of the provided `winlink` (`wl`).
    - Set the window (`w`) in `fs` to the window of the provided `winlink` (`wl`).
    - Set the window pane (`wp`) in `fs` to the provided `window_pane` (`wp`).
    - Call [`cmd_find_log_state`](#cmd_find_log_state) to log the current state of `fs`.
- **Output**:
    - The function does not return a value; it modifies the `cmd_find_state` structure pointed to by `fs`.
- **Functions called**:
    - [`cmd_find_clear_state`](#cmd_find_clear_state)
    - [`cmd_find_log_state`](#cmd_find_log_state)


---
### cmd_find_from_pane<!-- {{#callable:cmd_find_from_pane}} -->
The `cmd_find_from_pane` function updates a `cmd_find_state` structure to reflect the state associated with a specific `window_pane`.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that will be updated to reflect the state of the specified pane.
    - `wp`: A pointer to a `window_pane` structure representing the pane from which the state is to be derived.
    - `flags`: An integer representing flags that may modify the behavior of the function.
- **Control Flow**:
    - Call [`cmd_find_from_window`](#cmd_find_from_window) with the `window` of the given `window_pane` and the provided flags.
    - If [`cmd_find_from_window`](#cmd_find_from_window) returns a non-zero value, return -1 indicating failure.
    - Set the `wp` field of the `cmd_find_state` structure to the provided `window_pane`.
    - Log the state using [`cmd_find_log_state`](#cmd_find_log_state).
    - Return 0 indicating success.
- **Output**:
    - Returns 0 on success, or -1 if the state could not be derived from the window of the given pane.
- **Functions called**:
    - [`cmd_find_from_window`](#cmd_find_from_window)
    - [`cmd_find_log_state`](#cmd_find_log_state)


---
### cmd_find_from_nothing<!-- {{#callable:cmd_find_from_nothing}} -->
The `cmd_find_from_nothing` function initializes a `cmd_find_state` structure by finding the best available session and setting the current window, index, and active pane based on that session.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that will be initialized and populated with session, window, and pane information.
    - `flags`: An integer representing flags that modify the behavior of the function, such as preferences for session selection.
- **Control Flow**:
    - The function begins by clearing the state of the `cmd_find_state` structure using [`cmd_find_clear_state`](#cmd_find_clear_state) with the provided flags.
    - It attempts to find the best session using [`cmd_find_best_session`](#cmd_find_best_session) with no specific session list, a size of 0, and the provided flags.
    - If no session is found (`fs->s` is NULL), it clears the state again and returns -1 to indicate failure.
    - If a session is found, it sets the current window link (`fs->wl`), index (`fs->idx`), window (`fs->w`), and active pane (`fs->wp`) based on the found session.
    - The function logs the state using [`cmd_find_log_state`](#cmd_find_log_state) and returns 0 to indicate success.
- **Output**:
    - The function returns 0 on success, indicating that a session was found and the state was populated, or -1 on failure if no session could be found.
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
    - The function begins by clearing the current state in `fs` using [`cmd_find_clear_state`](#cmd_find_clear_state) with the provided flags.
    - It checks if the mouse event `m` is valid; if not, it returns -1 indicating failure.
    - If the mouse event is valid, it attempts to find the window pane associated with the mouse event using [`cmd_mouse_pane`](cmd.c.driver.md#cmd_mouse_pane), updating `fs` with the session and winlink information.
    - If no window pane is found, it clears the state again and returns -1.
    - If a window pane is found, it sets the window in `fs` to the window of the found winlink.
    - Finally, it logs the state using [`cmd_find_log_state`](#cmd_find_log_state) and returns 0 indicating success.
- **Output**:
    - The function returns 0 on success, indicating that the state was successfully found and set, or -1 on failure, indicating that the state could not be determined from the mouse event.
- **Functions called**:
    - [`cmd_find_clear_state`](#cmd_find_clear_state)
    - [`cmd_mouse_pane`](cmd.c.driver.md#cmd_mouse_pane)
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
    - If the client is not attached to a session, attempt to find the pane the client is running in using [`cmd_find_inside_pane`](#cmd_find_inside_pane).
    - If a pane is found, update `fs` with the window and session information associated with that pane.
    - If no pane is found or the session does not contain the pane, call [`cmd_find_from_nothing`](#cmd_find_from_nothing) to handle the state as if there is no client.
- **Output**:
    - The function returns 0 on success, indicating that the state was successfully determined and updated, or the result of [`cmd_find_from_nothing`](#cmd_find_from_nothing) if the client is NULL or the pane cannot be found.
- **Functions called**:
    - [`cmd_find_from_nothing`](#cmd_find_from_nothing)
    - [`cmd_find_clear_state`](#cmd_find_clear_state)
    - [`server_client_get_pane`](server-client.c.driver.md#server_client_get_pane)
    - [`cmd_find_from_session`](#cmd_find_from_session)
    - [`cmd_find_log_state`](#cmd_find_log_state)
    - [`cmd_find_inside_pane`](#cmd_find_inside_pane)
    - [`cmd_find_best_session_with_window`](#cmd_find_best_session_with_window)


---
### cmd_find_target<!-- {{#callable:cmd_find_target}} -->
The `cmd_find_target` function resolves a target string into a specific session, window, or pane state based on the given type and flags, updating the provided `cmd_find_state` structure accordingly.
- **Inputs**:
    - `fs`: A pointer to a `cmd_find_state` structure that will be updated with the resolved target state.
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item, used to retrieve the current state and client.
    - `target`: A string representing the target to be resolved, which can be a session, window, or pane identifier.
    - `type`: An enum `cmd_find_type` indicating the type of target to find (session, window, or pane).
    - `flags`: An integer representing various flags that modify the behavior of the function, such as `CMD_FIND_CANFAIL` or `CMD_FIND_QUIET`.
- **Control Flow**:
    - If the `CMD_FIND_CANFAIL` flag is set, the `CMD_FIND_QUIET` flag is also set.
    - The function logs the input arguments, including the target, type, and flags.
    - The current state is determined based on the marked pane, the current queue item, or the client associated with the item.
    - If the target is empty or NULL, the current state is used.
    - If the target is a mouse event or marked target, the corresponding state is resolved using mouse or marked pane information.
    - The target string is parsed to separate session, window, and pane components using ':' and '.' as delimiters.
    - Exact match flags are set if the session or window components start with '='.
    - The session, window, and pane components are mapped through conversion tables if applicable.
    - The function attempts to resolve the session, window, and pane components in various combinations, updating the `cmd_find_state` structure accordingly.
    - If any component cannot be resolved, an error is logged, and the function returns -1 unless `CMD_FIND_CANFAIL` is set, in which case it returns 0.
    - If the target is successfully resolved, the function logs the state and returns 0.
- **Output**:
    - The function returns 0 on success, indicating that the target was successfully resolved and the `cmd_find_state` structure was updated; it returns -1 on error unless the `CMD_FIND_CANFAIL` flag is set, in which case it returns 0.
- **Functions called**:
    - [`strlcat`](compat/strlcat.c.driver.md#strlcat)
    - [`cmd_find_clear_state`](#cmd_find_clear_state)
    - [`server_check_marked`](server.c.driver.md#server_check_marked)
    - [`cmd_find_valid_state`](#cmd_find_valid_state)
    - [`cmdq_get_current`](cmd-queue.c.driver.md#cmdq_get_current)
    - [`cmd_find_from_client`](#cmd_find_from_client)
    - [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client)
    - [`cmdq_get_event`](cmd-queue.c.driver.md#cmdq_get_event)
    - [`cmd_mouse_pane`](cmd.c.driver.md#cmd_mouse_pane)
    - [`cmd_mouse_window`](cmd.c.driver.md#cmd_mouse_window)
    - [`cmd_find_copy_state`](#cmd_find_copy_state)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
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
The `cmd_find_current_client` function attempts to find and return the current client associated with a given command queue item, or the best available client if no direct association is found.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, representing the command queue item from which the client is to be determined.
    - `quiet`: An integer flag indicating whether error messages should be suppressed (non-zero value) or not (zero value).
- **Control Flow**:
    - Initialize a `client` pointer `c` to NULL and attempt to retrieve the client from the `item` using [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client) if `item` is not NULL.
    - If `c` is not NULL and has an associated session, return `c` immediately as the current client.
    - Initialize a `client` pointer `found` to NULL to store the best client found.
    - If `c` is not NULL, attempt to find a window pane inside the client using [`cmd_find_inside_pane`](#cmd_find_inside_pane).
    - If a window pane is found, clear the `cmd_find_state` structure `fs` and set its window to the found pane's window.
    - Attempt to find the best session containing the window using [`cmd_find_best_session_with_window`](#cmd_find_best_session_with_window), and if successful, find the best client for that session using [`cmd_find_best_client`](#cmd_find_best_client).
    - If no window pane is found, attempt to find the best session using [`cmd_find_best_session`](#cmd_find_best_session) and then find the best client for that session.
    - If no client is found and `item` is not NULL and `quiet` is false, log an error message using `cmdq_error`.
    - Log a debug message indicating the result of the function and return the found client.
- **Output**:
    - Returns a pointer to the `client` structure representing the current or best client found, or NULL if no suitable client is found.
- **Functions called**:
    - [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client)
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
    - If `target` is NULL, the function calls [`cmd_find_current_client`](#cmd_find_current_client) to find the current client and returns it.
    - The `target` string is duplicated using [`xstrdup`](xmalloc.c.driver.md#xstrdup) to a local variable `copy`.
    - If `copy` ends with a colon, the colon is removed by setting the last character to a null terminator.
    - The function iterates over the list of clients using `TAILQ_FOREACH`, checking each client's session, name, and ttyname against `copy`.
    - If a matching client is found, the loop breaks and the client is returned.
    - If no client is found and `quiet` is false, an error message is reported using `cmdq_error`.
    - The `copy` string is freed before the function returns.
    - A debug log is generated with the target and the result of the search.
- **Output**:
    - Returns a pointer to the `client` structure if a matching client is found, or NULL if not found and `quiet` is true.
- **Functions called**:
    - [`cmd_find_current_client`](#cmd_find_current_client)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


