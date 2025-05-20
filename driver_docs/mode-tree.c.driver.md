# Purpose
This C source code file is part of the `tmux` project, specifically implementing a mode tree structure used for displaying hierarchical data within a terminal multiplexer environment. The code provides a comprehensive set of functionalities to manage and interact with a tree-like data structure, which is used to represent and navigate through nested items, such as sessions, windows, or panes in `tmux`. The file defines several key structures, including `mode_tree_data`, `mode_tree_item`, and `mode_tree_line`, which collectively manage the state and behavior of the tree, including item expansion, selection, and tagging.

The code includes functions for building, drawing, and interacting with the mode tree, such as [`mode_tree_build`](#mode_tree_build), [`mode_tree_draw`](#mode_tree_draw), and [`mode_tree_key`](#mode_tree_key). These functions handle the rendering of the tree on the screen, user input processing, and dynamic updates to the tree's structure. The file also defines callback mechanisms for custom behaviors, such as sorting, searching, and handling key events, allowing for flexible integration with other parts of the `tmux` system. Additionally, the code supports features like search and filter functionalities, preview modes, and menu interactions, enhancing the user experience when navigating complex data structures within `tmux`.
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
- **Use**: This variable is used to define the menu options available in a mode tree interface, allowing users to interact with the interface using specific key codes.


# Data Structures

---
### mode_tree_search_dir
- **Type**: `enum`
- **Members**:
    - `MODE_TREE_SEARCH_FORWARD`: Represents the forward search direction in the mode tree.
    - `MODE_TREE_SEARCH_BACKWARD`: Represents the backward search direction in the mode tree.
- **Description**: The `mode_tree_search_dir` is an enumeration that defines the possible directions for searching within a mode tree structure. It includes two possible values: `MODE_TREE_SEARCH_FORWARD` for forward searching and `MODE_TREE_SEARCH_BACKWARD` for backward searching. This enumeration is used to control the direction of search operations within the mode tree, allowing for flexible navigation and search functionality.


---
### mode_tree_preview
- **Type**: `enum`
- **Members**:
    - `MODE_TREE_PREVIEW_OFF`: Represents the state where the mode tree preview is turned off.
    - `MODE_TREE_PREVIEW_NORMAL`: Represents the state where the mode tree preview is in normal mode.
    - `MODE_TREE_PREVIEW_BIG`: Represents the state where the mode tree preview is in big mode.
- **Description**: The `mode_tree_preview` enumeration defines three possible states for the preview mode of a mode tree: off, normal, and big. This enumeration is used to control the display size of the preview in a mode tree, allowing for different levels of detail or visibility based on the selected state.


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
- **Description**: The `mode_tree_data` structure is a comprehensive data structure used to manage and display a hierarchical tree of items within a window pane in a terminal multiplexer environment. It includes various fields for managing the state and behavior of the tree, such as sorting, searching, and filtering capabilities. The structure also supports callbacks for custom actions like building, drawing, and handling user interactions with the tree. It maintains a list of child items, tracks the current selection, and manages the display properties like width, height, and scrolling offset. Additionally, it supports preview modes and can handle complex interactions through a series of callback functions.


---
### mode_tree_item
- **Type**: `struct`
- **Members**:
    - `parent`: Pointer to the parent mode_tree_item in the tree structure.
    - `itemdata`: Pointer to the data associated with this tree item.
    - `line`: Unsigned integer representing the line number of the item.
    - `key`: Key code associated with this item for navigation or selection.
    - `keystr`: String representation of the key code.
    - `keylen`: Length of the key string.
    - `tag`: Unique identifier for the item, used for searching and referencing.
    - `name`: Name of the item, used for display and identification.
    - `text`: Additional text associated with the item, often used for display.
    - `expanded`: Flag indicating whether the item's children are expanded in the view.
    - `tagged`: Flag indicating whether the item is tagged for selection or action.
    - `draw_as_parent`: Flag indicating if the item should be drawn as a parent node.
    - `no_tag`: Flag indicating if the item should not be tagged.
    - `align`: Alignment setting for the item's name display.
    - `children`: List of child mode_tree_items, forming a tree structure.
    - `entry`: Tail queue entry for linking items in a list.
- **Description**: The `mode_tree_item` structure represents a node in a hierarchical tree used in the tmux application. Each node can have a parent and multiple children, forming a tree structure. It contains metadata such as a unique tag, name, and text for display purposes, as well as flags for controlling its display state (e.g., expanded, tagged). The structure also includes key-related fields for user interaction, allowing navigation and selection within the tree. The `children` member is a list of child nodes, enabling the recursive nature of the tree, while the `entry` member facilitates the integration of the node into a linked list.


---
### mode_tree_line
- **Type**: `struct`
- **Members**:
    - `item`: Pointer to a mode_tree_item structure associated with this line.
    - `depth`: Unsigned integer representing the depth of the line in the tree.
    - `last`: Integer flag indicating if this line is the last child in its level.
    - `flat`: Integer flag indicating if the line is part of a flat structure.
- **Description**: The `mode_tree_line` structure represents a single line in a mode tree, which is a hierarchical data structure used to display and manage tree-like data in a user interface. Each line corresponds to an item in the tree and contains information about its depth, whether it is the last item at its level, and whether it is part of a flat structure. This structure is used to facilitate the rendering and interaction with the tree in a user interface, such as in a terminal application.


---
### mode_tree_menu
- **Type**: `struct`
- **Members**:
    - `data`: A pointer to a `mode_tree_data` structure, representing the data associated with the menu.
    - `c`: A pointer to a `client` structure, representing the client associated with the menu.
    - `line`: An unsigned integer representing the line number in the menu.
- **Description**: The `mode_tree_menu` structure is a simple data structure used to represent a menu within a mode tree context. It contains a pointer to the `mode_tree_data` structure, which holds the data for the mode tree, a pointer to a `client` structure, which represents the client interacting with the menu, and an unsigned integer `line` that indicates the current line number in the menu. This structure is part of a larger system for managing and displaying hierarchical data in a tree-like format, often used in terminal-based applications like tmux.


# Functions

---
### mode_tree_find_item<!-- {{#callable:mode_tree_find_item}} -->
The [`mode_tree_find_item`](#mode_tree_find_item) function recursively searches a tree structure for an item with a specific tag and returns it.
- **Inputs**:
    - `mtl`: A pointer to a `mode_tree_list` structure, which is a list of `mode_tree_item` structures representing the tree to search.
    - `tag`: A `uint64_t` value representing the tag of the item to find in the tree.
- **Control Flow**:
    - The function iterates over each `mode_tree_item` in the `mode_tree_list` using `TAILQ_FOREACH` macro.
    - For each item, it checks if the item's `tag` matches the given `tag`; if so, it returns the item.
    - If the tag does not match, it recursively calls [`mode_tree_find_item`](#mode_tree_find_item) on the item's children to search deeper in the tree.
    - If a matching item is found in the children, it returns that item.
    - If no matching item is found, the function returns `NULL`.
- **Output**:
    - A pointer to the `mode_tree_item` with the matching tag, or `NULL` if no such item is found.
- **Functions called**:
    - [`mode_tree_find_item`](#mode_tree_find_item)


---
### mode_tree_free_item<!-- {{#callable:mode_tree_free_item}} -->
The `mode_tree_free_item` function deallocates memory associated with a `mode_tree_item` structure and its children.
- **Inputs**:
    - `mti`: A pointer to a `mode_tree_item` structure that needs to be freed.
- **Control Flow**:
    - The function first calls [`mode_tree_free_items`](#mode_tree_free_items) to recursively free all child items of the given `mode_tree_item`.
    - It then frees the memory allocated for the `name`, `text`, and `keystr` fields of the `mode_tree_item`.
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
    - If true, set the offset (`mtd->offset`) to ensure the current line is visible by adjusting it to the last visible line.
- **Output**:
    - The function does not return a value; it modifies the `offset` field of the `mode_tree_data` structure in place.


---
### mode_tree_clear_lines<!-- {{#callable:mode_tree_clear_lines}} -->
The `mode_tree_clear_lines` function deallocates and resets the line list in a `mode_tree_data` structure.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure whose line list is to be cleared.
- **Control Flow**:
    - The function begins by freeing the memory allocated to `mtd->line_list` using the `free` function.
    - It then sets `mtd->line_list` to `NULL` to indicate that the list is now empty.
    - Finally, it sets `mtd->line_size` to 0, indicating that there are no lines in the list.
- **Output**:
    - This function does not return any value; it modifies the `mode_tree_data` structure in place.


---
### mode_tree_build_lines<!-- {{#callable:mode_tree_build_lines}} -->
The [`mode_tree_build_lines`](#mode_tree_build_lines) function constructs a list of lines representing a hierarchical tree structure, updating depth and key information for each item.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure that holds the state and configuration of the mode tree.
    - `mtl`: A pointer to a `mode_tree_list` structure, which is a list of `mode_tree_item` structures representing the current level of the tree.
    - `depth`: An unsigned integer representing the current depth level in the tree hierarchy.
- **Control Flow**:
    - Set the current depth in `mtd` and update `maxdepth` if the current depth is greater.
    - Iterate over each `mode_tree_item` in the `mtl` list using `TAILQ_FOREACH`.
    - For each item, reallocate the `line_list` in `mtd` to accommodate a new line and update the `line_size`.
    - Create a new `mode_tree_line` for the current item, setting its `item`, `depth`, and `last` properties.
    - Assign a line number to the current item and check if it has children to determine if the tree is flat.
    - If the item is expanded, recursively call [`mode_tree_build_lines`](#mode_tree_build_lines) for its children with an incremented depth.
    - Determine the key for the item using a callback if available, or by assigning a default key based on the line number.
    - Store the key string and its length in the item if a valid key is assigned.
    - After processing all items, iterate again to set the `flat` property for each line based on the presence of children.
- **Output**:
    - The function does not return a value; it modifies the `mode_tree_data` structure to reflect the current state of the tree lines.
- **Functions called**:
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`mode_tree_build_lines`](#mode_tree_build_lines)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`key_string_lookup_key`](key-string.c.driver.md#key_string_lookup_key)


---
### mode_tree_clear_tagged<!-- {{#callable:mode_tree_clear_tagged}} -->
The [`mode_tree_clear_tagged`](#mode_tree_clear_tagged) function recursively clears the 'tagged' status of all items in a mode tree list and its children.
- **Inputs**:
    - `mtl`: A pointer to a `mode_tree_list` structure, which is a list of `mode_tree_item` structures representing the nodes of a tree.
- **Control Flow**:
    - The function iterates over each `mode_tree_item` in the `mode_tree_list` using the `TAILQ_FOREACH` macro.
    - For each `mode_tree_item`, it sets the `tagged` field to 0, effectively untagging the item.
    - The function then recursively calls itself on the `children` list of the current `mode_tree_item`, ensuring that all descendant items are also untagged.
- **Output**:
    - The function does not return any value; it modifies the `tagged` status of items in the provided `mode_tree_list` and its children in place.
- **Functions called**:
    - [`mode_tree_clear_tagged`](#mode_tree_clear_tagged)


---
### mode_tree_up<!-- {{#callable:mode_tree_up}} -->
The `mode_tree_up` function navigates upwards in a mode tree structure, adjusting the current position and offset based on the wrap parameter.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure representing the current state of the mode tree.
    - `wrap`: An integer flag indicating whether to wrap around when reaching the top of the list.
- **Control Flow**:
    - Check if the current position (`mtd->current`) is at the top (0).
    - If at the top and `wrap` is true, set `mtd->current` to the last line (`mtd->line_size - 1`) and adjust `mtd->offset` if necessary.
    - If not at the top, decrement `mtd->current` to move up one line.
    - If the new `mtd->current` is less than `mtd->offset`, decrement `mtd->offset` to keep the current line visible.
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
    - Returns 1 if the current line was successfully moved down, or 0 if it was at the last line and wrapping was not enabled.


---
### mode_tree_swap<!-- {{#callable:mode_tree_swap}} -->
The `mode_tree_swap` function swaps the current line in a mode tree with another line at the same depth in a specified direction, if a swap callback is defined.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure representing the mode tree data.
    - `direction`: An integer indicating the direction to swap the current line; positive for forward and negative for backward.
- **Control Flow**:
    - Initialize `current_depth` with the depth of the current line in the mode tree.
    - Check if the swap callback (`swapcb`) is NULL; if so, return immediately as no swap can be performed.
    - Set `swap_with` to the current line index and enter a loop to find the next line at the same depth in the specified direction.
    - In the loop, check if moving in the specified direction would go out of bounds; if so, return.
    - Increment `swap_with` by `direction` and update `swap_with_depth` with the depth of the new line.
    - Continue the loop until a line with a depth less than or equal to `current_depth` is found.
    - If the depth of the found line is not equal to `current_depth`, return as no valid swap target was found.
    - Call the swap callback function with the item data of the current line and the found line.
    - If the swap callback returns true, update the current line index to `swap_with` and rebuild the mode tree.
- **Output**:
    - The function does not return a value; it modifies the mode tree data structure in place if a swap is performed.
- **Functions called**:
    - [`mode_tree_build`](#mode_tree_build)


---
### mode_tree_get_current<!-- {{#callable:mode_tree_get_current}} -->
The `mode_tree_get_current` function retrieves the data associated with the currently selected item in a mode tree structure.
- **Inputs**:
    - `mtd`: A pointer to a `struct mode_tree_data` which represents the mode tree data structure containing the list of items and the current selection index.
- **Control Flow**:
    - Access the `line_list` array within the `mode_tree_data` structure using the `current` index to get the current line.
    - Retrieve the `itemdata` from the `item` field of the current line's `mode_tree_item`.
    - Return the `itemdata` pointer.
- **Output**:
    - A void pointer to the `itemdata` of the currently selected item in the mode tree.


---
### mode_tree_get_current_name<!-- {{#callable:mode_tree_get_current_name}} -->
The function `mode_tree_get_current_name` retrieves the name of the currently selected item in a mode tree data structure.
- **Inputs**:
    - `mtd`: A pointer to a `struct mode_tree_data` which contains the mode tree data, including the list of lines and the current selection index.
- **Control Flow**:
    - Access the `line_list` array within the `mode_tree_data` structure using the `current` index to get the current line.
    - Retrieve the `item` from the current line, which is a pointer to a `mode_tree_item`.
    - Return the `name` field of the `mode_tree_item`, which is a string representing the name of the current item.
- **Output**:
    - The function returns a `const char *`, which is the name of the currently selected item in the mode tree.


---
### mode_tree_expand_current<!-- {{#callable:mode_tree_expand_current}} -->
The `mode_tree_expand_current` function expands the currently selected item in a mode tree if it is not already expanded and then rebuilds the mode tree.
- **Inputs**:
    - `mtd`: A pointer to a `struct mode_tree_data` which contains the mode tree data, including the list of items and the current selection.
- **Control Flow**:
    - Check if the current item in the mode tree (pointed to by `mtd->current`) is not expanded.
    - If the item is not expanded, set its `expanded` property to 1 (true).
    - Call `mode_tree_build(mtd)` to rebuild the mode tree with the updated expansion state.
- **Output**:
    - This function does not return a value; it modifies the state of the mode tree data structure in place.
- **Functions called**:
    - [`mode_tree_build`](#mode_tree_build)


---
### mode_tree_collapse_current<!-- {{#callable:mode_tree_collapse_current}} -->
The `mode_tree_collapse_current` function collapses the currently selected item in a mode tree if it is expanded.
- **Inputs**:
    - `mtd`: A pointer to a `struct mode_tree_data` which contains the mode tree data, including the list of items and the current selection.
- **Control Flow**:
    - Check if the current item in the mode tree (pointed to by `mtd->current`) is expanded.
    - If the item is expanded, set its `expanded` property to 0 (collapsed).
    - Call `mode_tree_build(mtd)` to rebuild the mode tree structure after collapsing the item.
- **Output**:
    - This function does not return a value; it modifies the state of the mode tree data structure in place.
- **Functions called**:
    - [`mode_tree_build`](#mode_tree_build)


---
### mode_tree_get_tag<!-- {{#callable:mode_tree_get_tag}} -->
The `mode_tree_get_tag` function searches for a specific tag within a list of mode tree lines and returns whether it was found, updating the index of the found item if successful.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure, which contains the list of mode tree lines to search through.
    - `tag`: A `uint64_t` value representing the tag to search for within the mode tree lines.
    - `found`: A pointer to a `u_int` where the index of the found item will be stored if the tag is found.
- **Control Flow**:
    - Initialize a loop variable `i` to iterate over the mode tree lines.
    - Loop through each line in `mtd->line_list` up to `mtd->line_size`.
    - Check if the `tag` of the current line's item matches the input `tag`.
    - If a match is found, break out of the loop.
    - After the loop, check if `i` is not equal to `mtd->line_size`, indicating a match was found.
    - If a match was found, set `*found` to `i` and return 1.
    - If no match was found, return 0.
- **Output**:
    - Returns 1 if the tag is found and updates `*found` with the index; otherwise, returns 0.


---
### mode_tree_expand<!-- {{#callable:mode_tree_expand}} -->
The `mode_tree_expand` function expands a specific item in a mode tree structure if it is not already expanded, based on a given tag.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure, which contains the mode tree's state and data.
    - `tag`: A `uint64_t` value representing the unique identifier of the item to be expanded in the mode tree.
- **Control Flow**:
    - The function first calls [`mode_tree_get_tag`](#mode_tree_get_tag) to find the index of the item with the specified tag in the `line_list` array of the `mode_tree_data` structure.
    - If the item is not found, the function returns immediately without making any changes.
    - If the item is found and is not already expanded, the function sets the `expanded` flag of the item to 1.
    - After setting the item as expanded, the function calls [`mode_tree_build`](#mode_tree_build) to rebuild the mode tree structure.
- **Output**:
    - The function does not return any value; it modifies the state of the `mode_tree_data` structure by expanding the specified item and rebuilding the tree.
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
    - If the item is not found, it checks if `mtd->current` is beyond the last item in the list (`mtd->line_size`).
    - If `mtd->current` is beyond the last item, it sets `mtd->current` to the last item and adjusts `mtd->offset` similarly, then returns 0.
    - If `mtd->current` is within bounds, it simply returns 0.
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
    - Iterate over each line in the `line_list` of the `mode_tree_data` structure using a for loop.
    - For each line, retrieve the associated `mode_tree_item`.
    - Check if the `tagged` field of the `mode_tree_item` is set to true.
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
    - For each item, check if it is tagged by evaluating the `tagged` field of the `mode_tree_item`.
    - If an item is tagged, set `fired` to 1 and call the callback function `cb` with the mode tree's `modedata`, the item's `itemdata`, the client `c`, and the key `key`.
    - After the loop, if no tagged items were found (`fired` is 0) and `current` is true, apply the callback to the current item in the mode tree.
- **Output**:
    - The function does not return a value; it performs actions via the callback function on tagged items or the current item.


---
### mode_tree_start<!-- {{#callable:mode_tree_start}} -->
The `mode_tree_start` function initializes and returns a new `mode_tree_data` structure for managing a tree-like data structure in a window pane.
- **Inputs**:
    - `wp`: A pointer to a `window_pane` structure representing the window pane where the mode tree will be displayed.
    - `args`: A pointer to an `args` structure containing command-line arguments that influence the mode tree's behavior.
    - `buildcb`: A callback function of type `mode_tree_build_cb` used to build the mode tree.
    - `drawcb`: A callback function of type `mode_tree_draw_cb` used to draw the mode tree.
    - `searchcb`: A callback function of type `mode_tree_search_cb` used for searching within the mode tree.
    - `menucb`: A callback function of type `mode_tree_menu_cb` used for handling menu interactions.
    - `heightcb`: A callback function of type `mode_tree_height_cb` used to determine the height of the mode tree.
    - `keycb`: A callback function of type `mode_tree_key_cb` used to handle key events.
    - `swapcb`: A callback function of type `mode_tree_swap_cb` used to swap items in the mode tree.
    - `modedata`: A pointer to user-defined data associated with the mode tree.
    - `menu`: A pointer to a `menu_item` structure representing the menu associated with the mode tree.
    - `sort_list`: An array of strings representing the sorting criteria for the mode tree.
    - `sort_size`: An unsigned integer representing the number of sorting criteria in `sort_list`.
    - `s`: A double pointer to a `screen` structure where the mode tree will be displayed.
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
    - Initialize the provided `screen` structure and set its mode to disable the cursor.
    - Return the initialized `mode_tree_data` structure.
- **Output**:
    - Returns a pointer to the newly initialized `mode_tree_data` structure.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`args_get`](arguments.c.driver.md#args_get)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`screen_init`](screen.c.driver.md#screen_init)


---
### mode_tree_zoom<!-- {{#callable:mode_tree_zoom}} -->
The `mode_tree_zoom` function manages the zoom state of a window pane within a mode tree, toggling zoom based on the presence of a specific argument.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure, which contains information about the mode tree and the associated window pane.
    - `args`: A pointer to an `args` structure, which holds command-line arguments, including flags that determine the function's behavior.
- **Control Flow**:
    - Retrieve the window pane associated with the mode tree data (`mtd->wp`).
    - Check if the 'Z' flag is present in the `args` using `args_has(args, 'Z')`.
    - If the 'Z' flag is present, set `mtd->zoomed` to the current zoom state of the window (`wp->window->flags & WINDOW_ZOOMED`).
    - If the window is not already zoomed (`!mtd->zoomed`) and `window_zoom(wp)` returns 0, call `server_redraw_window(wp->window)` to redraw the window.
    - If the 'Z' flag is not present, set `mtd->zoomed` to -1.
- **Output**:
    - The function does not return a value; it modifies the `zoomed` state of the `mode_tree_data` structure and potentially redraws the window.
- **Functions called**:
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`window_zoom`](window.c.driver.md#window_zoom)
    - [`server_redraw_window`](server-fn.c.driver.md#server_redraw_window)


---
### mode_tree_set_height<!-- {{#callable:mode_tree_set_height}} -->
The `mode_tree_set_height` function calculates and sets the height of a mode tree display based on the screen size and preview mode, with optional custom height adjustment.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure, which contains information about the mode tree, including its screen, height, preview mode, and optional height callback function.
- **Control Flow**:
    - Retrieve the screen associated with the mode tree data (`mtd`).
    - Check if a custom height callback (`heightcb`) is provided in `mtd`.
    - If a custom height callback is provided, call it to get a custom height and adjust `mtd->height` accordingly.
    - If no custom height callback is provided, determine the height based on the preview mode (`preview`) in `mtd`.
    - For `MODE_TREE_PREVIEW_NORMAL`, set `mtd->height` to two-thirds of the screen height, with adjustments if it exceeds `line_size` or is less than 10.
    - For `MODE_TREE_PREVIEW_BIG`, set `mtd->height` to one-fourth of the screen height, with adjustments if it exceeds `line_size` or is less than 2.
    - For other preview modes, set `mtd->height` to the full screen height.
    - Ensure that the height does not result in less than 2 lines of space on the screen by adjusting `mtd->height` if necessary.
- **Output**:
    - The function does not return a value; it modifies the `height` field of the `mode_tree_data` structure pointed to by `mtd`.


---
### mode_tree_build<!-- {{#callable:mode_tree_build}} -->
The `mode_tree_build` function constructs and updates a tree structure for a user interface, handling sorting, filtering, and layout adjustments.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure that contains the state and configuration for the mode tree.
- **Control Flow**:
    - Retrieve the current tag from the line list if available, otherwise set it to `UINT64_MAX`.
    - Concatenate the current children list to the saved list and reinitialize the children list.
    - Invoke the build callback function to populate the children list based on the current sort criteria and filter.
    - Check if there are no matches in the children list; if so, call the build callback again with a null filter.
    - Free the items in the saved list and reinitialize it.
    - Clear the current line list and reset the maximum depth.
    - Rebuild the line list from the children list starting at depth 0.
    - If the line list is not null and the tag is `UINT64_MAX`, update the tag with the current item's tag.
    - Set the current item in the mode tree using the updated tag.
    - Update the width of the mode tree to match the screen's width.
    - Adjust the height of the mode tree based on the preview mode setting.
    - Check and adjust the selected item to ensure it is visible on the screen.
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
    - Check if the `references` field is now zero.
    - If it is zero, free the memory allocated for the `mode_tree_data` structure.
- **Output**:
    - The function does not return any value; it performs an operation on the `mode_tree_data` structure.


---
### mode_tree_free<!-- {{#callable:mode_tree_free}} -->
The `mode_tree_free` function deallocates and cleans up resources associated with a `mode_tree_data` structure.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure that needs to be freed.
- **Control Flow**:
    - Retrieve the `window_pane` associated with the `mode_tree_data` structure.
    - If the `zoomed` flag is not set, call [`server_unzoom_window`](server-fn.c.driver.md#server_unzoom_window) to unzoom the window associated with the `window_pane`.
    - Call [`mode_tree_free_items`](#mode_tree_free_items) to free all items in the `children` list of the `mode_tree_data`.
    - Call [`mode_tree_clear_lines`](#mode_tree_clear_lines) to clear the lines associated with the `mode_tree_data`.
    - Call [`screen_free`](screen.c.driver.md#screen_free) to free the screen associated with the `mode_tree_data`.
    - Free the `search` and `filter` strings if they are allocated.
    - Set the `dead` flag of the `mode_tree_data` to 1, indicating it is no longer in use.
    - Call [`mode_tree_remove_ref`](#mode_tree_remove_ref) to decrease the reference count and potentially free the `mode_tree_data` structure.
- **Output**:
    - The function does not return a value; it performs cleanup operations on the `mode_tree_data` structure.
- **Functions called**:
    - [`server_unzoom_window`](server-fn.c.driver.md#server_unzoom_window)
    - [`mode_tree_free_items`](#mode_tree_free_items)
    - [`mode_tree_clear_lines`](#mode_tree_clear_lines)
    - [`screen_free`](screen.c.driver.md#screen_free)
    - [`mode_tree_remove_ref`](#mode_tree_remove_ref)


---
### mode_tree_resize<!-- {{#callable:mode_tree_resize}} -->
The `mode_tree_resize` function resizes the screen associated with a mode tree, rebuilds the mode tree, redraws it, and marks the window pane for redraw.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure, which contains the mode tree's state and associated data.
    - `sx`: An unsigned integer representing the new width of the screen.
    - `sy`: An unsigned integer representing the new height of the screen.
- **Control Flow**:
    - Retrieve the screen associated with the mode tree data (`mtd`).
    - Call [`screen_resize`](screen.c.driver.md#screen_resize) to resize the screen to the new dimensions (`sx` and `sy`).
    - Invoke [`mode_tree_build`](#mode_tree_build) to rebuild the mode tree structure based on the new screen size.
    - Call [`mode_tree_draw`](#mode_tree_draw) to redraw the mode tree on the resized screen.
    - Set the `PANE_REDRAW` flag on the window pane to indicate that it needs to be redrawn.
- **Output**:
    - The function does not return a value; it modifies the state of the mode tree and its associated screen and window pane.
- **Functions called**:
    - [`screen_resize`](screen.c.driver.md#screen_resize)
    - [`mode_tree_build`](#mode_tree_build)
    - [`mode_tree_draw`](#mode_tree_draw)


---
### mode_tree_add<!-- {{#callable:mode_tree_add}} -->
The `mode_tree_add` function creates and adds a new item to a mode tree, setting its properties and inserting it into the appropriate parent or root list.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure representing the mode tree to which the item will be added.
    - `parent`: A pointer to a `mode_tree_item` structure representing the parent item under which the new item will be added; can be NULL if the item is to be added at the root level.
    - `itemdata`: A void pointer to the data associated with the new item.
    - `tag`: A `uint64_t` value representing a unique identifier for the new item.
    - `name`: A constant character pointer to the name of the new item.
    - `text`: A constant character pointer to the text description of the new item; can be NULL.
    - `expanded`: An integer indicating whether the new item should be expanded (1), collapsed (0), or inherit the state from a saved item (-1).
- **Control Flow**:
    - Log the function call with the tag, name, and text of the new item.
    - Allocate memory for a new `mode_tree_item` and initialize its parent and itemdata fields.
    - Set the tag, name, and text fields of the new item, duplicating the name and text strings.
    - Search for a saved item with the same tag in the `mtd->saved` list.
    - If a saved item is found, copy its tagged and expanded states to the new item if applicable.
    - If no saved item is found, set the expanded state based on the `expanded` parameter.
    - Initialize the children list of the new item.
    - Insert the new item into the parent's children list if a parent is provided, otherwise insert it into the root children list of `mtd`.
- **Output**:
    - Returns a pointer to the newly created `mode_tree_item`.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`mode_tree_find_item`](#mode_tree_find_item)


---
### mode_tree_draw_as_parent<!-- {{#callable:mode_tree_draw_as_parent}} -->
The function `mode_tree_draw_as_parent` sets the `draw_as_parent` flag of a `mode_tree_item` structure to 1, indicating that the item should be drawn as a parent in a mode tree.
- **Inputs**:
    - `mti`: A pointer to a `mode_tree_item` structure whose `draw_as_parent` flag is to be set.
- **Control Flow**:
    - The function directly sets the `draw_as_parent` field of the `mode_tree_item` structure pointed to by `mti` to 1.
- **Output**:
    - The function does not return any value.


---
### mode_tree_no_tag<!-- {{#callable:mode_tree_no_tag}} -->
The function `mode_tree_no_tag` sets the `no_tag` attribute of a `mode_tree_item` structure to 1, indicating that the item should not be tagged.
- **Inputs**:
    - `mti`: A pointer to a `mode_tree_item` structure whose `no_tag` attribute is to be set.
- **Control Flow**:
    - The function takes a single argument, a pointer to a `mode_tree_item` structure.
    - It sets the `no_tag` attribute of the `mode_tree_item` structure to 1.
- **Output**:
    - The function does not return any value; it modifies the `no_tag` attribute of the provided `mode_tree_item` structure in place.


---
### mode_tree_align<!-- {{#callable:mode_tree_align}} -->
The `mode_tree_align` function sets the alignment of a mode tree item's name.
- **Inputs**:
    - `mti`: A pointer to a `mode_tree_item` structure whose alignment is to be set.
    - `align`: An integer representing the alignment setting: -1 for left, 0 for no alignment, or 1 for right alignment.
- **Control Flow**:
    - The function takes a `mode_tree_item` pointer and an integer alignment value as arguments.
    - It assigns the alignment value to the `align` field of the `mode_tree_item` structure.
- **Output**:
    - The function does not return any value; it modifies the `align` field of the provided `mode_tree_item` structure.


---
### mode_tree_remove<!-- {{#callable:mode_tree_remove}} -->
The `mode_tree_remove` function removes a specified item from a mode tree and frees its memory.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure representing the mode tree from which the item will be removed.
    - `mti`: A pointer to a `mode_tree_item` structure representing the item to be removed from the mode tree.
- **Control Flow**:
    - Retrieve the parent of the item `mti` to be removed.
    - If the parent is not NULL, remove `mti` from the parent's children list using `TAILQ_REMOVE`.
    - If the parent is NULL, remove `mti` from the root children list of `mtd` using `TAILQ_REMOVE`.
    - Call [`mode_tree_free_item`](#mode_tree_free_item) to free the memory associated with `mti`.
- **Output**:
    - The function does not return a value; it performs the removal and memory deallocation as a side effect.
- **Functions called**:
    - [`mode_tree_free_item`](#mode_tree_free_item)


---
### mode_tree_draw<!-- {{#callable:mode_tree_draw}} -->
The `mode_tree_draw` function renders a tree-like structure on a screen within a window pane, handling various display attributes and conditions.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure containing information about the tree to be drawn, including its lines, items, screen, and display settings.
- **Control Flow**:
    - Check if `mtd->line_size` is zero and return if true, as there is nothing to draw.
    - Initialize grid cell styles and screen dimensions from `mtd`.
    - Start writing to the screen and clear it.
    - Calculate the maximum key length for alignment purposes.
    - Determine the maximum alignment length for each depth level in the tree.
    - Iterate over each line in the tree, skipping lines outside the current viewable offset and height range.
    - For each line, calculate padding and symbols based on the item's properties (e.g., key, depth, expanded state).
    - Construct the display text for each line, including key, symbols, name, and optional text, ensuring it fits within the screen width.
    - Apply brightness attribute toggling for tagged items.
    - Draw each line on the screen, using different styles for the current line versus other lines.
    - If preview mode is enabled and conditions are met, draw a preview box below the tree with additional information.
    - Invoke a callback to draw additional content in the preview box if applicable.
    - Finalize the screen drawing by moving the cursor to the current line and stopping the screen write context.
- **Output**:
    - The function does not return a value; it directly modifies the screen associated with the `mode_tree_data` structure to display the tree.
- **Functions called**:
    - [`style_apply`](style.c.driver.md#style_apply)
    - [`screen_write_start`](screen-write.c.driver.md#screen_write_start)
    - [`screen_write_clearscreen`](screen-write.c.driver.md#screen_write_clearscreen)
    - [`screen_write_cursormove`](screen-write.c.driver.md#screen_write_cursormove)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`strlcat`](compat/strlcat.c.driver.md#strlcat)
    - [`utf8_cstrwidth`](utf8.c.driver.md#utf8_cstrwidth)
    - [`screen_write_clearendofline`](screen-write.c.driver.md#screen_write_clearendofline)
    - [`format_draw`](format-draw.c.driver.md#format_draw)
    - [`screen_write_box`](screen-write.c.driver.md#screen_write_box)
    - [`screen_write_stop`](screen-write.c.driver.md#screen_write_stop)


---
### mode_tree_search_backward<!-- {{#callable:mode_tree_search_backward}} -->
The `mode_tree_search_backward` function searches backward through a tree structure for an item matching a search string or callback condition.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure that contains the tree data and search parameters.
- **Control Flow**:
    - Check if the search string in `mtd` is NULL; if so, return NULL.
    - Initialize `mti` and `last` to the current item in the line list of `mtd`.
    - Enter an infinite loop to traverse the tree backward.
    - Attempt to find the previous item (`prev`) in the list; if found, navigate to the last child of the previous subtree.
    - If no previous item is found, move to the parent of the current item.
    - If the current item becomes NULL, navigate to the last child of the last root subtree.
    - If the current item is the same as the last item, break the loop.
    - If no search callback is provided, check if the current item's name contains the search string; if so, return the current item.
    - If a search callback is provided, use it to check if the current item matches the search condition; if so, return the current item.
    - If no match is found, continue the loop.
- **Output**:
    - Returns a pointer to the `mode_tree_item` that matches the search criteria, or NULL if no match is found.


---
### mode_tree_search_forward<!-- {{#callable:mode_tree_search_forward}} -->
The `mode_tree_search_forward` function searches through a tree structure starting from the current item and returns the first item that matches a given search string or satisfies a custom search callback.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure that contains the tree data, current position, search string, and search callback function.
- **Control Flow**:
    - Check if the search string (`mtd->search`) is NULL; if so, return NULL immediately.
    - Initialize `mti` and `last` to the current item in the tree (`mtd->line_list[mtd->current].item`).
    - Enter an infinite loop to traverse the tree structure.
    - If the current item has children, move to the first child.
    - If there are no children, attempt to move to the next sibling using `TAILQ_NEXT`.
    - If there is no next sibling, move up to the parent and try to find the next sibling of the parent, repeating until a next sibling is found or the root is reached.
    - If the current item becomes NULL, reset it to the first child of the root.
    - If the current item is the same as the starting item (`last`), break the loop to avoid infinite traversal.
    - If no custom search callback is provided, check if the current item's name contains the search string using `strstr`; if it does, return the current item.
    - If a custom search callback is provided, call it with the current item's data and the search string; if it returns true, return the current item.
    - If no match is found after traversing the entire tree, return NULL.
- **Output**:
    - Returns a pointer to the first `mode_tree_item` that matches the search criteria, or NULL if no match is found.


---
### mode_tree_search_set<!-- {{#callable:mode_tree_search_set}} -->
The `mode_tree_search_set` function performs a search in a mode tree data structure, expands the path to the found item, rebuilds the tree, sets the current item, redraws the tree, and marks the pane for redraw.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure, which contains the mode tree and its associated data.
- **Control Flow**:
    - Check the search direction in `mtd->search_dir` and call either [`mode_tree_search_forward`](#mode_tree_search_forward) or [`mode_tree_search_backward`](#mode_tree_search_backward) to find the next matching item in the tree.
    - If no matching item is found (`mti` is NULL), return immediately.
    - Retrieve the tag of the found item (`mti->tag`).
    - Iterate through the parent chain of the found item (`mti->parent`) and set each parent's `expanded` flag to 1 to ensure the path to the item is expanded.
    - Call [`mode_tree_build`](#mode_tree_build) to rebuild the tree structure.
    - Call [`mode_tree_set_current`](#mode_tree_set_current) with the found item's tag to set it as the current item in the tree.
    - Call [`mode_tree_draw`](#mode_tree_draw) to redraw the tree on the screen.
    - Set the `PANE_REDRAW` flag in `mtd->wp->flags` to indicate that the pane needs to be redrawn.
- **Output**:
    - The function does not return a value; it modifies the state of the `mode_tree_data` structure and its associated pane.
- **Functions called**:
    - [`mode_tree_search_forward`](#mode_tree_search_forward)
    - [`mode_tree_search_backward`](#mode_tree_search_backward)
    - [`mode_tree_build`](#mode_tree_build)
    - [`mode_tree_set_current`](#mode_tree_set_current)
    - [`mode_tree_draw`](#mode_tree_draw)


---
### mode_tree_search_callback<!-- {{#callable:mode_tree_search_callback}} -->
The `mode_tree_search_callback` function handles search input for a mode tree, updating the search string and triggering a search operation if the mode tree is not marked as dead.
- **Inputs**:
    - `c`: A pointer to a `client` structure, which is unused in this function.
    - `data`: A pointer to a `mode_tree_data` structure, which contains the state and data for the mode tree.
    - `s`: A constant character pointer representing the search string input.
    - `done`: An integer indicating if the search operation is complete, which is unused in this function.
- **Control Flow**:
    - Check if the mode tree is marked as dead using `mtd->dead`; if true, return 0 immediately.
    - Free the current search string stored in `mtd->search`.
    - Check if the input search string `s` is NULL or empty; if so, set `mtd->search` to NULL and return 0.
    - Duplicate the input search string `s` using [`xstrdup`](xmalloc.c.driver.md#xstrdup) and assign it to `mtd->search`.
    - Call `mode_tree_search_set(mtd)` to perform the search operation with the updated search string.
- **Output**:
    - The function returns an integer value of 0, indicating the operation was completed without errors.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`mode_tree_search_set`](#mode_tree_search_set)


---
### mode_tree_search_free<!-- {{#callable:mode_tree_search_free}} -->
The `mode_tree_search_free` function decreases the reference count of a `mode_tree_data` object and frees it if the count reaches zero.
- **Inputs**:
    - `data`: A pointer to a `mode_tree_data` object whose reference count is to be decreased.
- **Control Flow**:
    - The function calls [`mode_tree_remove_ref`](#mode_tree_remove_ref) with the `data` pointer as an argument.
    - Inside [`mode_tree_remove_ref`](#mode_tree_remove_ref), the reference count of the `mode_tree_data` object is decremented.
    - If the reference count reaches zero, the `mode_tree_data` object is freed.
- **Output**:
    - The function does not return any value.
- **Functions called**:
    - [`mode_tree_remove_ref`](#mode_tree_remove_ref)


---
### mode_tree_filter_callback<!-- {{#callable:mode_tree_filter_callback}} -->
The `mode_tree_filter_callback` function updates the filter string of a mode tree data structure and triggers a rebuild and redraw of the mode tree.
- **Inputs**:
    - `c`: A pointer to a `client` structure, which is unused in this function.
    - `data`: A pointer to a `mode_tree_data` structure, which holds the state and data for the mode tree.
    - `s`: A constant character pointer representing the new filter string to be applied to the mode tree.
    - `done`: An integer flag indicating if the operation is complete, which is unused in this function.
- **Control Flow**:
    - Check if the `dead` flag in the `mode_tree_data` structure is set; if so, return 0 immediately.
    - If a filter string already exists in the `mode_tree_data`, free it to avoid memory leaks.
    - If the input string `s` is NULL or empty, set the filter to NULL; otherwise, duplicate the string `s` and assign it to the filter.
    - Call [`mode_tree_build`](#mode_tree_build) to rebuild the mode tree with the new filter.
    - Call [`mode_tree_draw`](#mode_tree_draw) to redraw the mode tree on the screen.
    - Set the `PANE_REDRAW` flag on the associated window pane to indicate that it needs to be redrawn.
- **Output**:
    - The function returns an integer value of 0, indicating successful execution.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`mode_tree_build`](#mode_tree_build)
    - [`mode_tree_draw`](#mode_tree_draw)


---
### mode_tree_filter_free<!-- {{#callable:mode_tree_filter_free}} -->
The `mode_tree_filter_free` function decreases the reference count of a `mode_tree_data` object and frees it if the count reaches zero.
- **Inputs**:
    - `data`: A pointer to a `mode_tree_data` object whose reference count is to be decreased and potentially freed.
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
The `mode_tree_menu_callback` function handles menu interactions in a mode tree by executing a callback if conditions are met and then cleaning up resources.
- **Inputs**:
    - `menu`: A pointer to a `menu` structure, which is unused in this function.
    - `idx`: An unsigned integer representing the index of the menu item, which is unused in this function.
    - `key`: A `key_code` representing the key pressed during the menu interaction.
    - `data`: A pointer to a `mode_tree_menu` structure containing the mode tree data and client information.
- **Control Flow**:
    - Retrieve the `mode_tree_menu` and `mode_tree_data` structures from the `data` argument.
    - Check if the mode tree is marked as dead or if the key is `KEYC_NONE`; if so, skip to cleanup.
    - Check if the line number in `mtm` is valid within the mode tree's line size; if not, skip to cleanup.
    - Set the current line in the mode tree to the line specified in `mtm`.
    - Invoke the menu callback function `menucb` with the mode tree's data, client, and key.
    - Perform cleanup by removing a reference from the mode tree data and freeing the `mode_tree_menu` structure.
- **Output**:
    - The function does not return a value; it performs actions based on the menu interaction and cleans up resources.
- **Functions called**:
    - [`mode_tree_remove_ref`](#mode_tree_remove_ref)


---
### mode_tree_display_menu<!-- {{#callable:mode_tree_display_menu}} -->
The `mode_tree_display_menu` function displays a menu for a specific item in a mode tree at a given position on the screen, with options depending on whether the menu is triggered from inside or outside the tree.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure representing the mode tree data.
    - `c`: A pointer to a `client` structure representing the client for which the menu is being displayed.
    - `x`: An unsigned integer representing the x-coordinate on the screen where the menu should be displayed.
    - `y`: An unsigned integer representing the y-coordinate on the screen where the menu should be displayed.
    - `outside`: An integer flag indicating whether the menu is being displayed from outside the mode tree (non-zero) or inside (zero).
- **Control Flow**:
    - Calculate the line index based on the current offset and y-coordinate, defaulting to the current line if out of bounds.
    - Retrieve the mode tree item for the calculated line index.
    - Determine the menu items and title based on whether the menu is triggered from inside or outside the mode tree.
    - Create a menu with the determined title and add the menu items to it.
    - Allocate memory for a `mode_tree_menu` structure and set its fields with the mode tree data, client, and line index.
    - Increment the reference count of the mode tree data.
    - Adjust the x-coordinate to center the menu if necessary.
    - Display the menu at the specified coordinates using [`menu_display`](menu.c.driver.md#menu_display), passing a callback function and the `mode_tree_menu` structure.
    - If menu display fails, decrement the reference count, free the `mode_tree_menu` structure, and free the menu.
- **Output**:
    - The function does not return a value but displays a menu on the screen and manages memory and references for the mode tree data.
- **Functions called**:
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`menu_create`](menu.c.driver.md#menu_create)
    - [`menu_add_items`](menu.c.driver.md#menu_add_items)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`menu_display`](menu.c.driver.md#menu_display)
    - [`mode_tree_remove_ref`](#mode_tree_remove_ref)
    - [`menu_free`](menu.c.driver.md#menu_free)


---
### mode_tree_key<!-- {{#callable:mode_tree_key}} -->
The `mode_tree_key` function processes keyboard and mouse input events to navigate and manipulate a mode tree structure in a terminal interface.
- **Inputs**:
    - `mtd`: A pointer to a `mode_tree_data` structure representing the mode tree state.
    - `c`: A pointer to a `client` structure representing the client interacting with the mode tree.
    - `key`: A pointer to a `key_code` representing the key or mouse event to be processed.
    - `m`: A pointer to a `mouse_event` structure containing details about a mouse event, if applicable.
    - `xp`: A pointer to an unsigned integer to store the x-coordinate of a mouse event, if applicable.
    - `yp`: A pointer to an unsigned integer to store the y-coordinate of a mouse event, if applicable.
- **Control Flow**:
    - Check if the input key is a mouse event and handle mouse interactions, updating the current selection or displaying a menu if necessary.
    - If the key is not a mouse event, iterate through the mode tree lines to find a matching key and update the current selection if found.
    - Handle various keyboard inputs to navigate the mode tree, such as moving up or down, expanding or collapsing nodes, tagging items, and sorting.
    - Return 1 if the key is a quit command ('q', 'ESC', or 'Ctrl+g'), otherwise return 0.
- **Output**:
    - The function returns an integer indicating whether the operation was a quit command (1) or not (0).
- **Functions called**:
    - [`cmd_mouse_at`](cmd.c.driver.md#cmd_mouse_at)
    - [`mode_tree_display_menu`](#mode_tree_display_menu)
    - [`mode_tree_up`](#mode_tree_up)
    - [`mode_tree_down`](#mode_tree_down)
    - [`mode_tree_swap`](#mode_tree_swap)
    - [`mode_tree_clear_tagged`](#mode_tree_clear_tagged)
    - [`mode_tree_build`](#mode_tree_build)
    - [`status_prompt_set`](status.c.driver.md#status_prompt_set)
    - [`mode_tree_search_set`](#mode_tree_search_set)
    - [`mode_tree_check_selected`](#mode_tree_check_selected)


---
### mode_tree_run_command<!-- {{#callable:mode_tree_run_command}} -->
The `mode_tree_run_command` function executes a command template by replacing placeholders with a given name and handling any parsing errors.
- **Inputs**:
    - `c`: A pointer to a `struct client`, representing the client context in which the command is executed.
    - `fs`: A pointer to a `struct cmd_find_state`, representing the state used to find the command context.
    - `template`: A constant character pointer representing the command template string with placeholders.
    - `name`: A constant character pointer representing the name to replace placeholders in the template.
- **Control Flow**:
    - The function begins by replacing placeholders in the `template` with the `name` using [`cmd_template_replace`](cmd.c.driver.md#cmd_template_replace) and stores the result in `command`.
    - It checks if `command` is not NULL and not empty, indicating a valid command string.
    - A new command queue state is created using [`cmdq_new_state`](cmd-queue.c.driver.md#cmdq_new_state) with the provided `fs` and other parameters.
    - The command is parsed and appended to the command queue using `cmd_parse_and_append`, and the status is checked for errors.
    - If a parsing error occurs (`CMD_PARSE_ERROR`), and if `c` is not NULL, the error message is capitalized and displayed using `status_message_set`.
    - The error message memory is freed if it was allocated.
    - The command queue state is freed using [`cmdq_free_state`](cmd-queue.c.driver.md#cmdq_free_state).
    - Finally, the memory allocated for `command` is freed.
- **Output**:
    - The function does not return a value; it executes a command and handles any errors internally.
- **Functions called**:
    - [`cmd_template_replace`](cmd.c.driver.md#cmd_template_replace)
    - [`cmdq_new_state`](cmd-queue.c.driver.md#cmdq_new_state)
    - [`cmdq_free_state`](cmd-queue.c.driver.md#cmdq_free_state)


