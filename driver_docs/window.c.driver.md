# Purpose
The provided C source code is part of the `tmux` terminal multiplexer, specifically handling the management of windows and panes within a `tmux` session. This file is responsible for defining the structures and functions that manage the lifecycle and interactions of windows and their associated panes. Each window can contain multiple panes, which are essentially pseudo-terminals (pty) that can display different terminal sessions. The code includes functions for creating, destroying, and managing these windows and panes, as well as handling their layout, resizing, and input/output operations.

Key components of this code include the management of global lists and trees for windows and panes, functions for comparing and finding windows and panes by various criteria, and operations for manipulating the layout and focus of panes within a window. The code also handles the synchronization of input across panes, the management of pane modes, and the updating of pane states based on user interactions or changes in the environment. This file is integral to the core functionality of `tmux`, providing the necessary infrastructure to support the dynamic and flexible management of terminal sessions within a single windowed interface.
# Imports and Dependencies

---
- `sys/types.h`
- `sys/ioctl.h`
- `ctype.h`
- `errno.h`
- `fcntl.h`
- `fnmatch.h`
- `regex.h`
- `signal.h`
- `stdint.h`
- `stdlib.h`
- `string.h`
- `time.h`
- `unistd.h`
- `tmux.h`


# Global Variables

---
### windows
- **Type**: `struct windows`
- **Description**: The `windows` variable is a global instance of the `struct windows` data structure. This structure is used to manage a collection of window objects within the application, which are part of a larger system for handling terminal multiplexing.
- **Use**: The `windows` variable is used to store and manage all window instances globally, allowing for operations such as creation, destruction, and reference counting of windows.


---
### all_window_panes
- **Type**: `struct window_pane_tree`
- **Description**: The `all_window_panes` variable is a global data structure of type `struct window_pane_tree`. It is used to maintain a collection of all window panes in the application. This structure is likely implemented as a red-black tree, as indicated by the use of `RB_GENERATE` macros for managing window panes.
- **Use**: This variable is used to store and manage all window panes globally, allowing for operations such as insertion, deletion, and searching of window panes.


---
### next_window_pane_id
- **Type**: `u_int`
- **Description**: The `next_window_pane_id` is a static unsigned integer variable used to assign unique identifiers to window panes within the application. It is incremented each time a new window pane is created to ensure that each pane has a distinct ID.
- **Use**: This variable is used to generate unique IDs for new window panes as they are created.


---
### next_window_id
- **Type**: `u_int`
- **Description**: The `next_window_id` is a static unsigned integer variable used to assign unique identifiers to newly created windows in the application. It is incremented each time a new window is created, ensuring that each window has a distinct ID.
- **Use**: This variable is used to generate and track unique IDs for windows within the application.


---
### next_active_point
- **Type**: `u_int`
- **Description**: The `next_active_point` is a static unsigned integer variable used to track the next active point in the context of window panes within the application. It is likely used to assign a unique identifier or sequence number to active panes or events related to panes.
- **Use**: This variable is used to increment and assign a unique active point identifier to window panes when they become active.


---
### window_pane_create
- **Type**: ``struct window_pane *``
- **Description**: The `window_pane_create` function is a static function that returns a pointer to a `struct window_pane`. It is responsible for creating a new window pane within a given window, initializing its properties such as size, options, and screen buffers. The function also assigns a unique ID to the pane and inserts it into the global window pane tree.
- **Use**: This function is used to create and initialize a new window pane within a specified window, setting up its environment and adding it to the global pane management structures.


# Data Structures

---
### window_pane_input_data
- **Type**: `struct`
- **Members**:
    - `item`: A pointer to a `cmdq_item` structure, representing a command queue item associated with the window pane input.
    - `wp`: An unsigned integer representing the window pane identifier.
    - `file`: A pointer to a `client_file` structure, representing a file associated with the client.
- **Description**: The `window_pane_input_data` structure is used to encapsulate input data related to a window pane in a terminal multiplexer environment. It holds references to a command queue item, a window pane identifier, and a client file, which are essential for managing input operations and interactions within a specific window pane.


# Functions

---
### window_cmp<!-- {{#callable:window_cmp}} -->
The `window_cmp` function compares two `window` structures based on their `id` values.
- **Inputs**:
    - `w1`: A pointer to the first `window` structure to be compared.
    - `w2`: A pointer to the second `window` structure to be compared.
- **Control Flow**:
    - The function calculates the difference between the `id` fields of the two `window` structures.
    - It returns the result of the subtraction, which indicates the relative order of the two windows based on their IDs.
- **Output**:
    - An integer value representing the difference between the `id` of `w1` and `w2`, which can be negative, zero, or positive depending on their values.


---
### winlink_cmp<!-- {{#callable:winlink_cmp}} -->
The `winlink_cmp` function compares two `winlink` structures based on their index values.
- **Inputs**:
    - `wl1`: A pointer to the first `winlink` structure to compare.
    - `wl2`: A pointer to the second `winlink` structure to compare.
- **Control Flow**:
    - The function calculates the difference between the `idx` fields of the two `winlink` structures.
    - It returns the result of the subtraction, which indicates the relative order of the two `winlink` instances.
- **Output**:
    - An integer value representing the difference between the indices of the two `winlink` structures; a positive value indicates that `wl1` has a higher index than `wl2`, a negative value indicates the opposite, and zero indicates they are equal.


---
### window_pane_cmp<!-- {{#callable:window_pane_cmp}} -->
Compares two `window_pane` structures based on their IDs.
- **Inputs**:
    - `wp1`: A pointer to the first `window_pane` structure to compare.
    - `wp2`: A pointer to the second `window_pane` structure to compare.
- **Control Flow**:
    - The function calculates the difference between the IDs of `wp1` and `wp2`.
    - It returns the result of the subtraction, which indicates the relative order of the two panes based on their IDs.
- **Output**:
    - An integer representing the difference between the IDs of the two panes; a positive value indicates `wp1` has a higher ID than `wp2`, a negative value indicates the opposite, and zero indicates they are equal.


---
### winlink_find_by_window<!-- {{#callable:winlink_find_by_window}} -->
The `winlink_find_by_window` function searches for a `winlink` associated with a specific `window` in a given set of `winlinks`.
- **Inputs**:
    - `wwl`: A pointer to a `struct winlinks`, which is a collection of `winlink` structures.
    - `w`: A pointer to a `struct window`, which is the window to be searched for within the `winlinks`.
- **Control Flow**:
    - The function initializes a pointer `wl` to iterate through the `winlinks` using the `RB_FOREACH` macro.
    - For each `winlink` in the `winlinks`, it checks if the `window` pointer of the current `winlink` matches the provided `window` pointer `w`.
    - If a match is found, the function returns the corresponding `winlink` pointer.
    - If no match is found after iterating through all `winlinks`, the function returns `NULL`.
- **Output**:
    - The function returns a pointer to the `winlink` that corresponds to the specified `window`, or `NULL` if no such `winlink` exists.


---
### winlink_find_by_index<!-- {{#callable:winlink_find_by_index}} -->
The `winlink_find_by_index` function retrieves a `winlink` structure from a `winlinks` tree based on a specified index.
- **Inputs**:
    - `wwl`: A pointer to a `winlinks` structure that represents the tree of winlinks.
    - `idx`: An integer representing the index of the winlink to be found.
- **Control Flow**:
    - The function first checks if the provided index `idx` is less than 0, and if so, it calls `fatalx` to terminate the program with an error message.
    - If the index is valid, it initializes a `winlink` structure `wl` with the given index.
    - The function then uses `RB_FIND` to search for the `winlink` in the `winlinks` tree using the initialized `wl`.
- **Output**:
    - Returns a pointer to the found `winlink` structure if it exists, or NULL if it does not.


---
### winlink_find_by_window_id<!-- {{#callable:winlink_find_by_window_id}} -->
The `winlink_find_by_window_id` function searches for a `winlink` structure associated with a specific window ID.
- **Inputs**:
    - `wwl`: A pointer to a `struct winlinks`, which is a collection of `winlink` structures.
    - `id`: An unsigned integer representing the ID of the window to search for.
- **Control Flow**:
    - The function initializes a pointer `wl` to iterate through the `winlink` structures in the `wwl` collection.
    - It uses the `RB_FOREACH` macro to traverse each `winlink` in the `winlinks` tree.
    - For each `winlink`, it checks if the `window`'s ID matches the provided `id`.
    - If a match is found, it returns the corresponding `winlink` pointer.
    - If no match is found after traversing all `winlink` structures, it returns NULL.
- **Output**:
    - Returns a pointer to the `winlink` structure associated with the specified window ID, or NULL if no such `winlink` exists.


---
### winlink_next_index<!-- {{#callable:winlink_next_index}} -->
The `winlink_next_index` function finds the next available index for a window link.
- **Inputs**:
    - `wwl`: A pointer to a `struct winlinks` that contains the list of window links.
    - `idx`: An integer representing the starting index from which to search for the next available index.
- **Control Flow**:
    - Initializes a variable `i` with the value of `idx`.
    - Enters a do-while loop that continues until it has checked all indices.
    - Checks if the index `i` is available by calling [`winlink_find_by_index`](#winlink_find_by_index) with `wwl` and `i`.
    - If an available index is found (i.e., [`winlink_find_by_index`](#winlink_find_by_index) returns NULL), it returns the index `i`.
    - If `i` reaches `INT_MAX`, it wraps around to 0; otherwise, it increments `i`.
    - The loop continues until it returns to the original index `idx`, at which point it returns -1 if no available index is found.
- **Output**:
    - Returns the next available index for a window link, or -1 if no available index is found after checking all possible indices.
- **Functions called**:
    - [`winlink_find_by_index`](#winlink_find_by_index)


---
### winlink_count<!-- {{#callable:winlink_count}} -->
The `winlink_count` function counts the number of `winlink` structures in a given `winlinks` tree.
- **Inputs**:
    - `wwl`: A pointer to a `struct winlinks`, which is a red-black tree containing `winlink` structures.
- **Control Flow**:
    - Initializes a counter `n` to zero.
    - Iterates over each `winlink` in the `winlinks` tree using `RB_FOREACH` macro.
    - Increments the counter `n` for each `winlink` encountered.
    - Returns the final count of `winlink` structures.
- **Output**:
    - Returns an unsigned integer representing the total number of `winlink` structures in the provided `winlinks` tree.


---
### winlink_add<!-- {{#callable:winlink_add}} -->
The `winlink_add` function adds a new `winlink` to a `winlinks` structure at a specified index.
- **Inputs**:
    - `wwl`: A pointer to the `winlinks` structure where the new `winlink` will be added.
    - `idx`: An integer representing the index at which to add the new `winlink`. If negative, it is treated as a request to find the next available index.
- **Control Flow**:
    - If `idx` is negative, the function calls [`winlink_next_index`](#winlink_next_index) to find the next available index in the `winlinks` structure.
    - If `idx` is non-negative, the function checks if a `winlink` already exists at that index using [`winlink_find_by_index`](#winlink_find_by_index).
    - If the index is valid (either found or not already occupied), memory is allocated for a new `winlink` using `xcalloc`.
    - The new `winlink` is initialized with the specified index and inserted into the `winlinks` structure using `RB_INSERT`.
- **Output**:
    - Returns a pointer to the newly added `winlink` if successful, or NULL if the addition failed due to an invalid index.
- **Functions called**:
    - [`winlink_next_index`](#winlink_next_index)
    - [`winlink_find_by_index`](#winlink_find_by_index)


---
### winlink_set_window<!-- {{#callable:winlink_set_window}} -->
Sets the window associated with a given winlink.
- **Inputs**:
    - `wl`: A pointer to a `winlink` structure that represents the winlink to be updated.
    - `w`: A pointer to a `window` structure that represents the new window to associate with the winlink.
- **Control Flow**:
    - Checks if the `window` field of the `winlink` is not NULL.
    - If it is not NULL, removes the `winlink` from the current window's list of winlinks and decrements the reference count of the current window.
    - Inserts the `winlink` at the tail of the new window's list of winlinks.
    - Updates the `window` field of the `winlink` to point to the new window.
    - Increments the reference count of the new window.
- **Output**:
    - The function does not return a value; it modifies the state of the `winlink` and the associated `window` structures.
- **Functions called**:
    - [`window_remove_ref`](#window_remove_ref)
    - [`window_add_ref`](#window_add_ref)


---
### winlink_remove<!-- {{#callable:winlink_remove}} -->
The `winlink_remove` function removes a specified `winlink` from a `winlinks` structure and frees its memory.
- **Inputs**:
    - `wwl`: A pointer to a `winlinks` structure from which the `winlink` will be removed.
    - `wl`: A pointer to the `winlink` structure that is to be removed.
- **Control Flow**:
    - Check if the `window` associated with the `winlink` (`wl`) is not NULL.
    - If the `window` is not NULL, remove the `winlink` from the `winlinks` list of that `window` using `TAILQ_REMOVE`.
    - Decrement the reference count of the `window` using [`window_remove_ref`](#window_remove_ref).
    - Remove the `winlink` from the `winlinks` tree using `RB_REMOVE`.
    - Free the memory allocated for the `winlink`.
- **Output**:
    - The function does not return a value; it performs operations that modify the state of the `winlinks` structure and frees the memory of the removed `winlink`.
- **Functions called**:
    - [`window_remove_ref`](#window_remove_ref)


---
### winlink_next<!-- {{#callable:winlink_next}} -->
The `winlink_next` function retrieves the next `winlink` in a red-black tree of `winlinks`.
- **Inputs**:
    - `wl`: A pointer to a `winlink` structure representing the current `winlink`.
- **Control Flow**:
    - The function calls `RB_NEXT` macro to find the next `winlink` in the red-black tree.
    - It passes the `winlinks` tree, the type of the `winlink`, and the current `winlink` pointer to `RB_NEXT`.
- **Output**:
    - Returns a pointer to the next `winlink` in the tree, or NULL if there is no next `winlink`.


---
### winlink_previous<!-- {{#callable:winlink_previous}} -->
The `winlink_previous` function retrieves the previous `winlink` in a red-black tree.
- **Inputs**:
    - `wl`: A pointer to a `winlink` structure representing the current winlink.
- **Control Flow**:
    - The function calls `RB_PREV` macro to find the previous `winlink` in the red-black tree.
    - It passes the `winlinks` tree, the current `winlink` pointer, and the `wl` pointer to the macro.
- **Output**:
    - Returns a pointer to the previous `winlink` in the tree, or NULL if there is no previous winlink.


---
### winlink_next_by_number<!-- {{#callable:winlink_next_by_number}} -->
The `winlink_next_by_number` function retrieves the next `winlink` in a session's window list, wrapping around if necessary.
- **Inputs**:
    - `wl`: A pointer to the current `winlink` from which to start the search.
    - `s`: A pointer to the `session` structure that contains the list of windows.
    - `n`: An integer representing the number of `winlink` positions to advance.
- **Control Flow**:
    - The function enters a loop that continues as long as `n` is greater than 0.
    - Within the loop, it attempts to get the next `winlink` using `RB_NEXT`.
    - If `RB_NEXT` returns NULL (indicating the end of the list), it wraps around to the first `winlink` in the session's window list using `RB_MIN`.
- **Output**:
    - Returns a pointer to the `winlink` that is `n` positions ahead of the starting `winlink`, or the first `winlink` if the end of the list is reached.


---
### winlink_previous_by_number<!-- {{#callable:winlink_previous_by_number}} -->
The `winlink_previous_by_number` function retrieves the previous `winlink` in a session's window list based on a specified number of steps.
- **Inputs**:
    - `wl`: A pointer to the current `winlink` from which to start searching.
    - `s`: A pointer to the `session` containing the window list.
    - `n`: An integer representing the number of steps to move backwards in the window list.
- **Control Flow**:
    - The function enters a loop that continues as long as n is greater than 0.
    - Within the loop, it attempts to retrieve the previous `winlink` using `RB_PREV`.
    - If `RB_PREV` returns NULL, indicating that the current `winlink` is the first in the list, it resets `wl` to the last `winlink` in the session's window list using `RB_MAX`.
- **Output**:
    - Returns a pointer to the `winlink` that is `n` steps before the current `winlink`, or the last `winlink` if the end of the list is reached.


---
### winlink_stack_push<!-- {{#callable:winlink_stack_push}} -->
The `winlink_stack_push` function adds a `winlink` to the top of a `winlink_stack` after removing it if it already exists.
- **Inputs**:
    - `stack`: A pointer to the `winlink_stack` structure where the `winlink` will be pushed.
    - `wl`: A pointer to the `winlink` structure that is to be pushed onto the stack.
- **Control Flow**:
    - The function first checks if the `wl` pointer is NULL; if it is, the function returns immediately without making any changes.
    - If `wl` is not NULL, it calls [`winlink_stack_remove`](#winlink_stack_remove) to remove the `wl` from the stack if it is already present.
    - Then, it inserts `wl` at the head of the `stack` using `TAILQ_INSERT_HEAD`.
    - Finally, it sets the `WINLINK_VISITED` flag for the `wl` to indicate that it has been added to the stack.
- **Output**:
    - The function does not return a value; it modifies the `winlink_stack` and the `winlink` structure in place.
- **Functions called**:
    - [`winlink_stack_remove`](#winlink_stack_remove)


---
### winlink_stack_remove<!-- {{#callable:winlink_stack_remove}} -->
Removes a `winlink` from the `winlink_stack` if it has been visited.
- **Inputs**:
    - `stack`: A pointer to the `winlink_stack` structure from which the `winlink` will be removed.
    - `wl`: A pointer to the `winlink` structure that is to be removed from the stack.
- **Control Flow**:
    - Checks if the `wl` pointer is not NULL and if the `WINLINK_VISITED` flag is set for the `winlink`.
    - If both conditions are met, it removes the `winlink` from the `winlink_stack` using `TAILQ_REMOVE`.
    - Finally, it clears the `WINLINK_VISITED` flag for the `winlink`.
- **Output**:
    - The function does not return a value; it modifies the state of the `winlink` and the `winlink_stack` directly.


---
### window_find_by_id_str<!-- {{#callable:window_find_by_id_str}} -->
Finds a `window` by its string identifier.
- **Inputs**:
    - `s`: A string representing the identifier of the window, which must start with '@'.
- **Control Flow**:
    - Checks if the first character of the input string `s` is '@'. If not, returns NULL.
    - Uses `strtonum` to convert the substring starting from the second character of `s` into an unsigned integer `id`, checking for errors.
    - If `strtonum` returns an error, returns NULL.
    - Calls [`window_find_by_id`](#window_find_by_id) with the parsed `id` to retrieve the corresponding `window`.
- **Output**:
    - Returns a pointer to the `window` associated with the given identifier, or NULL if the identifier is invalid.
- **Functions called**:
    - [`window_find_by_id`](#window_find_by_id)


---
### window_find_by_id<!-- {{#callable:window_find_by_id}} -->
The `window_find_by_id` function retrieves a `window` structure from a red-black tree based on a given window ID.
- **Inputs**:
    - `id`: An unsigned integer representing the unique identifier of the window to be found.
- **Control Flow**:
    - A temporary `window` structure `w` is created and its `id` field is set to the input `id`.
    - The function then calls `RB_FIND` to search for the `window` in the global `windows` red-black tree using the temporary structure.
- **Output**:
    - Returns a pointer to the `window` structure if found; otherwise, it returns NULL.


---
### window_update_activity<!-- {{#callable:window_update_activity}} -->
Updates the activity time of a window and queues an alert for window activity.
- **Inputs**:
    - `w`: A pointer to a `struct window` representing the window whose activity is being updated.
- **Control Flow**:
    - Calls `gettimeofday` to retrieve the current time and updates the `activity_time` field of the window structure.
    - Invokes `alerts_queue` function to queue an alert indicating that there has been activity in the window.
- **Output**:
    - This function does not return a value; it modifies the state of the window and triggers an alert.


---
### window_create<!-- {{#callable:window_create}} -->
Creates a new window with specified dimensions and initializes its properties.
- **Inputs**:
    - `sx`: The width of the window in character cells.
    - `sy`: The height of the window in character cells.
    - `xpixel`: The width of the window in pixels, defaults to DEFAULT_XPIXEL if zero.
    - `ypixel`: The height of the window in pixels, defaults to DEFAULT_YPIXEL if zero.
- **Control Flow**:
    - Checks if `xpixel` and `ypixel` are zero and assigns default values if they are.
    - Allocates memory for a new `window` structure using `xcalloc`.
    - Initializes the `window` properties such as name, flags, dimensions, and options.
    - Increments the global window ID counter and inserts the new window into a red-black tree.
    - Calls [`window_set_fill_character`](#window_set_fill_character) to set the fill character for the window.
    - Calls [`window_update_activity`](#window_update_activity) to update the activity timestamp of the window.
    - Logs the creation of the window and returns a pointer to the newly created window.
- **Output**:
    - Returns a pointer to the newly created `window` structure.
- **Functions called**:
    - [`window_set_fill_character`](#window_set_fill_character)
    - [`window_update_activity`](#window_update_activity)


---
### window_destroy<!-- {{#callable:window_destroy}} -->
The `window_destroy` function cleans up and deallocates resources associated with a `window` structure.
- **Inputs**:
    - `w`: A pointer to the `window` structure that is to be destroyed.
- **Control Flow**:
    - Logs the destruction of the window along with its ID and reference count.
    - Calls [`window_unzoom`](#window_unzoom) to reset the window's zoom state.
    - Removes the window from the global list of windows using `RB_REMOVE`.
    - Frees the layout cells if they exist by calling `layout_free_cell` for both `layout_root` and `saved_layout_root`.
    - Frees the old layout pointer.
    - Calls [`window_destroy_panes`](#window_destroy_panes) to clean up all panes associated with the window.
    - Checks if the `name_event` timer is initialized and deletes it if so.
    - Checks if the `alerts_timer` is initialized and deletes it if so.
    - Checks if the `offset_timer` is initialized and deletes it if so.
    - Frees the options associated with the window.
    - Frees the fill character string.
    - Frees the name of the window.
    - Finally, frees the window structure itself.
- **Output**:
    - This function does not return a value; it performs cleanup and deallocation of resources associated with the specified `window`.
- **Functions called**:
    - [`window_unzoom`](#window_unzoom)
    - [`window_destroy_panes`](#window_destroy_panes)


---
### window_pane_destroy_ready<!-- {{#callable:window_pane_destroy_ready}} -->
The `window_pane_destroy_ready` function checks if a `window_pane` can be safely destroyed.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure that is being checked for readiness to be destroyed.
- **Control Flow**:
    - The function first checks if the `pipe_fd` of the `window_pane` is not -1, indicating that it is associated with a pipe.
    - If the `pipe_event` output buffer length is not zero, it returns 0, indicating that the pane is not ready to be destroyed.
    - Next, it checks if there are any bytes available to read from the file descriptor associated with the pane using `ioctl`.
    - If there are bytes available, it returns 0, indicating that the pane is still active.
    - Finally, it checks if the `PANE_EXITED` flag is not set; if it is not set, it returns 0, indicating the pane is not ready to be destroyed.
    - If all checks pass, it returns 1, indicating that the pane can be safely destroyed.
- **Output**:
    - The function returns 1 if the `window_pane` is ready to be destroyed, and 0 otherwise.


---
### window_add_ref<!-- {{#callable:window_add_ref}} -->
Increments the reference count of a `window` structure and logs the action.
- **Inputs**:
    - `w`: A pointer to the `window` structure whose reference count is to be incremented.
    - `from`: A string indicating the source or context from which the reference is being added.
- **Control Flow**:
    - The function increments the `references` field of the `window` structure pointed to by `w`.
    - It then calls `log_debug` to log the increment action, including the function name, window ID, source string, and the new reference count.
- **Output**:
    - The function does not return a value; it modifies the state of the `window` by increasing its reference count and logs the action.


---
### window_remove_ref<!-- {{#callable:window_remove_ref}} -->
Decrements the reference count of a `window` and destroys it if the count reaches zero.
- **Inputs**:
    - `w`: A pointer to the `window` structure whose reference count is to be decremented.
    - `from`: A string indicating the context or source from which the reference is being removed, used for logging.
- **Control Flow**:
    - Decrements the `references` field of the `window` structure pointed to by `w`.
    - Logs the current state of the `window` reference count using `log_debug`.
    - Checks if the `references` count has reached zero.
    - If the count is zero, calls the [`window_destroy`](#window_destroy) function to free the resources associated with the `window`.
- **Output**:
    - The function does not return a value; it modifies the state of the `window` and may lead to its destruction.
- **Functions called**:
    - [`window_destroy`](#window_destroy)


---
### window_set_name<!-- {{#callable:window_set_name}} -->
Sets the name of a window to a new specified name.
- **Inputs**:
    - `w`: A pointer to the `window` structure that represents the window whose name is to be set.
    - `new_name`: A pointer to a string containing the new name to be assigned to the window.
- **Control Flow**:
    - The function first frees the memory allocated for the current name of the window using `free(w->name)`.
    - It then calls `utf8_stravis` to convert the `new_name` into a UTF-8 encoded string and assigns it to `w->name`.
    - Finally, it invokes `notify_window` to send a notification that the window has been renamed.
- **Output**:
    - The function does not return a value; it modifies the state of the window by changing its name and notifying other components of the change.


---
### window_resize<!-- {{#callable:window_resize}} -->
The `window_resize` function adjusts the size and pixel dimensions of a specified window.
- **Inputs**:
    - `struct window *w`: A pointer to the `window` structure that is to be resized.
    - `u_int sx`: The new width of the window in character cells.
    - `u_int sy`: The new height of the window in character cells.
    - `int xpixel`: The new width of the window in pixels, or -1 to keep the current value.
    - `int ypixel`: The new height of the window in pixels, or -1 to keep the current value.
- **Control Flow**:
    - If `xpixel` is 0, it is set to `DEFAULT_XPIXEL`.
    - If `ypixel` is 0, it is set to `DEFAULT_YPIXEL`.
    - A debug log is generated to record the resizing action, including the new dimensions.
    - The width (`sx`) and height (`sy`) of the window are updated.
    - If `xpixel` is not -1, the pixel width of the window is updated.
    - If `ypixel` is not -1, the pixel height of the window is updated.
- **Output**:
    - The function does not return a value; it modifies the properties of the specified window directly.


---
### window_pane_send_resize<!-- {{#callable:window_pane_send_resize}} -->
The `window_pane_send_resize` function sends a resize request to a window pane by updating its terminal size.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane to be resized.
    - `sx`: An unsigned integer representing the new width (columns) of the window pane.
    - `sy`: An unsigned integer representing the new height (rows) of the window pane.
- **Control Flow**:
    - Check if the file descriptor (`fd`) of the window pane is valid (not -1); if invalid, exit the function.
    - Log the resize action with the pane ID and new dimensions.
    - Initialize a `winsize` structure to hold the new size information.
    - Set the `ws_col` and `ws_row` fields of the `winsize` structure to the new dimensions.
    - Calculate the pixel dimensions based on the window's pixel size and the new dimensions.
    - Use the `ioctl` system call to send the new size to the terminal associated with the pane.
    - If the `ioctl` call fails, check for specific errors on Solaris and log a fatal error if necessary.
- **Output**:
    - The function does not return a value; it modifies the terminal size of the specified window pane and logs any errors encountered during the process.


---
### window_has_pane<!-- {{#callable:window_has_pane}} -->
The `window_has_pane` function checks if a specific pane is part of a given window.
- **Inputs**:
    - `w`: A pointer to a `struct window` representing the window to check.
    - `wp`: A pointer to a `struct window_pane` representing the pane to search for within the window.
- **Control Flow**:
    - The function iterates over each pane in the linked list of panes associated with the given window `w`.
    - For each pane, it checks if the current pane (`wp1`) is the same as the pane being searched for (`wp`).
    - If a match is found, the function returns 1, indicating that the pane exists in the window.
    - If the iteration completes without finding a match, the function returns 0.
- **Output**:
    - The function returns 1 if the specified pane is found in the window, and 0 if it is not.


---
### window_update_focus<!-- {{#callable:window_update_focus}} -->
Updates the focus of a window by logging its ID and updating the active pane's focus.
- **Inputs**:
    - `w`: A pointer to a `struct window` that represents the window whose focus is to be updated.
- **Control Flow**:
    - Checks if the input window pointer `w` is not NULL.
    - Logs the function name and the ID of the window.
    - Calls [`window_pane_update_focus`](#window_pane_update_focus) with the active pane of the window.
- **Output**:
    - This function does not return a value; it performs actions to update the focus of the window's active pane.
- **Functions called**:
    - [`window_pane_update_focus`](#window_pane_update_focus)


---
### window_pane_update_focus<!-- {{#callable:window_pane_update_focus}} -->
Updates the focus state of a given window pane based on client activity.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane whose focus state is to be updated.
- **Control Flow**:
    - Checks if the `window_pane` pointer `wp` is not NULL and that the pane has not exited.
    - Determines if the pane is the active pane of its window.
    - Iterates through the list of clients to check if any client is focused and attached to the session of the window containing the pane.
    - If no client is focused and the pane is currently marked as focused, it logs a focus out event, sends a focus out notification, and updates the pane's flags.
    - If a client is focused and the pane is not marked as focused, it logs a focus in event, sends a focus in notification, and updates the pane's flags.
    - If the focus state remains unchanged, it logs that the focus state is unchanged.
- **Output**:
    - The function does not return a value; it modifies the state of the `window_pane` and logs focus changes.


---
### window_set_active_pane<!-- {{#callable:window_set_active_pane}} -->
Sets the active pane of a window and updates related states.
- **Inputs**:
    - `w`: A pointer to the `window` structure representing the window containing the panes.
    - `wp`: A pointer to the `window_pane` structure that is to be set as the active pane.
    - `notify`: An integer flag indicating whether to send a notification about the pane change.
- **Control Flow**:
    - Logs the function call with the ID of the pane being set as active.
    - Checks if the specified pane is already the active pane; if so, returns 0.
    - Stores the current active pane in `lastwp`.
    - Removes the specified pane from the last panes stack and pushes the current active pane onto it.
    - Updates the active pane to the specified pane and increments the active point.
    - Sets the `PANE_CHANGED` flag for the new active pane.
    - If focus events are enabled, updates the focus state for both the last and current active panes.
    - Updates the window's TTY offset.
    - If `notify` is true, sends a notification about the pane change.
    - Returns 1 to indicate a successful change.
- **Output**:
    - Returns 0 if the specified pane is already active; otherwise, returns 1 indicating the active pane was successfully changed.
- **Functions called**:
    - [`window_pane_stack_remove`](#window_pane_stack_remove)
    - [`window_pane_stack_push`](#window_pane_stack_push)
    - [`window_pane_update_focus`](#window_pane_update_focus)


---
### window_pane_get_palette<!-- {{#callable:window_pane_get_palette}} -->
Retrieves a color from the palette of a specified window pane.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure from which the color palette is accessed.
    - `c`: An integer representing the color index to retrieve from the palette.
- **Control Flow**:
    - Checks if the `wp` pointer is NULL; if it is, returns -1 to indicate an error.
    - If `wp` is valid, calls the `colour_palette_get` function with the palette of the window pane and the color index `c`.
- **Output**:
    - Returns the color value from the palette corresponding to the index `c`, or -1 if the window pane is NULL.


---
### window_redraw_active_switch<!-- {{#callable:window_redraw_active_switch}} -->
The `window_redraw_active_switch` function updates the redraw flags of window panes based on their active and inactive styles.
- **Inputs**:
    - `w`: A pointer to the `window` structure representing the current window.
    - `wp`: A pointer to the `window_pane` structure that needs to be checked for redraw.
- **Control Flow**:
    - The function first checks if the provided `window_pane` (`wp`) is the active pane of the `window` (`w`); if so, it returns immediately without making changes.
    - It enters an infinite loop where it compares the cached graphics cells of the `window_pane` (`wp`) with its active graphics cell.
    - If the graphics cells are not equal, it sets the `PANE_REDRAW` flag for the `window_pane`.
    - If the graphics cells are equal, it checks the foreground and background colors of the graphics cells against the palette; if they differ, it sets the `PANE_REDRAW` flag.
    - The loop continues until `wp` becomes the active pane of the `window`, at which point it breaks out of the loop.
- **Output**:
    - The function does not return a value; instead, it modifies the `flags` of the `window_pane` to indicate whether a redraw is necessary.
- **Functions called**:
    - [`window_pane_get_palette`](#window_pane_get_palette)


---
### window_get_active_at<!-- {{#callable:window_get_active_at}} -->
The `window_get_active_at` function retrieves the active `window_pane` at specified coordinates within a given `window`.
- **Inputs**:
    - `w`: A pointer to the `window` structure that contains multiple panes.
    - `x`: An unsigned integer representing the x-coordinate to check for an active pane.
    - `y`: An unsigned integer representing the y-coordinate to check for an active pane.
- **Control Flow**:
    - Iterates through each `window_pane` in the `window` using `TAILQ_FOREACH`.
    - Checks if the current `window_pane` is visible using [`window_pane_visible`](#window_pane_visible).
    - Calculates the full size and offset of the `window_pane` using [`window_pane_full_size_offset`](#window_pane_full_size_offset).
    - Compares the provided coordinates (x, y) against the calculated offsets and sizes to determine if the coordinates fall within the bounds of the `window_pane`.
    - If a matching `window_pane` is found, it is returned; otherwise, the function continues to the next pane.
    - If no active pane is found after checking all panes, the function returns NULL.
- **Output**:
    - Returns a pointer to the active `window_pane` that contains the specified coordinates, or NULL if no such pane exists.
- **Functions called**:
    - [`window_pane_visible`](#window_pane_visible)
    - [`window_pane_full_size_offset`](#window_pane_full_size_offset)


---
### window_find_string<!-- {{#callable:window_find_string}} -->
The `window_find_string` function locates a window pane based on a specified string that represents a position.
- **Inputs**:
    - `w`: A pointer to the `window` structure representing the window in which to search for the pane.
    - `s`: A string that specifies the position to find, which can be 'top', 'bottom', 'left', 'right', 'top-left', 'top-right', 'bottom-left', or 'bottom-right'.
- **Control Flow**:
    - The function initializes the coordinates `x` and `y` to the center of the window.
    - It retrieves the pane border status from the window options to adjust the `top` and `bottom` coordinates accordingly.
    - The function then compares the input string `s` against predefined position strings using `strcasecmp` to determine the appropriate `x` and `y` coordinates.
    - If the input string does not match any known positions, the function returns NULL.
    - Finally, it calls [`window_get_active_at`](#window_get_active_at) with the calculated `x` and `y` coordinates to retrieve the corresponding pane.
- **Output**:
    - The function returns a pointer to the `window_pane` structure that corresponds to the specified position, or NULL if no matching pane is found.
- **Functions called**:
    - [`window_get_active_at`](#window_get_active_at)


---
### window_zoom<!-- {{#callable:window_zoom}} -->
The `window_zoom` function zooms a specified window pane, adjusting its layout and saving the previous layout.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure that represents the pane to be zoomed.
- **Control Flow**:
    - Checks if the window is already zoomed; if so, returns -1.
    - Checks if the window has only one pane; if so, returns -1.
    - If the specified pane is not the active pane, it sets it as the active pane.
    - Iterates through all panes in the window, saving their current layout and setting their layout to NULL.
    - Saves the current layout root of the window.
    - Initializes the layout for the zoomed pane.
    - Sets the zoomed flag for the window and notifies about the layout change.
- **Output**:
    - Returns 0 on success, indicating the pane has been successfully zoomed, or -1 if the zoom operation could not be performed.
- **Functions called**:
    - [`window_count_panes`](#window_count_panes)
    - [`window_set_active_pane`](#window_set_active_pane)


---
### window_unzoom<!-- {{#callable:window_unzoom}} -->
The `window_unzoom` function restores a window's layout from a zoomed state.
- **Inputs**:
    - `w`: A pointer to the `window` structure representing the window to be unzoomed.
    - `notify`: An integer flag indicating whether to notify clients of the layout change.
- **Control Flow**:
    - Checks if the window is currently zoomed by evaluating the `WINDOW_ZOOMED` flag.
    - If the window is not zoomed, it returns -1 indicating failure.
    - Clears the `WINDOW_ZOOMED` flag and frees the current layout.
    - Restores the layout from `saved_layout_root` and sets `saved_layout_root` to NULL.
    - Iterates through each `window_pane` in the window's panes list, restoring their layout cells from saved states.
    - Calls `layout_fix_panes` to adjust the layout of the panes.
    - If `notify` is true, sends a notification about the layout change.
- **Output**:
    - Returns 0 on success, or -1 if the window was not zoomed.


---
### window_push_zoom<!-- {{#callable:window_push_zoom}} -->
The `window_push_zoom` function manages the zoom state of a window by unzooming it and potentially marking it as previously zoomed.
- **Inputs**:
    - `w`: A pointer to the `window` structure that represents the window to be manipulated.
    - `always`: An integer flag indicating whether to always mark the window as zoomed.
    - `flag`: An integer flag that indicates whether the window is currently zoomed.
- **Control Flow**:
    - Logs the current function call with the window ID and zoom status.
    - Checks if the `flag` is set and if either `always` is true or the window is already zoomed.
    - If the conditions are met, it sets the `WINDOW_WASZOOMED` flag for the window.
    - If not, it clears the `WINDOW_WASZOOMED` flag.
    - Calls the [`window_unzoom`](#window_unzoom) function to unzoom the window and returns the result.
- **Output**:
    - Returns 1 if the window was successfully unzoomed, otherwise returns 0.
- **Functions called**:
    - [`window_unzoom`](#window_unzoom)


---
### window_pop_zoom<!-- {{#callable:window_pop_zoom}} -->
The `window_pop_zoom` function restores the zoom state of a window if it was previously zoomed.
- **Inputs**:
    - `struct window *w`: A pointer to the `window` structure that represents the window whose zoom state is to be restored.
- **Control Flow**:
    - Logs the current function call along with the window ID and whether it was previously zoomed.
    - Checks if the `WINDOW_WASZOOMED` flag is set in the window's flags.
    - If the flag is set, it calls the [`window_zoom`](#window_zoom) function on the active pane of the window.
    - Returns the result of the [`window_zoom`](#window_zoom) function, which indicates success or failure.
    - If the flag is not set, it returns 0, indicating no action was taken.
- **Output**:
    - Returns 0 if the window was not previously zoomed or the result of the [`window_zoom`](#window_zoom) function if it was.
- **Functions called**:
    - [`window_zoom`](#window_zoom)


---
### window_add_pane<!-- {{#callable:window_add_pane}} -->
The `window_add_pane` function adds a new pane to a specified window, positioning it relative to an existing pane based on provided flags.
- **Inputs**:
    - `w`: A pointer to the `window` structure where the new pane will be added.
    - `other`: A pointer to another `window_pane` structure that serves as a reference for positioning the new pane; if NULL, the active pane is used.
    - `hlimit`: An unsigned integer representing the height limit for the new pane.
    - `flags`: An integer containing flags that dictate the positioning behavior of the new pane (e.g., whether to insert before or after the reference pane).
- **Control Flow**:
    - If `other` is NULL, it is set to the currently active pane in the window.
    - A new pane is created using [`window_pane_create`](#window_pane_create) with the specified dimensions and height limit.
    - If the pane list in the window is empty, the new pane is inserted at the head of the list.
    - If the `SPAWN_BEFORE` flag is set, the new pane is inserted before the `other` pane or at the head if `SPAWN_FULLSIZE` is also set.
    - If the `SPAWN_BEFORE` flag is not set, the new pane is inserted after the `other` pane or at the tail if `SPAWN_FULLSIZE` is set.
- **Output**:
    - Returns a pointer to the newly created `window_pane`.
- **Functions called**:
    - [`window_pane_create`](#window_pane_create)


---
### window_lost_pane<!-- {{#callable:window_lost_pane}} -->
The `window_lost_pane` function handles the removal of a pane from a window, updating the active pane and notifying relevant components.
- **Inputs**:
    - `struct window *w`: A pointer to the `window` structure from which the pane is being removed.
    - `struct window_pane *wp`: A pointer to the `window_pane` structure that is being lost.
- **Control Flow**:
    - Logs the action of losing a pane with its identifiers.
    - Checks if the pane being lost is marked and clears the mark if it is.
    - Removes the pane from the stack of last panes in the window.
    - If the lost pane is the active pane, it attempts to set a new active pane from the last panes.
    - If no last panes are available, it tries to set the active pane to the previous or next pane in the list.
    - If a new active pane is set, it updates its flags, notifies the window of the change, and updates the window's focus.
- **Output**:
    - The function does not return a value but modifies the state of the window and its panes, specifically updating the active pane and notifying any listeners of the change.
- **Functions called**:
    - [`window_pane_stack_remove`](#window_pane_stack_remove)
    - [`window_update_focus`](#window_update_focus)


---
### window_remove_pane<!-- {{#callable:window_remove_pane}} -->
The `window_remove_pane` function removes a specified pane from a window and cleans up its resources.
- **Inputs**:
    - `w`: A pointer to the `window` structure from which the pane is to be removed.
    - `wp`: A pointer to the `window_pane` structure that is to be removed from the window.
- **Control Flow**:
    - Calls [`window_lost_pane`](#window_lost_pane) to handle any necessary updates related to the pane being removed.
    - Uses `TAILQ_REMOVE` to remove the specified pane from the list of panes in the window.
    - Calls [`window_pane_destroy`](#window_pane_destroy) to free the resources associated with the pane.
- **Output**:
    - This function does not return a value; it performs operations that modify the state of the window and the pane.
- **Functions called**:
    - [`window_lost_pane`](#window_lost_pane)
    - [`window_pane_destroy`](#window_pane_destroy)


---
### window_pane_at_index<!-- {{#callable:window_pane_at_index}} -->
The `window_pane_at_index` function retrieves a pointer to a `window_pane` structure at a specified index.
- **Inputs**:
    - `w`: A pointer to a `window` structure that contains the panes.
    - `idx`: An unsigned integer representing the index of the desired pane.
- **Control Flow**:
    - The function first retrieves the base index for panes from the window's options using `options_get_number`.
    - It then iterates through the list of panes using `TAILQ_FOREACH`, comparing the current index with the provided index.
    - If a match is found, it returns the corresponding `window_pane` pointer.
    - If no match is found after iterating through all panes, it returns NULL.
- **Output**:
    - The function returns a pointer to the `window_pane` at the specified index if found, or NULL if no such pane exists.


---
### window_pane_next_by_number<!-- {{#callable:window_pane_next_by_number}} -->
The `window_pane_next_by_number` function retrieves the next `window_pane` in a circular manner based on a specified number of steps.
- **Inputs**:
    - `w`: A pointer to the `window` structure that contains the list of panes.
    - `wp`: A pointer to the current `window_pane` from which the next pane is to be found.
    - `n`: An unsigned integer representing the number of steps to move forward in the list of panes.
- **Control Flow**:
    - The function enters a loop that iterates `n` times.
    - In each iteration, it attempts to get the next pane using `TAILQ_NEXT`.
    - If `TAILQ_NEXT` returns NULL, indicating the end of the list, it wraps around to the first pane using `TAILQ_FIRST`.
- **Output**:
    - The function returns a pointer to the next `window_pane` after moving `n` steps from the current pane, wrapping around to the start if necessary.


---
### window_pane_previous_by_number<!-- {{#callable:window_pane_previous_by_number}} -->
The `window_pane_previous_by_number` function retrieves the previous `window_pane` in a linked list based on a specified number of steps.
- **Inputs**:
    - `w`: A pointer to the `window` structure that contains the list of panes.
    - `wp`: A pointer to the current `window_pane` from which to start the search.
    - `n`: An unsigned integer representing the number of steps to move backwards in the list of panes.
- **Control Flow**:
    - The function enters a loop that decrements `n` until it reaches zero.
    - Within the loop, it attempts to retrieve the previous `window_pane` using `TAILQ_PREV`.
    - If `TAILQ_PREV` returns NULL, indicating that there are no more previous panes, it resets `wp` to the last pane in the list using `TAILQ_LAST`.
- **Output**:
    - The function returns a pointer to the `window_pane` that is `n` steps before the given `wp`, or the last pane if the end of the list is reached.


---
### window_pane_index<!-- {{#callable:window_pane_index}} -->
The `window_pane_index` function retrieves the index of a specified `window_pane` within its parent `window`.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` whose index is to be determined.
    - `i`: A pointer to an unsigned integer where the base index will be stored.
- **Control Flow**:
    - The function first retrieves the base index for panes from the window's options using `options_get_number`.
    - It then iterates through the list of panes in the window using `TAILQ_FOREACH`.
    - During the iteration, if the current pane matches the specified `window_pane`, the function returns 0.
    - If the specified pane is not found, the function returns -1.
- **Output**:
    - The function outputs 0 if the specified `window_pane` is found and its index is updated in the provided pointer; otherwise, it returns -1 if the pane is not found.


---
### window_count_panes<!-- {{#callable:window_count_panes}} -->
Counts the number of panes in a given window.
- **Inputs**:
    - `w`: A pointer to a `window` structure whose panes are to be counted.
- **Control Flow**:
    - Initializes a counter `n` to zero.
    - Iterates over each `window_pane` in the `panes` list of the `window` using `TAILQ_FOREACH`.
    - Increments the counter `n` for each pane encountered.
    - Returns the final count of panes.
- **Output**:
    - Returns an unsigned integer representing the total number of panes in the specified window.


---
### window_destroy_panes<!-- {{#callable:window_destroy_panes}} -->
The `window_destroy_panes` function cleans up and destroys all panes associated with a given window.
- **Inputs**:
    - `w`: A pointer to a `struct window` that contains the panes to be destroyed.
- **Control Flow**:
    - The function first enters a loop that continues until the `last_panes` queue is empty, removing each pane from this queue.
    - Next, it enters another loop that continues until the `panes` queue is empty, removing each pane from this queue and calling [`window_pane_destroy`](#window_pane_destroy) on it.
- **Output**:
    - The function does not return a value; it performs cleanup by destroying the panes associated with the window.
- **Functions called**:
    - [`window_pane_stack_remove`](#window_pane_stack_remove)
    - [`window_pane_destroy`](#window_pane_destroy)


---
### window_printable_flags<!-- {{#callable:window_printable_flags}} -->
The `window_printable_flags` function generates a string of printable flags representing the state of a given window link.
- **Inputs**:
    - `wl`: A pointer to a `struct winlink` representing the window link whose flags are to be printed.
    - `escape`: An integer indicating whether to escape the activity flag with an additional '#' character.
- **Control Flow**:
    - Initialize a static character array `flags` to hold the printable flags.
    - Check if the `WINLINK_ACTIVITY` flag is set; if so, add '#' to `flags`, and if `escape` is true, add another '#'.
    - Check if the `WINLINK_BELL` flag is set; if so, add '!' to `flags`.
    - Check if the `WINLINK_SILENCE` flag is set; if so, add '~' to `flags`.
    - Check if the current window link is the active window in the session; if so, add '*' to `flags`.
    - Check if the current window link is the first in the last window list; if so, add '-' to `flags`.
    - Check if the window link is marked; if so, add 'M' to `flags`.
    - Check if the window is zoomed; if so, add 'Z' to `flags`.
    - Terminate the `flags` string with a null character.
- **Output**:
    - Returns a pointer to the `flags` string containing the printable representation of the window link's state.


---
### window_pane_find_by_id_str<!-- {{#callable:window_pane_find_by_id_str}} -->
Finds a `window_pane` by its string identifier.
- **Inputs**:
    - `s`: A string representing the identifier of the window pane, which must start with '%'.
- **Control Flow**:
    - Checks if the first character of the input string `s` is '%'. If not, returns NULL.
    - Uses `strtonum` to convert the substring starting from the second character to an unsigned integer, checking for errors.
    - If conversion is successful, calls [`window_pane_find_by_id`](#window_pane_find_by_id) with the parsed ID to retrieve the corresponding `window_pane`.
- **Output**:
    - Returns a pointer to the `window_pane` associated with the given ID, or NULL if the input string is invalid or the ID does not correspond to any pane.
- **Functions called**:
    - [`window_pane_find_by_id`](#window_pane_find_by_id)


---
### window_pane_find_by_id<!-- {{#callable:window_pane_find_by_id}} -->
Finds a `window_pane` by its unique identifier.
- **Inputs**:
    - `id`: An unsigned integer representing the unique identifier of the `window_pane` to be found.
- **Control Flow**:
    - Creates a temporary `window_pane` structure `wp` and assigns the input `id` to its `id` field.
    - Uses the `RB_FIND` macro to search for the `window_pane` in the `window_pane_tree` using the temporary `wp` structure.
- **Output**:
    - Returns a pointer to the `window_pane` if found; otherwise, returns NULL.


---
### window_pane_create<!-- {{#callable:window_pane_create}} -->
Creates and initializes a new `window_pane` structure.
- **Inputs**:
    - `w`: A pointer to the `window` structure that this pane will belong to.
    - `sx`: An unsigned integer representing the width of the pane.
    - `sy`: An unsigned integer representing the height of the pane.
    - `hlimit`: An unsigned integer representing the height limit for the pane.
- **Control Flow**:
    - Allocates memory for a new `window_pane` using `xcalloc`.
    - Initializes the `window` pointer of the pane to the provided window.
    - Creates a new options structure for the pane based on the window's options.
    - Sets initial flags for the pane indicating style and theme changes.
    - Increments the global pane ID and inserts the new pane into a red-black tree of all panes.
    - Initializes various properties of the pane, including file descriptors and dimensions.
    - Sets the scrollbar style and initializes the color palette based on the pane's options.
    - Initializes the screen for the pane and sets the default cursor.
    - If successful, retrieves the hostname and sets it as the title of the pane's screen.
- **Output**:
    - Returns a pointer to the newly created `window_pane` structure.
- **Functions called**:
    - [`window_pane_default_cursor`](#window_pane_default_cursor)


---
### window_pane_destroy<!-- {{#callable:window_pane_destroy}} -->
The `window_pane_destroy` function deallocates all resources associated with a `window_pane` structure.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure that is to be destroyed.
- **Control Flow**:
    - Calls [`window_pane_reset_mode_all`](#window_pane_reset_mode_all) to reset any modes associated with the pane.
    - Frees the `searchstr` string associated with the pane.
    - Checks if the file descriptor `fd` is valid and if so, removes the pane from the utempter, frees the event buffer, and closes the file descriptor.
    - If the input context `ictx` is not NULL, it frees the input context.
    - Frees the status and base screens associated with the pane.
    - Checks if the `pipe_fd` is valid, frees the associated pipe event, and closes the pipe file descriptor.
    - If the resize timer event is initialized, it deletes the event.
    - Iterates through the resize queue and frees each resize structure.
    - Removes the pane from the global tree of all window panes.
    - Frees the options, current working directory, shell, command arguments, and color palette associated with the pane.
    - Finally, frees the `window_pane` structure itself.
- **Output**:
    - The function does not return a value; it performs cleanup and deallocation of resources associated with the specified `window_pane`.
- **Functions called**:
    - [`window_pane_reset_mode_all`](#window_pane_reset_mode_all)


---
### window_pane_read_callback<!-- {{#callable:window_pane_read_callback}} -->
The `window_pane_read_callback` function processes input data from a window pane's event buffer and manages communication with connected clients.
- **Inputs**:
    - `bufev`: A pointer to a `bufferevent` structure, which is unused in this function.
    - `data`: A pointer to a `window_pane` structure that contains the state and data of the window pane.
- **Control Flow**:
    - Checks if the `pipe_fd` of the window pane is valid (not -1).
    - Calls [`window_pane_get_new_data`](#window_pane_get_new_data) to retrieve new data from the event buffer.
    - If new data is available, it writes this data to the `pipe_event` and updates the used data offset.
    - Logs the current size of the input buffer for debugging purposes.
    - Iterates through all connected clients and writes output to those that are in control mode.
    - Calls `input_parse_pane` to process the input data for the window pane.
    - Disables further read events on the window pane's event.
- **Output**:
    - The function does not return a value; it performs actions such as writing data to a pipe and updating the state of the window pane.
- **Functions called**:
    - [`window_pane_get_new_data`](#window_pane_get_new_data)
    - [`window_pane_update_used_data`](#window_pane_update_used_data)


---
### window_pane_error_callback<!-- {{#callable:window_pane_error_callback}} -->
The `window_pane_error_callback` function handles error events for a window pane.
- **Inputs**:
    - `bufev`: A pointer to a `bufferevent` structure, which is unused in this function.
    - `what`: A short integer representing the type of event that triggered the callback, which is also unused.
    - `data`: A pointer to user-defined data, which is expected to be a pointer to a `window_pane` structure.
- **Control Flow**:
    - The function retrieves the `window_pane` structure from the `data` pointer.
    - It logs a debug message indicating an error has occurred for the specific pane identified by its ID.
    - The function sets the `PANE_EXITED` flag in the `flags` member of the `window_pane` structure.
    - It checks if the pane is ready to be destroyed by calling [`window_pane_destroy_ready`](#window_pane_destroy_ready).
    - If the pane is ready to be destroyed, it calls `server_destroy_pane` to handle the destruction.
- **Output**:
    - The function does not return a value; it modifies the state of the `window_pane` and may trigger its destruction.
- **Functions called**:
    - [`window_pane_destroy_ready`](#window_pane_destroy_ready)


---
### window_pane_set_event<!-- {{#callable:window_pane_set_event}} -->
Sets up event handling for a window pane.
- **Inputs**:
    - `wp`: A pointer to a `window_pane` structure that represents the pane to set the event for.
- **Control Flow**:
    - Calls `setblocking` to set the file descriptor of the pane to non-blocking mode.
    - Creates a new buffered event using `bufferevent_new`, passing the file descriptor, a read callback, an error callback, and the pane structure.
    - Checks if the event creation was successful; if not, it calls `fatalx` to handle the out-of-memory error.
    - Initializes the input context for the pane using `input_init`.
    - Enables reading and writing on the buffered event using `bufferevent_enable`.
- **Output**:
    - The function does not return a value; it sets up the event handling for the specified window pane.


---
### window_pane_resize<!-- {{#callable:window_pane_resize}} -->
Resizes a window pane and updates its associated properties.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure that represents the pane to be resized.
    - `sx`: An unsigned integer representing the new width of the pane.
    - `sy`: An unsigned integer representing the new height of the pane.
- **Control Flow**:
    - Checks if the new size is the same as the current size; if so, the function returns early without making changes.
    - Allocates memory for a `window_pane_resize` structure to store the new size and the old size.
    - Inserts the new size information into the pane's resize queue.
    - Updates the pane's current size to the new dimensions.
    - Logs the resize action for debugging purposes.
    - Calls `screen_resize` to adjust the screen dimensions of the pane.
    - If there are any modes associated with the pane, it calls the resize function of the first mode, if it exists.
- **Output**:
    - The function does not return a value; it modifies the state of the `window_pane` and its associated structures.


---
### window_pane_set_mode<!-- {{#callable:window_pane_set_mode}} -->
Sets the mode for a specified window pane.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane to set the mode for.
    - `swp`: A pointer to another `window_pane` structure, possibly representing a source pane.
    - `mode`: A pointer to a `window_mode` structure that defines the mode to be set.
    - `fs`: A pointer to a `cmd_find_state` structure that holds the state of the command.
    - `args`: A pointer to an `args` structure containing additional arguments for the mode.
- **Control Flow**:
    - Checks if the current mode of the pane is already the desired mode; if so, returns 1.
    - Iterates through the list of modes in the pane to check if the desired mode is already present.
    - If the mode is found, it is moved to the head of the list; if not, a new mode entry is created and initialized.
    - The screen for the pane is updated to reflect the new mode.
    - Flags are set to indicate that the pane needs to be redrawn and its scrollbar updated.
    - The layout of the window is adjusted to accommodate the new mode.
    - The window borders and status are redrawn, and a notification is sent indicating the mode change.
- **Output**:
    - Returns 0 on success, indicating that the mode was successfully set, or 1 if the mode was already set.


---
### window_pane_reset_mode<!-- {{#callable:window_pane_reset_mode}} -->
Resets the current mode of a window pane and updates its state.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure that represents the pane whose mode is to be reset.
- **Control Flow**:
    - Checks if the `modes` list of the `window_pane` is empty; if so, it returns immediately.
    - Retrieves the first mode entry from the `modes` list and removes it.
    - Calls the `free` function of the mode to clean up resources associated with the mode.
    - If there are no more modes left after removal, it clears the `PANE_UNSEENCHANGES` flag, logs a debug message, and sets the screen to the base screen.
    - If there is a next mode, it updates the pane's screen to the next mode's screen and calls its resize function if it exists.
    - Sets flags to indicate that the pane needs to be redrawn and updates the layout of the window.
    - Redraws the window borders and updates the status of the window.
    - Notifies that the pane's mode has changed.
- **Output**:
    - The function does not return a value; it modifies the state of the `window_pane` and performs side effects such as logging and notifying other components.


---
### window_pane_reset_mode_all<!-- {{#callable:window_pane_reset_mode_all}} -->
Resets all modes of a given window pane.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure whose modes are to be reset.
- **Control Flow**:
    - The function enters a while loop that continues as long as the `modes` list of the `window_pane` is not empty.
    - Within the loop, it calls the [`window_pane_reset_mode`](#window_pane_reset_mode) function to reset the first mode in the list.
- **Output**:
    - The function does not return a value; it modifies the state of the `window_pane` by clearing all its modes.
- **Functions called**:
    - [`window_pane_reset_mode`](#window_pane_reset_mode)


---
### window_pane_copy_paste<!-- {{#callable:window_pane_copy_paste}} -->
The `window_pane_copy_paste` function synchronizes the paste operation across multiple visible panes in a window.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the current pane from which the paste operation is initiated.
    - `buf`: A pointer to a character buffer containing the data to be pasted.
    - `len`: The size of the data in the buffer to be pasted.
- **Control Flow**:
    - Iterates over each pane in the current window using `TAILQ_FOREACH`.
    - Checks if the current pane in the loop is not the same as the source pane (`wp`).
    - Ensures the pane is empty of modes, has a valid file descriptor, is not input-off, is visible, and has the 'synchronize-panes' option enabled.
    - If all conditions are met, logs the paste operation and writes the buffer data to the event associated with the pane.
- **Output**:
    - The function does not return a value; it performs actions that result in the specified data being pasted into other panes.
- **Functions called**:
    - [`window_pane_visible`](#window_pane_visible)


---
### window_pane_copy_key<!-- {{#callable:window_pane_copy_key}} -->
The `window_pane_copy_key` function synchronizes key input across all visible panes in a window, except the one that originated the input.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane that is sending the key input.
    - `key`: The key code representing the key input to be synchronized across other panes.
- **Control Flow**:
    - Iterates through each pane in the window using `TAILQ_FOREACH`.
    - Checks if the current pane is not the source pane (`wp`), is not in any mode, has a valid file descriptor, is not input-off, is visible, and has synchronization enabled.
    - If all conditions are met, it calls `input_key_pane` to send the key input to the current pane.
- **Output**:
    - The function does not return a value; it sends the key input to other panes that meet the specified conditions.
- **Functions called**:
    - [`window_pane_visible`](#window_pane_visible)


---
### window_pane_paste<!-- {{#callable:window_pane_paste}} -->
The `window_pane_paste` function handles pasting text into a window pane if certain conditions are met.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane where the paste operation is to occur.
    - `key`: A `key_code` representing the key that triggered the paste operation.
    - `buf`: A pointer to a character buffer containing the text to be pasted.
    - `len`: A size_t value representing the length of the text in the buffer.
- **Control Flow**:
    - The function first checks if the `window_pane` has any active modes; if so, it returns immediately without pasting.
    - It then checks if the pane's file descriptor is invalid or if the pane is in an input-off state, returning if either condition is true.
    - Next, it checks if the key pressed is a paste key and if the screen mode allows pasting; if not, it returns.
    - If all checks pass, it logs the paste operation and writes the buffer content to the pane's event.
    - Finally, if the 'synchronize-panes' option is enabled, it calls [`window_pane_copy_paste`](#window_pane_copy_paste) to paste the content into other panes.
- **Output**:
    - The function does not return a value; it performs actions based on the input conditions and modifies the state of the window pane.
- **Functions called**:
    - [`window_pane_copy_paste`](#window_pane_copy_paste)


---
### window_pane_key<!-- {{#callable:window_pane_key}} -->
Processes key events for a specific window pane.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane receiving the key event.
    - `c`: A pointer to the `client` structure representing the client that initiated the key event.
    - `s`: A pointer to the `session` structure associated with the client.
    - `wl`: A pointer to the `winlink` structure representing the window link.
    - `key`: The key code representing the key event.
    - `m`: A pointer to the `mouse_event` structure if the event is a mouse event, otherwise NULL.
- **Control Flow**:
    - Checks if the key event is a mouse event and if the mouse event structure is NULL, returning -1 if true.
    - Retrieves the first mode entry from the pane's modes list.
    - If a mode is active and it has a key handler, it calls the key handler with the relevant parameters.
    - If the pane's file descriptor is -1 or the input is turned off, it returns 0.
    - Processes the key input through the `input_key_pane` function, returning -1 if it fails.
    - If the key is a mouse event, it returns 0.
    - If the 'synchronize-panes' option is enabled, it copies the key event to other panes.
- **Output**:
    - Returns 0 on success, -1 if an error occurs, or if the key event is not processed.
- **Functions called**:
    - [`window_pane_copy_key`](#window_pane_copy_key)


---
### window_pane_visible<!-- {{#callable:window_pane_visible}} -->
The `window_pane_visible` function checks if a given window pane is visible based on its zoom state and whether it is the active pane.
- **Inputs**:
    - `wp`: A pointer to a `struct window_pane` representing the pane to check for visibility.
- **Control Flow**:
    - The function first checks if the `WINDOW_ZOOMED` flag is not set for the window associated with the pane.
    - If the window is not zoomed, it returns 1, indicating the pane is visible.
    - If the window is zoomed, it checks if the pane is the active pane of the window and returns the result of that comparison.
- **Output**:
    - Returns 1 if the pane is visible, otherwise returns 0.


---
### window_pane_exited<!-- {{#callable:window_pane_exited}} -->
The `window_pane_exited` function checks if a window pane has exited.
- **Inputs**:
    - `wp`: A pointer to a `struct window_pane` representing the pane to check.
- **Control Flow**:
    - The function evaluates the condition of the pane's file descriptor (`fd`) and its exit flag (`flags`).
    - It returns true if the file descriptor is -1 or if the exit flag indicates that the pane has exited.
- **Output**:
    - Returns an integer value: 1 if the pane has exited, otherwise 0.


---
### window_pane_search<!-- {{#callable:window_pane_search}} -->
The `window_pane_search` function searches for a specified term in the text of a window pane, optionally using regular expressions and case sensitivity.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane in which to search.
    - `term`: A string containing the term to search for within the pane.
    - `regex`: An integer flag indicating whether to treat the search term as a regular expression (1) or a simple string (0).
    - `ignore`: An integer flag indicating whether to ignore case when searching (1) or not (0).
- **Control Flow**:
    - The function first checks if the `regex` flag is set; if not, it prepares a wildcard search pattern by surrounding the term with asterisks and sets the case sensitivity flag if `ignore` is true.
    - If the `regex` flag is set, it compiles the regular expression using `regcomp`, applying the case sensitivity flag if `ignore` is true.
    - The function then iterates over each line of the pane's text, trimming trailing whitespace and logging the line for debugging.
    - For each line, it checks for a match using either `fnmatch` (for simple string searches) or `regexec` (for regex searches), freeing the line after checking.
    - If a match is found, the loop breaks, and the function prepares to return the line index; if no matches are found after checking all lines, it returns 0.
- **Output**:
    - The function returns the index of the first line where the term was found, or 0 if the term was not found in any line.


---
### window_pane_choose_best<!-- {{#callable:window_pane_choose_best}} -->
The `window_pane_choose_best` function selects the `window_pane` with the highest `active_point` from a given list.
- **Inputs**:
    - `list`: An array of pointers to `window_pane` structures.
    - `size`: An unsigned integer representing the number of panes in the list.
- **Control Flow**:
    - If the size is zero, the function returns NULL immediately.
    - The first pane in the list is initially considered the best.
    - A loop iterates through the remaining panes, comparing their `active_point` values to find the one with the highest value.
    - If a pane with a higher `active_point` is found, it becomes the new best pane.
    - After checking all panes, the function returns the best pane found.
- **Output**:
    - Returns a pointer to the `window_pane` with the highest `active_point`, or NULL if the list is empty.


---
### window_pane_full_size_offset<!-- {{#callable:window_pane_full_size_offset}} -->
Calculates the full size and offset of a window pane, accounting for the presence of scrollbars.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure whose size and offset are to be calculated.
    - `xoff`: A pointer to an unsigned integer where the calculated x-offset will be stored.
    - `yoff`: A pointer to an unsigned integer where the calculated y-offset will be stored.
    - `sx`: A pointer to an unsigned integer where the calculated width of the pane will be stored.
    - `sy`: A pointer to an unsigned integer where the calculated height of the pane will be stored.
- **Control Flow**:
    - Retrieve the number of scrollbars and their position from the pane's options.
    - Check if the scrollbar should be displayed using [`window_pane_show_scrollbar`](#window_pane_show_scrollbar).
    - Calculate the width of the scrollbar based on its style and padding.
    - Determine the x-offset and width of the pane based on the scrollbar's position (left or right).
    - Set the y-offset and height of the pane directly from the `window_pane` structure.
- **Output**:
    - The function does not return a value; instead, it modifies the values pointed to by `xoff`, `yoff`, `sx`, and `sy` to reflect the calculated dimensions and offsets of the pane.
- **Functions called**:
    - [`window_pane_show_scrollbar`](#window_pane_show_scrollbar)


---
### window_pane_find_up<!-- {{#callable:window_pane_find_up}} -->
Finds the pane directly above the specified pane.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the current pane.
- **Control Flow**:
    - Checks if the input pane `wp` is NULL and returns NULL if it is.
    - Retrieves the associated `window` structure from the input pane.
    - Gets the pane border status from the window options.
    - Calculates the full size and offset of the input pane.
    - Determines the edge position based on the pane border status.
    - Iterates through all panes in the window to find candidates for the pane above.
    - For each candidate pane, checks if it is adjacent to the top edge of the input pane.
    - If a candidate pane is found to be above, it is added to a list.
    - Chooses the best candidate pane from the list based on the active point.
    - Frees the list of candidate panes and returns the best pane found.
- **Output**:
    - Returns a pointer to the `window_pane` structure representing the pane directly above the input pane, or NULL if no such pane exists.
- **Functions called**:
    - [`window_pane_full_size_offset`](#window_pane_full_size_offset)
    - [`window_pane_choose_best`](#window_pane_choose_best)


---
### window_pane_find_down<!-- {{#callable:window_pane_find_down}} -->
Finds the pane directly below a given window pane.
- **Inputs**:
    - `wp`: A pointer to the current `window_pane` from which the search for the pane below will begin.
- **Control Flow**:
    - Checks if the input `wp` is NULL and returns NULL if it is.
    - Retrieves the associated `window` from the `window_pane` structure.
    - Gets the pane border status from the window options.
    - Calculates the edge position based on the current pane's offset and size.
    - Adjusts the edge position based on the pane border status.
    - Iterates through all panes in the window to find candidates for the pane below.
    - For each candidate pane, checks if it is adjacent to the calculated edge and within the horizontal bounds of the current pane.
    - If a candidate pane is found, it is added to a list.
    - After iterating through all panes, selects the best candidate from the list using [`window_pane_choose_best`](#window_pane_choose_best).
    - Frees the list of candidate panes and returns the best candidate.
- **Output**:
    - Returns a pointer to the best candidate `window_pane` found directly below the input pane, or NULL if no such pane exists.
- **Functions called**:
    - [`window_pane_full_size_offset`](#window_pane_full_size_offset)
    - [`window_pane_choose_best`](#window_pane_choose_best)


---
### window_pane_find_left<!-- {{#callable:window_pane_find_left}} -->
Finds the window pane directly to the left of a given pane.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure for which the left pane is to be found.
- **Control Flow**:
    - Checks if the input pane `wp` is NULL and returns NULL if it is.
    - Retrieves the associated `window` structure from the input pane.
    - Initializes variables for storing offsets and sizes of the panes.
    - Calls [`window_pane_full_size_offset`](#window_pane_full_size_offset) to get the full size and offset of the input pane.
    - Iterates through all panes in the window using `TAILQ_FOREACH`.
    - For each pane, checks if it is not the same as the input pane and if its right edge aligns with the left edge of the input pane.
    - Determines if the current pane overlaps vertically with the input pane.
    - If a valid adjacent pane is found, it is added to a list.
    - After iterating through all panes, calls [`window_pane_choose_best`](#window_pane_choose_best) to select the most suitable pane from the list.
    - Frees the list of candidate panes and returns the best pane found.
- **Output**:
    - Returns a pointer to the `window_pane` structure that is directly to the left of the input pane, or NULL if no such pane exists.
- **Functions called**:
    - [`window_pane_full_size_offset`](#window_pane_full_size_offset)
    - [`window_pane_choose_best`](#window_pane_choose_best)


---
### window_pane_find_right<!-- {{#callable:window_pane_find_right}} -->
The `window_pane_find_right` function locates the pane immediately to the right of a specified window pane.
- **Inputs**:
    - `wp`: A pointer to the current `window_pane` structure from which the right pane is to be found.
- **Control Flow**:
    - Check if the input pane `wp` is NULL; if so, return NULL.
    - Retrieve the associated `window` structure from the input pane.
    - Initialize variables for storing pane offsets and dimensions.
    - Calculate the right edge position based on the current pane's offset and size.
    - Iterate through all panes in the window using a loop.
    - For each pane, check if it is not the same as the input pane and if its left edge matches the calculated right edge.
    - Determine if the current pane overlaps vertically with the input pane.
    - If a valid adjacent pane is found, add it to a list.
    - After checking all panes, call [`window_pane_choose_best`](#window_pane_choose_best) to select the most suitable pane from the list.
    - Free the list of candidate panes and return the best pane found.
- **Output**:
    - Returns a pointer to the `window_pane` structure that is immediately to the right of the specified pane, or NULL if no such pane exists.
- **Functions called**:
    - [`window_pane_full_size_offset`](#window_pane_full_size_offset)
    - [`window_pane_choose_best`](#window_pane_choose_best)


---
### window_pane_stack_push<!-- {{#callable:window_pane_stack_push}} -->
The `window_pane_stack_push` function adds a `window_pane` to the head of a `window_panes` stack after removing it if it already exists.
- **Inputs**:
    - `stack`: A pointer to a `window_panes` structure representing the stack to which the pane will be added.
    - `wp`: A pointer to a `window_pane` structure that is to be pushed onto the stack.
- **Control Flow**:
    - The function first checks if the `window_pane` pointer `wp` is not NULL.
    - If `wp` is not NULL, it calls [`window_pane_stack_remove`](#window_pane_stack_remove) to remove `wp` from the stack if it is already present.
    - Then, it inserts `wp` at the head of the `stack` using `TAILQ_INSERT_HEAD`.
    - Finally, it sets the `PANE_VISITED` flag for the `window_pane`.
- **Output**:
    - The function does not return a value; it modifies the state of the `window_panes` stack and the `window_pane`.
- **Functions called**:
    - [`window_pane_stack_remove`](#window_pane_stack_remove)


---
### window_pane_stack_remove<!-- {{#callable:window_pane_stack_remove}} -->
Removes a `window_pane` from the `window_panes` stack if it has been visited.
- **Inputs**:
    - `stack`: A pointer to a `window_panes` structure representing the stack from which the pane will be removed.
    - `wp`: A pointer to the `window_pane` structure that is to be removed from the stack.
- **Control Flow**:
    - Checks if the `window_pane` pointer `wp` is not NULL and if the `PANE_VISITED` flag is set.
    - If both conditions are met, it removes `wp` from the `stack` using `TAILQ_REMOVE`.
    - Then, it clears the `PANE_VISITED` flag from `wp`.
- **Output**:
    - The function does not return a value; it modifies the state of the `window_pane` and the `window_panes` stack directly.


---
### winlink_clear_flags<!-- {{#callable:winlink_clear_flags}} -->
The `winlink_clear_flags` function clears alert flags from a specified `winlink` and its associated `winlinks`.
- **Inputs**:
    - `wl`: A pointer to a `struct winlink` whose alert flags are to be cleared.
- **Control Flow**:
    - The function first clears the `WINDOW_ALERTFLAGS` from the `window` associated with the provided `winlink`.
    - It then iterates over all `winlinks` associated with the same `window` using `TAILQ_FOREACH`.
    - For each `winlink`, if its `flags` contain `WINLINK_ALERTFLAGS`, it clears this flag and calls `server_status_session` with the `session` of the `winlink`.
- **Output**:
    - The function does not return a value; it modifies the state of the `winlink` and its associated `winlinks` by clearing alert flags.


---
### winlink_shuffle_up<!-- {{#callable:winlink_shuffle_up}} -->
The `winlink_shuffle_up` function adjusts the index of a specified `winlink` in a session by moving it up in the index order.
- **Inputs**:
    - `s`: A pointer to the `session` structure that contains the list of windows.
    - `wl`: A pointer to the `winlink` structure that needs to be shuffled.
    - `before`: An integer flag indicating whether to shuffle the `winlink` before its current index (if true) or after (if false).
- **Control Flow**:
    - Check if the `winlink` pointer `wl` is NULL; if so, return -1 indicating failure.
    - Determine the starting index `idx` based on the `before` flag.
    - Find the next available index `last` starting from `idx` by checking for free indices in the session's windows.
    - If no free index is found (i.e., `last` reaches INT_MAX), return -1.
    - Shift all `winlink` indices from `last - 1` down to `idx` by incrementing their indices to make space for the specified `winlink`.
- **Output**:
    - Returns the new index of the `winlink` after it has been shuffled up, or -1 if the operation failed.
- **Functions called**:
    - [`winlink_find_by_index`](#winlink_find_by_index)


---
### window_pane_input_callback<!-- {{#callable:window_pane_input_callback}} -->
The `window_pane_input_callback` function processes input data for a window pane, handling errors and state changes.
- **Inputs**:
    - `struct client *c`: A pointer to the `client` structure representing the client that initiated the input.
    - `__unused const char *path`: An unused parameter that may represent the path of the input source.
    - `int error`: An integer indicating if there was an error during input processing.
    - `int closed`: An integer flag indicating if the input stream has been closed.
    - `struct evbuffer *buffer`: A pointer to an `evbuffer` structure containing the input data.
    - `void *data`: A pointer to user-defined data, specifically a `window_pane_input_data` structure.
- **Control Flow**:
    - The function retrieves the `window_pane` associated with the input data using [`window_pane_find_by_id`](#window_pane_find_by_id).
    - If the `file` in `cdata` is not NULL and either the `window_pane` is NULL or the client is dead, it sets the client's return value to 1 and marks the client for exit.
    - If the `file` is NULL or if the input stream is closed or an error occurred, it continues the command queue, unrefs the client, and frees the input data.
    - If none of the above conditions are met, it parses the input buffer using `input_parse_buffer`.
    - Finally, it drains the processed data from the `evbuffer`.
- **Output**:
    - The function does not return a value; it modifies the state of the client and the window pane based on the input data.
- **Functions called**:
    - [`window_pane_find_by_id`](#window_pane_find_by_id)


---
### window_pane_start_input<!-- {{#callable:window_pane_start_input}} -->
Initializes input handling for a specified window pane.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane to start input for.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item associated with the input.
    - `cause`: A pointer to a string that will hold an error message if the function fails.
- **Control Flow**:
    - Checks if the pane is empty; if not, sets an error message and returns -1.
    - Checks if the client associated with the command item is dead or exited; if so, returns 1.
    - Checks if the client has an active session; if so, returns 1.
    - Allocates memory for `window_pane_input_data` and initializes it with the command item and pane ID.
    - Calls `file_read` to start reading input from the client and increments the client's reference count.
- **Output**:
    - Returns 0 on success, -1 if the pane is not empty, or 1 if the client is dead, exited, or has an active session.


---
### window_pane_get_new_data<!-- {{#callable:window_pane_get_new_data}} -->
Retrieves new data from a `window_pane`'s input buffer and updates the size of the available data.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure from which data is to be retrieved.
    - `wpo`: A pointer to a `window_pane_offset` structure that tracks the offset of used data.
    - `size`: A pointer to a size_t variable where the size of the new data will be stored.
- **Control Flow**:
    - Calculates the amount of data already used by subtracting the `base_offset` from the `used` field in `wpo`.
    - Determines the size of the new data available in the input buffer by subtracting the used data from the total length of the input buffer.
    - Returns a pointer to the start of the new data in the input buffer by adding the used offset to the data pointer.
- **Output**:
    - Returns a pointer to the new data in the input buffer, starting from the calculated offset.


---
### window_pane_update_used_data<!-- {{#callable:window_pane_update_used_data}} -->
Updates the used data size in a `window_pane_offset` structure based on the current input buffer size.
- **Inputs**:
    - `wp`: A pointer to a `window_pane` structure representing the pane whose data is being updated.
    - `wpo`: A pointer to a `window_pane_offset` structure that tracks the used data size.
    - `size`: A size_t value representing the amount of data to be added to the used size.
- **Control Flow**:
    - Calculates the current used data by subtracting the base offset from the used offset in `wpo`.
    - Checks if the provided size exceeds the available space in the input buffer of the `window_pane`.
    - If it does, adjusts the size to the maximum available space.
    - Increments the used size in `wpo` by the adjusted size.
- **Output**:
    - The function does not return a value; it modifies the `used` field in the `window_pane_offset` structure directly.


---
### window_set_fill_character<!-- {{#callable:window_set_fill_character}} -->
Sets the fill character for a window based on its options.
- **Inputs**:
    - `w`: A pointer to a `struct window` that represents the window whose fill character is to be set.
- **Control Flow**:
    - Free the current fill character of the window to avoid memory leaks.
    - Retrieve the fill character string from the window's options using `options_get_string`.
    - Check if the retrieved string is not empty and is a valid UTF-8 character using `utf8_isvalid`.
    - Convert the valid UTF-8 string to a `struct utf8_data` using `utf8_fromcstr`.
    - If the conversion is successful and the width of the character is 1, assign it to the window's fill character.
    - If the character is invalid or has a width greater than 1, free the allocated memory for the character.
- **Output**:
    - The function does not return a value; it modifies the `fill_character` field of the `struct window` directly.


---
### window_pane_default_cursor<!-- {{#callable:window_pane_default_cursor}} -->
Sets the default cursor for a given window pane.
- **Inputs**:
    - `wp`: A pointer to a `struct window_pane` which contains the screen and options for the pane.
- **Control Flow**:
    - Calls the `screen_set_default_cursor` function with the screen and options from the `window_pane` structure.
- **Output**:
    - This function does not return a value; it modifies the cursor settings of the specified window pane.


---
### window_pane_mode<!-- {{#callable:window_pane_mode}} -->
The `window_pane_mode` function determines the current mode of a given window pane.
- **Inputs**:
    - `wp`: A pointer to a `struct window_pane` representing the window pane whose mode is to be determined.
- **Control Flow**:
    - The function first checks if the `modes` list of the window pane is not empty.
    - If the first mode in the list is `window_copy_mode`, it returns `WINDOW_PANE_COPY_MODE`.
    - If the first mode in the list is `window_view_mode`, it returns `WINDOW_PANE_VIEW_MODE`.
    - If none of the above conditions are met, it returns `WINDOW_PANE_NO_MODE`.
- **Output**:
    - The function returns an integer indicating the current mode of the window pane: `WINDOW_PANE_COPY_MODE`, `WINDOW_PANE_VIEW_MODE`, or `WINDOW_PANE_NO_MODE`.


---
### window_pane_show_scrollbar<!-- {{#callable:window_pane_show_scrollbar}} -->
The `window_pane_show_scrollbar` function determines whether a scrollbar should be displayed for a given window pane based on the current mode and scrollbar options.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane for which the scrollbar visibility is being checked.
    - `sb_option`: An integer representing the scrollbar display option, which can dictate whether the scrollbar is always shown, shown in modal mode, or not shown at all.
- **Control Flow**:
    - The function first checks if the screen associated with the window pane is in alternate mode using the `SCREEN_IS_ALTERNATE` macro; if it is, the function returns 0, indicating no scrollbar should be shown.
    - Next, it evaluates the `sb_option` parameter: if it is set to `PANE_SCROLLBARS_ALWAYS`, the function returns 1, indicating that the scrollbar should be displayed.
    - If `sb_option` is `PANE_SCROLLBARS_MODAL`, the function checks if the current mode of the window pane is not `WINDOW_PANE_NO_MODE` using the [`window_pane_mode`](#window_pane_mode) function; if this condition is met, it returns 1.
    - If none of the conditions for showing the scrollbar are met, the function returns 0.
- **Output**:
    - The function returns an integer: 1 if the scrollbar should be displayed, and 0 if it should not.
- **Functions called**:
    - [`window_pane_mode`](#window_pane_mode)


---
### window_pane_get_bg<!-- {{#callable:window_pane_get_bg}} -->
Retrieves the background color for a given `window_pane`.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure whose background color is to be retrieved.
- **Control Flow**:
    - Calls [`window_pane_get_bg_control_client`](#window_pane_get_bg_control_client) to check if a control client has provided a background color.
    - If the control client returns -1, it retrieves default colors using `tty_default_colours`.
    - Checks if the default background color is set; if not, it calls [`window_get_bg_client`](#window_get_bg_client) to get the background color from the client.
- **Output**:
    - Returns an integer representing the background color; returns -1 if no valid background color is found.
- **Functions called**:
    - [`window_pane_get_bg_control_client`](#window_pane_get_bg_control_client)
    - [`window_get_bg_client`](#window_get_bg_client)


---
### window_get_bg_client<!-- {{#callable:window_get_bg_client}} -->
The `window_get_bg_client` function retrieves the background color of the first attached client associated with a given window pane.
- **Inputs**:
    - `wp`: A pointer to a `struct window_pane` representing the window pane for which the background client color is to be retrieved.
- **Control Flow**:
    - Iterates through the global list of clients using `TAILQ_FOREACH`.
    - Checks if the current client is unattached by evaluating the `CLIENT_UNATTACHEDFLAGS` flag.
    - Continues to the next client if the current client's session is NULL or does not match the window associated with the pane.
    - Skips to the next client if the current client's background color is set to -1.
    - Returns the background color of the first valid client found, or -1 if no valid client is found.
- **Output**:
    - Returns the background color of the first attached client that has a valid background color, or -1 if no such client exists.


---
### window_pane_get_bg_control_client<!-- {{#callable:window_pane_get_bg_control_client}} -->
Retrieves the background control client ID for a given window pane.
- **Inputs**:
    - `wp`: A pointer to a `window_pane` structure representing the pane for which the background control client ID is to be retrieved.
- **Control Flow**:
    - Checks if the `control_bg` field of the `window_pane` structure is set to -1, returning -1 if true.
    - Iterates through the list of clients using `TAILQ_FOREACH` to check if any client has the `CLIENT_CONTROL` flag set.
    - If a client with the `CLIENT_CONTROL` flag is found, returns the `control_bg` value of the `window_pane`.
- **Output**:
    - Returns the background control client ID if a control client exists; otherwise, returns -1.


---
### window_pane_get_fg<!-- {{#callable:window_pane_get_fg}} -->
The `window_pane_get_fg` function retrieves the foreground color of the first attached client associated with a given window pane.
- **Inputs**:
    - `wp`: A pointer to a `struct window_pane` representing the window pane for which the foreground color is to be retrieved.
- **Control Flow**:
    - The function starts by obtaining a pointer to the `window` associated with the provided `window_pane`.
    - It then iterates over the list of clients using `TAILQ_FOREACH`.
    - For each client, it checks if the client is attached (not having `CLIENT_UNATTACHEDFLAGS`) and if the client's session is associated with the window.
    - If the client's foreground color (`tty.fg`) is valid (not -1), it returns that color.
    - If no valid foreground color is found after checking all clients, the function returns -1.
- **Output**:
    - The function returns an integer representing the foreground color of the first valid attached client, or -1 if no valid client is found.


---
### window_pane_get_fg_control_client<!-- {{#callable:window_pane_get_fg_control_client}} -->
Retrieves the foreground control client ID for a given window pane.
- **Inputs**:
    - `wp`: A pointer to a `struct window_pane` representing the window pane from which to retrieve the foreground control client ID.
- **Control Flow**:
    - Checks if the `control_fg` field of the `window_pane` structure is set to -1, indicating no valid foreground control client; if so, returns -1.
    - Iterates through the list of clients using `TAILQ_FOREACH` to check if any client has the `CLIENT_CONTROL` flag set.
    - If a client with the `CLIENT_CONTROL` flag is found, returns the `control_fg` value of the `window_pane`.
- **Output**:
    - Returns the foreground control client ID if a valid client is found; otherwise, returns -1.


---
### window_pane_get_theme<!-- {{#callable:window_pane_get_theme}} -->
The `window_pane_get_theme` function determines the theme of a given window pane based on its background color and associated clients.
- **Inputs**:
    - `wp`: A pointer to a `struct window_pane` representing the pane whose theme is to be determined.
- **Control Flow**:
    - Checks if the input pane pointer `wp` is NULL and returns `THEME_UNKNOWN` if it is.
    - Retrieves the associated `struct window` from the pane.
    - Attempts to derive the theme from the pane's background color using `colour_totheme`.
    - If a valid theme is found from the background color, it is returned immediately.
    - Iterates through the global list of clients to check for any attached clients that have a defined theme.
    - Flags are set to indicate if light or dark themes are found among the clients.
    - Based on the flags, returns `THEME_DARK`, `THEME_LIGHT`, or `THEME_UNKNOWN`.
- **Output**:
    - Returns an enumeration value of type `enum client_theme`, indicating the theme of the window pane, which can be `THEME_LIGHT`, `THEME_DARK`, or `THEME_UNKNOWN`.
- **Functions called**:
    - [`window_pane_get_bg`](#window_pane_get_bg)


---
### window_pane_send_theme_update<!-- {{#callable:window_pane_send_theme_update}} -->
The `window_pane_send_theme_update` function sends a theme update to a window pane if the theme has changed and updates are allowed.
- **Inputs**:
    - `wp`: A pointer to a `struct window_pane` representing the pane whose theme update is to be sent.
- **Control Flow**:
    - Check if the `PANE_THEMECHANGED` flag is not set; if so, return immediately.
    - Check if the `screen` mode does not allow theme updates; if so, return immediately.
    - Retrieve the current theme of the pane using `window_pane_get_theme(wp)`.
    - Based on the retrieved theme, send the appropriate key input to the pane using `input_key_pane`.
    - Clear the `PANE_THEMECHANGED` flag to indicate that the theme update has been processed.
- **Output**:
    - The function does not return a value; it modifies the state of the `window_pane` and sends input to it based on the current theme.
- **Functions called**:
    - [`window_pane_get_theme`](#window_pane_get_theme)


# Function Declarations (Public API)

---
### window_pane_create<!-- {{#callable_declaration:window_pane_create}} -->
Creates a new window pane and attaches it to a specified window.
- **Description**: This function is used to create a new window pane and attach it to a specified window. It initializes the pane with the given dimensions and history limit. The function must be called with a valid window pointer, and the dimensions should be positive values. It sets up various properties and options for the pane, including its screen and palette. The function returns a pointer to the newly created window pane, which can be used for further operations. It is important to ensure that the window pointer is valid and that the dimensions are within acceptable ranges to avoid undefined behavior.
- **Inputs**:
    - `w`: A pointer to a window structure to which the new pane will be attached. Must not be null.
    - `sx`: The width of the new pane in columns. Must be a positive integer.
    - `sy`: The height of the new pane in rows. Must be a positive integer.
    - `hlimit`: The history limit for the pane's scrollback buffer. Must be a non-negative integer.
- **Output**: Returns a pointer to the newly created window pane structure.
- **See also**: [`window_pane_create`](#window_pane_create)  (Implementation)


---
### window_pane_destroy<!-- {{#callable_declaration:window_pane_destroy}} -->
Destroys a window pane and releases its resources.
- **Description**: Use this function to properly destroy a window pane and free all associated resources. It should be called when a window pane is no longer needed to ensure that all memory and file descriptors are released. This function handles the cleanup of various resources such as buffers, events, and options associated with the window pane. It is important to ensure that the window pane is not in use before calling this function to avoid undefined behavior.
- **Inputs**:
    - `wp`: A pointer to the window pane to be destroyed. Must not be null and should point to a valid window pane structure. The function assumes ownership of the resources and will free them, so the caller should not use the window pane after this call.
- **Output**: None
- **See also**: [`window_pane_destroy`](#window_pane_destroy)  (Implementation)


---
### window_pane_full_size_offset<!-- {{#callable_declaration:window_pane_full_size_offset}} -->
Calculate the full size and offset of a window pane including scrollbars.
- **Description**: This function calculates the full size and offset of a window pane, taking into account the presence and position of scrollbars. It is useful when you need to determine the actual display area of a pane, including any additional space occupied by scrollbars. This function should be called when you need precise layout information for rendering or interaction purposes. The function does not modify the window pane itself but writes the calculated values to the provided pointers.
- **Inputs**:
    - `wp`: A pointer to a `struct window_pane` representing the pane for which the size and offset are being calculated. Must not be null.
    - `xoff`: A pointer to an unsigned integer where the horizontal offset will be stored. Must not be null.
    - `yoff`: A pointer to an unsigned integer where the vertical offset will be stored. Must not be null.
    - `sx`: A pointer to an unsigned integer where the full width, including scrollbars, will be stored. Must not be null.
    - `sy`: A pointer to an unsigned integer where the full height will be stored. Must not be null.
- **Output**: None
- **See also**: [`window_pane_full_size_offset`](#window_pane_full_size_offset)  (Implementation)


