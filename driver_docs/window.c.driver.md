# Purpose
The provided C source code is part of the `tmux` terminal multiplexer, specifically handling the management of windows and panes within a `tmux` session. This file is responsible for the creation, destruction, and manipulation of windows and their associated panes, which are essentially pseudo-terminal (pty) interfaces. Each window can contain multiple panes, and this code manages their lifecycle, including resizing, focus management, and input/output handling. The code also includes functions for managing the layout of panes within a window, handling user input, and updating the display based on changes in pane state.

Key technical components include the use of red-black trees for efficient management of windows and panes, as well as the use of various data structures to track the state and properties of each window and pane. The code defines several functions for creating and destroying windows and panes, managing their layout and focus, and handling input and output. It also includes mechanisms for synchronizing input across multiple panes and for handling special modes such as zooming and copying. The file is part of the core `tmux` server functionality and does not define public APIs or external interfaces, but rather provides internal mechanisms for managing the terminal multiplexing environment.
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
- **Description**: The `windows` variable is a global instance of the `struct windows` data structure. This structure is used to manage and store all the windows in the application, which are collections of panes, each associated with a pseudo-terminal (pty).
- **Use**: The `windows` variable is used to maintain a global list of all windows, allowing for operations such as creation, destruction, and management of window references and their associated panes.


---
### all_window_panes
- **Type**: `struct window_pane_tree`
- **Description**: The `all_window_panes` variable is a global data structure of type `struct window_pane_tree`. It is used to maintain a collection of all window panes in the application. This structure is likely implemented as a red-black tree, as indicated by the use of the `RB_GENERATE` macro for `window_pane_tree`.
- **Use**: This variable is used to store and manage all window panes globally, allowing for efficient access and manipulation of panes throughout the application.


---
### next_window_pane_id
- **Type**: `u_int`
- **Description**: The `next_window_pane_id` is a static unsigned integer variable used to assign unique identifiers to window panes within the application. It is incremented each time a new window pane is created, ensuring that each pane has a distinct ID.
- **Use**: This variable is used to generate and track unique IDs for window panes as they are created.


---
### next_window_id
- **Type**: `u_int`
- **Description**: The `next_window_id` is a static unsigned integer variable used to assign unique identifiers to newly created windows within the application. It is incremented each time a new window is created, ensuring that each window has a distinct ID.
- **Use**: This variable is used to generate and track unique IDs for windows as they are created.


---
### next_active_point
- **Type**: `u_int`
- **Description**: The `next_active_point` is a static unsigned integer variable used to track the next active point in the system. It is likely used to assign a unique identifier or sequence number to active elements, such as window panes, within the application.
- **Use**: This variable is incremented each time a new active point is set, ensuring each active element has a unique identifier.


---
### window_pane_create
- **Type**: `static struct window_pane *`
- **Description**: The `window_pane_create` function is a static function that returns a pointer to a `struct window_pane`. It is responsible for creating a new window pane within a given window, with specified dimensions and history limit.
- **Use**: This function is used to initialize and allocate a new window pane structure, setting up its properties and inserting it into the global window pane tree.


# Data Structures

---
### window_pane_input_data
- **Type**: `struct`
- **Members**:
    - `item`: A pointer to a cmdq_item structure, representing a command queue item associated with the window pane input.
    - `wp`: An unsigned integer representing the window pane identifier.
    - `file`: A pointer to a client_file structure, representing a file associated with the client.
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
    - It returns the result of the subtraction, which indicates the relative order of the two `winlink` structures based on their indices.
- **Output**:
    - An integer representing the difference between the indices of the two `winlink` structures; a positive value indicates that `wl1` has a higher index than `wl2`, a negative value indicates the opposite, and zero indicates they are equal.


---
### window_pane_cmp<!-- {{#callable:window_pane_cmp}} -->
Compares two `window_pane` structures based on their IDs.
- **Inputs**:
    - `wp1`: A pointer to the first `window_pane` structure to compare.
    - `wp2`: A pointer to the second `window_pane` structure to compare.
- **Control Flow**:
    - The function calculates the difference between the IDs of `wp1` and `wp2`.
    - The result of the subtraction is returned directly.
- **Output**:
    - An integer representing the difference between the IDs of the two `window_pane` structures, which can be used to determine their relative order.


---
### winlink_find_by_window<!-- {{#callable:winlink_find_by_window}} -->
The `winlink_find_by_window` function searches for a `winlink` associated with a specific `window` in a given set of `winlinks`.
- **Inputs**:
    - `wwl`: A pointer to a `struct winlinks` which is a collection of `winlink` structures.
    - `w`: A pointer to a `struct window` that is being searched for within the `winlinks`.
- **Control Flow**:
    - The function initializes a pointer `wl` to iterate through the `winlinks` using the `RB_FOREACH` macro.
    - For each `winlink` in the `winlinks`, it checks if the `window` pointer of the `winlink` matches the provided `window` pointer `w`.
    - If a match is found, the function returns the corresponding `winlink` pointer.
    - If no match is found after iterating through all `winlinks`, the function returns `NULL`.
- **Output**:
    - The function returns a pointer to the `winlink` that corresponds to the specified `window`, or `NULL` if no such `winlink` exists.


---
### winlink_find_by_index<!-- {{#callable:winlink_find_by_index}} -->
The `winlink_find_by_index` function retrieves a `winlink` structure from a `winlinks` tree based on a specified index.
- **Inputs**:
    - `wwl`: A pointer to a `struct winlinks` which represents the tree of winlinks.
    - `idx`: An integer representing the index of the winlink to be found.
- **Control Flow**:
    - The function first checks if the provided index `idx` is less than 0, and if so, it calls `fatalx` to terminate the program with an error message.
    - If the index is valid, it initializes a `struct winlink` variable `wl` with the given index.
    - Finally, it uses the `RB_FIND` macro to search for the winlink in the `wwl` tree using the initialized `wl` structure.
- **Output**:
    - Returns a pointer to the `winlink` structure found at the specified index, or NULL if not found.


---
### winlink_find_by_window_id<!-- {{#callable:winlink_find_by_window_id}} -->
The `winlink_find_by_window_id` function searches for a `winlink` structure associated with a specific window ID within a collection of `winlink` structures.
- **Inputs**:
    - `wwl`: A pointer to a `struct winlinks` which is a collection of `winlink` structures.
    - `id`: An unsigned integer representing the ID of the window to search for.
- **Control Flow**:
    - The function initializes a pointer `wl` to iterate through the `winlink` structures.
    - It uses the `RB_FOREACH` macro to traverse each `winlink` in the `wwl` collection.
    - For each `winlink`, it checks if the `window`'s ID matches the provided `id`.
    - If a match is found, it returns the corresponding `winlink` pointer.
    - If no match is found after checking all `winlink` structures, it returns NULL.
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
Counts the number of `winlink` structures in a `winlinks` tree.
- **Inputs**:
    - `wwl`: A pointer to a `struct winlinks`, which is a red-black tree containing `winlink` structures.
- **Control Flow**:
    - Initializes a counter `n` to zero.
    - Iterates over each `winlink` in the `winlinks` tree using `RB_FOREACH`.
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
    - If the index is valid and not already occupied, memory is allocated for a new `winlink` using [`xcalloc`](xmalloc.c.driver.md#xcalloc).
    - The new `winlink` is initialized with the specified index and inserted into the `winlinks` structure using `RB_INSERT`.
- **Output**:
    - The function returns a pointer to the newly added `winlink`, or NULL if the addition was unsuccessful.
- **Functions called**:
    - [`winlink_next_index`](#winlink_next_index)
    - [`winlink_find_by_index`](#winlink_find_by_index)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)


---
### winlink_set_window<!-- {{#callable:winlink_set_window}} -->
Sets the window associated with a given winlink, updating references accordingly.
- **Inputs**:
    - `wl`: A pointer to a `winlink` structure representing the winlink to be updated.
    - `w`: A pointer to a `window` structure representing the new window to associate with the winlink.
- **Control Flow**:
    - Checks if the current window in the winlink (`wl->window`) is not NULL.
    - If it is not NULL, removes the winlink from the current window's list of winlinks and decrements the reference count for that window.
    - Inserts the winlink at the tail of the new window's list of winlinks.
    - Updates the winlink's window pointer to the new window.
    - Increments the reference count for the new window.
- **Output**:
    - The function does not return a value; it modifies the state of the winlink and the associated windows.
- **Functions called**:
    - [`window_remove_ref`](#window_remove_ref)
    - [`window_add_ref`](#window_add_ref)


---
### winlink_remove<!-- {{#callable:winlink_remove}} -->
The `winlink_remove` function removes a specified `winlink` from a `winlinks` structure and frees its memory.
- **Inputs**:
    - `wwl`: A pointer to a `struct winlinks` which contains the list of winlinks.
    - `wl`: A pointer to the `struct winlink` that needs to be removed.
- **Control Flow**:
    - Check if the `window` associated with the `winlink` (`wl`) is not NULL.
    - If the `window` is not NULL, remove the `winlink` from the `winlinks` list of that `window` using `TAILQ_REMOVE`.
    - Call [`window_remove_ref`](#window_remove_ref) to decrement the reference count of the `window`.
    - Remove the `winlink` from the `winlinks` tree using `RB_REMOVE`.
    - Free the memory allocated for the `winlink`.
- **Output**:
    - The function does not return a value; it modifies the state of the `winlinks` structure and frees the memory of the removed `winlink`.
- **Functions called**:
    - [`window_remove_ref`](#window_remove_ref)


---
### winlink_next<!-- {{#callable:winlink_next}} -->
The `winlink_next` function retrieves the next `winlink` in a red-black tree of `winlink` structures.
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
    - The function enters a loop that decrements `n` until it reaches zero.
    - Within the loop, it attempts to get the next `winlink` using `RB_NEXT`.
    - If `RB_NEXT` returns NULL (indicating the end of the list), it wraps around to the first `winlink` using `RB_MIN`.
- **Output**:
    - Returns a pointer to the next `winlink` after advancing `n` positions, or the first `winlink` if the end of the list is reached.


---
### winlink_previous_by_number<!-- {{#callable:winlink_previous_by_number}} -->
The `winlink_previous_by_number` function retrieves the previous `winlink` in a session's window list based on a specified number of steps.
- **Inputs**:
    - `wl`: A pointer to the current `winlink` from which to start searching.
    - `s`: A pointer to the `session` structure that contains the list of windows.
    - `n`: An integer representing the number of steps to move backwards in the `winlink` list.
- **Control Flow**:
    - The function enters a loop that continues as long as `n` is greater than 0.
    - Within the loop, it attempts to retrieve the previous `winlink` using `RB_PREV`.
    - If `RB_PREV` returns NULL, indicating that there are no more previous `winlinks`, it resets `wl` to the maximum `winlink` in the session's window list using `RB_MAX`.
- **Output**:
    - Returns a pointer to the `winlink` that is `n` steps before the given `wl`, or the maximum `winlink` if the end of the list is reached.


---
### winlink_stack_push<!-- {{#callable:winlink_stack_push}} -->
The `winlink_stack_push` function adds a `winlink` to the top of a `winlink_stack` after removing it if it already exists.
- **Inputs**:
    - `stack`: A pointer to the `winlink_stack` structure where the `winlink` will be pushed.
    - `wl`: A pointer to the `winlink` structure that is to be pushed onto the stack.
- **Control Flow**:
    - The function first checks if the `wl` pointer is NULL; if it is, the function returns immediately without making any changes.
    - If `wl` is not NULL, it calls [`winlink_stack_remove`](#winlink_stack_remove) to remove `wl` from the stack if it is already present.
    - Then, it inserts `wl` at the head of the `stack` using `TAILQ_INSERT_HEAD`.
    - Finally, it sets the `WINLINK_VISITED` flag for the `wl` to indicate that it has been added to the stack.
- **Output**:
    - The function does not return a value; it modifies the state of the `winlink_stack` and the `winlink` passed to it.
- **Functions called**:
    - [`winlink_stack_remove`](#winlink_stack_remove)


---
### winlink_stack_remove<!-- {{#callable:winlink_stack_remove}} -->
The `winlink_stack_remove` function removes a `winlink` from a `winlink_stack` if it has been visited.
- **Inputs**:
    - `stack`: A pointer to the `winlink_stack` from which the `winlink` will be removed.
    - `wl`: A pointer to the `winlink` that is to be removed from the stack.
- **Control Flow**:
    - Check if the `wl` pointer is not NULL and if the `WINLINK_VISITED` flag is set for the `winlink`.
    - If both conditions are true, remove the `winlink` from the `winlink_stack` using `TAILQ_REMOVE`.
    - Clear the `WINLINK_VISITED` flag for the `winlink`.
- **Output**:
    - The function does not return a value; it modifies the state of the `winlink_stack` and the `winlink` by removing it from the stack and updating its flags.


---
### window_find_by_id_str<!-- {{#callable:window_find_by_id_str}} -->
The `window_find_by_id_str` function retrieves a `window` structure based on a string identifier.
- **Inputs**:
    - `s`: A pointer to a string that represents the identifier of the window, which must start with '@'.
- **Control Flow**:
    - Checks if the first character of the input string `s` is not '@', returning NULL if true.
    - Uses [`strtonum`](compat/strtonum.c.driver.md#strtonum) to convert the substring starting from the second character of `s` into an unsigned integer `id`, checking for conversion errors.
    - If the conversion is successful, calls [`window_find_by_id`](#window_find_by_id) with the obtained `id` to retrieve the corresponding `window` structure.
- **Output**:
    - Returns a pointer to the `window` structure if found, or NULL if the input string is invalid or the conversion fails.
- **Functions called**:
    - [`strtonum`](compat/strtonum.c.driver.md#strtonum)
    - [`window_find_by_id`](#window_find_by_id)


---
### window_find_by_id<!-- {{#callable:window_find_by_id}} -->
The `window_find_by_id` function retrieves a `window` structure from a red-black tree based on a given window ID.
- **Inputs**:
    - `id`: An unsigned integer representing the unique identifier of the window to be found.
- **Control Flow**:
    - A local `window` structure is created and its `id` field is set to the provided `id` argument.
    - The function then calls `RB_FIND` to search for the window in the global `windows` red-black tree using the local `window` structure.
- **Output**:
    - Returns a pointer to the `window` structure if found; otherwise, it returns NULL.


---
### window_update_activity<!-- {{#callable:window_update_activity}} -->
Updates the activity time of a window and queues an alert for window activity.
- **Inputs**:
    - `w`: A pointer to a `struct window` representing the window whose activity is being updated.
- **Control Flow**:
    - Calls `gettimeofday` to retrieve the current time and updates the `activity_time` field of the window structure.
    - Invokes [`alerts_queue`](alerts.c.driver.md#alerts_queue) function to enqueue an alert indicating that there has been activity in the window.
- **Output**:
    - This function does not return a value; it modifies the state of the window and triggers an alert.
- **Functions called**:
    - [`alerts_queue`](alerts.c.driver.md#alerts_queue)


---
### window_create<!-- {{#callable:window_create}} -->
Creates a new `window` structure with specified dimensions and pixel sizes.
- **Inputs**:
    - `sx`: The width of the window in character cells.
    - `sy`: The height of the window in character cells.
    - `xpixel`: The width of the window in pixels, defaults to `DEFAULT_XPIXEL` if zero.
    - `ypixel`: The height of the window in pixels, defaults to `DEFAULT_YPIXEL` if zero.
- **Control Flow**:
    - Checks if `xpixel` and `ypixel` are zero and assigns default values if necessary.
    - Allocates memory for a new `window` structure using [`xcalloc`](xmalloc.c.driver.md#xcalloc).
    - Initializes various fields of the `window` structure, including name, flags, dimensions, and options.
    - Increments the global `next_window_id` and assigns it to the new window.
    - Inserts the new window into a global list of windows using a red-black tree.
    - Calls [`window_set_fill_character`](#window_set_fill_character) to set the fill character for the window.
    - Calls [`window_update_activity`](#window_update_activity) to update the activity timestamp of the window.
    - Logs the creation of the window and returns a pointer to the new window.
- **Output**:
    - Returns a pointer to the newly created `window` structure.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`options_create`](options.c.driver.md#options_create)
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
    - Removes the window from the global list of windows.
    - Frees the layout cells if they exist.
    - Calls [`window_destroy_panes`](#window_destroy_panes) to clean up all panes associated with the window.
    - Checks and deletes any initialized events related to the window.
    - Frees the options and fill character associated with the window.
    - Frees the name of the window and finally the window structure itself.
- **Output**:
    - The function does not return a value; it performs cleanup and deallocation of resources associated with the specified window.
- **Functions called**:
    - [`window_unzoom`](#window_unzoom)
    - [`layout_free_cell`](layout.c.driver.md#layout_free_cell)
    - [`window_destroy_panes`](#window_destroy_panes)
    - [`options_free`](options.c.driver.md#options_free)


---
### window_pane_destroy_ready<!-- {{#callable:window_pane_destroy_ready}} -->
The `window_pane_destroy_ready` function checks if a `window_pane` can be safely destroyed.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure that is being checked for readiness to be destroyed.
- **Control Flow**:
    - The function first checks if the `pipe_fd` of the `window_pane` is not -1, indicating that it is in use.
    - If the `pipe_fd` is valid, it checks if there is any data in the output buffer of the associated event; if there is, it returns 0, indicating that the pane is not ready to be destroyed.
    - Next, it checks if there are any bytes available to read from the file descriptor associated with the pane; if there are, it also returns 0.
    - If the `PANE_EXITED` flag is not set, it returns 0, indicating that the pane is still active.
    - If all checks pass, it returns 1, indicating that the pane is ready to be destroyed.
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
    - `from`: A string indicating the source of the reference removal, used for logging purposes.
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
Sets the name of a window after freeing the old name.
- **Inputs**:
    - `w`: A pointer to the `window` structure whose name is to be set.
    - `new_name`: A pointer to a string containing the new name for the window.
- **Control Flow**:
    - The function first frees the memory allocated for the current name of the window.
    - It then calls [`utf8_stravis`](utf8.c.driver.md#utf8_stravis) to convert the `new_name` into a safe UTF-8 string and assigns it to the window's name.
    - Finally, it calls [`notify_window`](notify.c.driver.md#notify_window) to notify that the window has been renamed.
- **Output**:
    - The function does not return a value; it modifies the window's name and triggers a notification.
- **Functions called**:
    - [`utf8_stravis`](utf8.c.driver.md#utf8_stravis)
    - [`notify_window`](notify.c.driver.md#notify_window)


---
### window_resize<!-- {{#callable:window_resize}} -->
The `window_resize` function updates the size and pixel dimensions of a specified window.
- **Inputs**:
    - `w`: A pointer to the `window` structure that represents the window to be resized.
    - `sx`: An unsigned integer representing the new width of the window.
    - `sy`: An unsigned integer representing the new height of the window.
    - `xpixel`: An integer representing the new pixel width; if set to 0, it defaults to `DEFAULT_XPIXEL`.
    - `ypixel`: An integer representing the new pixel height; if set to 0, it defaults to `DEFAULT_YPIXEL`.
- **Control Flow**:
    - If `xpixel` is 0, it is set to `DEFAULT_XPIXEL`.
    - If `ypixel` is 0, it is set to `DEFAULT_YPIXEL`.
    - A debug log is generated to record the resizing action, including the window ID and new dimensions.
    - The width (`sx`) and height (`sy`) of the window are updated to the new values.
    - If `xpixel` is not -1, the pixel width of the window is updated.
    - If `ypixel` is not -1, the pixel height of the window is updated.
- **Output**:
    - The function does not return a value; it modifies the properties of the specified window directly.


---
### window_pane_send_resize<!-- {{#callable:window_pane_send_resize}} -->
The `window_pane_send_resize` function sends a resize request to a window pane by updating its terminal size.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane to be resized.
    - `sx`: An unsigned integer representing the new width (in columns) of the window pane.
    - `sy`: An unsigned integer representing the new height (in rows) of the window pane.
- **Control Flow**:
    - Check if the file descriptor (`fd`) of the window pane is valid (not -1); if invalid, exit the function.
    - Log the resize action with the pane ID and new dimensions.
    - Initialize a `winsize` structure and set its fields for columns, rows, and pixel dimensions based on the window's pixel size.
    - Use the `ioctl` system call to send the new size to the terminal associated with the pane.
    - Handle potential errors from the `ioctl` call, specifically for Solaris systems, where certain errors are ignored.
- **Output**:
    - The function does not return a value; it modifies the terminal size of the specified window pane and may log an error if the resize fails.


---
### window_has_pane<!-- {{#callable:window_has_pane}} -->
The `window_has_pane` function checks if a specific pane is part of a given window.
- **Inputs**:
    - `w`: A pointer to the `struct window` that contains a list of panes.
    - `wp`: A pointer to the `struct window_pane` that needs to be checked for existence in the window.
- **Control Flow**:
    - The function iterates over each pane in the list of panes associated with the given window `w` using `TAILQ_FOREACH`.
    - For each pane `wp1`, it checks if it is equal to the pane `wp` being searched for.
    - If a match is found, the function returns 1, indicating that the pane exists in the window.
    - If the loop completes without finding a match, the function returns 0, indicating that the pane does not exist in the window.
- **Output**:
    - The function returns an integer: 1 if the specified pane is found in the window, and 0 if it is not.


---
### window_update_focus<!-- {{#callable:window_update_focus}} -->
Updates the focus of a window by logging its ID and updating the active pane's focus.
- **Inputs**:
    - `w`: A pointer to a `struct window` representing the window whose focus is to be updated.
- **Control Flow**:
    - Checks if the input window pointer `w` is not NULL.
    - Logs the function name and the ID of the window.
    - Calls [`window_pane_update_focus`](#window_pane_update_focus) with the active pane of the window.
- **Output**:
    - The function does not return a value; it performs actions to update the focus of the window's active pane.
- **Functions called**:
    - [`window_pane_update_focus`](#window_pane_update_focus)


---
### window_pane_update_focus<!-- {{#callable:window_pane_update_focus}} -->
Updates the focus state of a given `window_pane` based on the active client.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure that needs to be updated.
- **Control Flow**:
    - Checks if the `window_pane` is not NULL and has not exited.
    - Determines if the `window_pane` is the active pane of its parent window.
    - Iterates through the list of clients to check if any client is focused and attached to the same window as the `window_pane`.
    - If no client is focused and the pane is currently focused, it logs a focus out event, sends a focus out notification, and updates the pane's flags.
    - If a client is focused and the pane is not currently focused, it logs a focus in event, sends a focus in notification, and updates the pane's flags.
    - If the focus state remains unchanged, it logs that the focus is unchanged.
- **Output**:
    - The function does not return a value; it modifies the state of the `window_pane` and logs focus changes.
- **Functions called**:
    - [`notify_pane`](notify.c.driver.md#notify_pane)


---
### window_set_active_pane<!-- {{#callable:window_set_active_pane}} -->
Sets the active pane of a window and updates related states.
- **Inputs**:
    - `w`: A pointer to the `window` structure representing the window containing the panes.
    - `wp`: A pointer to the `window_pane` structure that is to be set as the active pane.
    - `notify`: An integer flag indicating whether to send a notification about the pane change.
- **Control Flow**:
    - Logs the function call with the ID of the pane to be activated.
    - Checks if the specified pane is already the active pane; if so, returns 0.
    - Stores the currently active pane in `lastwp`.
    - Removes the specified pane from the last panes stack and pushes the last active pane onto it.
    - Sets the specified pane as the new active pane and updates its active point and flags.
    - If focus events are enabled, updates the focus state of the last and new active panes.
    - Updates the window's offset in the terminal.
    - If `notify` is true, sends a notification about the pane change.
    - Returns 1 to indicate that the active pane has been changed.
- **Output**:
    - Returns 0 if the specified pane is already active; otherwise, returns 1 indicating that the active pane has been successfully changed.
- **Functions called**:
    - [`window_pane_stack_remove`](#window_pane_stack_remove)
    - [`window_pane_stack_push`](#window_pane_stack_push)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`window_pane_update_focus`](#window_pane_update_focus)
    - [`tty_update_window_offset`](tty.c.driver.md#tty_update_window_offset)
    - [`notify_window`](notify.c.driver.md#notify_window)


---
### window_pane_get_palette<!-- {{#callable:window_pane_get_palette}} -->
Retrieves a color from the palette of a specified window pane.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane from which to retrieve the color.
    - `c`: An integer representing the index of the color in the palette.
- **Control Flow**:
    - Checks if the `wp` pointer is NULL; if it is, returns -1 to indicate an error.
    - If `wp` is valid, calls the [`colour_palette_get`](colour.c.driver.md#colour_palette_get) function with the pane's palette and the color index `c` to retrieve the color.
- **Output**:
    - Returns the color value from the palette corresponding to the index `c`, or -1 if the pane pointer is NULL.
- **Functions called**:
    - [`colour_palette_get`](colour.c.driver.md#colour_palette_get)


---
### window_redraw_active_switch<!-- {{#callable:window_redraw_active_switch}} -->
The `window_redraw_active_switch` function updates the redraw flags of window panes based on their active and inactive styles.
- **Inputs**:
    - `w`: A pointer to the `window` structure representing the current window.
    - `wp`: A pointer to the `window_pane` structure that needs to be checked for redraw.
- **Control Flow**:
    - The function first checks if the provided `window_pane` (`wp`) is the active pane of the `window` (`w`); if so, it returns immediately without making any changes.
    - It enters an infinite loop where it compares the cached graphics cells of the `window_pane` (`wp`) with the active graphics cell.
    - If the graphics cells are not equal, it sets the `PANE_REDRAW` flag for the `window_pane`.
    - If the graphics cells are equal, it checks the foreground colors of the graphics cells; if they differ, it sets the `PANE_REDRAW` flag.
    - It then checks the background colors in a similar manner, setting the `PANE_REDRAW` flag if they differ.
    - The loop continues until `wp` becomes the active pane of the `window`, at which point it breaks out of the loop.
- **Output**:
    - The function does not return a value; instead, it modifies the `flags` of the `window_pane` to indicate whether it needs to be redrawn.
- **Functions called**:
    - [`grid_cells_look_equal`](grid.c.driver.md#grid_cells_look_equal)
    - [`window_pane_get_palette`](#window_pane_get_palette)


---
### window_get_active_at<!-- {{#callable:window_get_active_at}} -->
The `window_get_active_at` function retrieves the active `window_pane` at specified coordinates within a given `window`.
- **Inputs**:
    - `w`: A pointer to the `window` structure that contains the panes.
    - `x`: An unsigned integer representing the x-coordinate to check for an active pane.
    - `y`: An unsigned integer representing the y-coordinate to check for an active pane.
- **Control Flow**:
    - Iterates through each `window_pane` in the `window`'s list of panes using `TAILQ_FOREACH`.
    - Checks if the current `window_pane` is visible using the [`window_pane_visible`](#window_pane_visible) function.
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
The `window_find_string` function locates a `window_pane` based on a specified string that represents a position.
- **Inputs**:
    - `w`: A pointer to the `window` structure in which to search for the pane.
    - `s`: A string representing the desired position (e.g., 'top', 'bottom', 'left', 'right', etc.).
- **Control Flow**:
    - Initializes `x` and `y` coordinates to the center of the window.
    - Retrieves the pane border status from the window options to adjust the `top` and `bottom` boundaries.
    - Compares the input string `s` against predefined position strings using `strcasecmp` to determine the appropriate `x` and `y` coordinates.
    - If the input string does not match any known positions, the function returns NULL.
    - Calls [`window_get_active_at`](#window_get_active_at) with the calculated `x` and `y` to find the corresponding `window_pane`.
- **Output**:
    - Returns a pointer to the `window_pane` located at the specified position, or NULL if no valid position was provided.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`window_get_active_at`](#window_get_active_at)


---
### window_zoom<!-- {{#callable:window_zoom}} -->
The `window_zoom` function zooms a specified window pane, adjusting its layout and state.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure that represents the pane to be zoomed.
- **Control Flow**:
    - Checks if the window is already zoomed by evaluating the `WINDOW_ZOOMED` flag; if so, returns -1.
    - Checks if the window has only one pane; if true, returns -1.
    - If the specified pane is not the active pane, it sets it as the active pane.
    - Iterates through all panes in the window, saving their current layout and setting their layout to NULL.
    - Saves the current layout root of the window and initializes the layout for the zoomed pane.
    - Sets the `WINDOW_ZOOMED` flag to indicate that the window is now zoomed.
    - Notifies the system that the window layout has changed.
- **Output**:
    - Returns 0 on successful zooming of the pane, or -1 if the zoom operation could not be performed.
- **Functions called**:
    - [`window_count_panes`](#window_count_panes)
    - [`window_set_active_pane`](#window_set_active_pane)
    - [`layout_init`](layout.c.driver.md#layout_init)
    - [`notify_window`](notify.c.driver.md#notify_window)


---
### window_unzoom<!-- {{#callable:window_unzoom}} -->
The `window_unzoom` function restores a window's layout from a zoomed state.
- **Inputs**:
    - `w`: A pointer to the `window` structure representing the window to unzoom.
    - `notify`: An integer flag indicating whether to notify clients of the layout change.
- **Control Flow**:
    - Checks if the window is currently zoomed by evaluating the `WINDOW_ZOOMED` flag.
    - If the window is not zoomed, it returns -1 indicating failure.
    - Clears the `WINDOW_ZOOMED` flag from the window's flags.
    - Frees the current layout using [`layout_free`](layout.c.driver.md#layout_free).
    - Restores the layout root from `saved_layout_root` and sets `saved_layout_root` to NULL.
    - Iterates over each `window_pane` in the window's panes list, restoring their layout cells from `saved_layout_cell`.
    - Calls [`layout_fix_panes`](layout.c.driver.md#layout_fix_panes) to adjust the layout of the panes.
    - If the `notify` flag is set, it sends a notification about the layout change.
- **Output**:
    - Returns 0 on success, indicating the window has been successfully unzoomed, or -1 if the window was not zoomed.
- **Functions called**:
    - [`layout_free`](layout.c.driver.md#layout_free)
    - [`layout_fix_panes`](layout.c.driver.md#layout_fix_panes)
    - [`notify_window`](notify.c.driver.md#notify_window)


---
### window_push_zoom<!-- {{#callable:window_push_zoom}} -->
The `window_push_zoom` function manages the zoom state of a window by unzooming it and potentially marking it as previously zoomed.
- **Inputs**:
    - `w`: A pointer to the `window` structure that represents the window to be manipulated.
    - `always`: An integer flag indicating whether to always mark the window as previously zoomed.
    - `flag`: An integer flag that determines if the zoom state should be considered.
- **Control Flow**:
    - Logs the current function call with the window ID and whether it is currently zoomed.
    - Checks if the `flag` is set and either `always` is true or the window is currently zoomed.
    - If the conditions are met, it sets the `WINDOW_WASZOOMED` flag on the window.
    - If the conditions are not met, it clears the `WINDOW_WASZOOMED` flag.
    - Calls the [`window_unzoom`](#window_unzoom) function to unzoom the window and returns the result of that call.
- **Output**:
    - Returns 1 if the window was successfully unzoomed, otherwise returns 0.
- **Functions called**:
    - [`window_unzoom`](#window_unzoom)


---
### window_pop_zoom<!-- {{#callable:window_pop_zoom}} -->
The `window_pop_zoom` function restores the zoom state of a window if it was previously zoomed.
- **Inputs**:
    - `w`: A pointer to a `struct window` representing the window whose zoom state is to be restored.
- **Control Flow**:
    - Logs the current function name and the window ID along with its zoomed state.
    - Checks if the `WINDOW_WASZOOMED` flag is set in the window's flags.
    - If the flag is set, it calls the [`window_zoom`](#window_zoom) function on the active pane of the window and returns the result.
    - If the flag is not set, it returns 0.
- **Output**:
    - Returns 0 if the window was not previously zoomed or the result of the [`window_zoom`](#window_zoom) function if it was zoomed.
- **Functions called**:
    - [`window_zoom`](#window_zoom)


---
### window_add_pane<!-- {{#callable:window_add_pane}} -->
The `window_add_pane` function adds a new pane to a specified window, positioning it relative to an existing pane based on provided flags.
- **Inputs**:
    - `w`: A pointer to the `window` structure where the new pane will be added.
    - `other`: A pointer to another `window_pane` structure that serves as a reference for positioning the new pane; if NULL, the active pane is used.
    - `hlimit`: An unsigned integer representing the height limit for the new pane.
    - `flags`: An integer containing flags that dictate the positioning of the new pane, such as whether to insert it before or after the reference pane.
- **Control Flow**:
    - If `other` is NULL, it is set to the currently active pane in the window.
    - A new pane is created using [`window_pane_create`](#window_pane_create) with the specified dimensions and height limit.
    - If the list of panes in the window is empty, the new pane is inserted at the head of the list.
    - If the `SPAWN_BEFORE` flag is set, the new pane is inserted before the `other` pane or at the head if `SPAWN_FULLSIZE` is also set.
    - If the `SPAWN_BEFORE` flag is not set, the new pane is inserted after the `other` pane or at the tail if `SPAWN_FULLSIZE` is set.
- **Output**:
    - The function returns a pointer to the newly created `window_pane`.
- **Functions called**:
    - [`window_pane_create`](#window_pane_create)


---
### window_lost_pane<!-- {{#callable:window_lost_pane}} -->
The `window_lost_pane` function handles the removal of a pane from a window, updating the active pane and notifying relevant components.
- **Inputs**:
    - `w`: A pointer to the `window` structure representing the window from which the pane is being removed.
    - `wp`: A pointer to the `window_pane` structure representing the pane that is being lost.
- **Control Flow**:
    - Logs the action of losing a pane with its window and pane IDs.
    - Checks if the pane being lost is marked and clears the marked state if true.
    - Removes the pane from the stack of last panes in the window.
    - If the lost pane is the active pane, it attempts to set a new active pane from the last panes.
    - If no last panes are available, it tries to set the active pane to the previous or next pane in the list.
    - If a new active pane is set, it updates its flags, notifies the window of the change, and updates the window focus.
- **Output**:
    - The function does not return a value but modifies the state of the window and its panes, specifically updating the active pane and notifying any listeners of the change.
- **Functions called**:
    - [`server_clear_marked`](server.c.driver.md#server_clear_marked)
    - [`window_pane_stack_remove`](#window_pane_stack_remove)
    - [`notify_window`](notify.c.driver.md#notify_window)
    - [`window_update_focus`](#window_update_focus)


---
### window_remove_pane<!-- {{#callable:window_remove_pane}} -->
The `window_remove_pane` function removes a specified pane from a window and cleans up its resources.
- **Inputs**:
    - `w`: A pointer to the `window` structure from which the pane is to be removed.
    - `wp`: A pointer to the `window_pane` structure that is to be removed from the window.
- **Control Flow**:
    - Calls [`window_lost_pane`](#window_lost_pane) to handle any necessary updates related to the pane being removed.
    - Removes the specified pane from the linked list of panes in the window using `TAILQ_REMOVE`.
    - Destroys the pane by calling [`window_pane_destroy`](#window_pane_destroy) to free its resources.
- **Output**:
    - The function does not return a value; it performs operations that modify the state of the window and the pane.
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
    - The function first retrieves the base index for panes from the window's options using [`options_get_number`](options.c.driver.md#options_get_number).
    - It then iterates through the list of panes in the window using `TAILQ_FOREACH`.
    - During each iteration, it checks if the current index matches the provided index (`idx`).
    - If a match is found, it returns the pointer to the corresponding `window_pane`.
    - If no match is found after iterating through all panes, it returns NULL.
- **Output**:
    - The function returns a pointer to the `window_pane` at the specified index if found, otherwise it returns NULL.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)


---
### window_pane_next_by_number<!-- {{#callable:window_pane_next_by_number}} -->
The `window_pane_next_by_number` function retrieves the next `window_pane` in a circular manner based on a specified number.
- **Inputs**:
    - `w`: A pointer to the `window` structure containing the list of panes.
    - `wp`: A pointer to the current `window_pane` from which to start the search.
    - `n`: An unsigned integer representing the number of panes to skip.
- **Control Flow**:
    - The function enters a loop that iterates `n` times.
    - In each iteration, it attempts to move to the next pane using `TAILQ_NEXT`.
    - If the next pane is `NULL`, it wraps around to the first pane in the list using `TAILQ_FIRST`.
- **Output**:
    - Returns a pointer to the next `window_pane` after skipping `n` panes, wrapping around if necessary.


---
### window_pane_previous_by_number<!-- {{#callable:window_pane_previous_by_number}} -->
The `window_pane_previous_by_number` function retrieves the previous `window_pane` in a linked list based on a specified number of steps.
- **Inputs**:
    - `w`: A pointer to the `window` structure that contains the list of panes.
    - `wp`: A pointer to the current `window_pane` from which to start the search.
    - `n`: An unsigned integer representing the number of steps to move backwards in the list of panes.
- **Control Flow**:
    - The function enters a loop that iterates `n` times, decrementing `n` with each iteration.
    - In each iteration, it attempts to move to the previous `window_pane` using `TAILQ_PREV`.
    - If `TAILQ_PREV` returns NULL (indicating the start of the list), it wraps around to the last `window_pane` in the list using `TAILQ_LAST`.
- **Output**:
    - The function returns a pointer to the resulting `window_pane` after moving backwards `n` steps, or the last pane if the end of the list is reached.


---
### window_pane_index<!-- {{#callable:window_pane_index}} -->
The `window_pane_index` function calculates the index of a specified `window_pane` within its parent `window`.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` whose index is to be determined.
    - `i`: A pointer to an unsigned integer where the base index will be stored.
- **Control Flow**:
    - The function retrieves the base index for panes from the window's options using [`options_get_number`](options.c.driver.md#options_get_number).
    - It iterates through the list of panes in the window using `TAILQ_FOREACH`.
    - For each pane, it checks if the current pane (`wq`) is the same as the input pane (`wp`).
    - If a match is found, the function returns 0, indicating success.
    - If no match is found after iterating through all panes, the function returns -1.
- **Output**:
    - The function returns 0 on success (if the pane is found) or -1 if the pane is not found in the window.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)


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
The `window_printable_flags` function generates a string of flags representing the state of a given window link.
- **Inputs**:
    - `wl`: A pointer to a `winlink` structure representing the window link whose flags are to be printed.
    - `escape`: An integer indicating whether to escape the activity flag with an additional '#' character.
- **Control Flow**:
    - Initializes a static character array `flags` to hold the resulting flag characters.
    - Checks if the `WINLINK_ACTIVITY` flag is set; if so, adds '#' to `flags`, and if `escape` is true, adds another '#'.
    - Checks if the `WINLINK_BELL` flag is set; if so, adds '!' to `flags`.
    - Checks if the `WINLINK_SILENCE` flag is set; if so, adds '~' to `flags`.
    - Checks if the current window link is the active one in the session; if so, adds '*' to `flags`.
    - Checks if the window link is the first in the list of last windows; if so, adds '-' to `flags`.
    - Checks if the window link is marked; if so, adds 'M' to `flags`.
    - Checks if the window is zoomed; if so, adds 'Z' to `flags`.
    - Null-terminates the `flags` string and returns it.
- **Output**:
    - Returns a pointer to a string containing the concatenated flag characters representing the state of the window link.
- **Functions called**:
    - [`server_check_marked`](server.c.driver.md#server_check_marked)


---
### window_pane_find_by_id_str<!-- {{#callable:window_pane_find_by_id_str}} -->
The `window_pane_find_by_id_str` function retrieves a `window_pane` structure based on a string identifier.
- **Inputs**:
    - `s`: A pointer to a string that represents the identifier of the window pane, which must start with a '%' character.
- **Control Flow**:
    - The function first checks if the first character of the input string `s` is '%'. If not, it returns NULL.
    - It then calls [`strtonum`](compat/strtonum.c.driver.md#strtonum) to convert the substring starting from the second character into an unsigned integer, while also checking for conversion errors.
    - If the conversion is successful, it calls [`window_pane_find_by_id`](#window_pane_find_by_id) with the converted ID to retrieve the corresponding `window_pane`.
- **Output**:
    - Returns a pointer to the `window_pane` structure corresponding to the given ID, or NULL if the input string is invalid or the ID conversion fails.
- **Functions called**:
    - [`strtonum`](compat/strtonum.c.driver.md#strtonum)
    - [`window_pane_find_by_id`](#window_pane_find_by_id)


---
### window_pane_find_by_id<!-- {{#callable:window_pane_find_by_id}} -->
The `window_pane_find_by_id` function retrieves a `window_pane` structure from a red-black tree using a specified pane ID.
- **Inputs**:
    - `id`: An unsigned integer representing the unique identifier of the window pane to be found.
- **Control Flow**:
    - A temporary `window_pane` structure `wp` is created and its `id` field is set to the provided `id` argument.
    - The function then calls `RB_FIND` to search for the `window_pane` in the `all_window_panes` red-black tree using the temporary `wp` structure.
- **Output**:
    - Returns a pointer to the `window_pane` structure if found; otherwise, it returns NULL.


---
### window_pane_create<!-- {{#callable:window_pane_create}} -->
Creates and initializes a new `window_pane` structure.
- **Inputs**:
    - `w`: A pointer to the `window` structure that this pane will belong to.
    - `sx`: An unsigned integer representing the width of the pane.
    - `sy`: An unsigned integer representing the height of the pane.
    - `hlimit`: An unsigned integer representing the height limit for the pane.
- **Control Flow**:
    - Allocates memory for a new `window_pane` structure using [`xcalloc`](xmalloc.c.driver.md#xcalloc).
    - Initializes the `window` pointer and creates options for the pane based on the parent window's options.
    - Sets default flags for the pane indicating style and theme changes.
    - Increments the global pane ID and inserts the new pane into a red-black tree for tracking all panes.
    - Initializes various properties of the pane, including its dimensions, file descriptors, and color palette.
    - Sets the scrollbar style based on options and initializes the screen for the pane.
    - If successful, retrieves the hostname and sets it as the title of the pane's screen.
- **Output**:
    - Returns a pointer to the newly created `window_pane` structure.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`options_create`](options.c.driver.md#options_create)
    - [`style_set_scrollbar_style_from_option`](style.c.driver.md#style_set_scrollbar_style_from_option)
    - [`colour_palette_init`](colour.c.driver.md#colour_palette_init)
    - [`colour_palette_from_option`](colour.c.driver.md#colour_palette_from_option)
    - [`screen_init`](screen.c.driver.md#screen_init)
    - [`window_pane_default_cursor`](#window_pane_default_cursor)
    - [`screen_set_title`](screen.c.driver.md#screen_set_title)


---
### window_pane_destroy<!-- {{#callable:window_pane_destroy}} -->
The `window_pane_destroy` function deallocates all resources associated with a `window_pane` structure.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure that is to be destroyed.
- **Control Flow**:
    - Calls [`window_pane_reset_mode_all`](#window_pane_reset_mode_all) to reset any modes associated with the pane.
    - Frees the `searchstr` string associated with the pane.
    - Checks if the file descriptor `fd` is valid and, if so, removes the pane from the utempter, frees the associated event, and closes the file descriptor.
    - If the input context `ictx` is not NULL, it frees the input context.
    - Frees the status and base screens associated with the pane.
    - Checks if the `pipe_fd` is valid, frees the associated pipe event, and closes the pipe file descriptor.
    - If the resize timer event is initialized, it deletes the event.
    - Iterates through the resize queue and frees each resize request.
    - Removes the pane from the global tree of all window panes.
    - Frees the options associated with the pane, the current working directory, the shell, and the command arguments.
    - Frees the color palette and finally frees the `window_pane` structure itself.
- **Output**:
    - This function does not return a value; it performs cleanup and deallocation of resources associated with the specified `window_pane`.
- **Functions called**:
    - [`window_pane_reset_mode_all`](#window_pane_reset_mode_all)
    - [`input_free`](input.c.driver.md#input_free)
    - [`screen_free`](screen.c.driver.md#screen_free)
    - [`options_free`](options.c.driver.md#options_free)
    - [`cmd_free_argv`](cmd.c.driver.md#cmd_free_argv)
    - [`colour_palette_free`](colour.c.driver.md#colour_palette_free)


---
### window_pane_read_callback<!-- {{#callable:window_pane_read_callback}} -->
The `window_pane_read_callback` function processes input data from a window pane's event buffer and manages communication with connected clients.
- **Inputs**:
    - `bufev`: A pointer to a `bufferevent` structure, which is unused in this function.
    - `data`: A pointer to a `window_pane` structure that contains the state and data of the window pane.
- **Control Flow**:
    - Checks if the `pipe_fd` of the window pane is valid (not -1).
    - Retrieves new data from the event buffer using [`window_pane_get_new_data`](#window_pane_get_new_data).
    - If new data is available, it writes this data to the `pipe_event` and updates the used data offset.
    - Logs the size of the input buffer for debugging purposes.
    - Iterates through all connected clients and writes output to those that are in control mode.
    - Calls [`input_parse_pane`](input.c.driver.md#input_parse_pane) to process the input data from the event buffer.
    - Disables further read events on the window pane's event.
- **Output**:
    - The function does not return a value; it performs actions such as writing data to a pipe, logging, and updating the state of the window pane.
- **Functions called**:
    - [`window_pane_get_new_data`](#window_pane_get_new_data)
    - [`window_pane_update_used_data`](#window_pane_update_used_data)
    - [`control_write_output`](control.c.driver.md#control_write_output)
    - [`input_parse_pane`](input.c.driver.md#input_parse_pane)


---
### window_pane_error_callback<!-- {{#callable:window_pane_error_callback}} -->
Handles error events for a window pane and initiates its destruction if ready.
- **Inputs**:
    - `bufev`: A pointer to a `bufferevent` structure, which is unused in this function.
    - `what`: A short value indicating the type of error that occurred, which is also unused.
    - `data`: A pointer to user-defined data, expected to be a pointer to a `window_pane` structure.
- **Control Flow**:
    - Logs the error associated with the `window_pane` identified by its ID.
    - Sets the `PANE_EXITED` flag in the `window_pane` structure to indicate that the pane has exited.
    - Checks if the `window_pane` is ready to be destroyed using [`window_pane_destroy_ready`](#window_pane_destroy_ready).
    - If the pane is ready for destruction, calls [`server_destroy_pane`](server-fn.c.driver.md#server_destroy_pane) to handle the destruction process.
- **Output**:
    - No return value; the function performs actions based on the state of the `window_pane`.
- **Functions called**:
    - [`window_pane_destroy_ready`](#window_pane_destroy_ready)
    - [`server_destroy_pane`](server-fn.c.driver.md#server_destroy_pane)


---
### window_pane_set_event<!-- {{#callable:window_pane_set_event}} -->
Sets up event handling for a window pane.
- **Inputs**:
    - `wp`: A pointer to a `window_pane` structure that represents the pane to set the event for.
- **Control Flow**:
    - Calls [`setblocking`](tmux.c.driver.md#setblocking) to set the file descriptor of the pane to non-blocking mode.
    - Creates a new buffered event using `bufferevent_new`, passing the file descriptor, a read callback, an error callback, and the pane structure.
    - Checks if the event creation was successful; if not, it calls `fatalx` to terminate the program with an error message.
    - Initializes the input context for the pane using [`input_init`](input.c.driver.md#input_init).
    - Enables reading and writing on the buffered event using `bufferevent_enable`.
- **Output**:
    - The function does not return a value; it sets up the event handling for the specified window pane.
- **Functions called**:
    - [`setblocking`](tmux.c.driver.md#setblocking)
    - [`input_init`](input.c.driver.md#input_init)


---
### window_pane_resize<!-- {{#callable:window_pane_resize}} -->
The `window_pane_resize` function adjusts the size of a window pane and manages the associated resize events.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure that represents the pane to be resized.
    - `sx`: An unsigned integer representing the new width of the pane.
    - `sy`: An unsigned integer representing the new height of the pane.
- **Control Flow**:
    - The function first checks if the new size is the same as the current size; if so, it returns immediately without making changes.
    - If the size is different, it allocates memory for a new `window_pane_resize` structure and populates it with the new size and the old size.
    - The new resize request is added to the pane's resize queue.
    - The pane's size attributes are updated to the new dimensions.
    - A debug log entry is created to record the resize action.
    - The [`screen_resize`](screen.c.driver.md#screen_resize) function is called to update the screen representation of the pane with the new dimensions.
    - If there are any active modes associated with the pane, the resize function for that mode is called, if it exists.
- **Output**:
    - The function does not return a value; it modifies the state of the `window_pane` and its associated structures.
- **Functions called**:
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`screen_resize`](screen.c.driver.md#screen_resize)


---
### window_pane_set_mode<!-- {{#callable:window_pane_set_mode}} -->
Sets the mode for a specified window pane, managing its mode stack and updating the display accordingly.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane to set the mode for.
    - `swp`: A pointer to another `window_pane` structure, potentially used for mode context.
    - `mode`: A pointer to a `window_mode` structure that defines the new mode to be set.
    - `fs`: A pointer to a `cmd_find_state` structure that holds the state of the command being executed.
    - `args`: A pointer to an `args` structure containing additional arguments for the mode.
- **Control Flow**:
    - Checks if the current mode is already the desired mode; if so, returns 1.
    - Iterates through the list of modes in the pane to find if the desired mode is already present.
    - If the mode is found, it is moved to the head of the mode list.
    - If the mode is not found, a new `window_mode_entry` is created, initialized, and added to the head of the mode list.
    - Updates the pane's screen to the new mode's screen.
    - Sets flags to indicate that the pane needs to be redrawn and updates the layout.
    - Redraws the window borders and updates the status of the window.
    - Notifies that the pane's mode has changed.
- **Output**:
    - Returns 0 on success, or 1 if the mode was already set.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`layout_fix_panes`](layout.c.driver.md#layout_fix_panes)
    - [`server_redraw_window_borders`](server-fn.c.driver.md#server_redraw_window_borders)
    - [`server_status_window`](server-fn.c.driver.md#server_status_window)
    - [`notify_pane`](notify.c.driver.md#notify_pane)


---
### window_pane_reset_mode<!-- {{#callable:window_pane_reset_mode}} -->
Resets the current mode of a window pane and updates its state.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure that represents the pane whose mode is to be reset.
- **Control Flow**:
    - Checks if the `modes` list of the `window_pane` is empty; if so, it returns immediately.
    - Retrieves the first mode entry from the `modes` list and removes it.
    - Calls the `free` function of the mode associated with the removed entry and frees the entry itself.
    - Checks if there are any remaining modes; if none, it clears the `PANE_UNSEENCHANGES` flag, logs a debug message, and sets the screen to the base screen.
    - If there is a next mode, it updates the pane's screen to that mode's screen and calls its resize function if it exists.
    - Sets the pane's flags to indicate that it needs to be redrawn and updates the layout of the panes.
    - Redraws the window borders and updates the status of the window.
    - Notifies that the pane's mode has changed.
- **Output**:
    - The function does not return a value; it modifies the state of the `window_pane` and its associated window.
- **Functions called**:
    - [`layout_fix_panes`](layout.c.driver.md#layout_fix_panes)
    - [`server_redraw_window_borders`](server-fn.c.driver.md#server_redraw_window_borders)
    - [`server_status_window`](server-fn.c.driver.md#server_status_window)
    - [`notify_pane`](notify.c.driver.md#notify_pane)


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
The `window_pane_copy_paste` function synchronizes the paste buffer content across all visible panes in a window that have synchronization enabled.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the current pane from which the paste operation is initiated.
    - `buf`: A pointer to a character buffer containing the data to be pasted.
    - `len`: The size of the data in the buffer to be pasted.
- **Control Flow**:
    - Iterates over each pane in the window using `TAILQ_FOREACH`.
    - Checks if the current pane is not the source pane (`loop != wp`).
    - Ensures the pane is empty of modes (`TAILQ_EMPTY(&loop->modes)`), is open (`loop->fd != -1`), and is not in input-off state (`~loop->flags & PANE_INPUTOFF`).
    - Verifies that the pane is visible (`window_pane_visible(loop)`) and that synchronization is enabled for the pane (`options_get_number(loop->options, 'synchronize-panes')`).
    - If all conditions are met, logs the paste action and writes the buffer content to the pane's event using `bufferevent_write`.
- **Output**:
    - The function does not return a value; it performs side effects by writing the buffer content to other panes.
- **Functions called**:
    - [`window_pane_visible`](#window_pane_visible)
    - [`options_get_number`](options.c.driver.md#options_get_number)


---
### window_pane_copy_key<!-- {{#callable:window_pane_copy_key}} -->
The `window_pane_copy_key` function synchronizes key input across all visible panes in a window, except for the pane that originated the key event.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane from which the key event originated.
    - `key`: The key code representing the key that was pressed.
- **Control Flow**:
    - Iterates through each pane in the window using `TAILQ_FOREACH`.
    - Checks if the current pane is not the same as the originating pane (`wp`).
    - Ensures the current pane is not in any modes, is not closed (fd != -1), is not input-off, is visible, and has synchronization enabled.
    - If all conditions are met, it sends the key input to the current pane using [`input_key_pane`](input-keys.c.driver.md#input_key_pane).
- **Output**:
    - The function does not return a value; it sends key input to other panes as needed.
- **Functions called**:
    - [`window_pane_visible`](#window_pane_visible)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`input_key_pane`](input-keys.c.driver.md#input_key_pane)


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
    - Next, it checks if the key pressed is a paste key and if the pane is not in bracket paste mode; if both conditions are true, it returns.
    - If all checks pass, it logs the paste operation and writes the buffer content to the pane's event.
    - Finally, if the 'synchronize-panes' option is enabled, it calls [`window_pane_copy_paste`](#window_pane_copy_paste) to paste the content into other panes.
- **Output**:
    - The function does not return a value; it performs actions based on the input conditions, primarily writing the pasted content to the pane's event and potentially synchronizing it with other panes.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
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
    - Processes the key input through the [`input_key_pane`](input-keys.c.driver.md#input_key_pane) function, returning -1 if it fails.
    - If the key is a mouse event, it returns 0.
    - If the 'synchronize-panes' option is enabled, it copies the key event to other panes.
- **Output**:
    - Returns 0 on success, -1 if an error occurs, or if the key event is not processed.
- **Functions called**:
    - [`input_key_pane`](input-keys.c.driver.md#input_key_pane)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`window_pane_copy_key`](#window_pane_copy_key)


---
### window_pane_visible<!-- {{#callable:window_pane_visible}} -->
Determines if a given `window_pane` is visible based on its zoom state and whether it is the active pane.
- **Inputs**:
    - `wp`: A pointer to a `struct window_pane` which represents the pane whose visibility is being checked.
- **Control Flow**:
    - The function first checks if the `WINDOW_ZOOMED` flag is not set for the window associated with the pane.
    - If the window is not zoomed, it returns 1, indicating the pane is visible.
    - If the window is zoomed, it checks if the pane is the active pane of the window and returns the result of that comparison.
- **Output**:
    - Returns 1 if the pane is visible, otherwise returns 0.


---
### window_pane_exited<!-- {{#callable:window_pane_exited}} -->
The `window_pane_exited` function checks if a window pane has exited based on its file descriptor and exit flag.
- **Inputs**:
    - `wp`: A pointer to a `struct window_pane` representing the pane to check.
- **Control Flow**:
    - The function evaluates the file descriptor (`fd`) of the pane; if it is -1, it indicates that the pane is not active.
    - It also checks if the `flags` of the pane include the `PANE_EXITED` flag, which indicates that the pane has exited.
- **Output**:
    - The function returns an integer value: 1 if the pane has exited (either due to a -1 file descriptor or the exit flag being set), and 0 otherwise.


---
### window_pane_search<!-- {{#callable:window_pane_search}} -->
The `window_pane_search` function searches for a specified term in the text of a window pane, optionally using regular expressions and case sensitivity.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane in which to search.
    - `term`: A string containing the term to search for within the pane.
    - `regex`: An integer flag indicating whether to treat the search term as a regular expression (non-zero) or a simple string (zero).
    - `ignore`: An integer flag indicating whether to ignore case when searching (non-zero) or not (zero).
- **Control Flow**:
    - If `regex` is false, the function constructs a wildcard pattern from `term` and sets the appropriate flags based on `ignore`.
    - If `regex` is true, it compiles the regular expression using `regcomp` with the appropriate flags.
    - The function iterates over each line of the pane's text, trimming trailing whitespace and checking for matches against the search term or regex.
    - If a match is found, the loop breaks, and the function prepares to return the result.
    - After searching all lines, the function cleans up by freeing any allocated memory and returning the result.
- **Output**:
    - Returns the line number (1-based) where the term was found, or 0 if the term was not found in the pane.
- **Functions called**:
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`grid_view_string_cells`](grid-view.c.driver.md#grid_view_string_cells)


---
### window_pane_choose_best<!-- {{#callable:window_pane_choose_best}} -->
The `window_pane_choose_best` function selects the `window_pane` with the highest `active_point` from a given list.
- **Inputs**:
    - `list`: An array of pointers to `window_pane` structures.
    - `size`: The number of elements in the `list` array.
- **Control Flow**:
    - If the `size` is zero, the function returns NULL immediately.
    - The first `window_pane` in the list is assumed to be the best initially.
    - A loop iterates through the remaining panes in the list, comparing their `active_point` values.
    - If a pane with a higher `active_point` is found, it becomes the new best candidate.
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
    - Retrieve the scrollbar options from the `window` associated with the `window_pane`.
    - Determine if the scrollbar should be displayed and calculate its width accordingly.
    - Check the position of the scrollbar (left or right) and adjust the x-offset and width of the pane based on this position.
    - Set the y-offset and height of the pane directly from the `window_pane` structure.
- **Output**:
    - The function does not return a value; instead, it modifies the values pointed to by the input pointers to reflect the calculated offsets and sizes.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`window_pane_show_scrollbar`](#window_pane_show_scrollbar)


---
### window_pane_find_up<!-- {{#callable:window_pane_find_up}} -->
Finds the pane directly above the specified window pane.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the current pane.
- **Control Flow**:
    - Checks if the input pane `wp` is NULL and returns NULL if it is.
    - Retrieves the associated `window` structure from the input pane.
    - Gets the pane border status from the window options.
    - Calculates the full size and offset of the input pane.
    - Determines the edge position based on the pane border status.
    - Iterates through all panes in the window to find candidates above the input pane.
    - For each candidate pane, checks if it is adjacent to the edge of the input pane.
    - If a candidate pane is found, it is added to a list.
    - Chooses the best candidate pane from the list based on the active point.
    - Frees the list of candidate panes and returns the best pane found.
- **Output**:
    - Returns a pointer to the `window_pane` structure of the pane directly above the input pane, or NULL if no such pane exists.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`window_pane_full_size_offset`](#window_pane_full_size_offset)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`window_pane_choose_best`](#window_pane_choose_best)


---
### window_pane_find_down<!-- {{#callable:window_pane_find_down}} -->
Finds the pane directly below a given window pane.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the current pane.
- **Control Flow**:
    - Checks if the input pane `wp` is NULL and returns NULL if it is.
    - Retrieves the associated `window` structure from the input pane.
    - Gets the pane border status option to determine how to handle the edge case.
    - Calculates the edge position based on the current pane's offset and size.
    - Iterates through all panes in the window to find candidates that are directly below the current pane.
    - For each candidate pane, checks if it overlaps with the current pane's horizontal range.
    - If a candidate pane is found, it is added to a list.
    - After iterating, selects the best candidate pane from the list based on the active point.
    - Frees the list of candidate panes and returns the best candidate.
- **Output**:
    - Returns a pointer to the `window_pane` structure that is directly below the input pane, or NULL if no such pane exists.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`window_pane_full_size_offset`](#window_pane_full_size_offset)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`window_pane_choose_best`](#window_pane_choose_best)


---
### window_pane_find_left<!-- {{#callable:window_pane_find_left}} -->
The `window_pane_find_left` function finds the pane directly to the left of a given window pane.
- **Inputs**:
    - `wp`: A pointer to the current `window_pane` structure for which the left neighbor is to be found.
- **Control Flow**:
    - Checks if the input `wp` is NULL and returns NULL if it is.
    - Retrieves the associated `window` structure from the `window_pane`.
    - Initializes variables to store the full size and offset of the current pane.
    - Iterates through all panes in the window using a loop.
    - For each pane, checks if it is not the same as the input pane and if it is positioned directly to the left.
    - If a left neighbor is found, it checks if it overlaps vertically with the current pane.
    - Stores valid left neighbors in a dynamically allocated list.
    - After checking all panes, calls [`window_pane_choose_best`](#window_pane_choose_best) to select the most suitable left pane from the list.
    - Frees the list of found panes and returns the best left pane.
- **Output**:
    - Returns a pointer to the best `window_pane` found to the left of the input pane, or NULL if no such pane exists.
- **Functions called**:
    - [`window_pane_full_size_offset`](#window_pane_full_size_offset)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`window_pane_choose_best`](#window_pane_choose_best)


---
### window_pane_find_right<!-- {{#callable:window_pane_find_right}} -->
The `window_pane_find_right` function locates the window pane immediately to the right of a specified pane.
- **Inputs**:
    - `wp`: A pointer to the current `window_pane` structure from which the search for the right pane will begin.
- **Control Flow**:
    - If the input pane `wp` is NULL, the function returns NULL immediately.
    - The function retrieves the associated `window` structure from the input pane.
    - It initializes variables to store the edge position and dimensions of the current pane.
    - The function iterates over all panes in the window using a loop.
    - For each pane, it checks if it is the same as the input pane and skips it if so.
    - It checks if the current pane is positioned directly to the right of the input pane based on their offsets.
    - If a pane is found to be adjacent, it checks if it overlaps vertically with the input pane.
    - If it does, the pane is added to a list of potential candidates.
    - After iterating through all panes, the function selects the best candidate from the list based on a predefined criterion.
    - Finally, it frees the list of candidates and returns the best pane found.
- **Output**:
    - The function returns a pointer to the `window_pane` structure that is immediately to the right of the specified pane, or NULL if no such pane exists.
- **Functions called**:
    - [`window_pane_full_size_offset`](#window_pane_full_size_offset)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`window_pane_choose_best`](#window_pane_choose_best)


---
### window_pane_stack_push<!-- {{#callable:window_pane_stack_push}} -->
Pushes a `window_pane` onto the top of a `window_panes` stack.
- **Inputs**:
    - `stack`: A pointer to a `window_panes` structure representing the stack to which the pane will be added.
    - `wp`: A pointer to a `window_pane` structure that is to be pushed onto the stack.
- **Control Flow**:
    - Checks if the `window_pane` pointer `wp` is not NULL.
    - Calls [`window_pane_stack_remove`](#window_pane_stack_remove) to remove `wp` from the stack if it is already present.
    - Inserts `wp` at the head of the `stack` using `TAILQ_INSERT_HEAD`.
    - Sets the `PANE_VISITED` flag for the `window_pane`.
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
Clears alert flags for a given `winlink` and updates the status of associated sessions.
- **Inputs**:
    - `wl`: A pointer to a `winlink` structure whose alert flags are to be cleared.
- **Control Flow**:
    - The function first clears the `WINDOW_ALERTFLAGS` from the `window` associated with the provided `winlink`.
    - It then iterates over all `winlinks` associated with the same `window` using `TAILQ_FOREACH`.
    - For each `winlink`, if its `flags` contain `WINLINK_ALERTFLAGS`, it clears this flag and calls [`server_status_session`](server-fn.c.driver.md#server_status_session) with the associated `session`.
- **Output**:
    - The function does not return a value; it modifies the state of the `winlink` and its associated `window` and `session`.
- **Functions called**:
    - [`server_status_session`](server-fn.c.driver.md#server_status_session)


---
### winlink_shuffle_up<!-- {{#callable:winlink_shuffle_up}} -->
The `winlink_shuffle_up` function adjusts the index of a specified window link within a session, moving it up in the order of window links.
- **Inputs**:
    - `s`: A pointer to the `session` structure that contains the list of window links.
    - `wl`: A pointer to the `winlink` structure that represents the window link to be shuffled.
    - `before`: An integer flag indicating whether to shuffle the window link before its current index (1) or after (0).
- **Control Flow**:
    - The function first checks if the `wl` pointer is NULL, returning -1 if it is.
    - It determines the current index of the window link based on the `before` flag.
    - A loop is used to find the next available index greater than or equal to the current index.
    - If no available index is found (i.e., it reaches INT_MAX), the function returns -1.
    - Another loop moves all window links from the found index down to the current index, incrementing their indices.
    - Finally, the function returns the original index of the window link.
- **Output**:
    - The function returns the new index of the window link if successful, or -1 if an error occurs.
- **Functions called**:
    - [`winlink_find_by_index`](#winlink_find_by_index)


---
### window_pane_input_callback<!-- {{#callable:window_pane_input_callback}} -->
The `window_pane_input_callback` function processes input data for a window pane, handling errors and state changes.
- **Inputs**:
    - `struct client *c`: A pointer to the `client` structure representing the client that initiated the input.
    - `__unused const char *path`: An unused parameter that may represent the path from which the input is received.
    - `int error`: An integer indicating whether an error occurred during input processing.
    - `int closed`: An integer flag indicating if the input stream has been closed.
    - `struct evbuffer *buffer`: A pointer to an `evbuffer` structure containing the input data.
    - `void *data`: A pointer to user-defined data, specifically a `window_pane_input_data` structure.
- **Control Flow**:
    - The function retrieves the `window_pane` associated with the input data using [`window_pane_find_by_id`](#window_pane_find_by_id).
    - If the `file` in `cdata` is not NULL and the `window_pane` is NULL or the client is dead, it sets the client's return value and flags the client for exit.
    - If the `file` is NULL or if the input stream is closed or an error occurred, it continues the command queue, unreferences the client, and frees the input data.
    - If there are no issues, it parses the input buffer using [`input_parse_buffer`](input.c.driver.md#input_parse_buffer).
    - Finally, it drains the processed data from the `evbuffer`.
- **Output**:
    - The function does not return a value; it modifies the state of the client and the window pane based on the input received.
- **Functions called**:
    - [`window_pane_find_by_id`](#window_pane_find_by_id)
    - [`file_cancel`](file.c.driver.md#file_cancel)
    - [`cmdq_continue`](cmd-queue.c.driver.md#cmdq_continue)
    - [`server_client_unref`](server-client.c.driver.md#server_client_unref)
    - [`input_parse_buffer`](input.c.driver.md#input_parse_buffer)


---
### window_pane_start_input<!-- {{#callable:window_pane_start_input}} -->
Initializes input handling for a specified window pane.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane to start input for.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item associated with the input.
    - `cause`: A pointer to a character pointer that will hold an error message if the function fails.
- **Control Flow**:
    - Checks if the pane is empty by evaluating the `PANE_EMPTY` flag; if not, sets an error message and returns -1.
    - Checks if the client associated with the command queue item is dead or exited; if so, returns 1.
    - Checks if the client has an active session; if so, returns 1.
    - Allocates memory for `window_pane_input_data`, initializes it with the command queue item and pane ID, and starts reading input from the client using [`file_read`](file.c.driver.md#file_read).
    - Increments the reference count for the client and returns 0 to indicate success.
- **Output**:
    - Returns 0 on success, -1 if the pane is not empty or if an error occurs, and 1 if the client is dead, exited, or has an active session.
- **Functions called**:
    - [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`file_read`](file.c.driver.md#file_read)


---
### window_pane_get_new_data<!-- {{#callable:window_pane_get_new_data}} -->
Retrieves new data from a `window_pane`'s input buffer and updates the size of the available data.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure from which data is being retrieved.
    - `wpo`: A pointer to a `window_pane_offset` structure that tracks the used offset in the input buffer.
    - `size`: A pointer to a size_t variable where the size of the new data will be stored.
- **Control Flow**:
    - Calculates the amount of data already used by subtracting the base offset from the used offset in `wpo`.
    - Determines the size of the new data available in the input buffer by subtracting the used data from the total length of the input buffer.
    - Returns a pointer to the start of the new data in the input buffer by adding the used offset to the pointer to the input buffer.
- **Output**:
    - Returns a pointer to the new data available in the input buffer of the `window_pane`.


---
### window_pane_update_used_data<!-- {{#callable:window_pane_update_used_data}} -->
Updates the used data offset for a window pane based on the input buffer size.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane whose used data is being updated.
    - `wpo`: A pointer to the `window_pane_offset` structure that holds the current used offset for the pane.
    - `size`: A size_t value representing the amount of data to be added to the used offset.
- **Control Flow**:
    - Calculates the current used offset by subtracting the base offset from the used offset in `wpo`.
    - Checks if the provided size exceeds the available space in the input buffer of the pane's event; if so, it adjusts the size to fit.
    - Increments the used offset in `wpo` by the adjusted size.
- **Output**:
    - The function does not return a value; it modifies the `used` field in the `window_pane_offset` structure directly.


---
### window_set_fill_character<!-- {{#callable:window_set_fill_character}} -->
Sets the fill character for a window based on its options.
- **Inputs**:
    - `w`: A pointer to a `struct window` that represents the window for which the fill character is being set.
- **Control Flow**:
    - Free the current fill character if it exists.
    - Retrieve the fill character string from the window's options.
    - Check if the retrieved string is not empty and is a valid UTF-8 character.
    - Convert the valid UTF-8 string to a `struct utf8_data`.
    - If the conversion is successful and the width of the character is 1, assign it to the window's fill character; otherwise, free the allocated memory.
- **Output**:
    - The function does not return a value; it modifies the `fill_character` field of the `struct window`.
- **Functions called**:
    - [`options_get_string`](options.c.driver.md#options_get_string)
    - [`utf8_isvalid`](utf8.c.driver.md#utf8_isvalid)
    - [`utf8_fromcstr`](utf8.c.driver.md#utf8_fromcstr)


---
### window_pane_default_cursor<!-- {{#callable:window_pane_default_cursor}} -->
Sets the default cursor for a given window pane.
- **Inputs**:
    - `wp`: A pointer to a `window_pane` structure representing the pane for which the default cursor is to be set.
- **Control Flow**:
    - Calls the [`screen_set_default_cursor`](screen.c.driver.md#screen_set_default_cursor) function with the screen associated with the window pane and its options.
- **Output**:
    - This function does not return a value; it modifies the state of the window pane's screen to set the default cursor.
- **Functions called**:
    - [`screen_set_default_cursor`](screen.c.driver.md#screen_set_default_cursor)


---
### window_pane_mode<!-- {{#callable:window_pane_mode}} -->
The `window_pane_mode` function determines the current mode of a given window pane.
- **Inputs**:
    - `wp`: A pointer to a `struct window_pane` representing the window pane whose mode is to be determined.
- **Control Flow**:
    - The function first checks if the `modes` list of the window pane is not empty.
    - If the first mode in the list is `window_copy_mode`, it returns `WINDOW_PANE_COPY_MODE`.
    - If the first mode in the list is `window_view_mode`, it returns `WINDOW_PANE_VIEW_MODE`.
    - If no modes match, it returns `WINDOW_PANE_NO_MODE`.
- **Output**:
    - The function returns an integer indicating the current mode of the window pane: `WINDOW_PANE_COPY_MODE`, `WINDOW_PANE_VIEW_MODE`, or `WINDOW_PANE_NO_MODE`.


---
### window_pane_show_scrollbar<!-- {{#callable:window_pane_show_scrollbar}} -->
The `window_pane_show_scrollbar` function determines whether a scrollbar should be displayed for a given window pane based on its mode and the specified scrollbar option.
- **Inputs**:
    - `wp`: A pointer to a `struct window_pane` representing the pane for which the scrollbar visibility is being determined.
    - `sb_option`: An integer representing the scrollbar display option, which can dictate whether the scrollbar is always shown, shown in modal mode, or not shown.
- **Control Flow**:
    - The function first checks if the pane is in an alternate screen mode using `SCREEN_IS_ALTERNATE`, returning 0 (no scrollbar) if true.
    - Next, it evaluates the `sb_option` against two conditions: if it is set to `PANE_SCROLLBARS_ALWAYS`, or if it is set to `PANE_SCROLLBARS_MODAL` and the pane is not in `WINDOW_PANE_NO_MODE`.
    - If either condition is met, the function returns 1 (indicating that a scrollbar should be shown); otherwise, it returns 0.
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
    - If the control client returns -1, it initializes a `grid_cell` structure with default colors using [`tty_default_colours`](tty.c.driver.md#tty_default_colours).
    - Checks if the default background color is set; if so, it retrieves the background color from the client using [`window_get_bg_client`](#window_get_bg_client).
    - If the default background color is not set, it uses the background color from the `defaults` structure.
- **Output**:
    - Returns an integer representing the background color; returns -1 if no valid background color is found.
- **Functions called**:
    - [`window_pane_get_bg_control_client`](#window_pane_get_bg_control_client)
    - [`tty_default_colours`](tty.c.driver.md#tty_default_colours)
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
    - Returns an integer representing the background color of the first attached client associated with the window pane, or -1 if no such client exists.
- **Functions called**:
    - [`session_has`](session.c.driver.md#session_has)


---
### window_pane_get_bg_control_client<!-- {{#callable:window_pane_get_bg_control_client}} -->
Retrieves the background control client ID for a given window pane.
- **Inputs**:
    - `wp`: A pointer to a `window_pane` structure representing the pane for which the background control client ID is to be retrieved.
- **Control Flow**:
    - Checks if the `control_bg` field of the `window_pane` is set to -1, returning -1 if true.
    - Iterates through the list of clients using `TAILQ_FOREACH` to check if any client has the `CLIENT_CONTROL` flag set.
    - If a client with the `CLIENT_CONTROL` flag is found, returns the `control_bg` value of the `window_pane`.
    - If no such client is found, returns -1.
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
    - For each client, it checks if the client is attached (not having `CLIENT_UNATTACHEDFLAGS`), if it has an associated session, and if the session is linked to the window.
    - If the client's foreground color (`tty.fg`) is valid (not -1), it returns that color.
    - If no valid foreground color is found after checking all clients, the function returns -1.
- **Output**:
    - The function returns an integer representing the foreground color of the first valid attached client, or -1 if no valid client is found.
- **Functions called**:
    - [`session_has`](session.c.driver.md#session_has)


---
### window_pane_get_fg_control_client<!-- {{#callable:window_pane_get_fg_control_client}} -->
Retrieves the foreground control client ID for a given window pane.
- **Inputs**:
    - `wp`: A pointer to a `window_pane` structure representing the pane for which the foreground control client ID is to be retrieved.
- **Control Flow**:
    - Checks if the `control_fg` field of the `window_pane` structure is set to -1, indicating no valid foreground control client; if so, returns -1.
    - Iterates through the global list of clients using `TAILQ_FOREACH` to check if any client has the `CLIENT_CONTROL` flag set.
    - If a client with the `CLIENT_CONTROL` flag is found, returns the `control_fg` value of the `window_pane`.
    - If no such client is found, returns -1.
- **Output**:
    - Returns the foreground control client ID associated with the window pane if a control client exists; otherwise, returns -1.


---
### window_pane_get_theme<!-- {{#callable:window_pane_get_theme}} -->
The `window_pane_get_theme` function determines the theme of a given window pane based on its background color and associated clients.
- **Inputs**:
    - `wp`: A pointer to a `struct window_pane` representing the pane whose theme is to be determined.
- **Control Flow**:
    - If the input `wp` is NULL, the function returns `THEME_UNKNOWN`.
    - The function retrieves the background color of the pane using [`window_pane_get_bg`](#window_pane_get_bg) and attempts to derive the theme from it.
    - If the derived theme is not `THEME_UNKNOWN`, it is returned immediately.
    - The function iterates over all clients to check if any client associated with the window has a defined theme.
    - Flags `found_light` and `found_dark` are set based on the themes found in the clients.
    - Finally, the function returns `THEME_DARK` if only dark themes were found, `THEME_LIGHT` if only light themes were found, or `THEME_UNKNOWN` if neither was found.
- **Output**:
    - The function returns an enumeration of type `enum client_theme`, which can be `THEME_LIGHT`, `THEME_DARK`, or `THEME_UNKNOWN`.
- **Functions called**:
    - [`colour_totheme`](colour.c.driver.md#colour_totheme)
    - [`window_pane_get_bg`](#window_pane_get_bg)
    - [`session_has`](session.c.driver.md#session_has)


---
### window_pane_send_theme_update<!-- {{#callable:window_pane_send_theme_update}} -->
The `window_pane_send_theme_update` function sends a theme update to a window pane if the theme has changed and updates are allowed.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane to which the theme update will be sent.
- **Control Flow**:
    - The function first checks if the `PANE_THEMECHANGED` flag is not set for the pane; if so, it returns immediately without doing anything.
    - Next, it checks if the screen mode allows theme updates by verifying that `MODE_THEME_UPDATES` is set; if not, it returns.
    - The function then retrieves the current theme of the pane using `window_pane_get_theme(wp)` and sends a corresponding input key event based on the theme: `KEYC_REPORT_LIGHT_THEME` for light theme, `KEYC_REPORT_DARK_THEME` for dark theme.
    - If the theme is unknown, no action is taken.
    - Finally, the function clears the `PANE_THEMECHANGED` flag to indicate that the theme update has been processed.
- **Output**:
    - The function does not return a value; it modifies the state of the `window_pane` and sends input events based on the theme.
- **Functions called**:
    - [`window_pane_get_theme`](#window_pane_get_theme)
    - [`input_key_pane`](input-keys.c.driver.md#input_key_pane)


