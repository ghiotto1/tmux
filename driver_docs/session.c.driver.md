# Purpose
The provided C source code is part of the tmux project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. This file is responsible for managing sessions within tmux, providing a comprehensive set of functions to create, find, manipulate, and destroy sessions. It defines the data structures and operations necessary to handle sessions and session groups, including creating new sessions, attaching and detaching windows to sessions, and synchronizing session groups. The code uses red-black trees to manage collections of sessions and session groups, ensuring efficient insertion, deletion, and lookup operations.

Key components of this file include functions for session lifecycle management, such as [`session_create`](#session_create), [`session_destroy`](#session_destroy), and [`session_free`](#session_free), which handle the creation, destruction, and memory management of session objects. The file also includes functions for session navigation and manipulation, such as [`session_next`](#session_next), [`session_previous`](#session_previous), and [`session_select`](#session_select), which allow users to move between windows within a session. Additionally, the code provides mechanisms for session group management, allowing sessions to be grouped and synchronized, which is useful for managing related sessions collectively. The file does not define a public API but rather serves as an internal component of the tmux application, providing essential functionality for session management.
# Imports and Dependencies

---
- `sys/types.h`
- `sys/time.h`
- `string.h`
- `stdlib.h`
- `unistd.h`
- `time.h`
- `tmux.h`


# Global Variables

---
### sessions
- **Type**: `struct sessions`
- **Description**: The `sessions` variable is a global instance of the `struct sessions` data structure. It is used to manage and store all active sessions within the application, providing a centralized list of session objects.
- **Use**: This variable is used to maintain a global list of all active sessions, allowing for operations such as session creation, deletion, and searching.


---
### next_session_id
- **Type**: `u_int`
- **Description**: The `next_session_id` is a global variable of type `u_int` (unsigned integer) that is used to assign unique identifiers to new sessions. It is incremented each time a new session is created to ensure that each session has a distinct ID.
- **Use**: This variable is used to generate and assign unique session IDs when creating new sessions.


---
### session_groups
- **Type**: `struct session_groups`
- **Description**: The `session_groups` variable is a global instance of the `struct session_groups` data structure, which is initialized using the `RB_INITIALIZER` macro. This structure is used to manage a collection of session groups, where each session group can contain multiple sessions.
- **Use**: This variable is used to store and manage session groups within the application, allowing for operations such as adding, removing, and synchronizing sessions within groups.


---
### session_next_alert
- **Type**: `static struct winlink *`
- **Description**: The `session_next_alert` is a static function that returns a pointer to a `struct winlink`. It is designed to iterate through a list of winlinks and find the next winlink that has alert flags set. This function is used to manage alerts within a session, specifically to identify the next window link that requires attention.
- **Use**: This function is used to find the next winlink with alert flags set, aiding in session management by identifying windows that need attention.


---
### session_previous_alert
- **Type**: `function`
- **Description**: The `session_previous_alert` is a static function that takes a pointer to a `winlink` structure as an argument and returns a pointer to a `winlink` structure. It is designed to traverse the linked list of `winlink` structures in reverse order, searching for a `winlink` that has alert flags set.
- **Use**: This function is used to find the previous `winlink` with alert flags set in a session's window list.


# Functions

---
### session_cmp<!-- {{#callable:session_cmp}} -->
The `session_cmp` function compares two session structures based on their names using the `strcmp` function.
- **Inputs**:
    - `s1`: A pointer to the first session structure to be compared.
    - `s2`: A pointer to the second session structure to be compared.
- **Control Flow**:
    - The function takes two session pointers as input.
    - It calls the `strcmp` function to compare the `name` fields of the two session structures.
    - The result of the `strcmp` function is returned, which indicates the lexicographical order of the session names.
- **Output**:
    - The function returns an integer that is less than, equal to, or greater than zero if the name of the first session is found, respectively, to be less than, to match, or be greater than the name of the second session.


---
### session_group_cmp<!-- {{#callable:session_group_cmp}} -->
The `session_group_cmp` function compares two session groups by their names using the `strcmp` function.
- **Inputs**:
    - `s1`: A pointer to the first session_group structure to be compared.
    - `s2`: A pointer to the second session_group structure to be compared.
- **Control Flow**:
    - The function calls `strcmp` with the `name` fields of the two session_group structures `s1` and `s2`.
    - The result of `strcmp` is returned, which indicates the lexicographical order of the two names.
- **Output**:
    - The function returns an integer that is less than, equal to, or greater than zero if the name of `s1` is found, respectively, to be less than, to match, or be greater than the name of `s2`.


---
### session_alive<!-- {{#callable:session_alive}} -->
The `session_alive` function checks if a given session is still active by verifying its presence in the global sessions list.
- **Inputs**:
    - `s`: A pointer to a session structure that needs to be checked for its presence in the global sessions list.
- **Control Flow**:
    - Iterate over each session in the global sessions list using the `RB_FOREACH` macro.
    - For each session in the list, check if it matches the session `s` passed as an argument.
    - If a match is found, return 1 indicating the session is alive.
    - If no match is found after iterating through the list, return 0 indicating the session is not alive.
- **Output**:
    - Returns 1 if the session is found in the global sessions list, otherwise returns 0.


---
### session_find<!-- {{#callable:session_find}} -->
The `session_find` function searches for a session by its name in a global Red-Black tree of sessions and returns a pointer to the session if found.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the session to be found.
- **Control Flow**:
    - A temporary `session` structure `s` is created and its `name` field is set to the input `name`.
    - The function calls `RB_FIND` with the global `sessions` tree, the address of the `sessions` tree, and the address of the temporary `session` structure `s`.
    - `RB_FIND` searches the Red-Black tree for a session with the same name as `s` and returns a pointer to the session if found, or `NULL` if not found.
- **Output**:
    - A pointer to the `session` structure if a session with the specified name is found, otherwise `NULL`.


---
### session_find_by_id_str<!-- {{#callable:session_find_by_id_str}} -->
The `session_find_by_id_str` function attempts to find a session by parsing a session ID from a string that starts with a '$' character.
- **Inputs**:
    - `s`: A string that is expected to start with a '$' character followed by a numeric session ID.
- **Control Flow**:
    - Check if the first character of the input string `s` is '$'; if not, return NULL.
    - Use `strtonum` to convert the substring of `s` (excluding the '$') to an unsigned integer `id`, checking for conversion errors.
    - If there is an error in conversion (i.e., `errstr` is not NULL), return NULL.
    - Call [`session_find_by_id`](#session_find_by_id) with the parsed `id` to find and return the corresponding session.
- **Output**:
    - Returns a pointer to the session structure if found, or NULL if the session ID is invalid or the session does not exist.
- **Functions called**:
    - [`session_find_by_id`](#session_find_by_id)


---
### session_find_by_id<!-- {{#callable:session_find_by_id}} -->
The `session_find_by_id` function searches for and returns a session with a specific ID from a global list of sessions.
- **Inputs**:
    - `id`: An unsigned integer representing the ID of the session to be found.
- **Control Flow**:
    - Iterate over each session `s` in the global `sessions` list using the `RB_FOREACH` macro.
    - Check if the current session's ID (`s->id`) matches the input `id`.
    - If a match is found, return the session `s`.
    - If no session with the matching ID is found after iterating through the list, return `NULL`.
- **Output**:
    - A pointer to the `session` structure with the matching ID, or `NULL` if no such session is found.


---
### session_create<!-- {{#callable:session_create}} -->
The `session_create` function initializes and creates a new session with specified parameters, ensuring it has a unique name and ID, and adds it to the global session list.
- **Inputs**:
    - `prefix`: A string used as a prefix for generating a session name if the name is not provided.
    - `name`: A string representing the desired name of the session; if NULL, a name is generated.
    - `cwd`: A string representing the current working directory for the session.
    - `env`: A pointer to an `environ` structure containing environment variables for the session.
    - `oo`: A pointer to an `options` structure containing options for the session.
    - `tio`: A pointer to a `termios` structure containing terminal I/O settings; can be NULL.
- **Control Flow**:
    - Allocate memory for a new session structure and initialize its reference count and flags.
    - Duplicate the current working directory string and assign it to the session's `cwd` field.
    - Initialize the session's `lastw` and `windows` data structures.
    - Assign the provided environment and options structures to the session.
    - Update the session's status cache.
    - If terminal I/O settings are provided, allocate memory for them and copy the settings into the session.
    - If a session name is provided, duplicate it and assign it to the session; otherwise, generate a unique name using the prefix and a unique ID.
    - Insert the session into the global session list, ensuring the name is unique.
    - Log the creation of the new session with its name and ID.
    - Retrieve the current time and set it as the session's creation time, updating the session's activity.
    - Return the newly created session structure.
- **Output**:
    - Returns a pointer to the newly created `session` structure.
- **Functions called**:
    - [`session_update_activity`](#session_update_activity)


---
### session_add_ref<!-- {{#callable:session_add_ref}} -->
The `session_add_ref` function increments the reference count of a session and logs the action with a debug message.
- **Inputs**:
    - `s`: A pointer to a `session` structure whose reference count is to be incremented.
    - `from`: A string indicating the source or reason for the reference increment, used in the debug log.
- **Control Flow**:
    - Increment the `references` field of the session structure pointed to by `s`.
    - Log a debug message with the function name, session name, source string, and the updated reference count.
- **Output**:
    - This function does not return any value; it modifies the session's reference count in place and logs a debug message.


---
### session_remove_ref<!-- {{#callable:session_remove_ref}} -->
The `session_remove_ref` function decrements the reference count of a session and triggers its cleanup if the count reaches zero.
- **Inputs**:
    - `s`: A pointer to the session structure whose reference count is to be decremented.
    - `from`: A string indicating the source or reason for the reference removal, used for logging purposes.
- **Control Flow**:
    - Decrement the reference count of the session `s`.
    - Log the current state of the session, including its name, the source of the reference removal, and the updated reference count.
    - Check if the reference count has reached zero.
    - If the reference count is zero, schedule the `session_free` function to be called once with a timeout event to clean up the session.
- **Output**:
    - The function does not return a value; it performs operations on the session object and logs information.


---
### session_free<!-- {{#callable:session_free}} -->
The `session_free` function deallocates a session's resources if its reference count is zero.
- **Inputs**:
    - `fd`: An unused integer parameter, typically representing a file descriptor.
    - `events`: An unused short parameter, typically representing event flags.
    - `arg`: A pointer to a session structure that is to be freed.
- **Control Flow**:
    - The function begins by casting the `arg` parameter to a `struct session` pointer `s`.
    - A debug log is generated to indicate the session being freed and its current reference count.
    - The function checks if the session's reference count (`s->references`) is zero.
    - If the reference count is zero, it proceeds to free the session's environment and options using `environ_free` and `options_free` respectively.
    - The session's name is freed using `free(s->name)`.
    - Finally, the session structure itself is freed using `free(s)`.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the session structure.


---
### session_destroy<!-- {{#callable:session_destroy}} -->
The `session_destroy` function is responsible for cleaning up and removing a session from the system, including notifying relevant components and freeing associated resources.
- **Inputs**:
    - `s`: A pointer to the session structure that is to be destroyed.
    - `notify`: An integer flag indicating whether to notify other components about the session closure.
    - `from`: A string indicating the source or reason for the session destruction.
- **Control Flow**:
    - Log the destruction of the session with its name and source.
    - Check if the current window (`curw`) of the session is NULL; if so, return immediately.
    - Set the current window (`curw`) to NULL to indicate no active window.
    - Remove the session from the global sessions list using `RB_REMOVE`.
    - If `notify` is true, call `notify_session` to inform about the session closure.
    - Free the terminal I/O settings associated with the session (`tio`).
    - If the session's lock timer is initialized, delete the event using `event_del`.
    - Remove the session from any session group it belongs to using [`session_group_remove`](#session_group_remove).
    - Iterate over the session's last window stack (`lastw`) and remove each winlink using `winlink_stack_remove`.
    - Iterate over the session's windows and remove each winlink, notifying about window unlinking using `notify_session_window`.
    - Free the current working directory string (`cwd`) associated with the session.
    - Remove a reference from the session using [`session_remove_ref`](#session_remove_ref), which may lead to freeing the session if no references remain.
- **Output**:
    - The function does not return any value; it performs cleanup operations and modifies the state of the session and related structures.
- **Functions called**:
    - [`session_group_remove`](#session_group_remove)
    - [`session_remove_ref`](#session_remove_ref)


---
### session_check_name<!-- {{#callable:session_check_name}} -->
The `session_check_name` function sanitizes a session name by replacing certain characters and encoding it for safe use.
- **Inputs**:
    - `name`: A constant character pointer representing the session name to be sanitized.
- **Control Flow**:
    - Check if the input name is an empty string; if so, return NULL.
    - Duplicate the input name into a new string `copy`.
    - Iterate over each character in `copy`, replacing ':' and '.' with '_'.
    - Encode the modified `copy` string into `new_name` using `utf8_stravis` with specific flags for octal, C-style, tab, and newline visibility.
    - Free the memory allocated for `copy`.
    - Return the sanitized and encoded `new_name`.
- **Output**:
    - A newly allocated string containing the sanitized and encoded session name, or NULL if the input name is empty.


---
### session_lock_timer<!-- {{#callable:session_lock_timer}} -->
The `session_lock_timer` function locks a session if it is attached and logs the session's lock status and activity time.
- **Inputs**:
    - `fd`: An unused integer representing a file descriptor.
    - `events`: An unused short integer representing event flags.
    - `arg`: A pointer to a session structure (`struct session *`) that represents the session to be potentially locked.
- **Control Flow**:
    - The function checks if the session (`s`) is attached by evaluating `s->attached`; if it is not attached (`s->attached == 0`), the function returns immediately without further action.
    - If the session is attached, the function logs a debug message indicating that the session is locked and records the session's name and activity time.
    - The function then calls `server_lock_session(s)` to lock the session.
    - Finally, it calls `recalculate_sizes()` to adjust any necessary sizes after the session lock.
- **Output**:
    - The function does not return any value; it performs actions on the session and logs information.


---
### session_update_activity<!-- {{#callable:session_update_activity}} -->
The `session_update_activity` function updates the activity timestamp of a session and manages the session's lock timer based on its attachment status and configuration options.
- **Inputs**:
    - `s`: A pointer to a `session` structure representing the session whose activity is being updated.
    - `from`: A pointer to a `timeval` structure representing the time to set as the session's activity time, or NULL to use the current time.
- **Control Flow**:
    - If `from` is NULL, the current time is obtained and set as the session's activity time; otherwise, the provided `from` time is used.
    - A debug log entry is created with the session's ID, name, and updated activity time.
    - If the session's lock timer is already initialized, it is deleted; otherwise, it is set with the `session_lock_timer` callback and the session as its argument.
    - If the session is attached, the lock timer is configured to trigger after a specified time interval obtained from the session's options, unless the interval is zero.
- **Output**:
    - The function does not return a value; it updates the session's activity time and manages its lock timer as a side effect.


---
### session_next_session<!-- {{#callable:session_next_session}} -->
The `session_next_session` function retrieves the next session in the global sessions list after a given session, or returns the first session if the given session is the last one, ensuring the session is alive and not empty.
- **Inputs**:
    - `s`: A pointer to the current session from which the next session is to be found.
- **Control Flow**:
    - Check if the global sessions list is empty or if the given session is not alive; if either is true, return NULL.
    - Use `RB_NEXT` to find the next session in the sessions list after the given session.
    - If no next session is found, use `RB_MIN` to get the first session in the list.
    - If the next session found is the same as the given session, return NULL to indicate no other session is available.
    - Return the next session found.
- **Output**:
    - Returns a pointer to the next session in the list, or NULL if no valid next session is found.
- **Functions called**:
    - [`session_alive`](#session_alive)


---
### session_previous_session<!-- {{#callable:session_previous_session}} -->
The `session_previous_session` function retrieves the previous session in a global session list relative to a given session, or returns NULL if no such session exists.
- **Inputs**:
    - `s`: A pointer to the current session for which the previous session is to be found.
- **Control Flow**:
    - Check if the global session list is empty or if the given session is not alive; if either is true, return NULL.
    - Use `RB_PREV` to find the previous session in the global session list relative to the given session `s`.
    - If no previous session is found, use `RB_MAX` to get the last session in the list.
    - If the found session is the same as the given session `s`, return NULL to indicate no previous session exists.
    - Return the found session pointer `s2`.
- **Output**:
    - Returns a pointer to the previous session in the global session list relative to the given session, or NULL if no such session exists.
- **Functions called**:
    - [`session_alive`](#session_alive)


---
### session_attach<!-- {{#callable:session_attach}} -->
The `session_attach` function attaches a specified window to a session at a given index, handling potential index conflicts and synchronizing session groups.
- **Inputs**:
    - `s`: A pointer to the session to which the window will be attached.
    - `w`: A pointer to the window that is to be attached to the session.
    - `idx`: An integer representing the index at which the window should be attached within the session's window list.
    - `cause`: A pointer to a string that will store an error message if the function fails due to an index conflict.
- **Control Flow**:
    - Attempt to add a new winlink at the specified index in the session's window list using `winlink_add`.
    - If the index is already in use (i.e., `winlink_add` returns NULL), format an error message indicating the index conflict and return NULL.
    - If successful, set the session of the new winlink to the provided session `s`.
    - Associate the provided window `w` with the new winlink using `winlink_set_window`.
    - Notify that a window has been linked to the session using `notify_session_window`.
    - Synchronize the session group from the current session using [`session_group_synchronize_from`](#session_group_synchronize_from).
    - Return the newly created winlink.
- **Output**:
    - Returns a pointer to the newly created winlink if successful, or NULL if the index is already in use.
- **Functions called**:
    - [`session_group_synchronize_from`](#session_group_synchronize_from)


---
### session_detach<!-- {{#callable:session_detach}} -->
The `session_detach` function detaches a specified window link from a session and synchronizes the session group, returning a status indicating if the session is now empty.
- **Inputs**:
    - `s`: A pointer to the session from which the window link is to be detached.
    - `wl`: A pointer to the winlink (window link) that is to be detached from the session.
- **Control Flow**:
    - Check if the current window of the session is the same as the window link to be detached, and if so, attempt to move to the last or previous window, then to the next window if necessary.
    - Clear alert flags from the window link.
    - Notify that the window has been unlinked from the session.
    - Remove the window link from the session's last window stack and from the session's windows.
    - Synchronize the session group from the current session.
    - Check if the session's windows are empty and return 1 if true, otherwise return 0.
- **Output**:
    - Returns 1 if the session has no more windows after detachment, otherwise returns 0.
- **Functions called**:
    - [`session_last`](#session_last)
    - [`session_previous`](#session_previous)
    - [`session_next`](#session_next)
    - [`session_group_synchronize_from`](#session_group_synchronize_from)


---
### session_has<!-- {{#callable:session_has}} -->
The `session_has` function checks if a given session is associated with a specific window by iterating through the window's winlinks.
- **Inputs**:
    - `s`: A pointer to a `struct session` representing the session to check for association with the window.
    - `w`: A pointer to a `struct window` representing the window whose winlinks are to be checked for association with the session.
- **Control Flow**:
    - Iterate over each `winlink` in the `winlinks` list of the window `w` using the `TAILQ_FOREACH` macro.
    - For each `winlink`, check if its `session` member matches the session `s`.
    - If a match is found, return 1 indicating the session is associated with the window.
    - If no match is found after iterating through all winlinks, return 0 indicating the session is not associated with the window.
- **Output**:
    - Returns 1 if the session is associated with the window, otherwise returns 0.


---
### session_is_linked<!-- {{#callable:session_is_linked}} -->
The `session_is_linked` function checks if a window is linked outside of its session, excluding session groups.
- **Inputs**:
    - `s`: A pointer to a `struct session`, representing the session to check.
    - `w`: A pointer to a `struct window`, representing the window to check within the session.
- **Control Flow**:
    - Check if the session `s` is part of a session group using [`session_group_contains`](#session_group_contains).
    - If the session is part of a session group, compare the window's reference count with the session group count.
    - If the session is not part of a session group, compare the window's reference count with 1.
- **Output**:
    - Returns 1 if the window is linked outside the session (not including session groups), otherwise returns 0.
- **Functions called**:
    - [`session_group_contains`](#session_group_contains)
    - [`session_group_count`](#session_group_count)


---
### session_next_alert<!-- {{#callable:session_next_alert}} -->
The `session_next_alert` function iterates through a linked list of `winlink` structures to find the next one with alert flags set.
- **Inputs**:
    - `wl`: A pointer to a `winlink` structure, representing the starting point in the linked list of windows to search for an alert.
- **Control Flow**:
    - The function enters a while loop that continues as long as `wl` is not NULL.
    - Inside the loop, it checks if the current `winlink`'s flags include `WINLINK_ALERTFLAGS`.
    - If the alert flags are set, the loop breaks, and the function prepares to return the current `winlink`.
    - If the alert flags are not set, the function moves to the next `winlink` in the list using `winlink_next(wl)`.
- **Output**:
    - The function returns a pointer to the first `winlink` structure with alert flags set, or NULL if no such `winlink` is found.


---
### session_next<!-- {{#callable:session_next}} -->
The `session_next` function moves the current session to the next window, optionally considering alert flags.
- **Inputs**:
    - `s`: A pointer to the current session structure.
    - `alert`: An integer flag indicating whether to consider alert flags when selecting the next window.
- **Control Flow**:
    - Check if the current window (`curw`) in the session `s` is NULL; if so, return -1 indicating failure.
    - Retrieve the next window link (`wl`) from the current window using `winlink_next`.
    - If `alert` is true, call [`session_next_alert`](#session_next_alert) to find the next window with alert flags set.
    - If no next window is found (`wl` is NULL), set `wl` to the first window in the session's window list.
    - If `alert` is true and no alert window is found, return -1 indicating failure.
    - Set the current window of the session to `wl` using [`session_set_current`](#session_set_current) and return the result.
- **Output**:
    - Returns 0 on success, or -1 if no suitable next window is found.
- **Functions called**:
    - [`session_next_alert`](#session_next_alert)
    - [`session_set_current`](#session_set_current)


---
### session_previous_alert<!-- {{#callable:session_previous_alert}} -->
The `session_previous_alert` function traverses a linked list of `winlink` structures to find the previous `winlink` with alert flags set.
- **Inputs**:
    - `wl`: A pointer to a `winlink` structure, representing the starting point in the linked list to search for a previous alert.
- **Control Flow**:
    - The function enters a loop that continues as long as `wl` is not `NULL`.
    - Within the loop, it checks if the current `winlink`'s flags include `WINLINK_ALERTFLAGS`.
    - If the alert flags are set, the loop breaks, and the function returns the current `winlink`.
    - If the alert flags are not set, it moves to the previous `winlink` in the list using `winlink_previous(wl)`.
- **Output**:
    - Returns a pointer to the `winlink` structure with alert flags set, or `NULL` if no such `winlink` is found.


---
### session_previous<!-- {{#callable:session_previous}} -->
The `session_previous` function moves the current session to the previous window, optionally considering alert flags, and updates the session's current window accordingly.
- **Inputs**:
    - `s`: A pointer to the `session` structure representing the current session.
    - `alert`: An integer flag indicating whether to consider alert flags when selecting the previous window.
- **Control Flow**:
    - Check if the current window (`curw`) in the session `s` is NULL; if so, return -1 indicating failure.
    - Call `winlink_previous` to get the previous window link from the current window.
    - If `alert` is true, call [`session_previous_alert`](#session_previous_alert) to find the previous window with alert flags set.
    - If no previous window is found, set `wl` to the maximum window link in the session's windows.
    - If `alert` is true and no alert window is found, return -1 indicating failure.
    - Call [`session_set_current`](#session_set_current) to update the session's current window to `wl` and return its result.
- **Output**:
    - Returns 0 on success, 1 if the current window remains unchanged, or -1 if no previous window is found.
- **Functions called**:
    - [`session_previous_alert`](#session_previous_alert)
    - [`session_set_current`](#session_set_current)


---
### session_select<!-- {{#callable:session_select}} -->
The `session_select` function sets the current window in a session to the window specified by its index.
- **Inputs**:
    - `s`: A pointer to a `struct session` representing the session in which the window is to be selected.
    - `idx`: An integer representing the index of the window to be selected within the session.
- **Control Flow**:
    - The function calls `winlink_find_by_index` to find the window link (`struct winlink`) in the session's windows list that corresponds to the given index `idx`.
    - It then calls [`session_set_current`](#session_set_current) with the session and the found window link to set the current window of the session to the specified window.
- **Output**:
    - The function returns the result of [`session_set_current`](#session_set_current), which is an integer indicating success (0) or failure (-1 if the window link is not found).
- **Functions called**:
    - [`session_set_current`](#session_set_current)


---
### session_last<!-- {{#callable:session_last}} -->
The `session_last` function sets the current window of a session to the last used window, if possible.
- **Inputs**:
    - `s`: A pointer to a `struct session`, representing the session whose last used window is to be set as the current window.
- **Control Flow**:
    - Retrieve the first window link from the session's last used windows list using `TAILQ_FIRST`.
    - Check if the retrieved window link is `NULL`; if so, return -1 indicating failure.
    - Check if the retrieved window link is the same as the current window; if so, return 1 indicating no change is needed.
    - Call [`session_set_current`](#session_set_current) to set the retrieved window link as the current window and return its result.
- **Output**:
    - Returns -1 if there is no last used window, 1 if the last used window is already the current window, or the result of [`session_set_current`](#session_set_current) if the current window is successfully changed.
- **Functions called**:
    - [`session_set_current`](#session_set_current)


---
### session_set_current<!-- {{#callable:session_set_current}} -->
The `session_set_current` function updates the current window link (`curw`) of a session to a specified window link (`wl`) and performs necessary updates and notifications.
- **Inputs**:
    - `s`: A pointer to the `session` structure representing the session whose current window link is to be updated.
    - `wl`: A pointer to the `winlink` structure representing the new window link to be set as the current window link for the session.
- **Control Flow**:
    - Check if `wl` is NULL; if so, return -1 indicating an error.
    - Check if `wl` is already the current window link (`curw`) of the session; if so, return 1 indicating no change is needed.
    - Remove `wl` from the session's last window stack (`lastw`).
    - Push the current window link (`curw`) onto the session's last window stack (`lastw`).
    - Set the session's current window link (`curw`) to `wl`.
    - If the global option `focus-events` is enabled, update the focus for the old and new windows.
    - Clear any alert flags on `wl`.
    - Update the activity timestamp for the window associated with `wl`.
    - Update the window offset for the terminal display.
    - Notify that the session's window has changed.
    - Return 0 indicating successful update.
- **Output**:
    - Returns 0 on successful update, 1 if the window link is already current, or -1 if the window link is NULL.


---
### session_group_contains<!-- {{#callable:session_group_contains}} -->
The `session_group_contains` function checks if a given session is part of any session group and returns the session group if found.
- **Inputs**:
    - `target`: A pointer to the session (`struct session *`) that is being checked for membership in a session group.
- **Control Flow**:
    - Iterate over each session group in the global `session_groups` red-black tree using `RB_FOREACH`.
    - For each session group, iterate over its sessions using `TAILQ_FOREACH`.
    - Check if the current session in the iteration matches the `target` session.
    - If a match is found, return the current session group (`sg`).
    - If no match is found after all iterations, return `NULL`.
- **Output**:
    - Returns a pointer to the `struct session_group` that contains the `target` session, or `NULL` if the session is not part of any group.


---
### session_group_find<!-- {{#callable:session_group_find}} -->
The `session_group_find` function searches for a session group by its name within a red-black tree of session groups.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the session group to be found.
- **Control Flow**:
    - A `session_group` structure `sg` is declared and its `name` field is set to the input `name`.
    - The function returns the result of `RB_FIND`, which searches the `session_groups` red-black tree for a node matching `sg`.
- **Output**:
    - A pointer to the `session_group` structure if found, otherwise `NULL`.


---
### session_group_new<!-- {{#callable:session_group_new}} -->
The `session_group_new` function creates a new session group with a given name or returns an existing one if it already exists.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the session group to be created or found.
- **Control Flow**:
    - Check if a session group with the given name already exists using [`session_group_find`](#session_group_find); if it does, return the existing session group.
    - Allocate memory for a new session group using `xcalloc` and assign it to `sg`.
    - Duplicate the provided name using `xstrdup` and assign it to `sg->name`.
    - Initialize the session group's session list using `TAILQ_INIT`.
    - Insert the new session group into the global session groups tree using `RB_INSERT`.
    - Return the newly created session group.
- **Output**:
    - Returns a pointer to the newly created or found session group structure.
- **Functions called**:
    - [`session_group_find`](#session_group_find)


---
### session_group_add<!-- {{#callable:session_group_add}} -->
The `session_group_add` function adds a session to a session group if it is not already part of any group.
- **Inputs**:
    - `sg`: A pointer to the session group to which the session should be added.
    - `s`: A pointer to the session that is to be added to the session group.
- **Control Flow**:
    - Check if the session `s` is already contained in any session group using `session_group_contains(s)`.
    - If the session is not part of any group (i.e., `session_group_contains(s)` returns `NULL`), insert the session `s` at the tail of the session group's session list using `TAILQ_INSERT_TAIL`.
- **Output**:
    - The function does not return any value; it modifies the session group by adding the session to it if applicable.
- **Functions called**:
    - [`session_group_contains`](#session_group_contains)


---
### session_group_remove<!-- {{#callable:session_group_remove}} -->
The `session_group_remove` function removes a session from its session group and deletes the group if it becomes empty.
- **Inputs**:
    - `s`: A pointer to the session (`struct session *`) that is to be removed from its session group.
- **Control Flow**:
    - Check if the session `s` is part of a session group using [`session_group_contains`](#session_group_contains); if not, return immediately.
    - Remove the session `s` from the session group's list of sessions using `TAILQ_REMOVE`.
    - Check if the session group is now empty using `TAILQ_EMPTY`; if it is, remove the session group from the global session groups tree using `RB_REMOVE`.
    - Free the memory allocated for the session group's name and the session group itself.
- **Output**:
    - The function does not return a value; it performs operations to remove a session from its group and potentially deletes the group.
- **Functions called**:
    - [`session_group_contains`](#session_group_contains)


---
### session_group_count<!-- {{#callable:session_group_count}} -->
The `session_group_count` function counts the number of sessions in a given session group.
- **Inputs**:
    - `sg`: A pointer to a `struct session_group`, representing the session group whose sessions are to be counted.
- **Control Flow**:
    - Initialize a counter `n` to zero.
    - Iterate over each session `s` in the session group `sg` using the `TAILQ_FOREACH` macro.
    - Increment the counter `n` for each session found in the group.
    - Return the final count `n`.
- **Output**:
    - The function returns an unsigned integer representing the number of sessions in the specified session group.


---
### session_group_attached_count<!-- {{#callable:session_group_attached_count}} -->
The function `session_group_attached_count` calculates the total number of clients attached to all sessions within a given session group.
- **Inputs**:
    - `sg`: A pointer to a `struct session_group`, which represents a group of sessions.
- **Control Flow**:
    - Initialize a counter `n` to zero.
    - Iterate over each session `s` in the session group `sg` using the `TAILQ_FOREACH` macro.
    - For each session `s`, add the value of `s->attached` to the counter `n`.
    - Return the total count `n`.
- **Output**:
    - The function returns an unsigned integer representing the total number of clients attached to all sessions in the specified session group.


---
### session_group_synchronize_to<!-- {{#callable:session_group_synchronize_to}} -->
The `session_group_synchronize_to` function synchronizes a given session with another session in the same session group.
- **Inputs**:
    - `s`: A pointer to the session that needs to be synchronized with another session in its session group.
- **Control Flow**:
    - Check if the session `s` is part of a session group using [`session_group_contains`](#session_group_contains); if not, return immediately.
    - Iterate over the sessions in the session group to find a target session that is not the same as `s`.
    - If a target session is found, call [`session_group_synchronize1`](#session_group_synchronize1) to synchronize the target session with `s`.
- **Output**:
    - The function does not return any value; it performs synchronization as a side effect.
- **Functions called**:
    - [`session_group_contains`](#session_group_contains)
    - [`session_group_synchronize1`](#session_group_synchronize1)


---
### session_group_synchronize_from<!-- {{#callable:session_group_synchronize_from}} -->
The `session_group_synchronize_from` function synchronizes all sessions in a session group to match the state of a target session.
- **Inputs**:
    - `target`: A pointer to the session that serves as the reference for synchronization within its session group.
- **Control Flow**:
    - Check if the target session is part of a session group using [`session_group_contains`](#session_group_contains); if not, return immediately.
    - Iterate over each session `s` in the session group using `TAILQ_FOREACH`.
    - For each session `s` that is not the target, call [`session_group_synchronize1`](#session_group_synchronize1) to synchronize it with the target session.
- **Output**:
    - The function does not return a value; it performs synchronization operations on sessions within a session group.
- **Functions called**:
    - [`session_group_contains`](#session_group_contains)
    - [`session_group_synchronize1`](#session_group_synchronize1)


---
### session_group_synchronize1<!-- {{#callable:session_group_synchronize1}} -->
The `session_group_synchronize1` function synchronizes the windows and state of a session with a target session, ensuring they are aligned.
- **Inputs**:
    - `target`: A pointer to the target session whose windows and state will be used to synchronize the other session.
    - `s`: A pointer to the session that will be synchronized to match the target session.
- **Control Flow**:
    - Check if the target session's windows are empty; if so, return immediately as the session will be destroyed.
    - If the current window in session `s` is not found in the target's windows, attempt to move to the next available window.
    - Save the current windows of session `s` and reset its windows list.
    - Iterate over each window in the target session, adding them to session `s` and setting appropriate flags and notifications.
    - Update the current window of session `s` to match the target session's current window if necessary.
    - Copy the last window stack from session `s`, reinitialize it, and update it with windows from the target session.
    - Free the old windows list of session `s`, notifying if any windows are unlinked.
- **Output**:
    - The function does not return a value; it modifies the session `s` to synchronize its windows and state with the target session.
- **Functions called**:
    - [`session_last`](#session_last)
    - [`session_previous`](#session_previous)
    - [`session_next`](#session_next)


---
### session_renumber_windows<!-- {{#callable:session_renumber_windows}} -->
The `session_renumber_windows` function renumbers the windows in a session, starting from a specified base index, and updates the session's window list and current window accordingly.
- **Inputs**:
    - `s`: A pointer to a `struct session` representing the session whose windows are to be renumbered.
- **Control Flow**:
    - Copy the current window list from the session into a temporary structure `old_wins` and initialize a new window list for the session.
    - Retrieve the base index from the session's options to start renumbering the windows.
    - Iterate over each window link (`wl`) in the old window list (`old_wins`), create a new window link (`wl_new`) in the session's new window list with the new index, and copy the window and flags from the old link to the new link.
    - Check if the current window or marked pane matches the current window link and update their indices accordingly.
    - Increment the new index for the next window link.
    - Copy the old last window stack into a temporary structure `old_lastw` and initialize a new last window stack for the session.
    - Iterate over each window link in the old last window stack, find the corresponding new window link, and insert it into the new last window stack while updating its flags.
    - Set the session's current window to the new window link corresponding to the updated current window index.
    - Free the old window links, reducing window references.
- **Output**:
    - The function does not return a value; it modifies the session's window list and current window in place.


---
### session_theme_changed<!-- {{#callable:session_theme_changed}} -->
The `session_theme_changed` function sets the `PANE_THEMECHANGED` flag for every pane in a given session's windows.
- **Inputs**:
    - `s`: A pointer to a `session` structure, representing the session whose panes' theme change flags are to be updated.
- **Control Flow**:
    - Check if the session pointer `s` is not NULL.
    - Iterate over each `winlink` in the session's `windows` red-black tree using `RB_FOREACH`.
    - For each `winlink`, iterate over each `window_pane` in the `window`'s `panes` queue using `TAILQ_FOREACH`.
    - Set the `PANE_THEMECHANGED` flag for each `window_pane`.
- **Output**:
    - The function does not return any value; it modifies the `flags` of each `window_pane` in the session's windows.


# Function Declarations (Public API)

---
### session_free<!-- {{#callable_declaration:session_free}} -->
Frees a session if it has no remaining references.
- **Description**: This function is used to release the resources associated with a session when it is no longer needed. It should be called when the session's reference count reaches zero, indicating that no other parts of the program are using it. The function will free the session's environment and options, as well as the session itself. It is important to ensure that the session is not accessed after this function is called, as it will have been deallocated.
- **Inputs**:
    - `fd`: This parameter is unused and can be ignored.
    - `events`: This parameter is unused and can be ignored.
    - `arg`: A pointer to the session object to be freed. Must not be null and should point to a valid session structure.
- **Output**: None
- **See also**: [`session_free`](#session_free)  (Implementation)


---
### session_lock_timer<!-- {{#callable_declaration:session_lock_timer}} -->
Locks a session if it is attached and has timed out.
- **Description**: This function is used to lock a session when it has been inactive for a specified period and is currently attached. It should be called as part of a timer event to enforce session locking based on inactivity. The function checks if the session is attached before proceeding to lock it, ensuring that only active sessions are locked. This is useful in scenarios where session security is a concern, and automatic locking is required after a period of inactivity.
- **Inputs**:
    - `fd`: This parameter is unused and can be ignored.
    - `events`: This parameter is unused and can be ignored.
    - `arg`: A pointer to a session object. This must not be null and should point to a valid session structure. The function expects this to be a session that is potentially attached and subject to locking.
- **Output**: None
- **See also**: [`session_lock_timer`](#session_lock_timer)  (Implementation)


---
### session_next_alert<!-- {{#callable_declaration:session_next_alert}} -->
Finds the next winlink with alert flags set.
- **Description**: Use this function to iterate through a list of winlinks starting from a given winlink, searching for the next winlink that has alert flags set. This is useful when you need to handle or process winlinks that require attention due to alerts. The function will return the first winlink with alert flags set, or NULL if no such winlink is found. Ensure that the starting winlink is valid and part of a properly initialized list.
- **Inputs**:
    - `wl`: A pointer to the starting winlink from which the search begins. It must be a valid winlink pointer, and the list must be properly initialized. If NULL, the function will immediately return NULL.
- **Output**: Returns a pointer to the next winlink with alert flags set, or NULL if no such winlink is found.
- **See also**: [`session_next_alert`](#session_next_alert)  (Implementation)


---
### session_previous_alert<!-- {{#callable_declaration:session_previous_alert}} -->
Finds the previous winlink with alert flags set.
- **Description**: Use this function to traverse backwards through a list of winlinks starting from a given winlink, searching for the first winlink that has alert flags set. This function is useful when you need to identify the previous alerting winlink in a sequence. It returns the first winlink with alert flags encountered, or NULL if no such winlink is found. Ensure that the starting winlink is valid and part of a properly linked list.
- **Inputs**:
    - `wl`: A pointer to the starting winlink from which the search begins. It must be a valid winlink in a linked list. If NULL, the function immediately returns NULL.
- **Output**: Returns a pointer to the previous winlink with alert flags set, or NULL if no such winlink exists.
- **See also**: [`session_previous_alert`](#session_previous_alert)  (Implementation)


---
### session_group_remove<!-- {{#callable_declaration:session_group_remove}} -->
Removes a session from its session group.
- **Description**: This function is used to remove a session from its associated session group. If the session is part of a group, it will be removed from the group's list of sessions. If the group becomes empty as a result, the group itself is destroyed. This function should be called when a session is no longer needed to be part of a group, ensuring that resources are properly released.
- **Inputs**:
    - `s`: A pointer to the session to be removed. Must not be null. The session should be part of a session group; otherwise, the function will have no effect.
- **Output**: None
- **See also**: [`session_group_remove`](#session_group_remove)  (Implementation)


---
### session_group_synchronize1<!-- {{#callable_declaration:session_group_synchronize1}} -->
Synchronizes the windows of one session with another.
- **Description**: Use this function to synchronize the windows of a session with those of a target session. It is typically used when sessions are part of a session group and need to maintain consistent window states. The function will update the current window, last window stack, and alert flags of the session to match the target. It should be called when the target session's window configuration changes and needs to be reflected in the session. The function assumes that the target session is not empty, as an empty session will be destroyed.
- **Inputs**:
    - `target`: A pointer to the target session whose windows will be used for synchronization. Must not be null and should have at least one window.
    - `s`: A pointer to the session that will be synchronized with the target session. Must not be null.
- **Output**: None
- **See also**: [`session_group_synchronize1`](#session_group_synchronize1)  (Implementation)


