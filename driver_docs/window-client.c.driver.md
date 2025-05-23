# Purpose
This C source code file is part of the tmux terminal multiplexer project and implements a specific window mode called "client-mode." The primary purpose of this code is to manage and display a list of clients connected to a tmux session, allowing users to interact with these clients through a menu-driven interface. The code defines several static functions for initializing, freeing, resizing, updating, and handling key events within this mode. It also includes functionality for sorting clients based on various criteria such as name, size, creation time, and activity time. The code provides a structured way to manage client interactions, including detaching clients or executing commands on them.

The file defines a set of menu items and key bindings that allow users to perform actions like detaching clients or tagging them for batch operations. It uses a mode tree structure to organize and display client data, with functions to build, draw, and update the display as needed. The code is designed to be integrated into the larger tmux application, with functions that interact with other parts of the system, such as the server and client management components. This file does not define a standalone executable but rather a module that contributes to the overall functionality of tmux, specifically focusing on client management within a window pane.
# Imports and Dependencies

---
- `sys/types.h`
- `sys/time.h`
- `stdlib.h`
- `string.h`
- `time.h`
- `tmux.h`


# Global Variables

---
### window_client_init
- **Type**: `function pointer`
- **Description**: `window_client_init` is a static function pointer that initializes a screen structure for a window mode entry in the tmux client mode. It takes three parameters: a pointer to a `window_mode_entry`, a pointer to a `cmd_find_state`, and a pointer to an `args` structure.
- **Use**: This function is used to set up the initial state and data structures required for the client mode in a tmux window, including setting up format strings and command strings.


---
### window_client_menu_items
- **Type**: ``static const struct menu_item[]``
- **Description**: The `window_client_menu_items` is a static constant array of `menu_item` structures, each representing a menu option for a client window in the application. Each menu item consists of a label, a key code, and a NULL pointer, which likely serves as a placeholder for additional data or a callback function.
- **Use**: This array is used to define the menu options available to the user in the client window mode, allowing for actions such as detaching clients or tagging them.


---
### window_client_mode
- **Type**: `const struct window_mode`
- **Description**: The `window_client_mode` is a constant structure of type `window_mode` that defines the behavior and properties of a specific mode called 'client-mode' in the application. It includes a name, a default format for displaying information, and function pointers for initializing, freeing, resizing, updating, and handling key events in this mode.
- **Use**: This variable is used to manage and define the operations and display format for the 'client-mode' within the application, allowing for consistent handling of client-related interactions.


---
### window_client_sort_list
- **Type**: `const char *[]`
- **Description**: `window_client_sort_list` is a static constant array of strings that defines the sorting criteria for window clients. It contains four elements: "name", "size", "creation", and "activity". These strings represent the different attributes by which window clients can be sorted.
- **Use**: This array is used to provide a list of sorting options for window clients in the application.


---
### window_client_sort
- **Type**: `struct mode_tree_sort_criteria*`
- **Description**: The `window_client_sort` is a static pointer to a `struct mode_tree_sort_criteria`, which is used to define the sorting criteria for window clients in a mode tree. This structure likely contains fields that specify how clients should be sorted, such as by name, size, creation time, or activity time.
- **Use**: This variable is used to determine the sorting order of window clients when building and displaying the mode tree in the client mode.


# Data Structures

---
### window_client_sort_type
- **Type**: `enum`
- **Members**:
    - `WINDOW_CLIENT_BY_NAME`: Sorts window clients by their name.
    - `WINDOW_CLIENT_BY_SIZE`: Sorts window clients by their size.
    - `WINDOW_CLIENT_BY_CREATION_TIME`: Sorts window clients by their creation time.
    - `WINDOW_CLIENT_BY_ACTIVITY_TIME`: Sorts window clients by their activity time.
- **Description**: The `window_client_sort_type` is an enumeration that defines the criteria for sorting window clients in a user interface. It provides four sorting options: by name, by size, by creation time, and by activity time, allowing for flexible organization of client windows based on user preference or application requirements.


---
### window_client_itemdata
- **Type**: `struct`
- **Members**:
    - `c`: A pointer to a struct client, representing a client associated with the item data.
- **Description**: The `window_client_itemdata` structure is a simple data structure that encapsulates a single pointer to a `client` structure. It is used to associate a client with an item in the context of managing window client data within the application. This structure is part of a larger system that handles client interactions and operations in a windowed environment, likely within a terminal multiplexer or similar application.


---
### window_client_modedata
- **Type**: `struct`
- **Members**:
    - `wp`: A pointer to a window_pane structure, representing the pane associated with this mode data.
    - `data`: A pointer to a mode_tree_data structure, used for managing the mode tree.
    - `format`: A string representing the format for displaying client information.
    - `key_format`: A string representing the format for key bindings.
    - `command`: A string representing the command to be executed.
    - `item_list`: A pointer to an array of window_client_itemdata structures, representing the list of client items.
    - `item_size`: An unsigned integer representing the size of the item_list array.
- **Description**: The `window_client_modedata` structure is used to manage the state and data associated with a client mode in a window pane. It includes pointers to related structures for managing the mode tree and client items, as well as strings for formatting display and key bindings. The structure also maintains a list of client items and its size, allowing for dynamic management of client-related data within a window pane.


# Functions

---
### window_client_add_item<!-- {{#callable:window_client_add_item}} -->
The function `window_client_add_item` adds a new item to the list of client items in a `window_client_modedata` structure, reallocating memory as necessary.
- **Inputs**:
    - `data`: A pointer to a `window_client_modedata` structure, which contains the list of client items and its current size.
- **Control Flow**:
    - Reallocate memory for the `item_list` array in the `data` structure to accommodate one more item, increasing the size by one.
    - Allocate memory for a new `window_client_itemdata` structure and initialize it to zero.
    - Assign the newly allocated `window_client_itemdata` to the next position in the `item_list` array and increment the `item_size`.
    - Return the pointer to the newly added `window_client_itemdata`.
- **Output**:
    - A pointer to the newly added `window_client_itemdata` structure.
- **Functions called**:
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)


---
### window_client_free_item<!-- {{#callable:window_client_free_item}} -->
The `window_client_free_item` function releases the resources associated with a `window_client_itemdata` structure by decrementing the reference count of the client and freeing the memory allocated for the item.
- **Inputs**:
    - `item`: A pointer to a `window_client_itemdata` structure that contains a client reference to be freed.
- **Control Flow**:
    - Call [`server_client_unref`](server-client.c.driver.md#server_client_unref) with the client reference from the `item` to decrement its reference count.
    - Free the memory allocated for the `item` using `free`.
- **Output**:
    - This function does not return any value; it performs cleanup operations on the provided `window_client_itemdata` structure.
- **Functions called**:
    - [`server_client_unref`](server-client.c.driver.md#server_client_unref)


---
### window_client_cmp<!-- {{#callable:window_client_cmp}} -->
The `window_client_cmp` function compares two `window_client_itemdata` structures based on specified sorting criteria, such as size, creation time, activity time, or name, and returns an integer indicating their relative order.
- **Inputs**:
    - `a0`: A pointer to the first `window_client_itemdata` structure to be compared.
    - `b0`: A pointer to the second `window_client_itemdata` structure to be compared.
- **Control Flow**:
    - The function casts the input pointers `a0` and `b0` to pointers of type `const struct window_client_itemdata *const *` and dereferences them to obtain `itema` and `itemb`.
    - It retrieves the `client` structures `ca` and `cb` from `itema` and `itemb`, respectively.
    - The function initializes a variable `result` to zero, which will hold the comparison result.
    - A switch statement is used to determine the sorting field specified by `window_client_sort->field`.
    - If the field is `WINDOW_CLIENT_BY_SIZE`, it compares the `tty.sx` and `tty.sy` fields of `ca` and `cb` to determine the result.
    - If the field is `WINDOW_CLIENT_BY_CREATION_TIME`, it uses `timercmp` to compare the `creation_time` fields of `ca` and `cb`, setting `result` to -1, 0, or 1 based on the comparison.
    - If the field is `WINDOW_CLIENT_BY_ACTIVITY_TIME`, it similarly compares the `activity_time` fields using `timercmp`.
    - If `result` is still zero after the above comparisons, it defaults to comparing the `name` fields of `ca` and `cb` using `strcmp`.
    - If `window_client_sort->reversed` is true, the function negates `result` to reverse the order.
    - The function returns the final `result` value.
- **Output**:
    - An integer indicating the relative order of the two `window_client_itemdata` structures: negative if the first is less than the second, zero if they are equal, and positive if the first is greater than the second.


---
### window_client_build<!-- {{#callable:window_client_build}} -->
The `window_client_build` function constructs and sorts a list of client items based on specified criteria and applies a filter to format and add them to a mode tree.
- **Inputs**:
    - `modedata`: A pointer to `window_client_modedata` structure which holds data related to the window client mode.
    - `sort_crit`: A pointer to `mode_tree_sort_criteria` structure that specifies the sorting criteria for the client items.
    - `tag`: An unused pointer to a `uint64_t` value, presumably for tagging purposes.
    - `filter`: A constant character pointer representing a filter string to apply to the client items.
- **Control Flow**:
    - Iterates over the existing items in `data->item_list`, freeing each item and then the list itself, resetting `item_list` and `item_size` to zero.
    - Iterates over all clients in the global `clients` list, skipping those without a session or with unattached flags, and adds each valid client to `data->item_list` while incrementing their reference count.
    - Sets the global `window_client_sort` to the provided `sort_crit` and sorts `data->item_list` using `qsort` with `window_client_cmp` as the comparator.
    - Iterates over the sorted `data->item_list`, applying the `filter` if provided, and formats each client item using `data->format`, adding them to the mode tree with [`mode_tree_add`](mode-tree.c.driver.md#mode_tree_add).
- **Output**:
    - The function does not return a value but modifies the `data->item_list` and `data->item_size` in the `window_client_modedata` structure, and updates the mode tree with formatted client items.
- **Functions called**:
    - [`window_client_free_item`](#window_client_free_item)
    - [`window_client_add_item`](#window_client_add_item)
    - [`format_single`](format.c.driver.md#format_single)
    - [`format_true`](format.c.driver.md#format_true)
    - [`mode_tree_add`](mode-tree.c.driver.md#mode_tree_add)


---
### window_client_draw<!-- {{#callable:window_client_draw}} -->
The `window_client_draw` function renders the client window's content on the screen, adjusting for status lines and cursor position.
- **Inputs**:
    - `modedata`: Unused parameter, typically used for mode-specific data.
    - `itemdata`: Pointer to `window_client_itemdata` structure containing client-specific data.
    - `ctx`: Pointer to `screen_write_ctx` structure used for writing to the screen.
    - `sx`: Unsigned integer representing the width of the screen area to draw.
    - `sy`: Unsigned integer representing the height of the screen area to draw.
- **Control Flow**:
    - Check if the client is attached to a session and return if not.
    - Retrieve the active window pane from the client's current session.
    - Calculate the number of status lines and determine their position (top or bottom).
    - Move the cursor to the appropriate position based on the status line position.
    - Draw a preview of the window pane's base content, adjusting for status lines.
    - Draw a horizontal line at the bottom or top of the screen, depending on the status line position.
    - Move the cursor to the original position or adjusted position based on status lines.
    - Copy the status screen content to the screen context, adjusting for status lines.
- **Output**:
    - The function does not return a value; it modifies the screen context to display the client's window content.
- **Functions called**:
    - [`status_line_size`](status.c.driver.md#status_line_size)
    - [`status_at_line`](status.c.driver.md#status_at_line)
    - [`screen_write_cursormove`](screen-write.c.driver.md#screen_write_cursormove)
    - [`screen_write_preview`](screen-write.c.driver.md#screen_write_preview)
    - [`screen_write_hline`](screen-write.c.driver.md#screen_write_hline)
    - [`screen_write_fast_copy`](screen-write.c.driver.md#screen_write_fast_copy)


---
### window_client_menu<!-- {{#callable:window_client_menu}} -->
The `window_client_menu` function processes a key event for a client in a specific window pane mode, delegating the key handling to the [`window_client_key`](#window_client_key) function if the mode data matches.
- **Inputs**:
    - `modedata`: A pointer to the mode data associated with the window pane, specifically a `window_client_modedata` structure.
    - `c`: A pointer to the `client` structure representing the client for which the menu is being processed.
    - `key`: A `key_code` representing the key event that triggered the menu action.
- **Control Flow**:
    - Retrieve the `window_client_modedata` structure from the `modedata` pointer.
    - Access the `window_pane` associated with the mode data.
    - Get the first `window_mode_entry` from the pane's mode queue using `TAILQ_FIRST`.
    - Check if the mode entry is `NULL` or if its data does not match the provided `modedata`; if so, return immediately.
    - Call [`window_client_key`](#window_client_key) with the mode entry, client, and key to handle the key event.
- **Output**:
    - This function does not return a value; it performs actions based on the key event and the current mode state.
- **Functions called**:
    - [`window_client_key`](#window_client_key)


---
### window_client_get_key<!-- {{#callable:window_client_get_key}} -->
The `window_client_get_key` function generates a key code based on a formatted string derived from client and line information.
- **Inputs**:
    - `modedata`: A pointer to `window_client_modedata` structure containing mode-specific data, including the key format string.
    - `itemdata`: A pointer to `window_client_itemdata` structure containing item-specific data, particularly the client information.
    - `line`: An unsigned integer representing the line number to be used in the key format.
- **Control Flow**:
    - Create a format tree `ft` with default settings.
    - Set default values in the format tree using the client from `itemdata`.
    - Add the line number to the format tree with the key 'line'.
    - Expand the format tree using the key format from `modedata` to generate a string `expanded`.
    - Look up the key code corresponding to the expanded string using [`key_string_lookup_string`](key-string.c.driver.md#key_string_lookup_string).
    - Free the memory allocated for the expanded string and the format tree.
    - Return the generated key code.
- **Output**:
    - The function returns a `key_code` which is a code representing a key derived from the formatted string.
- **Functions called**:
    - [`format_create`](format.c.driver.md#format_create)
    - [`format_defaults`](format.c.driver.md#format_defaults)
    - [`format_expand`](format.c.driver.md#format_expand)
    - [`key_string_lookup_string`](key-string.c.driver.md#key_string_lookup_string)
    - [`format_free`](format.c.driver.md#format_free)


---
### window_client_init<!-- {{#callable:window_client_init}} -->
The `window_client_init` function initializes a window client mode by setting up the necessary data structures and configurations for displaying and interacting with a list of clients in a window pane.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure representing the mode entry for the window pane.
    - `fs`: An unused pointer to a `cmd_find_state` structure, which is typically used for command state management.
    - `args`: A pointer to an `args` structure containing command-line arguments that may specify custom formats or commands for the window client mode.
- **Control Flow**:
    - Allocate memory for a `window_client_modedata` structure and assign it to `wme->data` and `data`.
    - Set `data->wp` to the window pane `wp` from `wme`.
    - Check if `args` is NULL or does not have the 'F' flag; if so, use the default format, otherwise use the format specified by the 'F' flag in `args`.
    - Check if `args` is NULL or does not have the 'K' flag; if so, use the default key format, otherwise use the key format specified by the 'K' flag in `args`.
    - Check if `args` is NULL or has no arguments; if so, use the default command, otherwise use the command specified by the first argument in `args`.
    - Initialize the mode tree with [`mode_tree_start`](mode-tree.c.driver.md#mode_tree_start), passing the window pane, arguments, and various callback functions, and store the result in `data->data`.
    - Call [`mode_tree_zoom`](mode-tree.c.driver.md#mode_tree_zoom) to adjust the mode tree based on the arguments.
    - Build and draw the mode tree using [`mode_tree_build`](mode-tree.c.driver.md#mode_tree_build) and [`mode_tree_draw`](mode-tree.c.driver.md#mode_tree_draw).
    - Return the screen pointer `s` which is set by [`mode_tree_start`](mode-tree.c.driver.md#mode_tree_start).
- **Output**:
    - Returns a pointer to a `screen` structure, which represents the initialized screen for the window client mode.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`args_get`](arguments.c.driver.md#args_get)
    - [`args_count`](arguments.c.driver.md#args_count)
    - [`args_string`](arguments.c.driver.md#args_string)
    - [`mode_tree_start`](mode-tree.c.driver.md#mode_tree_start)
    - [`mode_tree_zoom`](mode-tree.c.driver.md#mode_tree_zoom)
    - [`mode_tree_build`](mode-tree.c.driver.md#mode_tree_build)
    - [`mode_tree_draw`](mode-tree.c.driver.md#mode_tree_draw)


---
### window_client_free<!-- {{#callable:window_client_free}} -->
The `window_client_free` function deallocates memory and resources associated with a `window_mode_entry` structure's client mode data.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` which contains the client mode data to be freed.
- **Control Flow**:
    - Retrieve the `window_client_modedata` from the `wme` structure.
    - Check if the `data` is NULL; if so, return immediately as there is nothing to free.
    - Call [`mode_tree_free`](mode-tree.c.driver.md#mode_tree_free) to free the mode tree data associated with the client mode.
    - Iterate over each item in the `item_list` and call [`window_client_free_item`](#window_client_free_item) to free each item.
    - Free the `item_list` array itself.
    - Free the `format`, `key_format`, and `command` strings associated with the client mode.
    - Finally, free the `window_client_modedata` structure itself.
- **Output**:
    - The function does not return any value; it performs cleanup operations to free allocated memory.
- **Functions called**:
    - [`mode_tree_free`](mode-tree.c.driver.md#mode_tree_free)
    - [`window_client_free_item`](#window_client_free_item)


---
### window_client_resize<!-- {{#callable:window_client_resize}} -->
The `window_client_resize` function adjusts the size of a mode tree associated with a window client mode entry based on the provided dimensions.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry`, which contains data related to the window client mode.
    - `sx`: An unsigned integer representing the new width to resize to.
    - `sy`: An unsigned integer representing the new height to resize to.
- **Control Flow**:
    - Retrieve the `window_client_modedata` structure from the `wme` parameter.
    - Call the [`mode_tree_resize`](mode-tree.c.driver.md#mode_tree_resize) function with the mode tree data and the new dimensions `sx` and `sy`.
- **Output**:
    - This function does not return a value; it performs an in-place resize operation on the mode tree data.
- **Functions called**:
    - [`mode_tree_resize`](mode-tree.c.driver.md#mode_tree_resize)


---
### window_client_update<!-- {{#callable:window_client_update}} -->
The `window_client_update` function updates the mode tree data and redraws the associated window pane in a window mode entry.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry`, which contains the mode data for a window.
- **Control Flow**:
    - Retrieve the `window_client_modedata` from the `wme` structure.
    - Call [`mode_tree_build`](mode-tree.c.driver.md#mode_tree_build) on the `data` field of the `window_client_modedata` to rebuild the mode tree.
    - Call [`mode_tree_draw`](mode-tree.c.driver.md#mode_tree_draw) on the `data` field to redraw the mode tree.
    - Set the `PANE_REDRAW` flag on the `wp` field of the `window_client_modedata` to indicate that the pane needs to be redrawn.
- **Output**:
    - This function does not return a value; it updates the mode tree and sets a redraw flag on the window pane.
- **Functions called**:
    - [`mode_tree_build`](mode-tree.c.driver.md#mode_tree_build)
    - [`mode_tree_draw`](mode-tree.c.driver.md#mode_tree_draw)


---
### window_client_do_detach<!-- {{#callable:window_client_do_detach}} -->
The `window_client_do_detach` function handles detaching or suspending a client based on a key input.
- **Inputs**:
    - `modedata`: A pointer to `window_client_modedata` structure containing mode-specific data.
    - `itemdata`: A pointer to `window_client_itemdata` structure representing the client item to be detached or suspended.
    - `c`: An unused pointer to a `client` structure.
    - `key`: A `key_code` representing the key pressed to trigger the detach or suspend action.
- **Control Flow**:
    - Check if the current item is the one selected in the mode tree; if so, move the selection down.
    - If the key is 'd' or 'D', call [`server_client_detach`](server-client.c.driver.md#server_client_detach) with `MSG_DETACH` to detach the client.
    - If the key is 'x' or 'X', call [`server_client_detach`](server-client.c.driver.md#server_client_detach) with `MSG_DETACHKILL` to detach and kill the client.
    - If the key is 'z' or 'Z', call [`server_client_suspend`](server-client.c.driver.md#server_client_suspend) to suspend the client.
- **Output**:
    - The function does not return a value; it performs actions on the client based on the key input.
- **Functions called**:
    - [`mode_tree_get_current`](mode-tree.c.driver.md#mode_tree_get_current)
    - [`mode_tree_down`](mode-tree.c.driver.md#mode_tree_down)
    - [`server_client_detach`](server-client.c.driver.md#server_client_detach)
    - [`server_client_suspend`](server-client.c.driver.md#server_client_suspend)


---
### window_client_key<!-- {{#callable:window_client_key}} -->
The `window_client_key` function processes key events in a window client mode, handling specific key actions such as detaching clients or executing commands.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode entry.
    - `c`: A pointer to the `client` structure representing the client that triggered the key event.
    - `s`: An unused pointer to the `session` structure.
    - `wl`: An unused pointer to the `winlink` structure.
    - `key`: A `key_code` representing the key that was pressed.
    - `m`: A pointer to the `mouse_event` structure representing any mouse event associated with the key press.
- **Control Flow**:
    - Retrieve the current window pane and mode data from the `window_mode_entry` structure.
    - Call [`mode_tree_key`](mode-tree.c.driver.md#mode_tree_key) to process the key event and determine if the mode should be finished.
    - Use a switch statement to handle specific key codes: 'd', 'x', 'z' for detaching the current client, 'D', 'X', 'Z' for detaching all tagged clients, and '\r' for running a command on the current client.
    - If the mode is finished or there are no more clients, reset the window pane mode.
    - Otherwise, redraw the mode tree and set the pane redraw flag.
- **Output**:
    - The function does not return a value; it modifies the state of the window pane and client based on the key event.
- **Functions called**:
    - [`mode_tree_key`](mode-tree.c.driver.md#mode_tree_key)
    - [`mode_tree_get_current`](mode-tree.c.driver.md#mode_tree_get_current)
    - [`window_client_do_detach`](#window_client_do_detach)
    - [`mode_tree_build`](mode-tree.c.driver.md#mode_tree_build)
    - [`mode_tree_each_tagged`](mode-tree.c.driver.md#mode_tree_each_tagged)
    - [`mode_tree_run_command`](mode-tree.c.driver.md#mode_tree_run_command)
    - [`server_client_how_many`](server-client.c.driver.md#server_client_how_many)
    - [`window_pane_reset_mode`](window.c.driver.md#window_pane_reset_mode)
    - [`mode_tree_draw`](mode-tree.c.driver.md#mode_tree_draw)


