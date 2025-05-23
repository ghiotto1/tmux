# Purpose
This C source code file is part of the tmux project, specifically implementing a mode for managing and interacting with paste buffers within a tmux session. The code defines a "buffer-mode" that allows users to view, edit, sort, and manipulate paste buffers, which are temporary storage areas for text data. The file includes functions for initializing, resizing, updating, and freeing the buffer mode, as well as handling user input through key events and mouse interactions. It also defines a menu interface for performing actions such as pasting, tagging, and deleting buffers.

The code is structured around several key components, including data structures for managing buffer items and mode data, functions for sorting and displaying buffer contents, and a set of menu items for user interaction. The file defines a public API for the buffer mode through the `window_buffer_mode` structure, which includes function pointers for mode operations. The code is designed to be integrated into the larger tmux application, providing a specialized interface for buffer management. It leverages tmux's existing infrastructure for handling user input and rendering content within a terminal pane, making it a cohesive part of the tmux ecosystem.
# Imports and Dependencies

---
- `sys/types.h`
- `stdio.h`
- `stdlib.h`
- `string.h`
- `time.h`
- `unistd.h`
- `tmux.h`


# Global Variables

---
### window_buffer_init
- **Type**: `struct screen*`
- **Description**: The `window_buffer_init` function is a static function that initializes a screen structure for a window mode entry in the tmux application. It sets up the necessary data structures and configurations for managing buffer items within a window pane, including command, format, and key format settings.
- **Use**: This function is used to initialize and configure the screen and associated data for buffer management in a tmux window mode.


---
### window_buffer_menu_items
- **Type**: `const struct menu_item[]`
- **Description**: `window_buffer_menu_items` is a static constant array of `menu_item` structures, each representing a menu option for a buffer window in the application. Each menu item consists of a label, a key code, and a NULL pointer, with the array terminated by a `menu_item` with a NULL label.
- **Use**: This variable is used to define the menu options available in the buffer window mode, allowing users to interact with buffer items through key commands.


---
### window_buffer_mode
- **Type**: `const struct window_mode`
- **Description**: The `window_buffer_mode` is a constant structure of type `window_mode` that defines a specific mode for handling buffer operations in a window. It includes a name, a default format, and function pointers for initialization, freeing resources, resizing, updating, and handling key events.
- **Use**: This variable is used to define and manage the behavior of a window mode specifically for buffer operations, providing the necessary functions and settings to handle buffer-related tasks.


---
### window_buffer_sort_list
- **Type**: `array of const char pointers`
- **Description**: `window_buffer_sort_list` is a static array of constant character pointers that holds the names of sorting criteria for window buffers. The array contains three elements: "time", "name", and "size".
- **Use**: This variable is used to define the available sorting options for window buffers in the application.


---
### window_buffer_sort
- **Type**: `struct mode_tree_sort_criteria*`
- **Description**: The `window_buffer_sort` is a static pointer to a `struct mode_tree_sort_criteria`, which is used to define the sorting criteria for window buffers in the application. This structure likely contains fields that specify how buffers should be sorted, such as by time, name, or size, and whether the sorting should be reversed.
- **Use**: This variable is used to determine the sorting order of window buffers when they are displayed or manipulated in the application.


# Data Structures

---
### window_buffer_sort_type
- **Type**: `enum`
- **Members**:
    - `WINDOW_BUFFER_BY_TIME`: Sorts the window buffer by the time of creation.
    - `WINDOW_BUFFER_BY_NAME`: Sorts the window buffer by the name of the buffer.
    - `WINDOW_BUFFER_BY_SIZE`: Sorts the window buffer by the size of the buffer.
- **Description**: The `window_buffer_sort_type` is an enumeration that defines the criteria for sorting window buffers in a program. It provides three sorting options: by time, by name, and by size, allowing the user to organize buffers based on these attributes. This enum is likely used in conjunction with sorting functions to determine the order in which buffers are displayed or processed.


---
### window_buffer_itemdata
- **Type**: `struct`
- **Members**:
    - `name`: A constant character pointer representing the name of the buffer item.
    - `order`: An unsigned integer representing the order of the buffer item.
    - `size`: A size_t variable representing the size of the buffer item.
- **Description**: The `window_buffer_itemdata` structure is used to represent an item in a window buffer, containing metadata about the buffer such as its name, order, and size. This structure is part of a larger system for managing and displaying buffer data within a window, allowing for operations such as sorting and displaying buffer items based on various criteria.


---
### window_buffer_modedata
- **Type**: `struct`
- **Members**:
    - `wp`: A pointer to a window_pane structure, representing the pane associated with this mode data.
    - `fs`: A cmd_find_state structure, used to store the state of a command find operation.
    - `data`: A pointer to a mode_tree_data structure, which holds the data for the mode tree.
    - `command`: A string representing the command to be executed.
    - `format`: A string representing the format for displaying buffer information.
    - `key_format`: A string representing the format for key bindings.
    - `item_list`: A pointer to an array of window_buffer_itemdata structures, representing the list of buffer items.
    - `item_size`: An unsigned integer representing the size of the item list.
- **Description**: The `window_buffer_modedata` structure is used to manage the state and data associated with a window buffer mode in a terminal multiplexer. It includes pointers to various structures and strings that define the command, format, and key bindings for the mode, as well as a list of buffer items and their size. This structure is integral to handling the display and interaction with buffer data within a window pane.


---
### window_buffer_editdata
- **Type**: `struct`
- **Members**:
    - `wp_id`: An unsigned integer representing the window pane ID.
    - `name`: A pointer to a character array (string) representing the name of the buffer.
    - `pb`: A pointer to a paste_buffer structure, representing the paste buffer associated with this data.
- **Description**: The `window_buffer_editdata` structure is used to store information related to editing a buffer in a window pane. It includes the ID of the window pane (`wp_id`), the name of the buffer (`name`), and a pointer to the associated paste buffer (`pb`). This structure is likely used in the context of managing and editing text buffers within a terminal multiplexer environment, such as tmux, where buffers can be edited, pasted, or manipulated.


# Functions

---
### window_buffer_add_item<!-- {{#callable:window_buffer_add_item}} -->
The function `window_buffer_add_item` adds a new item to the list of buffer items in a window buffer mode data structure.
- **Inputs**:
    - `data`: A pointer to a `window_buffer_modedata` structure, which contains the list of buffer items and its current size.
- **Control Flow**:
    - The function reallocates memory for the `item_list` array to accommodate one more item, increasing the size by one.
    - It then allocates memory for a new `window_buffer_itemdata` structure, initializes it to zero, and assigns it to the newly added position in the `item_list`.
    - The `item_size` is incremented to reflect the addition of the new item.
    - Finally, the function returns a pointer to the newly added `window_buffer_itemdata` structure.
- **Output**:
    - A pointer to the newly added `window_buffer_itemdata` structure.


---
### window_buffer_free_item<!-- {{#callable:window_buffer_free_item}} -->
The `window_buffer_free_item` function deallocates memory associated with a `window_buffer_itemdata` structure, including its `name` field.
- **Inputs**:
    - `item`: A pointer to a `window_buffer_itemdata` structure that needs to be freed.
- **Control Flow**:
    - The function first frees the memory allocated for the `name` field of the `window_buffer_itemdata` structure using `free((void *)item->name);`.
    - Then, it frees the memory allocated for the `window_buffer_itemdata` structure itself using `free(item);`.
- **Output**:
    - The function does not return any value; it performs memory deallocation for the given `window_buffer_itemdata` structure.


---
### window_buffer_cmp<!-- {{#callable:window_buffer_cmp}} -->
The `window_buffer_cmp` function compares two `window_buffer_itemdata` structures based on specified sorting criteria and returns an integer indicating their order.
- **Inputs**:
    - `a0`: A pointer to the first `window_buffer_itemdata` structure to be compared.
    - `b0`: A pointer to the second `window_buffer_itemdata` structure to be compared.
- **Control Flow**:
    - Cast the input pointers `a0` and `b0` to pointers to `window_buffer_itemdata` structures.
    - Initialize a result variable to zero.
    - Check if the sorting field is `WINDOW_BUFFER_BY_TIME`; if so, compare the `order` fields of the two structures and store the result.
    - If the sorting field is `WINDOW_BUFFER_BY_SIZE`, compare the `size` fields of the two structures and store the result.
    - If the result is still zero, use `strcmp` to compare the `name` fields of the two structures as a tie breaker.
    - If the sorting order is reversed, negate the result.
    - Return the result.
- **Output**:
    - An integer that indicates the order of the two `window_buffer_itemdata` structures: negative if `a0` should come before `b0`, positive if `b0` should come before `a0`, and zero if they are equivalent.


---
### window_buffer_build<!-- {{#callable:window_buffer_build}} -->
The `window_buffer_build` function initializes and populates a list of buffer items, sorts them according to specified criteria, and applies a filter to format and add them to a mode tree for display.
- **Inputs**:
    - `modedata`: A pointer to `window_buffer_modedata` structure which holds the mode data including item list and format information.
    - `sort_crit`: A pointer to `mode_tree_sort_criteria` structure that specifies the sorting criteria for the buffer items.
    - `tag`: An unused pointer to a `uint64_t` value, presumably for tagging purposes.
    - `filter`: A string representing a filter expression to apply to each buffer item.
- **Control Flow**:
    - Free all existing items in the `item_list` and reset the list and its size to zero.
    - Iterate over all paste buffers using `paste_walk`, adding each buffer as an item to `item_list` with its name, size, and order.
    - Assign the provided `sort_crit` to the global `window_buffer_sort` and sort the `item_list` using `qsort` and `window_buffer_cmp`.
    - Check if the `cmd_find_valid_state` is valid and retrieve session, winlink, and window pane information if available.
    - Iterate over the sorted `item_list`, retrieve the corresponding paste buffer, and create a format tree for each item.
    - Apply the provided `filter` to each item using `format_expand` and `format_true`; skip items that do not pass the filter.
    - Expand the format for each item using `format_expand` and add the formatted item to the mode tree using `mode_tree_add`.
    - Free the format tree and any temporary strings used during processing.
- **Output**:
    - The function does not return a value but modifies the `modedata` structure by populating and sorting its `item_list` and adding formatted items to the mode tree.
- **Functions called**:
    - [`window_buffer_free_item`](#window_buffer_free_item)
    - [`window_buffer_add_item`](#window_buffer_add_item)


---
### window_buffer_draw<!-- {{#callable:window_buffer_draw}} -->
The `window_buffer_draw` function renders the contents of a paste buffer onto a screen context, handling line breaks and special character encoding.
- **Inputs**:
    - `modedata`: Unused parameter, typically used for mode-specific data.
    - `itemdata`: Pointer to `window_buffer_itemdata` structure containing information about the buffer to be drawn.
    - `ctx`: Pointer to `screen_write_ctx` structure representing the screen context where the buffer will be drawn.
    - `sx`: Unsigned integer representing the width of the screen context.
    - `sy`: Unsigned integer representing the height of the screen context.
- **Control Flow**:
    - Retrieve the paste buffer using the name from `itemdata`; if not found, return immediately.
    - Initialize pointers to the start and end of the buffer data and determine its size.
    - Iterate over each line up to `sy` (screen height), processing each line separately.
    - For each line, allocate memory for a buffer to hold the processed line data.
    - Convert the line data to a visible format using `utf8_strvis`, handling special characters with specific flags.
    - If the processed line is not empty, move the cursor to the appropriate position and write the line to the screen context.
    - Continue to the next line until the end of the buffer data is reached or all lines are processed.
    - Free the allocated buffer memory after processing.
- **Output**:
    - The function does not return a value; it modifies the screen context `ctx` to display the buffer contents.


---
### window_buffer_search<!-- {{#callable:window_buffer_search}} -->
The `window_buffer_search` function searches for a specified substring within a paste buffer's name and data.
- **Inputs**:
    - `modedata`: Unused parameter, typically used for mode-specific data.
    - `itemdata`: Pointer to a `window_buffer_itemdata` structure representing the buffer item to be searched.
    - `ss`: The substring to search for within the buffer's name and data.
- **Control Flow**:
    - Retrieve the paste buffer associated with the item's name using `paste_get_name`.
    - If the paste buffer is not found, return 0 indicating the search failed.
    - Check if the substring `ss` is present in the item's name using `strstr`; if found, return 1 indicating a match.
    - Retrieve the buffer data and its size using `paste_buffer_data`.
    - Use `memmem` to search for the substring `ss` within the buffer data; return 1 if found, otherwise return 0.
- **Output**:
    - Returns 1 if the substring is found in the buffer's name or data, otherwise returns 0.


---
### window_buffer_menu<!-- {{#callable:window_buffer_menu}} -->
The `window_buffer_menu` function handles key events for a buffer menu in a window pane, delegating the key handling to the [`window_buffer_key`](#window_buffer_key) function if the mode data matches.
- **Inputs**:
    - `modedata`: A pointer to the mode data, specifically a `window_buffer_modedata` structure, which contains information about the current window pane and buffer mode.
    - `c`: A pointer to a `client` structure, representing the client interacting with the buffer menu.
    - `key`: A `key_code` representing the key event that triggered the function call.
- **Control Flow**:
    - Retrieve the `window_buffer_modedata` from the `modedata` pointer.
    - Access the `window_pane` associated with the mode data.
    - Get the first `window_mode_entry` from the pane's mode queue using `TAILQ_FIRST`.
    - Check if the mode entry is `NULL` or if its data does not match the provided `modedata`; if so, return immediately.
    - Call [`window_buffer_key`](#window_buffer_key) with the mode entry, client, and key to handle the key event.
- **Output**:
    - The function does not return a value; it performs actions based on the key event and the current mode data.
- **Functions called**:
    - [`window_buffer_key`](#window_buffer_key)


---
### window_buffer_get_key<!-- {{#callable:window_buffer_get_key}} -->
The `window_buffer_get_key` function retrieves a key code based on a formatted string derived from a paste buffer and session context.
- **Inputs**:
    - `modedata`: A pointer to `window_buffer_modedata` structure containing mode-specific data, including format strings and session state.
    - `itemdata`: A pointer to `window_buffer_itemdata` structure representing an item in the buffer, including its name.
    - `line`: An unsigned integer representing the line number to be used in the format string.
- **Control Flow**:
    - Check if the session state in `data->fs` is valid and retrieve session, winlink, and window pane pointers if so.
    - Retrieve the paste buffer using the name from `item->name`; return `KEYC_NONE` if the buffer is not found.
    - Create a format tree and set default values for session, winlink, and window pane if available.
    - Add the line number to the format tree.
    - Expand the format string using the format tree and look up the corresponding key code.
    - Free the expanded string and format tree resources.
    - Return the key code.
- **Output**:
    - The function returns a `key_code` which is derived from the expanded format string, or `KEYC_NONE` if the paste buffer is not found.


---
### window_buffer_init<!-- {{#callable:window_buffer_init}} -->
The `window_buffer_init` function initializes a window buffer mode entry by setting up its data structures and configurations based on provided arguments.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure representing the mode entry to be initialized.
    - `fs`: A pointer to a `cmd_find_state` structure used to copy the find state into the mode data.
    - `args`: A pointer to an `args` structure containing command-line arguments that may specify format and command options.
- **Control Flow**:
    - Allocate memory for `window_buffer_modedata` and assign it to `wme->data`.
    - Copy the window pane from `wme` to the mode data.
    - Copy the find state from `fs` to the mode data.
    - Check if `args` is NULL or does not have the 'F' flag; if so, use the default format, otherwise use the format specified by the 'F' flag.
    - Check if `args` is NULL or does not have the 'K' flag; if so, use the default key format, otherwise use the key format specified by the 'K' flag.
    - Check if `args` is NULL or has no arguments; if so, use the default command, otherwise use the command specified by the first argument.
    - Initialize the mode tree with the specified callbacks and data, and store the result in `data->data`.
    - Apply zoom settings to the mode tree based on `args`.
    - Build and draw the mode tree.
    - Return the screen pointer `s` which is set during mode tree initialization.
- **Output**:
    - Returns a pointer to a `screen` structure that represents the initialized screen for the window buffer mode.


---
### window_buffer_free<!-- {{#callable:window_buffer_free}} -->
The `window_buffer_free` function deallocates memory and resources associated with a `window_mode_entry` structure's buffer mode data.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` which contains the buffer mode data to be freed.
- **Control Flow**:
    - Retrieve the `window_buffer_modedata` from the `wme` structure.
    - Check if the `data` is `NULL`; if so, return immediately as there is nothing to free.
    - Call `mode_tree_free` to free the mode tree data associated with the buffer mode.
    - Iterate over each item in the `item_list` and call [`window_buffer_free_item`](#window_buffer_free_item) to free each item.
    - Free the `item_list` array itself.
    - Free the `format`, `key_format`, and `command` strings associated with the buffer mode.
    - Finally, free the `window_buffer_modedata` structure itself.
- **Output**:
    - The function does not return any value; it performs cleanup operations to free allocated memory.
- **Functions called**:
    - [`window_buffer_free_item`](#window_buffer_free_item)


---
### window_buffer_resize<!-- {{#callable:window_buffer_resize}} -->
The `window_buffer_resize` function adjusts the size of a mode tree associated with a window buffer mode entry to new dimensions.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry`, which contains data related to the window mode entry being resized.
    - `sx`: An unsigned integer representing the new width (size x) for the mode tree.
    - `sy`: An unsigned integer representing the new height (size y) for the mode tree.
- **Control Flow**:
    - Retrieve the `window_buffer_modedata` structure from the `wme` parameter, which contains the mode tree data.
    - Call the `mode_tree_resize` function with the mode tree data and the new dimensions `sx` and `sy` to resize the mode tree.
- **Output**:
    - This function does not return a value; it performs an in-place resize of the mode tree associated with the window buffer mode entry.


---
### window_buffer_update<!-- {{#callable:window_buffer_update}} -->
The `window_buffer_update` function updates the mode tree data and redraws the pane associated with a window mode entry.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry`, which contains the mode data for a window.
- **Control Flow**:
    - Retrieve the `window_buffer_modedata` from the `window_mode_entry`.
    - Call `mode_tree_build` to rebuild the mode tree data structure using the current data.
    - Call `mode_tree_draw` to render the mode tree on the screen.
    - Set the `PANE_REDRAW` flag on the associated window pane to indicate that it needs to be redrawn.
- **Output**:
    - This function does not return a value; it performs updates and redraws the pane as a side effect.


---
### window_buffer_do_delete<!-- {{#callable:window_buffer_do_delete}} -->
The `window_buffer_do_delete` function deletes a paste buffer item from a mode tree and frees its associated resources if it is the current item and cannot be moved further down the list.
- **Inputs**:
    - `modedata`: A pointer to `window_buffer_modedata`, which contains the mode tree data and other related information.
    - `itemdata`: A pointer to `window_buffer_itemdata`, which represents the specific item in the mode tree to be deleted.
    - `c`: A pointer to a `client` structure, which is unused in this function.
    - `key`: A `key_code` representing the key pressed, which is unused in this function.
- **Control Flow**:
    - Check if the item to be deleted is the current item in the mode tree and if it cannot be moved further down the list.
    - If the item is at the end of the list, move one element up to maintain a valid selection.
    - Retrieve the paste buffer associated with the item's name using `paste_get_name`.
    - If the paste buffer is found, free it using `paste_free`.
- **Output**:
    - The function does not return a value; it performs operations to delete a paste buffer item and manage the mode tree state.


---
### window_buffer_do_paste<!-- {{#callable:window_buffer_do_paste}} -->
The `window_buffer_do_paste` function executes a paste command for a specified buffer item if the buffer exists.
- **Inputs**:
    - `modedata`: A pointer to the window buffer mode data structure, which contains information about the current mode and command to execute.
    - `itemdata`: A pointer to the window buffer item data structure, which contains information about the specific buffer item to be pasted.
    - `c`: A pointer to the client structure, representing the client that initiated the paste action.
    - `key`: A key code representing the key pressed, marked as unused in this function.
- **Control Flow**:
    - Retrieve the window buffer mode data and item data from the provided pointers.
    - Check if the buffer item name exists using `paste_get_name`.
    - If the buffer item exists, execute the command associated with the mode using `mode_tree_run_command`, passing the client, command, and item name.
- **Output**:
    - The function does not return a value; it performs an action by executing a command if the buffer item exists.


---
### window_buffer_finish_edit<!-- {{#callable:window_buffer_finish_edit}} -->
The `window_buffer_finish_edit` function deallocates memory associated with a `window_buffer_editdata` structure.
- **Inputs**:
    - `ed`: A pointer to a `window_buffer_editdata` structure that contains the data to be freed.
- **Control Flow**:
    - The function calls `free` on the `name` member of the `window_buffer_editdata` structure pointed to by `ed`.
    - The function then calls `free` on the `window_buffer_editdata` structure itself.
- **Output**:
    - The function does not return any value; it performs memory deallocation.


---
### window_buffer_edit_close_cb<!-- {{#callable:window_buffer_edit_close_cb}} -->
The `window_buffer_edit_close_cb` function finalizes the editing of a buffer by updating the paste buffer and redrawing the window pane if necessary.
- **Inputs**:
    - `buf`: A pointer to the character buffer containing the new data to be saved.
    - `len`: The length of the buffer `buf`.
    - `arg`: A pointer to a `window_buffer_editdata` structure containing information about the buffer being edited.
- **Control Flow**:
    - Check if `buf` is NULL or `len` is 0; if so, call [`window_buffer_finish_edit`](#window_buffer_finish_edit) and return.
    - Retrieve the paste buffer using `ed->name` and check if it matches `ed->pb`; if not, call [`window_buffer_finish_edit`](#window_buffer_finish_edit) and return.
    - Get the old buffer data and its length from the paste buffer.
    - If the old buffer does not end with a newline and the new buffer does, decrement `len` to remove the newline.
    - If `len` is not zero, replace the paste buffer data with the new buffer data.
    - Find the window pane by `ed->wp_id`; if found, check if the current mode is `window_buffer_mode`.
    - If in `window_buffer_mode`, rebuild and redraw the mode tree using `data->data`.
    - Set the `PANE_REDRAW` flag on the window pane.
    - Call [`window_buffer_finish_edit`](#window_buffer_finish_edit) to clean up the edit data.
- **Output**:
    - The function does not return a value; it performs operations to update the paste buffer and redraw the window pane as needed.
- **Functions called**:
    - [`window_buffer_finish_edit`](#window_buffer_finish_edit)


---
### window_buffer_start_edit<!-- {{#callable:window_buffer_start_edit}} -->
The `window_buffer_start_edit` function initiates the editing of a paste buffer item in a popup editor for a specified client.
- **Inputs**:
    - `data`: A pointer to a `window_buffer_modedata` structure containing mode data for the window buffer.
    - `item`: A pointer to a `window_buffer_itemdata` structure representing the buffer item to be edited.
    - `c`: A pointer to a `client` structure representing the client for whom the edit operation is being performed.
- **Control Flow**:
    - Retrieve the paste buffer associated with the item's name using `paste_get_name`.
    - If the paste buffer is not found, return immediately without performing any operations.
    - Retrieve the data and length of the paste buffer using `paste_buffer_data`.
    - Allocate memory for a `window_buffer_editdata` structure and initialize it with the window pane ID, buffer name, and paste buffer pointer.
    - Invoke `popup_editor` to open the buffer data in a popup editor for the client, passing the buffer data, length, a callback function `window_buffer_edit_close_cb`, and the edit data structure.
    - If `popup_editor` returns a non-zero value, indicating failure, call [`window_buffer_finish_edit`](#window_buffer_finish_edit) to clean up the edit data structure.
- **Output**:
    - The function does not return a value; it performs operations to start editing a buffer item in a popup editor.
- **Functions called**:
    - [`window_buffer_finish_edit`](#window_buffer_finish_edit)


---
### window_buffer_key<!-- {{#callable:window_buffer_key}} -->
The `window_buffer_key` function processes key events in a window buffer mode, handling actions like editing, deleting, and pasting buffer items based on the key pressed.
- **Inputs**:
    - `wme`: A pointer to the `window_mode_entry` structure representing the current window mode entry.
    - `c`: A pointer to the `client` structure representing the client that triggered the key event.
    - `s`: An unused pointer to the `session` structure.
    - `wl`: An unused pointer to the `winlink` structure.
    - `key`: A `key_code` representing the key that was pressed.
    - `m`: A pointer to the `mouse_event` structure representing any mouse event associated with the key press.
- **Control Flow**:
    - Check if the paste buffer is empty; if so, set `finished` to 1 and jump to the `out` label.
    - Call `mode_tree_key` to process the key event and determine if the mode tree should be finished.
    - Use a switch statement to handle specific key actions: 'e' for editing, 'd' for deleting the current item, 'D' for deleting all tagged items, 'P' for pasting all tagged items, and 'p' or '\r' for pasting the current item.
    - If the key action results in finishing the mode or the paste buffer is empty, reset the window pane mode.
    - Otherwise, redraw the mode tree and set the pane's redraw flag.
- **Output**:
    - The function does not return a value but modifies the state of the window buffer mode and potentially the window pane based on the key event.
- **Functions called**:
    - [`window_buffer_start_edit`](#window_buffer_start_edit)
    - [`window_buffer_do_delete`](#window_buffer_do_delete)
    - [`window_buffer_do_paste`](#window_buffer_do_paste)


# Function Declarations (Public API)

---
### window_buffer_init<!-- {{#callable_declaration:window_buffer_init}} -->
Initialize a screen for buffer mode in a window.
- **Description**: This function sets up a screen for buffer mode within a window, initializing necessary data structures and configurations based on the provided arguments. It should be called when entering buffer mode in a window, ensuring that the window mode entry and command find state are properly set up. The function handles default values for format, key format, and command if they are not specified in the arguments. It is important to ensure that the window mode entry and command find state are valid and properly initialized before calling this function.
- **Inputs**:
    - `wme`: A pointer to a window_mode_entry structure representing the current window mode entry. Must not be null and should be properly initialized.
    - `fs`: A pointer to a cmd_find_state structure representing the command find state. Must not be null and should be properly initialized.
    - `args`: A pointer to an args structure containing command-line arguments. Can be null, in which case default values are used for format, key format, and command.
- **Output**: Returns a pointer to a screen structure initialized for buffer mode.
- **See also**: [`window_buffer_init`](#window_buffer_init)  (Implementation)


---
### window_buffer_free<!-- {{#callable_declaration:window_buffer_free}} -->
Releases resources associated with a window mode entry.
- **Description**: This function is used to free all resources associated with a given window mode entry, specifically those related to buffer management. It should be called when the window mode entry is no longer needed to ensure that all allocated memory is properly released. The function checks if the data associated with the window mode entry is non-null before proceeding with the cleanup. It is important to call this function to prevent memory leaks in applications that manage window buffers.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that contains the data to be freed. The pointer must not be null, and the function assumes ownership of the data, freeing all associated resources.
- **Output**: None
- **See also**: [`window_buffer_free`](#window_buffer_free)  (Implementation)


---
### window_buffer_resize<!-- {{#callable_declaration:window_buffer_resize}} -->
Resize the window buffer to the specified dimensions.
- **Description**: This function adjusts the size of the window buffer associated with a given window mode entry to the specified width and height. It should be called whenever the dimensions of the window buffer need to be updated, such as when the window is resized. The function assumes that the window mode entry and its associated data are properly initialized before being passed to it.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the window mode entry whose buffer is to be resized. Must not be null and should be properly initialized.
    - `sx`: An unsigned integer representing the new width of the window buffer. There are no explicit constraints on the value, but it should be a valid size for the intended use.
    - `sy`: An unsigned integer representing the new height of the window buffer. Similar to `sx`, it should be a valid size for the intended use.
- **Output**: None
- **See also**: [`window_buffer_resize`](#window_buffer_resize)  (Implementation)


---
### window_buffer_update<!-- {{#callable_declaration:window_buffer_update}} -->
Update the display and redraw the window pane.
- **Description**: This function is used to refresh the display of a window mode entry by rebuilding and redrawing the mode tree associated with the window buffer. It should be called when the contents of the buffer have changed and need to be reflected in the user interface. The function also marks the window pane for redrawing, ensuring that the visual representation is updated accordingly.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that represents the window mode entry to be updated. This parameter must not be null, and it is expected to have a valid `data` field pointing to a `struct window_buffer_modedata`.
- **Output**: None
- **See also**: [`window_buffer_update`](#window_buffer_update)  (Implementation)


---
### window_buffer_key<!-- {{#callable_declaration:window_buffer_key}} -->
Handles key events in buffer mode.
- **Description**: This function processes key events within a buffer mode context, allowing for operations such as editing, deleting, and pasting buffer items. It should be called when a key event occurs while in buffer mode. The function manages the current selection and tagged items, performing actions based on the key pressed. It requires a valid window mode entry and client, and it assumes that the buffer is not empty. The function may reset the window pane mode or redraw the pane as needed.
- **Inputs**:
    - `wme`: A pointer to a window_mode_entry structure representing the current mode entry. Must not be null.
    - `c`: A pointer to a client structure representing the client triggering the key event. Must not be null.
    - `s`: A pointer to a session structure, which is unused in this function. Can be null.
    - `wl`: A pointer to a winlink structure, which is unused in this function. Can be null.
    - `key`: A key_code representing the key event to handle. Valid key codes correspond to specific buffer operations.
    - `m`: A pointer to a mouse_event structure, which may be used for mouse-related events. Can be null if not applicable.
- **Output**: None
- **See also**: [`window_buffer_key`](#window_buffer_key)  (Implementation)


