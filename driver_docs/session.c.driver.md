# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file primarily manages session-related functionalities, providing a comprehensive set of operations for creating, managing, and synchronizing sessions and session groups. It defines a variety of functions to handle session lifecycle events, such as creation, destruction, and reference counting, as well as session-specific operations like attaching and detaching windows, updating session activity, and managing session groups.

The code utilizes data structures such as red-black trees and linked lists to efficiently manage sessions and their associated windows. Key components include functions for session comparison, session group management, and session synchronization, which ensure that sessions within a group remain consistent. The file also includes mechanisms for session locking based on inactivity and renumbering windows within a session. Overall, this file provides essential backend functionality for managing sessions in tmux, enabling users to seamlessly switch between and manage multiple terminal sessions.
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
- **Use**: This variable is used to maintain a global list of all session instances, allowing for operations such as session creation, deletion, and lookup.


---
### next_session_id
- **Type**: `u_int`
- **Description**: The `next_session_id` is a global variable of type `u_int` (unsigned integer) that is used to assign unique identifiers to new sessions. It is incremented each time a new session is created to ensure that each session has a distinct ID.
- **Use**: This variable is used to generate and assign unique session IDs when creating new sessions.


---
### session_groups
- **Type**: `struct session_groups`
- **Description**: The `session_groups` variable is a global instance of the `struct session_groups` data structure, which is initialized using the `RB_INITIALIZER` macro. This structure is used to manage a collection of session groups, where each session group can contain multiple sessions.
- **Use**: This variable is used to store and manage session groups, allowing operations such as adding, removing, and synchronizing sessions within groups.


---
### session_next_alert
- **Type**: `static struct winlink *`
- **Description**: The `session_next_alert` is a static function that returns a pointer to a `winlink` structure. It is designed to iterate through a list of `winlink` structures and find the next one that has alert flags set.
- **Use**: This function is used to locate the next `winlink` in a session that requires attention due to alert flags being set.


---
### session_previous_alert
- **Type**: `function`
- **Description**: The `session_previous_alert` is a static function that takes a pointer to a `struct winlink` as an argument and returns a pointer to a `struct winlink`. It is designed to find the previous winlink in a session that has alert flags set.
- **Use**: This function is used to navigate to the previous window link in a session that has an alert, allowing the session to respond to alerts in a backward direction.


# Functions

---
### session_cmp<!-- {{#callable:session_cmp}} -->
The `session_cmp` function compares two session structures based on their names using the `strcmp` function.
- **Inputs**:
    - `s1`: A pointer to the first session structure to be compared.
    - `s2`: A pointer to the second session structure to be compared.
- **Control Flow**:
    - The function calls `strcmp` with the `name` fields of the two session structures `s1` and `s2`.
    - The result of `strcmp` is returned, which indicates the lexicographical order of the session names.
- **Output**:
    - The function returns an integer less than, equal to, or greater than zero if the name of `s1` is found, respectively, to be less than, to match, or be greater than the name of `s2`.


---
### session_group_cmp<!-- {{#callable:session_group_cmp}} -->
The `session_group_cmp` function compares two session groups by their names using the `strcmp` function.
- **Inputs**:
    - `s1`: A pointer to the first session_group structure to be compared.
    - `s2`: A pointer to the second session_group structure to be compared.
- **Control Flow**:
    - The function takes two pointers to session_group structures as input.
    - It uses the `strcmp` function to compare the `name` fields of the two session_group structures.
    - The result of the `strcmp` function is returned, which indicates the lexicographical order of the names.
- **Output**:
    - The function returns an integer that is less than, equal to, or greater than zero if the name of the first session group is found, respectively, to be less than, to match, or be greater than the name of the second session group.


---
### session_alive<!-- {{#callable:session_alive}} -->
The `session_alive` function checks if a given session is still present in the global sessions list.
- **Inputs**:
    - `s`: A pointer to a session structure that needs to be checked for its presence in the global sessions list.
- **Control Flow**:
    - Iterate over each session in the global sessions list using the `RB_FOREACH` macro.
    - For each session in the list, check if it matches the session `s` passed as an argument.
    - If a match is found, return 1 indicating the session is alive.
    - If no match is found after iterating through the list, return 0 indicating the session is not alive.
- **Output**:
    - Returns an integer: 1 if the session is found in the global sessions list, otherwise 0.


---
### session_find<!-- {{#callable:session_find}} -->
The `session_find` function searches for a session by its name in a global Red-Black tree of sessions and returns a pointer to the session if found.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the session to be searched for.
- **Control Flow**:
    - A temporary `session` structure `s` is created and its `name` field is set to the input `name`.
    - The function uses `RB_FIND` to search for the session in the global `sessions` Red-Black tree using the temporary `session` structure `s`.
    - If a session with the matching name is found, a pointer to the session is returned; otherwise, `NULL` is returned.
- **Output**:
    - A pointer to the `session` structure if a session with the specified name is found, otherwise `NULL`.


---
### session_find_by_id_str<!-- {{#callable:session_find_by_id_str}} -->
The `session_find_by_id_str` function attempts to find a session by parsing a session ID from a string that starts with a '$' character.
- **Inputs**:
    - `s`: A string representing the session ID prefixed with a '$' character.
- **Control Flow**:
    - Check if the first character of the input string `s` is '$'; if not, return NULL.
    - Use [`strtonum`](compat/strtonum.c.driver.md#strtonum) to convert the substring of `s` (excluding the '$') to an unsigned integer `id`, checking for conversion errors.
    - If there is a conversion error, return NULL.
    - Call [`session_find_by_id`](#session_find_by_id) with the parsed `id` to find and return the corresponding session.
- **Output**:
    - Returns a pointer to the session structure if found, otherwise returns NULL.
- **Functions called**:
    - [`strtonum`](compat/strtonum.c.driver.md#strtonum)
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
The `session_create` function initializes and creates a new session with specified parameters, ensuring a unique session name and ID, and updates its activity and creation time.
- **Inputs**:
    - `prefix`: A string used as a prefix for generating a session name if the name is not provided.
    - `name`: A string representing the desired name of the session; if NULL, a name is generated.
    - `cwd`: A string representing the current working directory for the session.
    - `env`: A pointer to an `environ` structure containing environment variables for the session.
    - `oo`: A pointer to an `options` structure containing options for the session.
    - `tio`: A pointer to a `termios` structure for terminal I/O settings; can be NULL.
- **Control Flow**:
    - Allocate memory for a new session structure and initialize its reference count and flags.
    - Duplicate the current working directory string and assign it to the session's `cwd` field.
    - Initialize the session's `lastw` and `windows` data structures.
    - Assign the provided environment and options structures to the session.
    - Update the session's status cache.
    - If a `termios` structure is provided, allocate memory for it and copy its contents to the session's `tio` field.
    - If a session name is provided, duplicate it and assign it to the session's `name` field, then assign a new session ID.
    - If no session name is provided, generate a unique session name using the prefix and ensure it is unique by checking against existing sessions.
    - Insert the new session into the global sessions tree.
    - Log the creation of the new session with its name and ID.
    - Retrieve the current time and assign it to the session's creation time, handling any errors.
    - Update the session's activity time with the creation time.
- **Output**:
    - Returns a pointer to the newly created `session` structure.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`status_update_cache`](status.c.driver.md#status_update_cache)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`session_update_activity`](#session_update_activity)


---
### session_add_ref<!-- {{#callable:session_add_ref}} -->
The `session_add_ref` function increments the reference count of a session and logs the updated reference count along with the session name and a source identifier.
- **Inputs**:
    - `s`: A pointer to a `session` structure whose reference count is to be incremented.
    - `from`: A string indicating the source or reason for the reference increment, used for logging purposes.
- **Control Flow**:
    - Increment the `references` field of the session structure pointed to by `s`.
    - Log a debug message with the function name, session name, source identifier, and the new reference count.
- **Output**:
    - The function does not return any value; it modifies the session's reference count in place and logs a message.


---
### session_remove_ref<!-- {{#callable:session_remove_ref}} -->
The `session_remove_ref` function decrements the reference count of a session and triggers its cleanup if the count reaches zero.
- **Inputs**:
    - `s`: A pointer to the session structure whose reference count is to be decremented.
    - `from`: A string indicating the source or context from which the reference is being removed, used for logging purposes.
- **Control Flow**:
    - Decrement the `references` field of the session structure `s`.
    - Log the current state of the session, including its name, the source of the reference removal, and the updated reference count.
    - Check if the `references` count has reached zero.
    - If the `references` count is zero, schedule a one-time event to free the session using `event_once` with `session_free` as the callback.
- **Output**:
    - The function does not return a value; it performs operations on the session structure and logs information.


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
    - If the reference count is zero, it proceeds to free the session's environment and options using [`environ_free`](environ.c.driver.md#environ_free) and [`options_free`](options.c.driver.md#options_free) respectively.
    - The session's name is freed using `free(s->name)`.
    - Finally, the session structure itself is freed using `free(s)`.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the session structure.
- **Functions called**:
    - [`environ_free`](environ.c.driver.md#environ_free)
    - [`options_free`](options.c.driver.md#options_free)


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
    - If `notify` is true, call [`notify_session`](notify.c.driver.md#notify_session) to inform about the session closure.
    - Free the terminal I/O settings associated with the session (`s->tio`).
    - If the session's lock timer is initialized, delete the event using `event_del`.
    - Remove the session from any session group it belongs to using [`session_group_remove`](#session_group_remove).
    - Iterate over the session's last window stack (`lastw`) and remove each winlink using [`winlink_stack_remove`](window.c.driver.md#winlink_stack_remove).
    - Iterate over the session's windows and remove each winlink, notifying about window unlinking using [`notify_session_window`](notify.c.driver.md#notify_session_window).
    - Free the current working directory string (`s->cwd`).
    - Remove a reference from the session using [`session_remove_ref`](#session_remove_ref), which may lead to freeing the session if no references remain.
- **Output**:
    - The function does not return any value; it performs cleanup operations and modifies the state of the session and related structures.
- **Functions called**:
    - [`notify_session`](notify.c.driver.md#notify_session)
    - [`session_group_remove`](#session_group_remove)
    - [`winlink_stack_remove`](window.c.driver.md#winlink_stack_remove)
    - [`notify_session_window`](notify.c.driver.md#notify_session_window)
    - [`winlink_remove`](window.c.driver.md#winlink_remove)
    - [`session_remove_ref`](#session_remove_ref)


---
### session_check_name<!-- {{#callable:session_check_name}} -->
The `session_check_name` function sanitizes a session name by replacing certain characters and encoding it for safe use.
- **Inputs**:
    - `name`: A constant character pointer representing the session name to be sanitized.
- **Control Flow**:
    - Check if the input name is an empty string; if so, return NULL.
    - Duplicate the input name into a new string `copy` using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - Iterate over each character in `copy`, replacing ':' and '.' with '_'.
    - Use [`utf8_stravis`](utf8.c.driver.md#utf8_stravis) to encode `copy` into `new_name` with specific visibility flags.
    - Free the memory allocated for `copy`.
    - Return the newly created `new_name`.
- **Output**:
    - A newly allocated string with sanitized and encoded session name, or NULL if the input name is empty.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`utf8_stravis`](utf8.c.driver.md#utf8_stravis)


---
### session_lock_timer<!-- {{#callable:session_lock_timer}} -->
The `session_lock_timer` function locks a session if it is attached and logs the session's lock activity time.
- **Inputs**:
    - `fd`: An unused integer representing a file descriptor.
    - `events`: An unused short integer representing event flags.
    - `arg`: A pointer to a session structure (`struct session *`) that represents the session to be locked.
- **Control Flow**:
    - The function begins by casting the `arg` parameter to a `struct session *` and assigns it to the variable `s`.
    - It checks if the session `s` is attached by evaluating `s->attached`. If `s->attached` is 0, the function returns immediately, doing nothing further.
    - If the session is attached, it logs a debug message indicating that the session is locked and includes the session's name and activity time.
    - The function then calls `server_lock_session(s)` to lock the session.
    - Finally, it calls `recalculate_sizes()` to adjust any necessary sizes after the session is locked.
- **Output**:
    - The function does not return any value; it performs actions such as logging, locking the session, and recalculating sizes.
- **Functions called**:
    - [`server_lock_session`](server-fn.c.driver.md#server_lock_session)
    - [`recalculate_sizes`](resize.c.driver.md#recalculate_sizes)


---
### session_update_activity<!-- {{#callable:session_update_activity}} -->
The `session_update_activity` function updates the activity timestamp of a session and manages the session's lock timer based on its attachment status and lock timeout settings.
- **Inputs**:
    - `s`: A pointer to a `session` structure representing the session whose activity is being updated.
    - `from`: A pointer to a `timeval` structure representing the time to set as the session's activity time, or NULL to use the current time.
- **Control Flow**:
    - If `from` is NULL, the current time is obtained using `gettimeofday` and set as the session's activity time; otherwise, the time from `from` is copied to the session's activity time.
    - A debug log entry is created with the session's ID, name, and updated activity time.
    - If the session's lock timer is already initialized, it is deleted using `evtimer_del`; otherwise, it is set up with `evtimer_set` to call `session_lock_timer`.
    - If the session is attached (i.e., `s->attached` is not zero), the lock timeout is retrieved from the session's options using [`options_get_number`](options.c.driver.md#options_get_number).
    - If the lock timeout is non-zero, the lock timer is added with `evtimer_add` using the timeout value.
- **Output**:
    - The function does not return a value; it updates the session's activity time and manages the lock timer as a side effect.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)


---
### session_next_session<!-- {{#callable:session_next_session}} -->
The `session_next_session` function retrieves the next session in the global sessions list, or the first session if the current session is the last, ensuring the session is alive and not the same as the input session.
- **Inputs**:
    - `s`: A pointer to the current session from which the next session is to be found.
- **Control Flow**:
    - Check if the global sessions list is empty or if the session `s` is not alive; if either is true, return NULL.
    - Use `RB_NEXT` to find the next session in the sessions list after `s`.
    - If no next session is found, use `RB_MIN` to get the first session in the list.
    - If the found session is the same as the input session `s`, return NULL.
    - Return the found session `s2`.
- **Output**:
    - A pointer to the next session in the list, or NULL if no valid next session is found.
- **Functions called**:
    - [`session_alive`](#session_alive)


---
### session_previous_session<!-- {{#callable:session_previous_session}} -->
The `session_previous_session` function retrieves the previous session in a global session list relative to a given session, or returns NULL if no such session exists.
- **Inputs**:
    - `s`: A pointer to the current session from which the previous session is to be found.
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
The `session_attach` function attaches a window to a session at a specified index, handling potential index conflicts and synchronizing session groups.
- **Inputs**:
    - `s`: A pointer to the session to which the window will be attached.
    - `w`: A pointer to the window that is to be attached to the session.
    - `idx`: An integer representing the index at which the window should be attached within the session's window list.
    - `cause`: A pointer to a string that will be set with an error message if the function fails due to an index conflict.
- **Control Flow**:
    - Attempt to add a new winlink at the specified index in the session's window list using [`winlink_add`](window.c.driver.md#winlink_add).
    - If the index is already in use (i.e., [`winlink_add`](window.c.driver.md#winlink_add) returns NULL), format an error message indicating the index conflict and return NULL.
    - If the winlink is successfully added, set the session of the winlink to the provided session `s`.
    - Associate the provided window `w` with the newly created winlink using [`winlink_set_window`](window.c.driver.md#winlink_set_window).
    - Trigger a notification that a window has been linked to the session using [`notify_session_window`](notify.c.driver.md#notify_session_window).
    - Synchronize the session group from the current session using [`session_group_synchronize_from`](#session_group_synchronize_from).
    - Return the newly created winlink.
- **Output**:
    - Returns a pointer to the newly created winlink if successful, or NULL if the specified index is already in use.
- **Functions called**:
    - [`winlink_add`](window.c.driver.md#winlink_add)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`winlink_set_window`](window.c.driver.md#winlink_set_window)
    - [`notify_session_window`](notify.c.driver.md#notify_session_window)
    - [`session_group_synchronize_from`](#session_group_synchronize_from)


---
### session_detach<!-- {{#callable:session_detach}} -->
The `session_detach` function detaches a specified window link from a session and synchronizes the session group, returning a status indicating if the session is now empty.
- **Inputs**:
    - `s`: A pointer to the session from which the window link is to be detached.
    - `wl`: A pointer to the window link that is to be detached from the session.
- **Control Flow**:
    - Check if the current window link of the session is the one to be detached and if so, attempt to move to the last or previous window, then to the next window if necessary.
    - Clear alert flags from the window link to be detached.
    - Notify that the window has been unlinked from the session.
    - Remove the window link from the session's last window stack and from the session's windows.
    - Synchronize the session group from the current session to ensure consistency across the group.
    - Check if the session's windows are empty and return 1 if true, otherwise return 0.
- **Output**:
    - Returns 1 if the session has no more windows after detachment, otherwise returns 0.
- **Functions called**:
    - [`session_last`](#session_last)
    - [`session_previous`](#session_previous)
    - [`session_next`](#session_next)
    - [`notify_session_window`](notify.c.driver.md#notify_session_window)
    - [`winlink_stack_remove`](window.c.driver.md#winlink_stack_remove)
    - [`winlink_remove`](window.c.driver.md#winlink_remove)
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
    - If no match is found after iterating through all winlinks, return 0.
- **Output**:
    - Returns 1 if the session `s` is associated with the window `w`, otherwise returns 0.


---
### session_is_linked<!-- {{#callable:session_is_linked}} -->
The `session_is_linked` function checks if a given window is linked outside of its session, excluding session groups.
- **Inputs**:
    - `s`: A pointer to a `struct session` representing the session to check.
    - `w`: A pointer to a `struct window` representing the window to check within the session.
- **Control Flow**:
    - Check if the session `s` is part of a session group using [`session_group_contains`](#session_group_contains) function.
    - If the session is part of a session group, compare the window's reference count `w->references` with the session group count obtained from `session_group_count(sg)`.
    - If the session is not part of a session group, compare the window's reference count `w->references` with 1.
- **Output**:
    - Returns 1 if the window is linked outside the session (not including session groups), otherwise returns 0.
- **Functions called**:
    - [`session_group_contains`](#session_group_contains)
    - [`session_group_count`](#session_group_count)


---
### session_next_alert<!-- {{#callable:session_next_alert}} -->
The `session_next_alert` function iterates through a linked list of `winlink` structures to find the next one with alert flags set.
- **Inputs**:
    - `wl`: A pointer to a `winlink` structure, representing the starting point in the linked list to search for the next alert.
- **Control Flow**:
    - The function enters a while loop that continues as long as `wl` is not NULL.
    - Inside the loop, it checks if the current `winlink`'s flags include `WINLINK_ALERTFLAGS`.
    - If the alert flags are set, the loop breaks, and the current `winlink` is returned.
    - If not, the function moves to the next `winlink` in the list using `winlink_next(wl)`.
- **Output**:
    - Returns a pointer to the next `winlink` structure with alert flags set, or NULL if no such `winlink` is found.
- **Functions called**:
    - [`winlink_next`](window.c.driver.md#winlink_next)


---
### session_next<!-- {{#callable:session_next}} -->
The `session_next` function moves the current session to the next window, optionally considering alert flags, and updates the session's current window.
- **Inputs**:
    - `s`: A pointer to the current session structure.
    - `alert`: An integer flag indicating whether to consider alert flags when selecting the next window.
- **Control Flow**:
    - Check if the current window (`curw`) of the session `s` is NULL; if so, return -1 indicating failure.
    - Get the next window link (`wl`) from the current window link (`curw`).
    - If `alert` is true, call [`session_next_alert`](#session_next_alert) to find the next window with alert flags set.
    - If no next window link is found (`wl` is NULL), set `wl` to the minimum window link in the session's window tree.
    - If `alert` is true and no alert window is found, return -1 indicating failure.
    - Call [`session_set_current`](#session_set_current) to update the session's current window to `wl`.
- **Output**:
    - Returns 0 on success, or -1 if no suitable next window is found.
- **Functions called**:
    - [`winlink_next`](window.c.driver.md#winlink_next)
    - [`session_next_alert`](#session_next_alert)
    - [`session_set_current`](#session_set_current)


---
### session_previous_alert<!-- {{#callable:session_previous_alert}} -->
The function `session_previous_alert` traverses a linked list of `winlink` structures to find the previous `winlink` with alert flags set.
- **Inputs**:
    - `wl`: A pointer to a `winlink` structure, representing the starting point in the linked list to search for a previous alert.
- **Control Flow**:
    - The function enters a while loop that continues as long as `wl` is not `NULL`.
    - Inside the loop, it checks if the current `winlink`'s flags include `WINLINK_ALERTFLAGS`.
    - If the alert flags are set, the loop breaks, and the function returns the current `winlink`.
    - If the alert flags are not set, it moves to the previous `winlink` in the list using `winlink_previous(wl)`.
- **Output**:
    - Returns a pointer to the `winlink` structure with alert flags set, or `NULL` if no such `winlink` is found.
- **Functions called**:
    - [`winlink_previous`](window.c.driver.md#winlink_previous)


---
### session_previous<!-- {{#callable:session_previous}} -->
The `session_previous` function moves the current session to the previous window in the session's window list, optionally considering alert flags.
- **Inputs**:
    - `s`: A pointer to the `session` structure representing the current session.
    - `alert`: An integer flag indicating whether to consider alert flags when selecting the previous window.
- **Control Flow**:
    - Check if the current window (`curw`) in the session `s` is NULL; if so, return -1 indicating failure.
    - Call [`winlink_previous`](window.c.driver.md#winlink_previous) to get the previous window link from the current window.
    - If `alert` is true, call [`session_previous_alert`](#session_previous_alert) to find the previous window with alert flags set.
    - If no previous window is found, set `wl` to the maximum window link in the session's window list.
    - If `alert` is true and no alert window is found, return -1 indicating failure.
    - Call [`session_set_current`](#session_set_current) to set the current window to `wl` and return its result.
- **Output**:
    - Returns 0 on success, or -1 if no previous window could be set as current.
- **Functions called**:
    - [`winlink_previous`](window.c.driver.md#winlink_previous)
    - [`session_previous_alert`](#session_previous_alert)
    - [`session_set_current`](#session_set_current)


---
### session_select<!-- {{#callable:session_select}} -->
The `session_select` function sets the current window in a session to the window at a specified index.
- **Inputs**:
    - `s`: A pointer to a `session` structure representing the session in which the window is to be selected.
    - `idx`: An integer representing the index of the window to be selected within the session.
- **Control Flow**:
    - The function calls [`winlink_find_by_index`](window.c.driver.md#winlink_find_by_index) to find the window link (`winlink`) in the session's windows list that corresponds to the given index `idx`.
    - It then calls [`session_set_current`](#session_set_current) with the session and the found `winlink` to set the current window of the session to the specified window.
- **Output**:
    - The function returns the result of [`session_set_current`](#session_set_current), which is an integer indicating success (0), no change needed (1), or failure (-1).
- **Functions called**:
    - [`winlink_find_by_index`](window.c.driver.md#winlink_find_by_index)
    - [`session_set_current`](#session_set_current)


---
### session_last<!-- {{#callable:session_last}} -->
The `session_last` function sets the current window of a session to the last used window, if possible.
- **Inputs**:
    - `s`: A pointer to a `struct session` representing the session whose last used window is to be set as the current window.
- **Control Flow**:
    - Retrieve the first window link from the session's last used windows list using `TAILQ_FIRST`.
    - Check if the retrieved window link is `NULL`; if so, return -1 indicating failure to set the last window.
    - Check if the retrieved window link is the same as the current window; if so, return 1 indicating no change is needed.
    - Call [`session_set_current`](#session_set_current) to set the retrieved window link as the current window and return its result.
- **Output**:
    - Returns -1 if there is no last window to switch to, 1 if the last window is already the current window, or the result of [`session_set_current`](#session_set_current) if the switch is successful.
- **Functions called**:
    - [`session_set_current`](#session_set_current)


---
### session_set_current<!-- {{#callable:session_set_current}} -->
The `session_set_current` function updates the current window link (`curw`) of a session to a specified window link (`wl`) and performs necessary updates and notifications.
- **Inputs**:
    - `s`: A pointer to the `session` structure representing the session whose current window link is to be updated.
    - `wl`: A pointer to the `winlink` structure representing the new window link to be set as the current window link for the session.
- **Control Flow**:
    - Check if `wl` is NULL, return -1 if true.
    - Check if `wl` is already the current window link (`curw`), return 1 if true.
    - Remove `wl` from the session's last window stack (`lastw`).
    - Push the current window link (`curw`) onto the session's last window stack (`lastw`).
    - Set the session's current window link (`curw`) to `wl`.
    - If the global option `focus-events` is enabled, update the focus for the old and new windows.
    - Clear any alert flags on `wl`.
    - Update the activity timestamp for the window associated with `wl`.
    - Update the window offset for the terminal display.
    - Notify that the session's window has changed.
    - Return 0 to indicate successful update.
- **Output**:
    - Returns 0 on successful update, -1 if `wl` is NULL, and 1 if `wl` is already the current window link.
- **Functions called**:
    - [`winlink_stack_remove`](window.c.driver.md#winlink_stack_remove)
    - [`winlink_stack_push`](window.c.driver.md#winlink_stack_push)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`window_update_focus`](window.c.driver.md#window_update_focus)
    - [`winlink_clear_flags`](window.c.driver.md#winlink_clear_flags)
    - [`window_update_activity`](window.c.driver.md#window_update_activity)
    - [`tty_update_window_offset`](tty.c.driver.md#tty_update_window_offset)
    - [`notify_session`](notify.c.driver.md#notify_session)


---
### session_group_contains<!-- {{#callable:session_group_contains}} -->
The `session_group_contains` function checks if a given session is part of any session group and returns the session group if found.
- **Inputs**:
    - `target`: A pointer to the session that needs to be checked for membership in any session group.
- **Control Flow**:
    - Iterate over each session group in the global `session_groups` tree using `RB_FOREACH`.
    - For each session group, iterate over its sessions using `TAILQ_FOREACH`.
    - Check if the current session in the iteration matches the `target` session.
    - If a match is found, return the current session group.
    - If no match is found after all iterations, return `NULL`.
- **Output**:
    - Returns a pointer to the `session_group` containing the `target` session, or `NULL` if the session is not part of any group.


---
### session_group_find<!-- {{#callable:session_group_find}} -->
The `session_group_find` function searches for a session group by its name within a red-black tree of session groups.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the session group to be found.
- **Control Flow**:
    - A `session_group` structure `sg` is initialized and its `name` field is set to the input `name`.
    - The function then calls `RB_FIND` on the `session_groups` red-black tree, passing the address of `sg` to locate a session group with the matching name.
- **Output**:
    - Returns a pointer to the `session_group` structure if found, otherwise returns `NULL` if no matching session group is found.


---
### session_group_new<!-- {{#callable:session_group_new}} -->
The `session_group_new` function creates a new session group with a given name or returns an existing one if it already exists.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the session group to be created or found.
- **Control Flow**:
    - Check if a session group with the given name already exists using [`session_group_find`](#session_group_find); if it does, return the existing session group.
    - Allocate memory for a new session group using [`xcalloc`](xmalloc.c.driver.md#xcalloc) and assign it to `sg`.
    - Duplicate the provided name using [`xstrdup`](xmalloc.c.driver.md#xstrdup) and assign it to the `name` field of the session group.
    - Initialize the `sessions` queue within the session group using `TAILQ_INIT`.
    - Insert the new session group into the global `session_groups` red-black tree using `RB_INSERT`.
    - Return the newly created session group.
- **Output**:
    - Returns a pointer to the newly created or found session group structure.
- **Functions called**:
    - [`session_group_find`](#session_group_find)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### session_group_add<!-- {{#callable:session_group_add}} -->
The `session_group_add` function adds a session to a session group if it is not already part of any group.
- **Inputs**:
    - `sg`: A pointer to the session group to which the session should be added.
    - `s`: A pointer to the session that needs to be added to the session group.
- **Control Flow**:
    - Check if the session `s` is already part of any session group using `session_group_contains(s)`.
    - If the session is not part of any group (i.e., `session_group_contains(s)` returns NULL), insert the session `s` at the tail of the session group's session list using `TAILQ_INSERT_TAIL`.
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
    - Check if the session group is now empty using `TAILQ_EMPTY`; if it is, remove the session group from the global list of session groups using `RB_REMOVE`.
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
    - Initialize a counter variable `n` to 0.
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
    - The function does not return a value; it performs synchronization between sessions as a side effect.
- **Functions called**:
    - [`session_group_contains`](#session_group_contains)
    - [`session_group_synchronize1`](#session_group_synchronize1)


---
### session_group_synchronize_from<!-- {{#callable:session_group_synchronize_from}} -->
The function `session_group_synchronize_from` synchronizes all sessions in a session group to match the state of a target session.
- **Inputs**:
    - `target`: A pointer to the session that serves as the reference for synchronization.
- **Control Flow**:
    - Check if the target session is part of a session group using [`session_group_contains`](#session_group_contains); if not, return immediately.
    - Iterate over each session in the session group using `TAILQ_FOREACH`.
    - For each session that is not the target, call [`session_group_synchronize1`](#session_group_synchronize1) to synchronize it with the target session.
- **Output**:
    - The function does not return any value; it performs synchronization as a side effect.
- **Functions called**:
    - [`session_group_contains`](#session_group_contains)
    - [`session_group_synchronize1`](#session_group_synchronize1)


---
### session_group_synchronize1<!-- {{#callable:session_group_synchronize1}} -->
The `session_group_synchronize1` function synchronizes the windows and state of one session with another target session.
- **Inputs**:
    - `target`: A pointer to the target session whose windows and state will be used to synchronize the other session.
    - `s`: A pointer to the session that will be synchronized to match the target session.
- **Control Flow**:
    - Check if the target session's windows are empty; if so, return immediately as the session will be destroyed.
    - If the current window in session `s` is not found in the target's windows, attempt to move to the next window in session `s`.
    - Save the current windows of session `s` and initialize its windows to an empty state.
    - Iterate over each window in the target session's windows, adding them to session `s` and setting appropriate flags and notifications.
    - Update the current window of session `s` to match the target session's current window if necessary.
    - Copy the last window stack from session `s`, reinitialize it, and update it to reflect the new window links.
    - Free the old windows of session `s`, notifying if any windows are unlinked.
- **Output**:
    - The function does not return a value; it modifies the state of the session `s` to match the target session.
- **Functions called**:
    - [`winlink_find_by_index`](window.c.driver.md#winlink_find_by_index)
    - [`session_last`](#session_last)
    - [`session_previous`](#session_previous)
    - [`session_next`](#session_next)
    - [`winlink_add`](window.c.driver.md#winlink_add)
    - [`winlink_set_window`](window.c.driver.md#winlink_set_window)
    - [`notify_session_window`](notify.c.driver.md#notify_session_window)
    - [`winlink_find_by_window_id`](window.c.driver.md#winlink_find_by_window_id)
    - [`winlink_remove`](window.c.driver.md#winlink_remove)


---
### session_renumber_windows<!-- {{#callable:session_renumber_windows}} -->
The `session_renumber_windows` function renumbers the windows in a session, starting from a specified base index, and updates the session's window list and current window accordingly.
- **Inputs**:
    - `s`: A pointer to a `struct session` representing the session whose windows are to be renumbered.
- **Control Flow**:
    - Copy the current window list from the session into a temporary structure `old_wins` and initialize the session's window list to be empty.
    - Retrieve the base index from the session's options to start renumbering the windows.
    - Iterate over each window link (`winlink`) in the old window list, creating a new window link in the session's window list with the new index, and copy the window and flags from the old link to the new link.
    - If the current window or a marked pane is encountered, store their new indices for later use.
    - Increment the new index for the next window link.
    - Copy the stack of last windows from the session into a temporary structure `old_lastw` and initialize the session's last window stack to be empty.
    - Iterate over each window link in the old last window stack, find the corresponding new window link, and insert it into the session's last window stack, updating its flags.
    - Set the session's current window to the new window link corresponding to the previously stored current window index.
    - If a marked pane was found, update the marked pane to the new window link or clear it if not found.
    - Iterate over the old window list and remove each window link, freeing resources.
- **Output**:
    - The function does not return a value; it modifies the session's window list and current window in place.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`winlink_add`](window.c.driver.md#winlink_add)
    - [`winlink_set_window`](window.c.driver.md#winlink_set_window)
    - [`winlink_find_by_window`](window.c.driver.md#winlink_find_by_window)
    - [`winlink_find_by_index`](window.c.driver.md#winlink_find_by_index)
    - [`server_clear_marked`](server.c.driver.md#server_clear_marked)
    - [`winlink_remove`](window.c.driver.md#winlink_remove)


---
### session_theme_changed<!-- {{#callable:session_theme_changed}} -->
The `session_theme_changed` function sets the `PANE_THEMECHANGED` flag for every pane in a given session's windows.
- **Inputs**:
    - `s`: A pointer to a `session` structure, representing the session whose panes' theme change flags are to be updated.
- **Control Flow**:
    - Check if the session pointer `s` is not NULL.
    - Iterate over each `winlink` in the session's `windows` red-black tree using `RB_FOREACH`.
    - For each `winlink`, iterate over its `window`'s `panes` using `TAILQ_FOREACH`.
    - Set the `PANE_THEMECHANGED` flag for each `window_pane` in the `panes` list.
- **Output**:
    - The function does not return any value; it modifies the `flags` of each `window_pane` in the session's windows to include `PANE_THEMECHANGED`.


