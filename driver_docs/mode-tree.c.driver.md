# Purpose
This C source code file is part of the `tmux` project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file implements a tree-based mode for displaying and interacting with hierarchical data structures within `tmux`. It provides functionality for building, displaying, and navigating a tree of items, which can be expanded or collapsed, tagged, and sorted based on various criteria. The code defines several structures, such as `mode_tree_data`, `mode_tree_item`, and `mode_tree_line`, which represent the tree's data, individual items, and lines, respectively. It also includes functions for managing the tree's lifecycle, such as building the tree, handling user input, and rendering the tree on the screen.

The file is not an executable but rather a component of the `tmux` codebase that provides specific functionality related to tree-based data visualization and interaction. It defines internal APIs for managing tree data structures, handling user interactions, and rendering the tree within a `tmux` pane. The code includes callback mechanisms for customizing the behavior of the tree, such as building and drawing the tree, handling key events, and managing search and filter operations. The file also includes a menu system for interacting with tree items, allowing users to perform actions like expanding, collapsing, and tagging items. Overall, this file is a specialized component that enhances `tmux` by providing a structured way to display and interact with hierarchical data.
# Imports and Dependencies

---
- `sys/types.h`
- `ctype.h`
- `stdio.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### mode_tree_menu_items
- **Type**: ``static const struct menu_item[]``
- **Description**: The `mode_tree_menu_items` is a static constant array of `struct menu_item` that defines a set of menu items for a mode tree interface. Each element in the array represents a menu item with a name, a key code, and a NULL pointer for additional data. The array is terminated with a NULL entry to indicate the end of the menu items.
- **Use**: This variable is used to define the menu options available in a mode tree interface, providing functionality for scrolling and canceling actions.


# Data Structures

---
### mode_tree_search_dir
- **Type**: `enum`
- **Members**:
    - `MODE_TREE_SEARCH_FORWARD`: Represents the forward direction for searching in a mode tree.
    - `MODE_TREE_SEARCH_BACKWARD`: Represents the backward direction for searching in a mode tree.
- **Description**: The `mode_tree_search_dir` is an enumeration that defines the possible directions for searching within a mode tree structure. It includes two possible values: `MODE_TREE_SEARCH_FORWARD` for forward searching and `MODE_TREE_SEARCH_BACKWARD` for backward searching. This enumeration is used to control the direction of search operations in the mode tree, allowing for flexible navigation and search functionality.


---
### mode_tree_preview
- **Type**: `enum`
- **Members**:
    - `MODE_TREE_PREVIEW_OFF`: Represents the state where the preview is turned off.
    - `MODE_TREE_PREVIEW_NORMAL`: Represents the state where the preview is in normal mode.
    - `MODE_TREE_PREVIEW_BIG`: Represents the state where the preview is in big mode.
- **Description**: The `mode_tree_preview` enumeration defines three possible states for a preview mode in a tree structure: off, normal, and big. This enum is used to control the display size or visibility of a preview in a user interface, allowing for different levels of detail or screen space usage depending on the selected mode.


---
### mode_tree_data
- **Type**: `struct`
- **Members**:
    - `dead`: Indicates if the mode tree data is inactive or terminated.
    - `references`: Counts the number of references to this data structure.
    - `zoomed`: Indicates if the window pane is zoomed.
    - `wp`: Pointer to the associated window pane.
    - `modedata`: Pointer to custom mode-specific data.
    - `menu`: Pointer to a menu item structure.
    - `sort_list`: Array of strings representing sorting options.
    - `sort_size`: Number of sorting options available.
    - `sort_crit`: Criteria used for sorting the mode tree.
    - `buildcb`: Callback function for building the mode tree.
    - `drawcb`: Callback function for drawing the mode tree.
    - `searchcb`: Callback function for searching within the mode tree.
    - `menucb`: Callback function for handling menu actions.
    - `heightcb`: Callback function for determining the height of the mode tree.
    - `keycb`: Callback function for handling key events.
    - `swapcb`: Callback function for swapping items in the mode tree.
    - `children`: List of child items in the mode tree.
    - `saved`: List of saved items in the mode tree.
    - `line_list`: Array of lines representing the mode tree structure.
    - `line_size`: Number of lines in the mode tree.
    - `depth`: Current depth of the mode tree.
    - `maxdepth`: Maximum depth reached by the mode tree.
    - `width`: Width of the mode tree display.
    - `height`: Height of the mode tree display.
    - `offset`: Offset for scrolling through the mode tree.
    - `current`: Index of the currently selected line in the mode tree.
    - `screen`: Screen structure for rendering the mode tree.
    - `preview`: Indicates the preview mode for the mode tree.
    - `search`: String used for searching within the mode tree.
    - `filter`: String used for filtering the mode tree items.
    - `no_matches`: Indicates if there are no matches for the current filter.
    - `search_dir`: Direction of the search within the mode tree.
- **Description**: The `mode_tree_data` structure is a comprehensive data structure used to manage and display a hierarchical mode tree within a window pane. It includes various fields for managing the state, sorting, and display of the tree, as well as callback functions for building, drawing, searching, and interacting with the tree. The structure supports features like zooming, filtering, and previewing, and maintains lists of child and saved items, as well as a line list for rendering the tree. It is designed to be flexible and extensible, allowing for complex interactions and customizations within the mode tree.


---
### mode_tree_item
- **Type**: `struct`
- **Members**:
    - `parent`: Pointer to the parent mode_tree_item.
    - `itemdata`: Pointer to the data associated with this item.
    - `line`: Line number associated with this item.
    - `key`: Key code associated with this item.
    - `keystr`: String representation of the key code.
    - `keylen`: Length of the key string.
    - `tag`: Unique identifier for the item.
    - `name`: Name of the item.
    - `text`: Additional text associated with the item.
    - `expanded`: Flag indicating if the item is expanded.
    - `tagged`: Flag indicating if the item is tagged.
    - `draw_as_parent`: Flag indicating if the item should be drawn as a parent.
    - `no_tag`: Flag indicating if the item should not be tagged.
    - `align`: Alignment setting for the item's name.
    - `children`: List of child mode_tree_items.
    - `entry`: Entry in the tail queue for mode_tree_items.
- **Description**: The `mode_tree_item` structure represents a node in a tree-like data structure used for managing hierarchical data in a mode tree. Each item can have a parent, a list of children, and associated data such as a name, text, and a unique tag. The structure also includes flags for managing the item's state, such as whether it is expanded or tagged, and settings for how it should be displayed, such as alignment and key bindings. This structure is part of a larger system for managing and displaying hierarchical data in a user interface, likely within a terminal or text-based application.


---
### mode_tree_line
- **Type**: `struct`
- **Members**:
    - `item`: A pointer to a `mode_tree_item` structure, representing the item associated with this line.
    - `depth`: An unsigned integer representing the depth of the line in the tree structure.
    - `last`: An integer indicating whether this line is the last child in its level (1 if true, 0 otherwise).
    - `flat`: An integer indicating whether the line is part of a flat structure (1 if true, 0 otherwise).
- **Description**: The `mode_tree_line` structure is used to represent a single line within a mode tree, which is a hierarchical data structure used in the tmux application. Each line corresponds to an item in the tree and contains information about its depth, whether it is the last item at its level, and whether it is part of a flat structure. This structure is essential for managing and displaying the hierarchical relationships between items in the mode tree.


---
### mode_tree_menu
- **Type**: `struct`
- **Members**:
    - `data`: A pointer to a `mode_tree_data` structure, which holds the main data for the mode tree.
    - `c`: A pointer to a `client` structure, representing the client associated with the menu.
    - `line`: An unsigned integer representing the current line in the menu.
- **Description**: The `mode_tree_menu` structure is used to represent a menu within a mode tree in the tmux application. It contains a reference to the main mode tree data, a client associated with the menu, and the current line being displayed or interacted with in the menu. This structure is part of the larger mode tree system, which is used to manage and display hierarchical data structures within tmux.


# Functions

---
### mode_tree_find_item<!-- {{#callable:mode_tree_find_item}} -->
The [`mode_tree_find_item`](#mode_tree_find_item) function recursively searches a tree structure for an item with a specific tag and returns it.
- **Inputs**:
    - `mtl`: A pointer to a `mode_tree_list` structure, which is a list of `mode_tree_item` structures representing the tree to search.
    - `tag`: A `uint64_t` value representing the tag of the item to find in the tree.
- **Control Flow**:
    - Iterate over each `mode_tree_item` in the `mode_tree_list` using `TAILQ_FOREACH` macro.
    - Check if the current item's tag matches the given tag; if so, return the current item.
    - If the current item has children, recursively call [`mode_tree_find_item`](#mode_tree_find_item) on the children list.
    - If a matching item is found in the children, return it.
    - If no matching item is found, return `NULL`.
- **Output**:
    - Returns a pointer to the `mode_tree_item` with the matching tag, or `NULL` if no such item is found.
- **Functions called**:
    - [`mode_tree_find_item`](#mode_tree_find_item)


---
### mode_tree_free_item<!-- {{#callable:mode_tree_free_item}} -->
The `mode_tree_free_item` function deallocates memory associated with a `mode_tree_item` structure, including its children and associated strings.
- **Inputs**:
    - `mti`: A pointer to a `mode_tree_item` structure that is to be freed.
- **Control Flow**:
    - The function first calls [`mode_tree_free_items`](#mode_tree_free_items) to recursively free all child items in the `children` list of the given `mode_tree_item`.
    - It then frees the memory allocated for the `name`, `text`, and `keystr` strings of the `mode_tree_item`.
    - Finally, it frees the memory allocated for the `mode_tree_item` itself.
- **Output**:
    - The function does not return any value; it performs memory deallocation for the specified `mode_tree_item` and its associated resources.
- **Functions called**:
    - [`mode_tree_free_items`](#mode_tree_free_items)


---
### mode_tree_free_items<!-- {{#callable:mode_tree_free_items}} -->
The `mode_tree_free_items` function iterates over a list of mode tree items, removes each item from the list, and frees the memory associated with each item.
- **Inputs**:
    - `mtl`: A pointer to a `mode_tree_list` structure, which is a tail queue of `mode_tree_item` structures.
- **Control Flow**:
    - The function uses `TAILQ_FOREACH_SAFE` to iterate over each `mode_tree_item` in the `mode_tree_list` pointed to by `mtl`, allowing safe removal of items during iteration.
    - For each item `mti` in the list, it removes `mti` from the list using `TAILQ_REMOVE`.
    - It then calls [`mode_tree_free_item`](#mode_tree_free_item) on `mti` to free the memory associated with the item.
- **Output**:
    - The function does not return any value; it performs its operations directly on the input list and its items.
- **Functions called**:
    - [`mode_tree_free_item`](#mode_tree_free_item)


---
### mode_tree_check_selected<!-- {{#callable:mode_tree_check_selected}} -->
The function `mode_tree_check_selected` adjusts the offset of a mode tree data structure if the current line is off-screen.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure, which contains information about the mode tree, including the current line, height, and offset.
- **Control Flow**:
    - Check if the current line (`mtd->current`) is greater than the last visible line (`mtd->height - 1`).
    - If true, set the offset (`mtd->offset`) to ensure the current line is visible by adjusting it to `mtd->current - mtd->height + 1`.
- **Output**:
    - The function does not return a value; it modifies the `offset` field of the `mode_tree_data` structure to ensure the current line is visible.


---
### mode_tree_clear_lines<!-- {{#callable:mode_tree_clear_lines}} -->
The `mode_tree_clear_lines` function deallocates memory for the line list in a `mode_tree_data` structure and resets its size and pointer.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure, which contains the line list to be cleared.
- **Control Flow**:
    - The function calls `free` on `mtd->line_list` to deallocate the memory used by the line list.
    - It sets `mtd->line_list` to `NULL` to indicate that the line list is no longer allocated.
    - It sets `mtd->line_size` to `0` to reflect that there are no lines in the list.
- **Output**:
    - The function does not return any value; it modifies the `mode_tree_data` structure in place.


---
### mode_tree_build_lines<!-- {{#callable:mode_tree_build_lines}} -->
The [`mode_tree_build_lines`](#mode_tree_build_lines) function constructs a list of lines representing a hierarchical tree structure, updating depth and key information for each item.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure that holds the state and configuration of the mode tree.
    - `mtl`: A pointer to a `mode_tree_list` structure representing the current list of tree items to process.
    - `depth`: An unsigned integer representing the current depth level in the tree hierarchy.
- **Control Flow**:
    - Set the current depth in `mtd` and update `mtd->maxdepth` if the current depth is greater.
    - Iterate over each `mode_tree_item` in the `mtl` list using `TAILQ_FOREACH`.
    - For each item, reallocate the `line_list` in `mtd` to accommodate a new line and update the line's properties such as item reference, depth, and last status.
    - Check if the item has children; if so, set `flat` to 0 and recursively call [`mode_tree_build_lines`](#mode_tree_build_lines) if the item is expanded.
    - Determine the key for the item using a callback if available, or assign a default key based on the line index.
    - Store the key string and its length if a valid key is assigned.
    - After processing all items, iterate again to set the `flat` property for each line in `line_list` based on the `flat` variable.
- **Output**:
    - The function does not return a value but modifies the `mtd` structure to reflect the updated list of lines and their properties.
- **Functions called**:
    - [`mode_tree_build_lines`](#mode_tree_build_lines)


---
### mode_tree_clear_tagged<!-- {{#callable:mode_tree_clear_tagged}} -->
The [`mode_tree_clear_tagged`](#mode_tree_clear_tagged) function recursively clears the 'tagged' status of all items in a mode tree list and its children.
- **Inputs**:
    - `mtl`: A pointer to a `mode_tree_list` structure, which is a list of `mode_tree_item` structures representing the nodes of a tree.
- **Control Flow**:
    - Iterate over each `mode_tree_item` in the `mode_tree_list` using `TAILQ_FOREACH`.
    - Set the `tagged` attribute of the current `mode_tree_item` to 0, effectively untagging it.
    - Recursively call [`mode_tree_clear_tagged`](#mode_tree_clear_tagged) on the `children` list of the current `mode_tree_item` to clear the 'tagged' status of all its descendants.
- **Output**:
    - The function does not return a value; it modifies the 'tagged' status of items in the provided mode tree list in place.
- **Functions called**:
    - [`mode_tree_clear_tagged`](#mode_tree_clear_tagged)


---
### mode_tree_up<!-- {{#callable:mode_tree_up}} -->
The `mode_tree_up` function navigates upwards in a mode tree structure, adjusting the current position and offset based on the wrap parameter.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure, which contains information about the current state of the mode tree, including the current position, line size, and offset.
    - `wrap`: An integer flag indicating whether the navigation should wrap around when reaching the top of the list (1 for wrap, 0 for no wrap).
- **Control Flow**:
    - Check if the current position (`mtd->current`) is at the top (0).
    - If at the top and wrap is enabled, set the current position to the last line (`mtd->line_size - 1`) and adjust the offset if the line size exceeds the height.
    - If not at the top, decrement the current position.
    - If the new current position is less than the offset, decrement the offset.
- **Output**:
    - The function does not return a value; it modifies the `mtd` structure in place to update the current position and offset.


---
### mode_tree_down<!-- {{#callable:mode_tree_down}} -->
The `mode_tree_down` function moves the current selection down one line in a mode tree, with optional wrapping to the top if the end is reached.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure representing the current state of the mode tree.
    - `wrap`: An integer flag indicating whether to wrap the selection to the top when the end is reached (non-zero to enable wrapping).
- **Control Flow**:
    - Check if the current line is the last line in the mode tree (`mtd->current == mtd->line_size - 1`).
    - If it is the last line and wrapping is enabled (`wrap` is non-zero), set the current line to the first line (`mtd->current = 0`) and reset the offset (`mtd->offset = 0`).
    - If it is the last line and wrapping is not enabled, return 0 to indicate no movement was made.
    - If it is not the last line, increment the current line (`mtd->current++`).
    - Check if the new current line exceeds the visible area (`mtd->current > mtd->offset + mtd->height - 1`), and if so, increment the offset (`mtd->offset++`).
    - Return 1 to indicate successful movement.
- **Output**:
    - Returns 1 if the current line was successfully moved down, or 0 if no movement was made due to reaching the end without wrapping.


---
### mode_tree_swap<!-- {{#callable:mode_tree_swap}} -->
The `mode_tree_swap` function swaps the current item in a mode tree with another item at the same depth in a specified direction, if a swap callback is defined.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure representing the mode tree data.
    - `direction`: An integer indicating the direction to swap the current item, where a positive value moves forward and a negative value moves backward.
- **Control Flow**:
    - Check if the swap callback (`swapcb`) is defined; if not, return immediately.
    - Initialize `current_depth` to the depth of the current line in the mode tree.
    - Set `swap_with` to the current index and enter a loop to find the next line at the same depth with the same parent in the specified direction.
    - In the loop, check if moving in the specified direction would go out of bounds; if so, return.
    - Increment `swap_with` by `direction` and update `swap_with_depth` to the depth of the line at `swap_with`.
    - Continue the loop until a line with a depth less than or equal to `current_depth` is found.
    - If the depth of the found line is not equal to `current_depth`, return.
    - Call the swap callback with the item data of the current and found lines; if the callback returns true, update the current index to `swap_with` and rebuild the mode tree.
- **Output**:
    - The function does not return a value; it modifies the `current` index of the `mode_tree_data` structure and potentially rebuilds the mode tree.
- **Functions called**:
    - [`mode_tree_build`](#mode_tree_build)


---
### mode_tree_get_current<!-- {{#callable:mode_tree_get_current}} -->
The `mode_tree_get_current` function retrieves the data associated with the currently selected item in a mode tree structure.
- **Inputs**:
    - `mtd`: A pointer to a `struct mode_tree_data` which represents the mode tree data structure containing the current state and items of the mode tree.
- **Control Flow**:
    - Access the `line_list` array within the `mode_tree_data` structure using the `current` index to get the current line.
    - Retrieve the `item` from the current line and access its `itemdata` field.
    - Return the `itemdata` which represents the data associated with the current item.
- **Output**:
    - A void pointer to the data associated with the currently selected item in the mode tree.


---
### mode_tree_get_current_name<!-- {{#callable:mode_tree_get_current_name}} -->
The function `mode_tree_get_current_name` retrieves the name of the currently selected item in a mode tree data structure.
- **Inputs**:
    - `mtd`: A pointer to a `struct mode_tree_data` which contains the mode tree data, including the list of lines and the current selection index.
- **Control Flow**:
    - Access the `line_list` array of the `mode_tree_data` structure using the `current` index to get the current line.
    - Retrieve the `item` from the current line, which is a pointer to a `mode_tree_item`.
    - Return the `name` field of the `mode_tree_item`.
- **Output**:
    - Returns a `const char *` which is the name of the currently selected item in the mode tree.


---
### mode_tree_expand_current<!-- {{#callable:mode_tree_expand_current}} -->
The `mode_tree_expand_current` function expands the currently selected item in a mode tree if it is not already expanded and then rebuilds the mode tree.
- **Inputs**:
    - `mtd`: A pointer to a `struct mode_tree_data` which contains the mode tree's data, including the list of items and the current selection.
- **Control Flow**:
    - Check if the current item in the mode tree (pointed to by `mtd->current`) is not expanded.
    - If the item is not expanded, set its `expanded` property to 1 (true).
    - Call `mode_tree_build(mtd)` to rebuild the mode tree with the updated expansion state.
- **Output**:
    - The function does not return a value; it modifies the state of the mode tree by expanding the current item and rebuilding the tree.
- **Functions called**:
    - [`mode_tree_build`](#mode_tree_build)


---
### mode_tree_collapse_current<!-- {{#callable:mode_tree_collapse_current}} -->
The `mode_tree_collapse_current` function collapses the currently selected item in a mode tree if it is expanded.
- **Inputs**:
    - `mtd`: A pointer to a `struct mode_tree_data` which contains the mode tree data, including the list of items and the current selection.
- **Control Flow**:
    - Check if the current item in the mode tree (pointed by `mtd->current`) is expanded.
    - If the item is expanded, set its `expanded` property to 0 (collapsed).
    - Call `mode_tree_build(mtd)` to rebuild the mode tree structure after collapsing the item.
- **Output**:
    - The function does not return a value; it modifies the state of the mode tree data structure in place.
- **Functions called**:
    - [`mode_tree_build`](#mode_tree_build)


---
### mode_tree_get_tag<!-- {{#callable:mode_tree_get_tag}} -->
The `mode_tree_get_tag` function searches for a specific tag within a list of mode tree lines and returns whether it was found, updating the found index if successful.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure, which contains the list of mode tree lines to search through.
    - `tag`: A `uint64_t` value representing the tag to search for within the mode tree lines.
    - `found`: A pointer to a `u_int` where the index of the found tag will be stored if the tag is found.
- **Control Flow**:
    - Initialize a loop variable `i` to iterate over the `line_list` in `mtd` up to `line_size`.
    - For each line, check if the `tag` of the current line's item matches the input `tag`.
    - If a match is found, break out of the loop.
    - After the loop, check if `i` is not equal to `line_size`, indicating a match was found.
    - If a match was found, set `*found` to `i` and return 1.
    - If no match was found, return 0.
- **Output**:
    - Returns 1 if the tag is found and updates `*found` with the index; returns 0 if the tag is not found.


---
### mode_tree_expand<!-- {{#callable:mode_tree_expand}} -->
The `mode_tree_expand` function expands a specific item in a mode tree structure if it is not already expanded, based on a given tag.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure representing the mode tree.
    - `tag`: A `uint64_t` value representing the unique tag of the item to be expanded.
- **Control Flow**:
    - Call [`mode_tree_get_tag`](#mode_tree_get_tag) to find the index of the item with the specified tag in the mode tree's line list.
    - If the item is found and is not already expanded, set its `expanded` property to 1.
    - Call [`mode_tree_build`](#mode_tree_build) to rebuild the mode tree structure.
- **Output**:
    - The function does not return a value; it modifies the mode tree structure in place.
- **Functions called**:
    - [`mode_tree_get_tag`](#mode_tree_get_tag)
    - [`mode_tree_build`](#mode_tree_build)


---
### mode_tree_set_current<!-- {{#callable:mode_tree_set_current}} -->
The `mode_tree_set_current` function sets the current item in a mode tree to the item with a specified tag, adjusting the offset if necessary, and returns whether the operation was successful.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure representing the mode tree.
    - `tag`: A `uint64_t` value representing the tag of the item to be set as current.
- **Control Flow**:
    - The function first attempts to find the item with the specified tag using [`mode_tree_get_tag`](#mode_tree_get_tag) and stores the result in `found`.
    - If the item is found, it sets `mtd->current` to `found` and adjusts `mtd->offset` based on whether the current item is beyond the visible height of the tree, then returns 1.
    - If the item is not found and `mtd->current` is greater than or equal to `mtd->line_size`, it sets `mtd->current` to the last item and adjusts `mtd->offset` similarly, then returns 0.
    - If the item is not found and `mtd->current` is within bounds, it simply returns 0.
- **Output**:
    - Returns an integer: 1 if the item with the specified tag was found and set as current, 0 otherwise.
- **Functions called**:
    - [`mode_tree_get_tag`](#mode_tree_get_tag)


---
### mode_tree_count_tagged<!-- {{#callable:mode_tree_count_tagged}} -->
The `mode_tree_count_tagged` function counts the number of tagged items in a mode tree data structure.
- **Inputs**:
    - `mtd`: A pointer to a `struct mode_tree_data` which contains the mode tree data structure, including a list of lines each associated with a mode tree item.
- **Control Flow**:
    - Initialize a counter `tagged` to zero.
    - Iterate over each line in the `line_list` of the `mode_tree_data` structure, using the `line_size` to determine the number of iterations.
    - For each line, retrieve the associated `mode_tree_item`.
    - Check if the `tagged` field of the `mode_tree_item` is set (non-zero).
    - If the item is tagged, increment the `tagged` counter.
    - After iterating through all lines, return the `tagged` counter.
- **Output**:
    - The function returns an unsigned integer representing the count of tagged items in the mode tree.


---
### mode_tree_each_tagged<!-- {{#callable:mode_tree_each_tagged}} -->
The `mode_tree_each_tagged` function iterates over tagged items in a mode tree and applies a callback function to each, with an option to apply the callback to the current item if no tagged items are found.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure representing the mode tree data.
    - `cb`: A callback function of type `mode_tree_each_cb` to be applied to each tagged item.
    - `c`: A pointer to a `client` structure, representing the client context for the callback.
    - `key`: A `key_code` value representing a key code to be passed to the callback.
    - `current`: An integer flag indicating whether to apply the callback to the current item if no tagged items are found.
- **Control Flow**:
    - Initialize a variable `fired` to 0 to track if any tagged items are found.
    - Iterate over each item in the `line_list` of the `mode_tree_data` structure.
    - For each item, check if it is tagged by evaluating the `tagged` field of the `mode_tree_item` structure.
    - If an item is tagged, set `fired` to 1 and call the callback function `cb` with the mode tree's `modedata`, the item's `itemdata`, the client `c`, and the key `key`.
    - After the loop, if no tagged items were found (`fired` is 0) and the `current` flag is set, call the callback function `cb` on the current item in the mode tree.
- **Output**:
    - The function does not return a value; it performs actions via the callback function on tagged items or the current item.


---
### mode_tree_start<!-- {{#callable:mode_tree_start}} -->
The `mode_tree_start` function initializes and returns a new `mode_tree_data` structure for managing a tree-like data structure in a window pane, with various callbacks and configurations.
- **Inputs**:
    - `wp`: A pointer to a `window_pane` structure representing the window pane where the mode tree will be displayed.
    - `args`: A pointer to an `args` structure containing command-line arguments that influence the mode tree's behavior.
    - `buildcb`: A callback function of type `mode_tree_build_cb` used to build the mode tree.
    - `drawcb`: A callback function of type `mode_tree_draw_cb` used to draw the mode tree.
    - `searchcb`: A callback function of type `mode_tree_search_cb` used for searching within the mode tree.
    - `menucb`: A callback function of type `mode_tree_menu_cb` used for handling menu interactions.
    - `heightcb`: A callback function of type `mode_tree_height_cb` used to determine the height of the mode tree.
    - `keycb`: A callback function of type `mode_tree_key_cb` used to handle key events.
    - `swapcb`: A callback function of type `mode_tree_swap_cb` used to handle swapping of items in the mode tree.
    - `modedata`: A pointer to user-defined data associated with the mode tree.
    - `menu`: A pointer to a `menu_item` structure representing the menu items available in the mode tree.
    - `sort_list`: An array of strings representing the sorting criteria available for the mode tree.
    - `sort_size`: An unsigned integer representing the number of sorting criteria in `sort_list`.
    - `s`: A double pointer to a `screen` structure where the mode tree will be rendered.
- **Control Flow**:
    - Allocate memory for a new `mode_tree_data` structure and initialize its reference count to 1.
    - Assign the provided `window_pane`, `modedata`, and `menu` to the new `mode_tree_data` structure.
    - Set the sorting list and size from the provided `sort_list` and `sort_size`.
    - Determine the preview mode based on the presence of the 'N' argument in `args`.
    - Set the sorting criteria field based on the 'O' argument in `args` and the provided `sort_list`.
    - Set the sorting criteria to be reversed if the 'r' argument is present in `args`.
    - Set the filter string if the 'f' argument is present in `args`, otherwise set it to NULL.
    - Assign the provided callback functions to the corresponding fields in the `mode_tree_data` structure.
    - Initialize the `children` list in the `mode_tree_data` structure.
    - Initialize the provided `screen` pointer and set its mode to disable the cursor.
    - Return the initialized `mode_tree_data` structure.
- **Output**:
    - Returns a pointer to the newly initialized `mode_tree_data` structure.


---
### mode_tree_zoom<!-- {{#callable:mode_tree_zoom}} -->
The `mode_tree_zoom` function manages the zoom state of a window pane within a mode tree, toggling zoom based on the presence of a specific argument.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure representing the mode tree data, which includes the window pane to be zoomed.
    - `args`: A pointer to an `args` structure containing command-line arguments, specifically checking for the presence of the 'Z' flag to determine zoom action.
- **Control Flow**:
    - Retrieve the window pane (`wp`) from the `mode_tree_data` structure (`mtd`).
    - Check if the 'Z' flag is present in the `args` structure using `args_has(args, 'Z')`.
    - If the 'Z' flag is present, set `mtd->zoomed` to the current zoom state of the window (`wp->window->flags & WINDOW_ZOOMED`).
    - If the window is not already zoomed (`!mtd->zoomed`) and `window_zoom(wp)` returns 0, call `server_redraw_window(wp->window)` to redraw the window.
    - If the 'Z' flag is not present, set `mtd->zoomed` to -1.
- **Output**:
    - The function does not return a value; it modifies the `zoomed` state within the `mode_tree_data` structure and potentially triggers a window redraw.


---
### mode_tree_set_height<!-- {{#callable:mode_tree_set_height}} -->
The `mode_tree_set_height` function adjusts the height of a mode tree display based on the screen size and preview mode settings.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure, which contains information about the mode tree, including its current height, preview mode, and callback functions.
- **Control Flow**:
    - Check if a custom height callback (`heightcb`) is provided in `mtd`; if so, use it to calculate the height and adjust `mtd->height` accordingly.
    - If no custom height callback is provided, determine the height based on the `preview` mode in `mtd`.
    - For `MODE_TREE_PREVIEW_NORMAL`, set `mtd->height` to two-thirds of the screen height, adjusting further if it exceeds `mtd->line_size` or is less than 10.
    - For `MODE_TREE_PREVIEW_BIG`, set `mtd->height` to one-fourth of the screen height, adjusting further if it exceeds `mtd->line_size` or is less than 2.
    - If the preview mode is off, set `mtd->height` to the full screen height.
    - Ensure that the height is not set such that the remaining screen space is less than 2 lines.
- **Output**:
    - The function does not return a value; it modifies the `height` field of the `mode_tree_data` structure pointed to by `mtd`.


---
### mode_tree_build<!-- {{#callable:mode_tree_build}} -->
The `mode_tree_build` function constructs and updates a tree-like data structure for a user interface, managing its display and interaction properties.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure, which contains the state and configuration of the mode tree.
- **Control Flow**:
    - Initialize a `screen` pointer `s` to the screen within `mtd`.
    - Determine the current tag from `mtd->line_list` if it exists, otherwise set it to `UINT64_MAX`.
    - Concatenate the `children` list into the `saved` list and reinitialize `children`.
    - Invoke the `buildcb` callback to populate `children` based on the current sort criteria and filter.
    - Check if `children` is empty and, if so, call `buildcb` again with a null filter.
    - Free the items in `saved` and reinitialize it.
    - Clear the current lines in `mtd` and reset `maxdepth`.
    - Build the lines for the tree using [`mode_tree_build_lines`](#mode_tree_build_lines).
    - If `line_list` is not null and the tag is `UINT64_MAX`, update the tag to the current item's tag.
    - Set the current item in the tree using [`mode_tree_set_current`](#mode_tree_set_current).
    - Update the width of the tree to the screen's width.
    - Adjust the height of the tree based on the preview mode and screen size.
    - Check and adjust the selected item to ensure it is visible.
- **Output**:
    - The function does not return a value; it modifies the `mode_tree_data` structure in place to reflect the updated tree state.
- **Functions called**:
    - [`mode_tree_free_items`](#mode_tree_free_items)
    - [`mode_tree_clear_lines`](#mode_tree_clear_lines)
    - [`mode_tree_build_lines`](#mode_tree_build_lines)
    - [`mode_tree_set_current`](#mode_tree_set_current)
    - [`mode_tree_set_height`](#mode_tree_set_height)
    - [`mode_tree_check_selected`](#mode_tree_check_selected)


---
### mode_tree_remove_ref<!-- {{#callable:mode_tree_remove_ref}} -->
The `mode_tree_remove_ref` function decrements the reference count of a `mode_tree_data` structure and frees it if the count reaches zero.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure whose reference count is to be decremented.
- **Control Flow**:
    - Decrement the `references` field of the `mode_tree_data` structure pointed to by `mtd`.
    - Check if the `references` field is zero after decrementing.
    - If the `references` field is zero, free the memory allocated for the `mode_tree_data` structure.
- **Output**:
    - The function does not return any value.


---
### mode_tree_free<!-- {{#callable:mode_tree_free}} -->
The `mode_tree_free` function deallocates and cleans up resources associated with a `mode_tree_data` structure.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure that needs to be freed.
- **Control Flow**:
    - Retrieve the `window_pane` associated with the `mode_tree_data` structure.
    - If the `zoomed` flag is not set, call `server_unzoom_window` to unzoom the window associated with the `window_pane`.
    - Call [`mode_tree_free_items`](#mode_tree_free_items) to free all items in the `children` list of the `mode_tree_data`.
    - Call [`mode_tree_clear_lines`](#mode_tree_clear_lines) to clear the lines associated with the `mode_tree_data`.
    - Free the screen resources associated with the `mode_tree_data`.
    - Free the `search` and `filter` strings if they are allocated.
    - Set the `dead` flag of the `mode_tree_data` to 1 to indicate it is no longer in use.
    - Call [`mode_tree_remove_ref`](#mode_tree_remove_ref) to decrease the reference count and potentially free the `mode_tree_data` structure.
- **Output**:
    - The function does not return a value; it performs cleanup operations on the `mode_tree_data` structure.
- **Functions called**:
    - [`mode_tree_free_items`](#mode_tree_free_items)
    - [`mode_tree_clear_lines`](#mode_tree_clear_lines)
    - [`mode_tree_remove_ref`](#mode_tree_remove_ref)


---
### mode_tree_resize<!-- {{#callable:mode_tree_resize}} -->
The `mode_tree_resize` function resizes the screen associated with a mode tree, rebuilds the mode tree, redraws it, and marks the window pane for redraw.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure, which contains information about the mode tree and its associated screen.
    - `sx`: An unsigned integer representing the new width of the screen.
    - `sy`: An unsigned integer representing the new height of the screen.
- **Control Flow**:
    - Retrieve the screen associated with the mode tree data structure `mtd`.
    - Call `screen_resize` to resize the screen to the new dimensions `sx` and `sy`.
    - Invoke [`mode_tree_build`](#mode_tree_build) to rebuild the mode tree structure after resizing.
    - Call [`mode_tree_draw`](#mode_tree_draw) to redraw the mode tree on the resized screen.
    - Set the `PANE_REDRAW` flag on the window pane to indicate that it needs to be redrawn.
- **Output**:
    - This function does not return a value; it performs operations on the `mode_tree_data` structure and its associated screen.
- **Functions called**:
    - [`mode_tree_build`](#mode_tree_build)
    - [`mode_tree_draw`](#mode_tree_draw)


---
### mode_tree_add<!-- {{#callable:mode_tree_add}} -->
The `mode_tree_add` function creates and adds a new item to a mode tree, initializing its properties and inserting it into the appropriate position in the tree structure.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure representing the mode tree to which the item will be added.
    - `parent`: A pointer to a `mode_tree_item` structure representing the parent item under which the new item will be added; can be NULL if the item is a root item.
    - `itemdata`: A void pointer to the data associated with the new item.
    - `tag`: A `uint64_t` value representing a unique identifier for the new item.
    - `name`: A constant character pointer to the name of the new item.
    - `text`: A constant character pointer to additional text associated with the new item; can be NULL.
    - `expanded`: An integer indicating whether the new item should be expanded (1), collapsed (0), or inherit the state from a saved item (-1).
- **Control Flow**:
    - Log the function call with the tag, name, and text of the new item.
    - Allocate memory for a new `mode_tree_item` structure and initialize its parent and itemdata fields.
    - Set the tag and name fields of the new item, duplicating the name string.
    - If text is provided, duplicate it and set the text field of the new item.
    - Search for a saved item with the same tag in the saved list of the mode tree data.
    - If a saved item is found, copy its tagged and expanded states to the new item if applicable.
    - If no saved item is found and expanded is -1, set the new item to expanded; otherwise, use the provided expanded value.
    - Initialize the children list of the new item.
    - Insert the new item into the children list of the parent if provided, or into the root children list of the mode tree data if no parent is provided.
    - Return a pointer to the newly created `mode_tree_item`.
- **Output**:
    - Returns a pointer to the newly created `mode_tree_item` structure.
- **Functions called**:
    - [`mode_tree_find_item`](#mode_tree_find_item)


---
### mode_tree_draw_as_parent<!-- {{#callable:mode_tree_draw_as_parent}} -->
The function `mode_tree_draw_as_parent` sets the `draw_as_parent` flag of a `mode_tree_item` to 1, indicating that the item should be drawn as a parent in a mode tree structure.
- **Inputs**:
    - `mti`: A pointer to a `mode_tree_item` structure whose `draw_as_parent` flag is to be set.
- **Control Flow**:
    - The function takes a single argument, a pointer to a `mode_tree_item` structure.
    - It sets the `draw_as_parent` member of the `mode_tree_item` structure to 1.
- **Output**:
    - The function does not return any value; it modifies the `draw_as_parent` field of the input `mode_tree_item` structure.


---
### mode_tree_no_tag<!-- {{#callable:mode_tree_no_tag}} -->
The `mode_tree_no_tag` function sets the `no_tag` attribute of a `mode_tree_item` structure to 1, indicating that the item should not be tagged.
- **Inputs**:
    - `mti`: A pointer to a `mode_tree_item` structure whose `no_tag` attribute is to be set.
- **Control Flow**:
    - The function takes a pointer to a `mode_tree_item` structure as an argument.
    - It sets the `no_tag` attribute of the `mode_tree_item` structure to 1.
- **Output**:
    - The function does not return any value.


---
### mode_tree_align<!-- {{#callable:mode_tree_align}} -->
The `mode_tree_align` function sets the alignment of a mode tree item's name.
- **Inputs**:
    - `mti`: A pointer to a `mode_tree_item` structure whose name alignment is to be set.
    - `align`: An integer indicating the alignment direction: -1 for left, 0 for no alignment, and 1 for right alignment.
- **Control Flow**:
    - The function takes a `mode_tree_item` pointer and an integer alignment value as parameters.
    - It assigns the alignment value to the `align` field of the `mode_tree_item` structure.
- **Output**:
    - The function does not return any value.


---
### mode_tree_remove<!-- {{#callable:mode_tree_remove}} -->
The `mode_tree_remove` function removes a specified item from a mode tree and frees its associated memory.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure representing the mode tree from which the item will be removed.
    - `mti`: A pointer to a `mode_tree_item` structure representing the item to be removed from the mode tree.
- **Control Flow**:
    - Retrieve the parent of the item `mti` to be removed.
    - Check if the parent is not NULL; if true, remove `mti` from the parent's children list using `TAILQ_REMOVE`.
    - If the parent is NULL, remove `mti` from the root children list of `mtd` using `TAILQ_REMOVE`.
    - Call [`mode_tree_free_item`](#mode_tree_free_item) to free the memory associated with `mti`.
- **Output**:
    - The function does not return a value; it performs the removal and memory deallocation as a side effect.
- **Functions called**:
    - [`mode_tree_free_item`](#mode_tree_free_item)


---
### mode_tree_draw<!-- {{#callable:mode_tree_draw}} -->
The `mode_tree_draw` function renders a tree-like structure on a screen, handling the display of items, their alignment, and additional preview information if enabled.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure containing the data and state necessary for drawing the tree, including the list of items, screen dimensions, and style options.
- **Control Flow**:
    - Check if `mtd->line_size` is zero and return if true, as there is nothing to draw.
    - Initialize grid cell styles and screen dimensions from `mtd`.
    - Start a screen write context and clear the screen.
    - Calculate the maximum key length for alignment purposes.
    - Determine the maximum alignment length for each depth level in the tree.
    - Iterate over each line in `mtd->line_list` within the visible range, adjusting the cursor position and preparing the text to be displayed for each item.
    - For each item, determine the appropriate symbol to display based on its expansion state and whether it has children.
    - Format the text for each item, including its key, symbol, name, and any additional text, ensuring it fits within the screen width.
    - Draw each line on the screen, applying different styles if the item is tagged or is the current item.
    - If preview mode is enabled and conditions are met, draw a preview box below the tree with additional information about the current item.
    - Finalize the screen write context and update the cursor position to the current item.
- **Output**:
    - The function does not return a value; it directly modifies the screen buffer associated with the `mode_tree_data` structure to display the tree.


---
### mode_tree_search_backward<!-- {{#callable:mode_tree_search_backward}} -->
The `mode_tree_search_backward` function searches backward through a tree structure to find an item matching a search criterion.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure, which contains the tree data and search parameters.
- **Control Flow**:
    - Check if the search string in `mtd` is NULL; if so, return NULL.
    - Initialize `mti` and `last` to the current item in the line list of `mtd`.
    - Enter an infinite loop to traverse the tree backward.
    - Attempt to get the previous item in the list; if found, navigate to the last child of this subtree.
    - If no previous item is found, move to the parent of the current item.
    - If the current item becomes NULL, set it to the last child of the last root subtree.
    - If the current item is the same as the starting item (`last`), break the loop.
    - If no custom search callback is provided, check if the item's name contains the search string; if so, return the item.
    - If a custom search callback is provided, use it to determine if the item matches the search criteria; if so, return the item.
    - If no matching item is found after traversing the entire tree, return NULL.
- **Output**:
    - A pointer to a `mode_tree_item` that matches the search criteria, or NULL if no match is found.


---
### mode_tree_search_forward<!-- {{#callable:mode_tree_search_forward}} -->
The `mode_tree_search_forward` function searches through a tree structure starting from the current item and returns the first item that matches a given search string or satisfies a custom search callback.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure, which contains the tree data and search parameters.
- **Control Flow**:
    - Check if the search string in `mtd` is NULL; if so, return NULL immediately.
    - Initialize `mti` and `last` to the current item in the tree based on `mtd->current`.
    - Enter a loop to traverse the tree structure.
    - If the current item has children, move to the first child.
    - If there are no children, attempt to move to the next sibling using `TAILQ_NEXT`.
    - If there are no siblings, move up to the parent and attempt to find the next sibling of the parent, repeating until a sibling is found or the root is reached.
    - If the root is reached and no next item is found, start again from the first child of the root.
    - If the current item is the same as the starting item (`last`), break the loop to avoid infinite traversal.
    - If a custom search callback is not provided, check if the current item's name contains the search string; if it does, return the current item.
    - If a custom search callback is provided, use it to determine if the current item matches the search criteria; if it does, return the current item.
    - If no matching item is found after traversing the entire tree, return NULL.
- **Output**:
    - Returns a pointer to the first `mode_tree_item` that matches the search criteria, or NULL if no match is found.


---
### mode_tree_search_set<!-- {{#callable:mode_tree_search_set}} -->
The `mode_tree_search_set` function performs a search in a mode tree data structure, expands the path to the found item, rebuilds the tree, sets the current item, and redraws the pane.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure, which contains the mode tree's state and configuration.
- **Control Flow**:
    - Check the search direction in `mtd->search_dir` and call either [`mode_tree_search_forward`](#mode_tree_search_forward) or [`mode_tree_search_backward`](#mode_tree_search_backward) to find the next matching item, storing the result in `mti`.
    - If no matching item is found (`mti` is NULL), return immediately.
    - Retrieve the tag of the found item (`mti->tag`) and store it in `tag`.
    - Iterate through the parent chain of `mti`, setting each parent's `expanded` field to 1 to ensure the path to the found item is expanded.
    - Call [`mode_tree_build`](#mode_tree_build) to rebuild the mode tree structure.
    - Call [`mode_tree_set_current`](#mode_tree_set_current) with `tag` to set the found item as the current item in the mode tree.
    - Call [`mode_tree_draw`](#mode_tree_draw) to redraw the mode tree display.
    - Set the `PANE_REDRAW` flag in `mtd->wp->flags` to indicate that the pane needs to be redrawn.
- **Output**:
    - The function does not return a value; it modifies the state of the `mode_tree_data` structure and updates the display.
- **Functions called**:
    - [`mode_tree_search_forward`](#mode_tree_search_forward)
    - [`mode_tree_search_backward`](#mode_tree_search_backward)
    - [`mode_tree_build`](#mode_tree_build)
    - [`mode_tree_set_current`](#mode_tree_set_current)
    - [`mode_tree_draw`](#mode_tree_draw)


---
### mode_tree_search_callback<!-- {{#callable:mode_tree_search_callback}} -->
The `mode_tree_search_callback` function handles search input for a mode tree, updating the search term and triggering a search operation if the mode tree is not marked as dead.
- **Inputs**:
    - `c`: A pointer to a `client` structure, which is unused in this function.
    - `data`: A pointer to a `mode_tree_data` structure, which contains the state and data for the mode tree.
    - `s`: A constant character pointer representing the search string input.
    - `done`: An integer indicating if the search operation is complete, which is unused in this function.
- **Control Flow**:
    - Check if the mode tree is marked as dead; if so, return 0 immediately.
    - Free the current search string stored in the mode tree data.
    - If the input search string `s` is NULL or empty, set the search string in the mode tree data to NULL and return 0.
    - Duplicate the input search string `s` and store it in the mode tree data's search field.
    - Call [`mode_tree_search_set`](#mode_tree_search_set) to perform the search operation with the updated search string.
- **Output**:
    - The function returns an integer value of 0, indicating the completion of the callback operation.
- **Functions called**:
    - [`mode_tree_search_set`](#mode_tree_search_set)


---
### mode_tree_search_free<!-- {{#callable:mode_tree_search_free}} -->
The `mode_tree_search_free` function decreases the reference count of a `mode_tree_data` object and frees it if the count reaches zero.
- **Inputs**:
    - `data`: A pointer to a `mode_tree_data` object whose reference count is to be decreased.
- **Control Flow**:
    - The function calls [`mode_tree_remove_ref`](#mode_tree_remove_ref) with the `data` pointer.
    - [`mode_tree_remove_ref`](#mode_tree_remove_ref) decreases the reference count of the `mode_tree_data` object pointed to by `data`.
    - If the reference count reaches zero, the `mode_tree_data` object is freed.
- **Output**:
    - The function does not return any value.
- **Functions called**:
    - [`mode_tree_remove_ref`](#mode_tree_remove_ref)


---
### mode_tree_filter_callback<!-- {{#callable:mode_tree_filter_callback}} -->
The `mode_tree_filter_callback` function updates the filter string for a mode tree and triggers a rebuild and redraw of the tree.
- **Inputs**:
    - `c`: A pointer to a `client` structure, which is unused in this function.
    - `data`: A pointer to a `mode_tree_data` structure, which contains the mode tree's state and data.
    - `s`: A constant character pointer representing the new filter string to be applied to the mode tree.
    - `done`: An integer indicating whether the operation is complete, which is unused in this function.
- **Control Flow**:
    - Check if the mode tree is marked as dead; if so, return 0 immediately.
    - If the current filter is not NULL, free the existing filter string.
    - If the input string `s` is NULL or empty, set the filter to NULL; otherwise, duplicate the string `s` and assign it to the filter.
    - Call [`mode_tree_build`](#mode_tree_build) to rebuild the mode tree with the new filter.
    - Call [`mode_tree_draw`](#mode_tree_draw) to redraw the mode tree.
    - Set the `PANE_REDRAW` flag on the associated window pane to indicate that it needs to be redrawn.
- **Output**:
    - The function returns an integer value of 0, indicating successful execution.
- **Functions called**:
    - [`mode_tree_build`](#mode_tree_build)
    - [`mode_tree_draw`](#mode_tree_draw)


---
### mode_tree_filter_free<!-- {{#callable:mode_tree_filter_free}} -->
The `mode_tree_filter_free` function decreases the reference count of a `mode_tree_data` object and frees it if the count reaches zero.
- **Inputs**:
    - `data`: A pointer to a `mode_tree_data` object whose reference count is to be decreased.
- **Control Flow**:
    - The function calls [`mode_tree_remove_ref`](#mode_tree_remove_ref) with the `data` pointer.
    - Inside [`mode_tree_remove_ref`](#mode_tree_remove_ref), the reference count of the `mode_tree_data` object is decremented.
    - If the reference count reaches zero, the `mode_tree_data` object is freed.
- **Output**:
    - The function does not return any value.
- **Functions called**:
    - [`mode_tree_remove_ref`](#mode_tree_remove_ref)


---
### mode_tree_menu_callback<!-- {{#callable:mode_tree_menu_callback}} -->
The `mode_tree_menu_callback` function handles menu interactions in a mode tree by processing a key event and updating the current line or invoking a callback, then cleaning up resources.
- **Inputs**:
    - `menu`: A pointer to a `menu` structure, which is unused in this function.
    - `idx`: An unsigned integer representing the index of the menu item, which is unused in this function.
    - `key`: A `key_code` representing the key that was pressed.
    - `data`: A pointer to a `mode_tree_menu` structure containing the mode tree data and client information.
- **Control Flow**:
    - Retrieve the `mode_tree_menu` and `mode_tree_data` structures from the `data` pointer.
    - Check if the mode tree is marked as dead or if the key is `KEYC_NONE`; if so, jump to the cleanup section labeled `out`.
    - Check if the line number in `mtm` is greater than or equal to the line size in `mtd`; if so, jump to the cleanup section labeled `out`.
    - Set the current line in `mtd` to the line number in `mtm`.
    - Invoke the menu callback function `menucb` with the mode data, client, and key.
    - In the cleanup section `out`, call [`mode_tree_remove_ref`](#mode_tree_remove_ref) to decrease the reference count of `mtd` and free the `mtm` structure.
- **Output**:
    - The function does not return a value; it performs actions based on the key event and cleans up resources.
- **Functions called**:
    - [`mode_tree_remove_ref`](#mode_tree_remove_ref)


---
### mode_tree_display_menu<!-- {{#callable:mode_tree_display_menu}} -->
The `mode_tree_display_menu` function displays a menu for a specific item in a mode tree, either using a custom menu or a default set of menu items, depending on the context.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure, which contains the mode tree's state and data.
    - `c`: A pointer to a `client` structure, representing the client for which the menu is being displayed.
    - `x`: An unsigned integer representing the x-coordinate where the menu should be displayed.
    - `y`: An unsigned integer representing the y-coordinate where the menu should be displayed.
    - `outside`: An integer flag indicating whether the menu is being displayed outside the normal context (non-zero) or not (zero).
- **Control Flow**:
    - Calculate the line index based on the current offset and y-coordinate, defaulting to the current line if out of bounds.
    - Retrieve the mode tree item (`mti`) for the calculated line.
    - Determine the menu items and title based on whether the menu is outside the normal context.
    - Create a menu with the determined title and add the menu items to it.
    - Allocate and initialize a `mode_tree_menu` structure to store the menu context.
    - Adjust the x-coordinate to center the menu if necessary.
    - Display the menu using `menu_display`, passing the `mode_tree_menu` structure for callback handling.
    - If menu display fails, clean up by removing a reference from `mtd`, freeing the `mode_tree_menu` structure, and freeing the menu.
- **Output**:
    - The function does not return a value, but it displays a menu on the client interface and manages the associated resources and references.
- **Functions called**:
    - [`mode_tree_remove_ref`](#mode_tree_remove_ref)


---
### mode_tree_key<!-- {{#callable:mode_tree_key}} -->
The `mode_tree_key` function processes keyboard and mouse input events to navigate and manipulate a tree-like data structure in a user interface.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure representing the current state of the mode tree.
    - `c`: A pointer to a `client` structure representing the client interacting with the mode tree.
    - `key`: A pointer to a `key_code` representing the key or mouse event being processed.
    - `m`: A pointer to a `mouse_event` structure containing details about a mouse event, if applicable.
    - `xp`: A pointer to an unsigned integer to store the x-coordinate of a mouse event, if applicable.
    - `yp`: A pointer to an unsigned integer to store the y-coordinate of a mouse event, if applicable.
- **Control Flow**:
    - Check if the input is a mouse event and handle mouse-specific logic, including updating coordinates and displaying context menus.
    - If the input is a key event, iterate through the list of mode tree lines to find a matching key and update the current selection if found.
    - Handle specific key commands for navigation (e.g., up, down, home, end), tagging, sorting, expanding/collapsing nodes, and searching within the mode tree.
    - Return 1 if the key event is a quit command ('q', 'ESC', or 'Ctrl+g'), otherwise return 0.
- **Output**:
    - The function returns an integer indicating whether the operation was a quit command (1) or not (0).
- **Functions called**:
    - [`mode_tree_display_menu`](#mode_tree_display_menu)
    - [`mode_tree_up`](#mode_tree_up)
    - [`mode_tree_down`](#mode_tree_down)
    - [`mode_tree_swap`](#mode_tree_swap)
    - [`mode_tree_clear_tagged`](#mode_tree_clear_tagged)
    - [`mode_tree_build`](#mode_tree_build)
    - [`mode_tree_search_set`](#mode_tree_search_set)
    - [`mode_tree_check_selected`](#mode_tree_check_selected)


---
### mode_tree_run_command<!-- {{#callable:mode_tree_run_command}} -->
The `mode_tree_run_command` function executes a command generated from a template and a name, handling any errors that occur during parsing.
- **Inputs**:
    - `c`: A pointer to a `struct client`, representing the client context in which the command is executed.
    - `fs`: A pointer to a `struct cmd_find_state`, representing the state used to find the command context.
    - `template`: A constant character pointer representing the command template to be used.
    - `name`: A constant character pointer representing the name to replace in the template.
- **Control Flow**:
    - The function begins by replacing placeholders in the `template` with the `name` using `cmd_template_replace`, storing the result in `command`.
    - If `command` is not NULL and not empty, a new command queue state is created using `cmdq_new_state`.
    - The command is parsed and appended to the command queue using `cmd_parse_and_append`, and the status is checked.
    - If a parse error occurs (`CMD_PARSE_ERROR`), the error message is capitalized and displayed to the client using `status_message_set`.
    - The error message memory is freed if it was allocated.
    - The command queue state is freed using `cmdq_free_state`.
    - Finally, the memory allocated for `command` is freed.
- **Output**:
    - The function does not return a value; it performs actions based on the command execution and error handling.


# Function Declarations (Public API)

---
### mode_tree_free_items<!-- {{#callable_declaration:mode_tree_free_items}} -->
Frees all items in a mode tree list.
- **Description**: Use this function to release all resources associated with the items in a mode tree list. It should be called when the list is no longer needed to prevent memory leaks. The function iterates over each item in the list, removes it, and frees its associated memory. Ensure that the list is properly initialized before calling this function.
- **Inputs**:
    - `mtl`: A pointer to a `mode_tree_list` structure representing the list of mode tree items to be freed. Must not be null, and should point to a valid, initialized list. The function will handle an empty list gracefully.
- **Output**: None
- **See also**: [`mode_tree_free_items`](#mode_tree_free_items)  (Implementation)


