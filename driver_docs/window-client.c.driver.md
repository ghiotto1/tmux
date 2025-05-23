# Purpose
This C source code file is part of the tmux terminal multiplexer project and implements a specific window mode called "client-mode." The primary functionality of this code is to manage and display a list of clients connected to a tmux server, allowing users to interact with these clients through a menu-driven interface. The code defines several static functions for initializing, freeing, resizing, updating, and handling key events within this mode. It also includes structures and functions for sorting and managing client data, as well as rendering the client list on the screen.

The file defines a set of menu items and key bindings that allow users to perform actions such as detaching clients, tagging them, or executing commands. The code is structured around a mode tree, which is a common pattern in tmux for managing hierarchical data and user interactions. The mode tree is used to build, sort, and display the list of clients, and it supports filtering and custom formatting of the displayed information. This file is not an executable on its own but is intended to be part of the larger tmux application, providing a specific mode that can be activated within a tmux session.
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
- **Description**: `window_client_init` is a static function pointer that returns a pointer to a `struct screen`. It is used to initialize the client mode in a window, setting up necessary data structures and configurations for the mode.
- **Use**: This function is used to initialize the client mode by setting up the screen and associated data structures when entering the mode.


---
### window_client_menu_items
- **Type**: `static const struct menu_item[]`
- **Description**: The `window_client_menu_items` is a static constant array of `struct menu_item` that defines a set of menu options for a client window in the application. Each menu item consists of a label, a key code, and a NULL pointer, with some items serving as separators (empty labels with `KEYC_NONE`). The array is terminated with a NULL entry to indicate the end of the menu items.
- **Use**: This variable is used to define the menu options available to the user in the client window mode, allowing interaction through specific key bindings.


---
### window_client_mode
- **Type**: `const struct window_mode`
- **Description**: The `window_client_mode` is a constant structure of type `window_mode` that defines a specific mode for handling client windows in the application. It includes a name, a default format, and function pointers for initialization, freeing resources, resizing, updating, and handling key events.
- **Use**: This variable is used to define and manage the behavior and lifecycle of client windows within the application.


---
### window_client_sort_list
- **Type**: `const char *[]`
- **Description**: `window_client_sort_list` is a static constant array of strings that defines the sorting criteria for window clients. It contains four elements: "name", "size", "creation", and "activity". These strings represent the different attributes by which window clients can be sorted.
- **Use**: This variable is used to provide a list of sorting options for window clients, allowing the system to sort clients based on name, size, creation time, or activity time.


---
### window_client_sort
- **Type**: `struct mode_tree_sort_criteria*`
- **Description**: The `window_client_sort` is a static pointer to a `struct mode_tree_sort_criteria`, which is used to define the criteria for sorting window clients in the application. This structure likely contains fields that specify the sorting field and order (e.g., ascending or descending).
- **Use**: This variable is used to determine the sorting order of window clients based on specified criteria such as name, size, creation time, or activity time.


# Data Structures

---
### window_client_sort_type
- **Type**: `enum`
- **Members**:
    - `WINDOW_CLIENT_BY_NAME`: Sorts window clients by their name.
    - `WINDOW_CLIENT_BY_SIZE`: Sorts window clients by their size.
    - `WINDOW_CLIENT_BY_CREATION_TIME`: Sorts window clients by their creation time.
    - `WINDOW_CLIENT_BY_ACTIVITY_TIME`: Sorts window clients by their activity time.
- **Description**: The `window_client_sort_type` is an enumeration that defines the criteria for sorting window clients in a user interface. It provides four sorting options: by name, by size, by creation time, and by activity time, allowing for flexible organization of window clients based on different attributes.


---
### window_client_itemdata
- **Type**: `struct`
- **Members**:
    - `c`: A pointer to a `client` structure, representing a client associated with the item data.
- **Description**: The `window_client_itemdata` structure is a simple data structure that encapsulates a single pointer to a `client` structure. It is used within the context of managing client-related data in a window client mode, allowing for operations such as sorting and managing client items in a list. This structure is primarily used to associate a client with specific item data in the window client mode, facilitating operations like detaching or suspending clients.


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
- **Description**: The `window_client_modedata` structure is used to manage the state and data associated with a client mode in a window pane. It includes pointers to related structures for managing the mode tree and client items, as well as strings for formatting display and key bindings. The structure also maintains a list of client items and tracks the size of this list, facilitating operations such as sorting and filtering clients within the mode.


# Functions

---
### window_client_add_item<!-- {{#callable:window_client_add_item}} -->
The `window_client_add_item` function adds a new item to the list of client items in a `window_client_modedata` structure, reallocating memory as necessary.
- **Inputs**:
    - `data`: A pointer to a `window_client_modedata` structure, which contains the list of client items and its current size.
- **Control Flow**:
    - The function begins by reallocating the `item_list` array in the `data` structure to accommodate one more item, increasing the size by one.
    - It then allocates memory for a new `window_client_itemdata` structure using `xcalloc`, initializing it to zero.
    - The newly allocated item is added to the `item_list` at the position indicated by the current `item_size`, which is then incremented.
    - Finally, the function returns a pointer to the newly added `window_client_itemdata` item.
- **Output**:
    - A pointer to the newly added `window_client_itemdata` structure.


---
### window_client_free_item<!-- {{#callable:window_client_free_item}} -->
The `window_client_free_item` function releases the resources associated with a `window_client_itemdata` structure by decrementing the reference count of the client and freeing the memory allocated for the item.
- **Inputs**:
    - `item`: A pointer to a `window_client_itemdata` structure that contains a client reference to be freed.
- **Control Flow**:
    - Call `server_client_unref` to decrement the reference count of the client associated with the `item`.
    - Use `free` to deallocate the memory occupied by the `item`.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the provided `window_client_itemdata` structure.


---
### window_client_cmp<!-- {{#callable:window_client_cmp}} -->
The `window_client_cmp` function compares two `window_client_itemdata` structures based on specified sorting criteria, such as size, creation time, activity time, or name, and returns an integer indicating their relative order.
- **Inputs**:
    - `a0`: A pointer to the first `window_client_itemdata` structure to be compared.
    - `b0`: A pointer to the second `window_client_itemdata` structure to be compared.
- **Control Flow**:
    - The function casts the input pointers `a0` and `b0` to pointers of type `const struct window_client_itemdata *const *` and dereferences them to obtain `itema` and `itemb`, respectively.
    - It retrieves the `client` structures `ca` and `cb` from `itema` and `itemb`.
    - The function initializes a result variable to zero, which will hold the comparison result.
    - A switch statement is used to determine the sorting field specified by `window_client_sort->field`.
    - If the sorting field is `WINDOW_CLIENT_BY_SIZE`, it compares the `tty.sx` and `tty.sy` fields of `ca` and `cb` to determine the result.
    - If the sorting field is `WINDOW_CLIENT_BY_CREATION_TIME`, it uses `timercmp` to compare the `creation_time` fields of `ca` and `cb`, setting the result to -1 or 1 based on the comparison.
    - If the sorting field is `WINDOW_CLIENT_BY_ACTIVITY_TIME`, it uses `timercmp` to compare the `activity_time` fields of `ca` and `cb`, setting the result to -1 or 1 based on the comparison.
    - If the result is still zero after the above comparisons, it defaults to comparing the `name` fields of `ca` and `cb` using `strcmp`.
    - If `window_client_sort->reversed` is true, the result is negated to reverse the order.
    - The function returns the final result.
- **Output**:
    - An integer indicating the relative order of the two `window_client_itemdata` structures: negative if the first is less than the second, zero if they are equal, and positive if the first is greater than the second.


---
### window_client_build<!-- {{#callable:window_client_build}} -->
The `window_client_build` function constructs and sorts a list of client items based on specified criteria and applies a filter to format and add them to a mode tree.
- **Inputs**:
    - `modedata`: A pointer to `window_client_modedata` structure which holds data related to the window client mode.
    - `sort_crit`: A pointer to `mode_tree_sort_criteria` structure that specifies the criteria for sorting the client items.
    - `tag`: An unused pointer to a `uint64_t` value, presumably for tagging purposes.
    - `filter`: A constant character pointer representing a filter string used to determine which clients to include in the mode tree.
- **Control Flow**:
    - Iterates over the existing items in `data->item_list`, freeing each item and then the list itself, resetting `item_list` and `item_size` to zero.
    - Iterates over all clients in the global `clients` list, skipping those without a session or with unattached flags, and adds the remaining clients to `data->item_list` while incrementing their reference count.
    - Sets the global `window_client_sort` to the provided `sort_crit` and sorts `data->item_list` using `qsort` and the `window_client_cmp` comparator function.
    - Iterates over the sorted `data->item_list`, applying the `filter` if provided, and formats each client item using `format_single` before adding it to the mode tree with `mode_tree_add`.
- **Output**:
    - The function does not return a value but modifies the `data->item_list` and `data->data` to reflect the sorted and filtered list of client items.
- **Functions called**:
    - [`window_client_free_item`](#window_client_free_item)
    - [`window_client_add_item`](#window_client_add_item)


---
### window_client_draw<!-- {{#callable:window_client_draw}} -->
The `window_client_draw` function renders the client window's content on the screen, adjusting for status lines and cursor position.
- **Inputs**:
    - `modedata`: Unused parameter, typically used for mode-specific data.
    - `itemdata`: Pointer to `window_client_itemdata` structure containing client-specific data.
    - `ctx`: Pointer to `screen_write_ctx` structure used for writing to the screen.
    - `sx`: Unsigned integer representing the width of the screen.
    - `sy`: Unsigned integer representing the height of the screen.
- **Control Flow**:
    - Check if the client is attached to a session and return if not.
    - Retrieve the active window pane from the client's current session.
    - Calculate the number of status lines and determine their position (top or bottom).
    - Move the cursor to the appropriate position based on the status line position.
    - Render a preview of the window pane's base content, adjusting for status lines.
    - Draw a horizontal line at the bottom or top of the screen, depending on the status line position.
    - Move the cursor to the appropriate position for rendering the status line.
    - Copy the status line content from the client's status screen to the current screen context.
- **Output**:
    - The function does not return a value; it modifies the screen context to display the client's window content.


---
### window_client_menu<!-- {{#callable:window_client_menu}} -->
The `window_client_menu` function processes a key event for a client in a specific window pane mode, delegating the key handling to the [`window_client_key`](#window_client_key) function if the mode data matches.
- **Inputs**:
    - `modedata`: A pointer to the mode data associated with the window pane, specifically of type `struct window_client_modedata`.
    - `c`: A pointer to the client structure, representing the client for which the menu is being processed.
    - `key`: The key code representing the key event that triggered the menu action.
- **Control Flow**:
    - Retrieve the window pane from the mode data using `data->wp`.
    - Get the first mode entry from the window pane's mode queue using `TAILQ_FIRST(&wp->modes)`.
    - Check if the mode entry is NULL or if its data does not match the provided mode data; if either condition is true, return immediately.
    - Call [`window_client_key`](#window_client_key) with the mode entry, client, and key to handle the key event.
- **Output**:
    - The function does not return a value; it performs actions based on the key event and mode data.
- **Functions called**:
    - [`window_client_key`](#window_client_key)


---
### window_client_get_key<!-- {{#callable:window_client_get_key}} -->
The `window_client_get_key` function generates a key code based on a formatted string derived from client and line data.
- **Inputs**:
    - `modedata`: A pointer to `window_client_modedata` structure containing mode-specific data, including the key format string.
    - `itemdata`: A pointer to `window_client_itemdata` structure representing a client item, which includes a reference to a client.
    - `line`: An unsigned integer representing the line number to be used in the key format.
- **Control Flow**:
    - Create a format tree `ft` with no specific format context.
    - Set default values in the format tree using the client from `itemdata`.
    - Add the line number to the format tree with the key 'line'.
    - Expand the format tree using the key format from `modedata` to generate a formatted string.
    - Look up the key code corresponding to the expanded string using `key_string_lookup_string`.
    - Free the memory allocated for the expanded string and the format tree.
    - Return the generated key code.
- **Output**:
    - The function returns a `key_code`, which is a code representing a key derived from the formatted string.


---
### window_client_init<!-- {{#callable:window_client_init}} -->
The `window_client_init` function initializes a window client mode by setting up the necessary data structures and configurations for displaying and interacting with a list of clients in a window pane.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure representing the mode entry for the window pane.
    - `fs`: An unused pointer to a `cmd_find_state` structure, which is not utilized in this function.
    - `args`: A pointer to an `args` structure containing command-line arguments that may specify custom formats or commands for the window client mode.
- **Control Flow**:
    - Allocate memory for a `window_client_modedata` structure and assign it to `wme->data` and `data`.
    - Set `data->wp` to the window pane from `wme`.
    - Check if `args` is NULL or does not have the 'F' flag; if so, use the default format, otherwise use the format specified by the 'F' flag in `args`.
    - Check if `args` is NULL or does not have the 'K' flag; if so, use the default key format, otherwise use the key format specified by the 'K' flag in `args`.
    - Check if `args` is NULL or has no arguments; if so, use the default command, otherwise use the command specified by the first argument in `args`.
    - Initialize the mode tree with `mode_tree_start`, passing the window pane, arguments, and various callback functions, and store the result in `data->data`.
    - Call `mode_tree_zoom` to adjust the mode tree based on the arguments.
    - Build and draw the mode tree using `mode_tree_build` and `mode_tree_draw`.
    - Return the screen pointer `s` which is set by `mode_tree_start`.
- **Output**:
    - Returns a pointer to a `screen` structure that represents the initialized screen for the window client mode.


---
### window_client_free<!-- {{#callable:window_client_free}} -->
The `window_client_free` function deallocates memory and resources associated with a `window_mode_entry` structure's client mode data.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` which contains the client mode data to be freed.
- **Control Flow**:
    - Retrieve the `window_client_modedata` from the `wme` structure.
    - Check if the `data` is `NULL`, and return immediately if it is.
    - Call `mode_tree_free` to free the mode tree data associated with `data->data`.
    - Iterate over each item in `data->item_list` and call [`window_client_free_item`](#window_client_free_item) to free each item.
    - Free the `data->item_list` array itself.
    - Free the `data->format`, `data->key_format`, and `data->command` strings.
    - Finally, free the `data` structure itself.
- **Output**:
    - This function does not return any value; it performs cleanup operations to free allocated memory.
- **Functions called**:
    - [`window_client_free_item`](#window_client_free_item)


---
### window_client_resize<!-- {{#callable:window_client_resize}} -->
The `window_client_resize` function adjusts the size of a mode tree associated with a window client based on the provided dimensions.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure, which contains data related to the window mode entry.
    - `sx`: An unsigned integer representing the new width (size x) for the mode tree.
    - `sy`: An unsigned integer representing the new height (size y) for the mode tree.
- **Control Flow**:
    - Retrieve the `window_client_modedata` structure from the `data` field of the `window_mode_entry` structure pointed to by `wme`.
    - Call the `mode_tree_resize` function with the `data` field of the `window_client_modedata` structure and the new dimensions `sx` and `sy`.
- **Output**:
    - This function does not return a value; it performs an in-place resize operation on the mode tree data structure.


---
### window_client_update<!-- {{#callable:window_client_update}} -->
The `window_client_update` function updates the mode tree data and redraws the associated window pane.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` which contains the mode data to be updated.
- **Control Flow**:
    - Retrieve the `window_client_modedata` from the `wme` structure.
    - Call `mode_tree_build` on the `data` field of the `window_client_modedata` to rebuild the mode tree.
    - Call `mode_tree_draw` on the `data` field to redraw the mode tree.
    - Set the `PANE_REDRAW` flag on the `wp` field of the `window_client_modedata` to indicate that the pane needs to be redrawn.
- **Output**:
    - This function does not return a value; it updates the mode tree and sets a redraw flag on the window pane.


---
### window_client_do_detach<!-- {{#callable:window_client_do_detach}} -->
The `window_client_do_detach` function handles the detachment or suspension of a client based on the provided key input.
- **Inputs**:
    - `modedata`: A pointer to `window_client_modedata` structure, which contains mode-specific data for the window client.
    - `itemdata`: A pointer to `window_client_itemdata` structure, representing the client item data to be processed.
    - `c`: An unused pointer to a `client` structure, representing the client context.
    - `key`: A `key_code` representing the key input that determines the action to be performed on the client.
- **Control Flow**:
    - Check if the current item is the one selected in the mode tree; if so, move the selection down in the mode tree.
    - If the key is 'd' or 'D', call `server_client_detach` to detach the client with a normal detach message.
    - If the key is 'x' or 'X', call `server_client_detach` to detach the client with a detach kill message.
    - If the key is 'z' or 'Z', call `server_client_suspend` to suspend the client.
- **Output**:
    - The function does not return a value; it performs actions on the client based on the key input.


---
### window_client_key<!-- {{#callable:window_client_key}} -->
The `window_client_key` function processes key events in a window client mode, handling specific keys to detach clients, run commands, or update the mode tree display.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure representing the current window mode entry.
    - `c`: A pointer to a `client` structure representing the client that triggered the key event.
    - `s`: An unused pointer to a `session` structure.
    - `wl`: An unused pointer to a `winlink` structure.
    - `key`: A `key_code` representing the key that was pressed.
    - `m`: A pointer to a `mouse_event` structure representing any mouse event associated with the key press.
- **Control Flow**:
    - Retrieve the current window pane and mode data from the `window_mode_entry` structure.
    - Call `mode_tree_key` to process the key event and determine if the mode should be finished.
    - Use a switch statement to handle specific key codes: 'd', 'x', 'z' for detaching the current client, 'D', 'X', 'Z' for detaching all tagged clients, and '\r' for running a command associated with the current item.
    - If the mode is finished or there are no more clients, reset the window pane mode.
    - Otherwise, redraw the mode tree and set the pane's redraw flag.
- **Output**:
    - The function does not return a value; it modifies the state of the window pane and mode tree based on the key event.
- **Functions called**:
    - [`window_client_do_detach`](#window_client_do_detach)


# Function Declarations (Public API)

---
### window_client_init<!-- {{#callable_declaration:window_client_init}} -->
Initializes a window client mode entry with specified arguments.
- **Description**: This function sets up a window client mode entry, initializing its data structures and configuring it based on the provided arguments. It should be called when a new window client mode is being created. The function expects a valid window mode entry and may use optional arguments to customize the format, key format, and command. If arguments are not provided, default values are used. The function returns a screen structure representing the initialized mode.
- **Inputs**:
    - `wme`: A pointer to a struct window_mode_entry representing the window mode entry to initialize. Must not be null.
    - `fs`: A pointer to a struct cmd_find_state, which is unused in this function. Can be null.
    - `args`: A pointer to a struct args containing optional arguments for initialization. Can be null. If null or if specific flags are not set, default values are used for format, key format, and command.
- **Output**: Returns a pointer to a struct screen representing the initialized window client mode.
- **See also**: [`window_client_init`](#window_client_init)  (Implementation)


---
### window_client_free<!-- {{#callable_declaration:window_client_free}} -->
Frees resources associated with a window mode entry.
- **Description**: Use this function to release all resources and memory associated with a given window mode entry when it is no longer needed. This function should be called to prevent memory leaks after a window mode entry has been used. It is important to ensure that the `wme` parameter is valid and that the function is called only once per entry to avoid undefined behavior.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that must not be null. The function assumes ownership of the resources associated with this entry and will free them. If `wme->data` is null, the function returns immediately without performing any operations.
- **Output**: None
- **See also**: [`window_client_free`](#window_client_free)  (Implementation)


---
### window_client_resize<!-- {{#callable_declaration:window_client_resize}} -->
Resize the window client mode entry.
- **Description**: This function adjusts the size of the window client mode entry to the specified dimensions. It should be called whenever the dimensions of the window need to be updated, such as when the window is resized by the user. The function requires a valid window mode entry and the new dimensions as input. It does not return any value or modify the input parameters directly.
- **Inputs**:
    - `wme`: A pointer to a struct window_mode_entry representing the window mode entry to be resized. Must not be null.
    - `sx`: An unsigned integer representing the new width of the window. Must be a valid non-negative value.
    - `sy`: An unsigned integer representing the new height of the window. Must be a valid non-negative value.
- **Output**: None
- **See also**: [`window_client_resize`](#window_client_resize)  (Implementation)


---
### window_client_update<!-- {{#callable_declaration:window_client_update}} -->
Updates the client mode display and marks the pane for redraw.
- **Description**: This function is used to refresh the display of a client mode entry within a window. It should be called when the data associated with the client mode needs to be rebuilt and redrawn, ensuring that the display reflects the current state. This function also marks the associated window pane for redraw, which is necessary to update the visual representation on the screen. It is typically used in environments where the client mode's data or display needs to be updated dynamically.
- **Inputs**:
    - `wme`: A pointer to a 'struct window_mode_entry' that represents the mode entry to be updated. This parameter must not be null, and it is expected to be a valid mode entry with associated data.
- **Output**: None
- **See also**: [`window_client_update`](#window_client_update)  (Implementation)


---
### window_client_key<!-- {{#callable_declaration:window_client_key}} -->
Handle key events in client mode.
- **Description**: This function processes key events for a client in a specific window mode, allowing for actions such as detaching clients or executing commands. It should be called when a key event occurs in the client mode. The function manages the current mode tree state and updates the display accordingly. It expects valid pointers for the window mode entry and client, and handles specific key codes to perform actions like detaching clients or running commands. The function does not return a value but may modify the state of the window pane or client.
- **Inputs**:
    - `wme`: A pointer to a struct window_mode_entry representing the current mode entry. Must not be null.
    - `c`: A pointer to a struct client representing the client for which the key event is being processed. Must not be null.
    - `s`: A pointer to a struct session, which is unused in this function. Can be null.
    - `wl`: A pointer to a struct winlink, which is unused in this function. Can be null.
    - `key`: A key_code representing the key event to be processed. Specific key codes trigger different actions.
    - `m`: A pointer to a struct mouse_event representing any associated mouse event. Can be null if no mouse event is associated.
- **Output**: None
- **See also**: [`window_client_key`](#window_client_key)  (Implementation)


