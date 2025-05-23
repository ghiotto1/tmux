# Purpose
The provided C source code is part of the `tmux` project, specifically implementing a "tree mode" for displaying and interacting with sessions, windows, and panes in a hierarchical manner. This code defines a window mode called "tree-mode" that allows users to navigate through a structured view of their tmux environment, providing functionalities such as selecting, expanding, marking, tagging, and killing sessions, windows, or panes. The code is structured around several static functions that handle the initialization, updating, resizing, and key handling of the tree view, as well as functions for building and sorting the tree items based on different criteria like index, name, or time.

The file is a C source file intended to be part of the tmux application, providing a specific mode that can be activated within a tmux session. It defines internal functions and structures to manage the tree view, including the `window_tree_modedata` and `window_tree_itemdata` structures, which store the state and data of the tree mode. The code does not define public APIs or external interfaces but rather integrates into the tmux application to enhance its functionality. The tree mode is designed to be interactive, responding to user inputs such as key presses and mouse events to manipulate the view and perform actions on the selected items.
# Imports and Dependencies

---
- `sys/types.h`
- `ctype.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### window_tree_init
- **Type**: ``struct screen *``
- **Description**: The `window_tree_init` function is a static function that initializes a window tree mode entry. It returns a pointer to a `struct screen`, which represents the screen associated with the window tree mode. This function sets up the necessary data structures and configurations for the window tree mode, including the format, key format, and command to be used.
- **Use**: This variable is used to initialize and configure the window tree mode, returning a screen structure that represents the visual representation of the window tree.


---
### window_tree_menu_items
- **Type**: ``const struct menu_item[]``
- **Description**: `window_tree_menu_items` is a static constant array of `struct menu_item` that defines a set of menu items for a window tree interface. Each menu item consists of a label, a key code, and a function pointer, which is set to `NULL` in this case.
- **Use**: This array is used to define the menu options available in the window tree interface, allowing users to interact with the interface using specific key bindings.


---
### window_tree_mode
- **Type**: ``const struct window_mode``
- **Description**: The `window_tree_mode` is a constant structure of type `window_mode` that defines a specific mode for a window, named "tree-mode". It includes function pointers for initialization, freeing resources, resizing, updating, and handling key events, as well as a default format for displaying the window tree.
- **Use**: This variable is used to define and manage the behavior and display format of the window tree mode in the application.


---
### window_tree_sort_list
- **Type**: `const char *[]`
- **Description**: `window_tree_sort_list` is a static constant array of strings that contains the sorting criteria options for a window tree structure. The array includes three strings: "index", "name", and "time".
- **Use**: This array is used to define the possible sorting criteria for organizing window tree items in the application.


---
### window_tree_sort
- **Type**: `struct mode_tree_sort_criteria*`
- **Description**: The `window_tree_sort` is a static pointer to a `struct mode_tree_sort_criteria`, which is used to define the criteria for sorting items in a window tree. This structure likely contains fields that specify the sorting field and order (e.g., ascending or descending).
- **Use**: This variable is used to determine the sorting behavior of sessions, windows, or panes in the window tree, affecting how they are displayed or processed.


# Data Structures

---
### window_tree_sort_type
- **Type**: `enum`
- **Members**:
    - `WINDOW_TREE_BY_INDEX`: Sorts the window tree by index.
    - `WINDOW_TREE_BY_NAME`: Sorts the window tree by name.
    - `WINDOW_TREE_BY_TIME`: Sorts the window tree by time.
- **Description**: The `window_tree_sort_type` is an enumeration that defines the sorting criteria for a window tree structure. It provides three options for sorting: by index, by name, and by time. This enumeration is used to determine how the window tree should be organized and displayed, allowing for different views based on the selected sorting method.


---
### window_tree_type
- **Type**: `enum`
- **Members**:
    - `WINDOW_TREE_NONE`: Represents a state where no specific window tree type is selected.
    - `WINDOW_TREE_SESSION`: Represents a session in the window tree.
    - `WINDOW_TREE_WINDOW`: Represents a window in the window tree.
    - `WINDOW_TREE_PANE`: Represents a pane in the window tree.
- **Description**: The `window_tree_type` is an enumeration that defines different types of entities that can be represented in a window tree structure. It includes types for none, session, window, and pane, allowing the system to categorize and manage different levels of the window hierarchy in a structured manner.


---
### window_tree_itemdata
- **Type**: `struct`
- **Members**:
    - `type`: An enum indicating the type of the window tree item, such as session, window, or pane.
    - `session`: An integer representing the session ID associated with the item.
    - `winlink`: An integer representing the window link index associated with the item.
    - `pane`: An integer representing the pane ID associated with the item.
- **Description**: The `window_tree_itemdata` structure is used to represent an item in a window tree, which can be a session, window, or pane. It contains information about the type of item it represents, as well as identifiers for the session, window link, and pane associated with the item. This structure is part of a larger system for managing and displaying a hierarchical view of sessions, windows, and panes in a terminal multiplexer environment.


---
### window_tree_modedata
- **Type**: `struct`
- **Members**:
    - `wp`: Pointer to a window pane associated with the mode data.
    - `dead`: Flag indicating if the mode data is no longer active.
    - `references`: Count of references to this mode data structure.
    - `data`: Pointer to mode tree data structure used for managing tree items.
    - `format`: String format used for displaying tree items.
    - `key_format`: String format used for key bindings in the tree.
    - `command`: Command string to execute when a tree item is selected.
    - `squash_groups`: Flag indicating if session groups should be squashed into a single entry.
    - `prompt_flags`: Flags for configuring prompt behavior.
    - `item_list`: Array of pointers to window tree item data structures.
    - `item_size`: Number of items in the item_list array.
    - `entered`: String representing the last entered command or input.
    - `fs`: Command find state structure for locating sessions, windows, or panes.
    - `type`: Enumeration indicating the type of tree (session, window, or pane).
    - `offset`: Offset for scrolling or navigating within the tree.
    - `left`: Position of the left navigation indicator.
    - `right`: Position of the right navigation indicator.
    - `start`: Index of the first visible item in the tree.
    - `end`: Index of the last visible item in the tree.
    - `each`: Width of each item in the tree display.
- **Description**: The `window_tree_modedata` structure is used to manage and display a hierarchical tree of sessions, windows, and panes within a terminal multiplexer environment. It contains pointers to associated window panes and mode tree data, as well as configuration strings for formatting and commands. The structure also maintains state information such as the number of references, navigation offsets, and display parameters for rendering the tree. It supports operations like expanding, collapsing, and executing commands on tree items, and is integral to the user interface for navigating and managing terminal sessions.


# Functions

---
### window_tree_pull_item<!-- {{#callable:window_tree_pull_item}} -->
The `window_tree_pull_item` function retrieves and sets the session, winlink, and window pane pointers based on the type of the window tree item data provided.
- **Inputs**:
    - `item`: A pointer to a `window_tree_itemdata` structure that contains information about the session, winlink, and pane IDs, as well as the type of the item (session, window, or pane).
    - `sp`: A pointer to a pointer to a `session` structure, which will be set to the session found by the session ID in the item data.
    - `wlp`: A pointer to a pointer to a `winlink` structure, which will be set to the winlink found by the winlink index in the item data.
    - `wp`: A pointer to a pointer to a `window_pane` structure, which will be set to the window pane found by the pane ID in the item data.
- **Control Flow**:
    - Initialize `wp` and `wlp` to NULL.
    - Find the session using the session ID from the item data and assign it to `sp`.
    - If the session is not found, return immediately.
    - If the item type is `WINDOW_TREE_SESSION`, set `wlp` to the current winlink of the session and `wp` to the active window pane of that winlink, then return.
    - Find the winlink using the winlink index from the item data and assign it to `wlp`.
    - If the winlink is not found, set `sp` to NULL and return.
    - If the item type is `WINDOW_TREE_WINDOW`, set `wp` to the active window pane of the winlink and return.
    - Find the window pane using the pane ID from the item data and assign it to `wp`.
    - If the window does not have the pane or the pane is not found, set `wp` to NULL.
    - If `wp` is NULL, set `sp` and `wlp` to NULL and return.
- **Output**:
    - The function does not return a value, but it sets the pointers `sp`, `wlp`, and `wp` to the appropriate session, winlink, and window pane based on the item data.
- **Functions called**:
    - [`session_find_by_id`](session.c.driver.md#session_find_by_id)
    - [`winlink_find_by_index`](window.c.driver.md#winlink_find_by_index)
    - [`window_pane_find_by_id`](window.c.driver.md#window_pane_find_by_id)
    - [`window_has_pane`](window.c.driver.md#window_has_pane)


---
### window_tree_add_item<!-- {{#callable:window_tree_add_item}} -->
The `window_tree_add_item` function adds a new item to the `item_list` of a `window_tree_modedata` structure, reallocating memory as necessary.
- **Inputs**:
    - `data`: A pointer to a `window_tree_modedata` structure, which contains the list of items (`item_list`) and its current size (`item_size`).
- **Control Flow**:
    - Reallocate memory for `data->item_list` to accommodate one more `window_tree_itemdata` structure, increasing the size by one.
    - Allocate memory for a new `window_tree_itemdata` structure and initialize it to zero using [`xcalloc`](xmalloc.c.driver.md#xcalloc).
    - Assign the newly allocated `window_tree_itemdata` to the last position in `data->item_list` and increment `data->item_size`.
    - Return the pointer to the newly added `window_tree_itemdata`.
- **Output**:
    - Returns a pointer to the newly added `window_tree_itemdata` structure.
- **Functions called**:
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)


---
### window_tree_free_item<!-- {{#callable:window_tree_free_item}} -->
The function `window_tree_free_item` deallocates memory for a `window_tree_itemdata` structure.
- **Inputs**:
    - `item`: A pointer to a `window_tree_itemdata` structure that needs to be freed.
- **Control Flow**:
    - The function calls the `free` function on the `item` pointer to deallocate the memory associated with it.
- **Output**:
    - The function does not return any value.


---
### window_tree_cmp_session<!-- {{#callable:window_tree_cmp_session}} -->
The function `window_tree_cmp_session` compares two session objects based on a specified sorting criterion and returns an integer indicating their relative order.
- **Inputs**:
    - `a0`: A pointer to the first session object to be compared.
    - `b0`: A pointer to the second session object to be compared.
- **Control Flow**:
    - The function casts the input pointers `a0` and `b0` to pointers to session objects `a` and `b`, respectively.
    - It then dereferences these pointers to obtain the session objects `sa` and `sb`.
    - The function initializes a variable `result` to 0 to store the comparison result.
    - A switch statement is used to determine the sorting criterion based on `window_tree_sort->field`.
    - If the sorting criterion is `WINDOW_TREE_BY_INDEX`, it compares the session IDs and assigns the difference to `result`.
    - If the sorting criterion is `WINDOW_TREE_BY_TIME`, it uses `timercmp` to compare the activity times of the sessions and assigns -1, 1, or 0 to `result` based on the comparison.
    - If the sorting criterion is `WINDOW_TREE_BY_NAME`, it uses `strcmp` to compare the session names and assigns the result to `result`.
    - If `window_tree_sort->reversed` is true, the function negates `result` to reverse the order.
    - The function returns the `result` value.
- **Output**:
    - An integer indicating the relative order of the two session objects: negative if the first is less than the second, positive if the first is greater, and zero if they are equal.


---
### window_tree_cmp_window<!-- {{#callable:window_tree_cmp_window}} -->
The function `window_tree_cmp_window` compares two `winlink` structures based on a specified sorting criterion and returns an integer indicating their relative order.
- **Inputs**:
    - `a0`: A pointer to the first `winlink` structure to be compared.
    - `b0`: A pointer to the second `winlink` structure to be compared.
- **Control Flow**:
    - The function casts the input pointers `a0` and `b0` to pointers to `winlink` structures.
    - It dereferences these pointers to obtain the `winlink` structures `wla` and `wlb`.
    - It retrieves the `window` structures `wa` and `wb` from `wla` and `wlb`, respectively.
    - The function initializes a result variable to 0.
    - It uses a switch statement to determine the sorting criterion based on `window_tree_sort->field`.
    - If the sorting criterion is `WINDOW_TREE_BY_INDEX`, it calculates the difference between the indices of `wla` and `wlb`.
    - If the sorting criterion is `WINDOW_TREE_BY_TIME`, it compares the activity times of `wa` and `wb` using `timercmp` and sets the result accordingly.
    - If the sorting criterion is `WINDOW_TREE_BY_NAME`, it compares the names of `wa` and `wb` using `strcmp`.
    - If `window_tree_sort->reversed` is true, it negates the result to reverse the sorting order.
    - The function returns the result.
- **Output**:
    - An integer indicating the relative order of the two `winlink` structures: negative if the first is less than the second, zero if they are equal, and positive if the first is greater than the second.


---
### window_tree_cmp_pane<!-- {{#callable:window_tree_cmp_pane}} -->
The `window_tree_cmp_pane` function compares two window panes based on a specified sorting criterion, either by active time or by index, and returns an integer indicating their relative order.
- **Inputs**:
    - `a0`: A pointer to the first window pane to be compared.
    - `b0`: A pointer to the second window pane to be compared.
- **Control Flow**:
    - The function casts the input pointers `a0` and `b0` to pointers to pointers of `struct window_pane`.
    - It checks if the sorting field is `WINDOW_TREE_BY_TIME`; if so, it calculates the difference between the `active_point` of the two panes.
    - If the sorting field is not `WINDOW_TREE_BY_TIME`, it retrieves the indices of the panes using [`window_pane_index`](window.c.driver.md#window_pane_index) and calculates the difference between these indices.
    - If the sorting order is reversed, it negates the result.
    - Finally, it returns the result, which indicates the relative order of the two panes.
- **Output**:
    - An integer that is negative, zero, or positive, indicating if the first pane is less than, equal to, or greater than the second pane, respectively, based on the sorting criteria.
- **Functions called**:
    - [`window_pane_index`](window.c.driver.md#window_pane_index)


---
### window_tree_build_pane<!-- {{#callable:window_tree_build_pane}} -->
The `window_tree_build_pane` function constructs a tree item representing a window pane and adds it to a mode tree structure for display or manipulation.
- **Inputs**:
    - `s`: A pointer to the session structure containing the window pane.
    - `wl`: A pointer to the winlink structure representing the window link in the session.
    - `wp`: A pointer to the window pane structure to be added to the tree.
    - `modedata`: A pointer to the mode tree data structure that holds the tree's state and configuration.
    - `parent`: A pointer to the parent mode tree item under which the new pane item will be added.
- **Control Flow**:
    - Retrieve the index of the window pane using [`window_pane_index`](window.c.driver.md#window_pane_index) and store it in `idx`.
    - Create a new item in the mode tree using [`window_tree_add_item`](#window_tree_add_item) and set its type to `WINDOW_TREE_PANE`.
    - Assign the session ID, winlink index, and pane ID to the new item.
    - Create a format tree with [`format_create`](format.c.driver.md#format_create) and set default values using [`format_defaults`](format.c.driver.md#format_defaults).
    - Expand the format tree to a string using [`format_expand`](format.c.driver.md#format_expand) and store it in `text`.
    - Format the pane index into a string `name` using [`xasprintf`](xmalloc.c.driver.md#xasprintf).
    - Free the format tree using [`format_free`](format.c.driver.md#format_free).
    - Add the new item to the mode tree with [`mode_tree_add`](mode-tree.c.driver.md#mode_tree_add), using the parent item, pane ID, name, and text.
    - Free the allocated `text` and `name` strings.
    - Align the mode tree item using [`mode_tree_align`](mode-tree.c.driver.md#mode_tree_align).
- **Output**:
    - The function does not return a value; it modifies the mode tree structure by adding a new pane item.
- **Functions called**:
    - [`window_pane_index`](window.c.driver.md#window_pane_index)
    - [`window_tree_add_item`](#window_tree_add_item)
    - [`format_create`](format.c.driver.md#format_create)
    - [`format_defaults`](format.c.driver.md#format_defaults)
    - [`format_expand`](format.c.driver.md#format_expand)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`format_free`](format.c.driver.md#format_free)
    - [`mode_tree_add`](mode-tree.c.driver.md#mode_tree_add)
    - [`mode_tree_align`](mode-tree.c.driver.md#mode_tree_align)


---
### window_tree_filter_pane<!-- {{#callable:window_tree_filter_pane}} -->
The function `window_tree_filter_pane` evaluates a filter string against a specific window pane and returns whether the filter condition is met.
- **Inputs**:
    - `s`: A pointer to a `struct session`, representing the current session.
    - `wl`: A pointer to a `struct winlink`, representing the current window link.
    - `wp`: A pointer to a `struct window_pane`, representing the current window pane.
    - `filter`: A constant character pointer to a filter string that is used to evaluate the window pane.
- **Control Flow**:
    - Check if the `filter` is `NULL`; if so, return `1` indicating the filter condition is met by default.
    - Use [`format_single`](format.c.driver.md#format_single) to format the `filter` string with the session, winlink, and window pane, storing the result in `cp`.
    - Evaluate the formatted string `cp` using [`format_true`](format.c.driver.md#format_true) to determine if the filter condition is true, storing the result in `result`.
    - Free the memory allocated for `cp`.
    - Return the value of `result`, indicating whether the filter condition is met.
- **Output**:
    - Returns an integer, `1` if the filter condition is met, otherwise `0`.
- **Functions called**:
    - [`format_single`](format.c.driver.md#format_single)
    - [`format_true`](format.c.driver.md#format_true)


---
### window_tree_build_window<!-- {{#callable:window_tree_build_window}} -->
The `window_tree_build_window` function constructs a tree representation of a window's panes within a session, applying filters and sorting criteria, and returns whether any panes were successfully added.
- **Inputs**:
    - `s`: A pointer to the session structure representing the current session.
    - `wl`: A pointer to the winlink structure representing the window link within the session.
    - `modedata`: A pointer to the window_tree_modedata structure containing mode-specific data.
    - `sort_crit`: A pointer to the mode_tree_sort_criteria structure specifying the sorting criteria for the panes.
    - `parent`: A pointer to the mode_tree_item structure representing the parent item in the mode tree.
    - `filter`: A string representing the filter criteria to apply to the panes.
- **Control Flow**:
    - Initialize a new window_tree_itemdata structure and set its type, session, and winlink properties.
    - Create a format tree and expand it to generate a text representation of the window, then free the format tree.
    - Determine if the window should be expanded based on the type of data and add it to the mode tree.
    - Check if the window has any panes; if not, clean up and return 0.
    - Iterate over the panes in the window, applying the filter to each pane and collecting those that pass the filter.
    - Sort the collected panes using the provided sorting criteria.
    - For each sorted pane, call [`window_tree_build_pane`](#window_tree_build_pane) to add it to the mode tree.
    - Free the list of panes and return 1 if any panes were added, otherwise clean up and return 0.
- **Output**:
    - Returns an integer indicating success (1) if panes were added to the mode tree, or failure (0) if no panes were added.
- **Functions called**:
    - [`window_tree_add_item`](#window_tree_add_item)
    - [`format_create`](format.c.driver.md#format_create)
    - [`format_defaults`](format.c.driver.md#format_defaults)
    - [`format_expand`](format.c.driver.md#format_expand)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`format_free`](format.c.driver.md#format_free)
    - [`mode_tree_add`](mode-tree.c.driver.md#mode_tree_add)
    - [`mode_tree_align`](mode-tree.c.driver.md#mode_tree_align)
    - [`window_tree_filter_pane`](#window_tree_filter_pane)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`window_tree_build_pane`](#window_tree_build_pane)
    - [`window_tree_free_item`](#window_tree_free_item)
    - [`mode_tree_remove`](mode-tree.c.driver.md#mode_tree_remove)


---
### window_tree_build_session<!-- {{#callable:window_tree_build_session}} -->
The `window_tree_build_session` function constructs a tree representation of a session's windows and panes, applying sorting and filtering criteria.
- **Inputs**:
    - `s`: A pointer to the session structure representing the current session.
    - `modedata`: A pointer to the window tree mode data structure, which holds information about the tree's state and configuration.
    - `sort_crit`: A pointer to the mode tree sort criteria structure, which specifies how the windows should be sorted.
    - `filter`: A string representing a filter to apply to the windows and panes, or NULL if no filter is applied.
- **Control Flow**:
    - Initialize a new item for the session in the tree and set its type, session ID, winlink, and pane to default values.
    - Create a format tree for the session's current window and expand it to generate a text representation of the session.
    - Determine whether the session should be expanded based on the mode data type and add it to the mode tree.
    - Iterate over all windows in the session, reallocating memory to store pointers to each window link.
    - Sort the windows using the provided sort criteria and a comparison function.
    - Iterate over the sorted windows, building each window's tree representation and counting how many are empty after filtering.
    - If all windows are empty, remove the session item from the tree and adjust the item size.
    - Free the allocated memory for the window link pointers.
- **Output**:
    - The function does not return a value; it modifies the mode tree data structure to include the session's windows and panes.
- **Functions called**:
    - [`window_tree_add_item`](#window_tree_add_item)
    - [`format_create`](format.c.driver.md#format_create)
    - [`format_defaults`](format.c.driver.md#format_defaults)
    - [`format_expand`](format.c.driver.md#format_expand)
    - [`format_free`](format.c.driver.md#format_free)
    - [`mode_tree_add`](mode-tree.c.driver.md#mode_tree_add)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`window_tree_build_window`](#window_tree_build_window)
    - [`window_tree_free_item`](#window_tree_free_item)
    - [`mode_tree_remove`](mode-tree.c.driver.md#mode_tree_remove)


---
### window_tree_build<!-- {{#callable:window_tree_build}} -->
The `window_tree_build` function constructs a tree structure of sessions, windows, and panes based on specified criteria, and assigns a tag to the current focus item.
- **Inputs**:
    - `modedata`: A pointer to `window_tree_modedata` structure containing the current mode data and state.
    - `sort_crit`: A pointer to `mode_tree_sort_criteria` structure that defines the sorting criteria for the sessions, windows, and panes.
    - `tag`: A pointer to a `uint64_t` where the function will store the tag of the current focus item.
    - `filter`: A string used to filter the items in the tree based on certain criteria.
- **Control Flow**:
    - Retrieve the current session group for the session in `modedata` and store it in `current`.
    - Free all existing items in `data->item_list` and reset `data->item_size` to 0.
    - Iterate over all sessions using `RB_FOREACH` and apply group squashing logic if `data->squash_groups` is true.
    - Allocate memory for a list of sessions and populate it with sessions that pass the group squashing filter.
    - Sort the list of sessions using `qsort` and the provided `sort_crit`.
    - Iterate over the sorted list of sessions and call [`window_tree_build_session`](#window_tree_build_session) for each session to build the tree structure.
    - Free the allocated list of sessions.
    - Set the `tag` based on the type of the current focus item in `modedata` (session, window, or pane).
- **Output**:
    - The function outputs a tree structure of sessions, windows, and panes, and assigns a tag to the current focus item based on its type.
- **Functions called**:
    - [`session_group_contains`](session.c.driver.md#session_group_contains)
    - [`window_tree_free_item`](#window_tree_free_item)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`window_tree_build_session`](#window_tree_build_session)
    - [`window_count_panes`](window.c.driver.md#window_count_panes)


---
### window_tree_draw_label<!-- {{#callable:window_tree_draw_label}} -->
The `window_tree_draw_label` function draws a centered label on a screen context, optionally surrounded by a box, if the dimensions allow.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure, which represents the screen context where the label will be drawn.
    - `px`: An unsigned integer representing the x-coordinate of the starting position for drawing the label.
    - `py`: An unsigned integer representing the y-coordinate of the starting position for drawing the label.
    - `sx`: An unsigned integer representing the width of the area where the label will be drawn.
    - `sy`: An unsigned integer representing the height of the area where the label will be drawn.
    - `gc`: A pointer to a `grid_cell` structure that defines the style (such as color) for the label text.
    - `label`: A constant character pointer to the string that will be drawn as the label.
- **Control Flow**:
    - Calculate the length of the label string using `strlen`.
    - Check if the width (`sx`) is zero, the height (`sy`) is one, or the label length exceeds the width; if any condition is true, return immediately without drawing.
    - Calculate the horizontal offset (`ox`) to center the label within the width, and the vertical offset (`oy`) to center it within the height.
    - If the label can be surrounded by a box (based on offsets and dimensions), move the cursor to the top-left corner of the box and draw a box around the label.
    - Move the cursor to the calculated position for the label and draw the label using `screen_write_puts`.
- **Output**:
    - The function does not return a value; it performs drawing operations on the provided screen context.
- **Functions called**:
    - [`screen_write_cursormove`](screen-write.c.driver.md#screen_write_cursormove)
    - [`screen_write_box`](screen-write.c.driver.md#screen_write_box)


---
### window_tree_draw_session<!-- {{#callable:window_tree_draw_session}} -->
The `window_tree_draw_session` function is responsible for rendering a visual representation of a session's windows in a tree-like structure on the screen.
- **Inputs**:
    - `data`: A pointer to a `window_tree_modedata` structure that holds the state and configuration for the window tree display.
    - `s`: A pointer to a `session` structure representing the session whose windows are to be drawn.
    - `ctx`: A pointer to a `screen_write_ctx` structure used for writing to the screen.
    - `sx`: An unsigned integer representing the width of the screen area available for drawing.
    - `sy`: An unsigned integer representing the height of the screen area available for drawing.
- **Control Flow**:
    - Calculate the total number of windows in the session using [`winlink_count`](window.c.driver.md#winlink_count).
    - Determine the number of windows that can be displayed based on the available screen width (`sx`).
    - Identify the current window index and calculate the start and end indices for visible windows.
    - Adjust the start and end indices based on the `data->offset` to handle scrolling.
    - Determine if left and right navigation indicators should be displayed based on the start and end indices.
    - Calculate the width of each window representation and handle any remaining space.
    - Draw left and right navigation indicators if applicable.
    - Iterate over the windows in the session, drawing each window's preview and label within the calculated bounds.
    - Adjust the grid cell color based on whether the window is the current window.
    - Draw vertical lines between window representations if necessary.
- **Output**:
    - The function does not return a value; it directly modifies the screen context to display the session's windows.
- **Functions called**:
    - [`winlink_count`](window.c.driver.md#winlink_count)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`screen_write_cursormove`](screen-write.c.driver.md#screen_write_cursormove)
    - [`screen_write_vline`](screen-write.c.driver.md#screen_write_vline)
    - [`screen_write_preview`](screen-write.c.driver.md#screen_write_preview)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`window_tree_draw_label`](#window_tree_draw_label)


---
### window_tree_draw_window<!-- {{#callable:window_tree_draw_window}} -->
The function `window_tree_draw_window` is responsible for rendering a visual representation of a window's panes within a given screen context, adjusting for visibility and active pane highlighting.
- **Inputs**:
    - `data`: A pointer to a `window_tree_modedata` structure that holds state information for the window tree mode.
    - `s`: A pointer to a `session` structure representing the current session.
    - `w`: A pointer to a `window` structure representing the window to be drawn.
    - `ctx`: A pointer to a `screen_write_ctx` structure used for writing to the screen.
    - `sx`: An unsigned integer representing the width of the screen area available for drawing.
    - `sy`: An unsigned integer representing the height of the screen area available for drawing.
- **Control Flow**:
    - Initialize variables for options, pane iteration, and screen context coordinates.
    - Calculate the total number of panes in the window and determine the color settings for inactive and active panes.
    - Determine the number of panes that can be visible based on the screen width (`sx`).
    - Identify the current active pane and calculate the start and end indices for visible panes, adjusting for the current active pane's position.
    - Adjust the start and end indices based on the `data->offset` to account for scrolling or offset adjustments.
    - Determine if left and right navigation indicators should be displayed based on the start and end indices and screen width.
    - Calculate the width of each pane's display area and handle any remaining space.
    - If the calculated width for each pane (`each`) is zero, exit the function early.
    - Draw left and right navigation indicators if applicable, updating `data->left` and `data->right` accordingly.
    - Iterate over the panes within the start and end indices, drawing each pane's preview and label, and updating the screen context accordingly.
- **Output**:
    - The function does not return a value; it modifies the screen context to visually represent the window's panes.
- **Functions called**:
    - [`window_count_panes`](window.c.driver.md#window_count_panes)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`screen_write_cursormove`](screen-write.c.driver.md#screen_write_cursormove)
    - [`screen_write_vline`](screen-write.c.driver.md#screen_write_vline)
    - [`screen_write_preview`](screen-write.c.driver.md#screen_write_preview)
    - [`window_pane_index`](window.c.driver.md#window_pane_index)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`window_tree_draw_label`](#window_tree_draw_label)


---
### window_tree_draw<!-- {{#callable:window_tree_draw}} -->
The `window_tree_draw` function renders a specific item in a window tree structure onto a screen context based on the item's type.
- **Inputs**:
    - `modedata`: A pointer to mode-specific data, which contains information about the current state and configuration of the window tree.
    - `itemdata`: A pointer to a `window_tree_itemdata` structure that represents the item to be drawn, including its type and identifiers for session, window, or pane.
    - `ctx`: A pointer to a `screen_write_ctx` structure, which is used to write to the screen.
    - `sx`: An unsigned integer representing the width of the area to draw.
    - `sy`: An unsigned integer representing the height of the area to draw.
- **Control Flow**:
    - The function begins by extracting session, winlink, and window pane information from the `itemdata` using [`window_tree_pull_item`](#window_tree_pull_item).
    - If the window pane (`wp`) is `NULL`, the function returns immediately, indicating nothing to draw.
    - A switch statement is used to determine the type of the item (`WINDOW_TREE_NONE`, `WINDOW_TREE_SESSION`, `WINDOW_TREE_WINDOW`, or `WINDOW_TREE_PANE`).
    - For `WINDOW_TREE_NONE`, no action is taken.
    - For `WINDOW_TREE_SESSION`, the function [`window_tree_draw_session`](#window_tree_draw_session) is called to draw the session on the screen context.
    - For `WINDOW_TREE_WINDOW`, the function [`window_tree_draw_window`](#window_tree_draw_window) is called to draw the window on the screen context.
    - For `WINDOW_TREE_PANE`, the function [`screen_write_preview`](screen-write.c.driver.md#screen_write_preview) is called to draw a preview of the pane on the screen context.
- **Output**:
    - The function does not return a value; it performs its operations directly on the provided screen context (`ctx`).
- **Functions called**:
    - [`window_tree_pull_item`](#window_tree_pull_item)
    - [`window_tree_draw_session`](#window_tree_draw_session)
    - [`window_tree_draw_window`](#window_tree_draw_window)
    - [`screen_write_preview`](screen-write.c.driver.md#screen_write_preview)


---
### window_tree_search<!-- {{#callable:window_tree_search}} -->
The `window_tree_search` function searches for a substring within the names of sessions, windows, or panes in a window tree structure and returns whether the substring is found.
- **Inputs**:
    - `modedata`: This is an unused parameter in the function, typically used for passing additional data, but not utilized here.
    - `itemdata`: A pointer to a `window_tree_itemdata` structure that contains information about the type of item (session, window, or pane) being searched.
    - `ss`: A constant character pointer representing the substring to search for within the item names.
- **Control Flow**:
    - The function begins by extracting session, window link, and window pane information from the `itemdata` using [`window_tree_pull_item`](#window_tree_pull_item).
    - It then uses a switch statement to determine the type of item (session, window, or pane) and performs the search accordingly.
    - For `WINDOW_TREE_NONE`, it returns 0 immediately as there is nothing to search.
    - For `WINDOW_TREE_SESSION`, it checks if the session is valid and searches for the substring in the session's name.
    - For `WINDOW_TREE_WINDOW`, it checks if both session and window link are valid and searches for the substring in the window's name.
    - For `WINDOW_TREE_PANE`, it checks if session, window link, and window pane are valid, retrieves the pane's command name, and searches for the substring in this command name.
    - If the command name is found, it frees the allocated memory for the command name and returns the result of the search.
    - If none of the cases match or the conditions are not met, it returns 0.
- **Output**:
    - The function returns an integer indicating whether the substring `ss` is found in the relevant item name (1 if found, 0 if not found or if the item is invalid).
- **Functions called**:
    - [`window_tree_pull_item`](#window_tree_pull_item)


---
### window_tree_menu<!-- {{#callable:window_tree_menu}} -->
The `window_tree_menu` function handles key events for a window tree mode entry in a client, ensuring the correct mode entry is active and passing the key event to the [`window_tree_key`](#window_tree_key) function.
- **Inputs**:
    - `modedata`: A pointer to the window tree mode data, which contains information about the current window pane and mode.
    - `c`: A pointer to the client structure, representing the client that triggered the event.
    - `key`: The key code representing the key event that occurred.
- **Control Flow**:
    - Retrieve the window pane from the mode data.
    - Get the first window mode entry from the window pane's modes queue.
    - Check if the mode entry is null or if its data does not match the provided mode data; if so, return immediately.
    - Call the [`window_tree_key`](#window_tree_key) function with the mode entry, client, and key to handle the key event.
- **Output**:
    - The function does not return any value; it performs actions based on the key event and the current mode entry.
- **Functions called**:
    - [`window_tree_key`](#window_tree_key)


---
### window_tree_get_key<!-- {{#callable:window_tree_get_key}} -->
The `window_tree_get_key` function retrieves a key code based on the format and context of a specific item in a window tree structure.
- **Inputs**:
    - `modedata`: A pointer to `window_tree_modedata` structure containing mode-specific data, including the key format string.
    - `itemdata`: A pointer to `window_tree_itemdata` structure representing the item in the window tree for which the key is being retrieved.
    - `line`: An unsigned integer representing the line number used in formatting the key.
- **Control Flow**:
    - Create a format tree `ft` with no specific format context.
    - Call [`window_tree_pull_item`](#window_tree_pull_item) to extract session, winlink, and window pane information from `itemdata`.
    - Depending on the type of `itemdata`, set default format values in `ft` for session, window, or pane.
    - Add the line number to the format tree `ft` using `format_add`.
    - Expand the format tree `ft` using the key format from `modedata` to get a string representation of the key.
    - Look up the key code from the expanded string using [`key_string_lookup_string`](key-string.c.driver.md#key_string_lookup_string).
    - Free the expanded string and the format tree `ft`.
    - Return the key code.
- **Output**:
    - Returns a `key_code` which is the key code corresponding to the formatted key string for the given item and line.
- **Functions called**:
    - [`format_create`](format.c.driver.md#format_create)
    - [`window_tree_pull_item`](#window_tree_pull_item)
    - [`format_defaults`](format.c.driver.md#format_defaults)
    - [`format_expand`](format.c.driver.md#format_expand)
    - [`key_string_lookup_string`](key-string.c.driver.md#key_string_lookup_string)
    - [`format_free`](format.c.driver.md#format_free)


---
### window_tree_swap<!-- {{#callable:window_tree_swap}} -->
The `window_tree_swap` function swaps two window items within the same session if they are of the same type and meet certain conditions.
- **Inputs**:
    - `cur_itemdata`: A pointer to the current window tree item data structure representing the first window item to be swapped.
    - `other_itemdata`: A pointer to the other window tree item data structure representing the second window item to be swapped.
- **Control Flow**:
    - Check if the types of the current and other items are the same; if not, return 0.
    - Check if the type of the items is WINDOW_TREE_WINDOW; if not, return 0.
    - Retrieve session, winlink, and pane information for both current and other items using [`window_tree_pull_item`](#window_tree_pull_item).
    - Check if both items belong to the same session; if not, return 0.
    - Check if the window tree sort field is not by index and if the windows are not comparable; if so, return 0 to prevent swapping.
    - Remove the winlinks from their respective window's winlink list.
    - Swap the windows between the winlinks and reinsert them into the winlink lists of the new windows.
    - Update the current window of the session if it matches one of the swapped winlinks.
    - Synchronize the session group and redraw the session group.
    - Recalculate the sizes of the windows.
    - Return 1 to indicate a successful swap.
- **Output**:
    - Returns an integer value: 1 if the swap was successful, or 0 if it was not possible to swap the items.
- **Functions called**:
    - [`window_tree_pull_item`](#window_tree_pull_item)
    - [`window_tree_cmp_window`](#window_tree_cmp_window)
    - [`session_set_current`](session.c.driver.md#session_set_current)
    - [`session_group_synchronize_from`](session.c.driver.md#session_group_synchronize_from)
    - [`server_redraw_session_group`](server-fn.c.driver.md#server_redraw_session_group)
    - [`recalculate_sizes`](resize.c.driver.md#recalculate_sizes)


---
### window_tree_init<!-- {{#callable:window_tree_init}} -->
The `window_tree_init` function initializes a window tree mode for a given window mode entry, setting up the necessary data structures and configurations based on provided arguments.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` which represents the window mode entry to be initialized.
    - `fs`: A pointer to a `struct cmd_find_state` which contains the state information for command finding.
    - `args`: A pointer to a `struct args` which holds the arguments that dictate the configuration of the window tree mode.
- **Control Flow**:
    - Allocate memory for `window_tree_modedata` and assign it to `wme->data` and `data`.
    - Set `data->wp` to the window pane from `wme` and initialize `data->references` to 1.
    - Determine the type of window tree (session, window, or pane) based on the presence of specific flags ('s', 'w') in `args` and set `data->type` accordingly.
    - Copy the command find state `fs` into `data->fs`.
    - Set `data->format`, `data->key_format`, and `data->command` based on the presence of 'F', 'K', and other arguments in `args`, using default values if not specified.
    - Set `data->squash_groups` and `data->prompt_flags` based on the presence of 'G' and 'y' flags in `args`.
    - Initialize `data->data` by calling [`mode_tree_start`](mode-tree.c.driver.md#mode_tree_start) with various parameters including function pointers for building, drawing, searching, and handling menu interactions.
    - Call [`mode_tree_zoom`](mode-tree.c.driver.md#mode_tree_zoom) to adjust the zoom level of the mode tree based on `args`.
    - Build and draw the mode tree using [`mode_tree_build`](mode-tree.c.driver.md#mode_tree_build) and [`mode_tree_draw`](mode-tree.c.driver.md#mode_tree_draw).
    - Set `data->type` to `WINDOW_TREE_NONE` to indicate the initialization is complete.
    - Return the screen pointer `s` which is set during [`mode_tree_start`](mode-tree.c.driver.md#mode_tree_start).
- **Output**:
    - Returns a pointer to a `struct screen` which represents the initialized screen for the window tree mode.
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
### window_tree_destroy<!-- {{#callable:window_tree_destroy}} -->
The `window_tree_destroy` function deallocates memory and resources associated with a `window_tree_modedata` structure when its reference count reaches zero.
- **Inputs**:
    - `data`: A pointer to a `window_tree_modedata` structure that holds the data and resources to be freed.
- **Control Flow**:
    - Decrement the reference count of the `data` structure.
    - If the reference count is not zero, return immediately without freeing resources.
    - Iterate over each item in the `item_list` array and free each item using [`window_tree_free_item`](#window_tree_free_item).
    - Free the `item_list` array itself.
    - Free the `format`, `key_format`, and `command` strings.
    - Free the `data` structure itself.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the provided data structure.
- **Functions called**:
    - [`window_tree_free_item`](#window_tree_free_item)


---
### window_tree_free<!-- {{#callable:window_tree_free}} -->
The `window_tree_free` function is responsible for cleaning up and freeing resources associated with a window mode entry in a tree structure.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` which contains the data to be freed.
- **Control Flow**:
    - Retrieve the `window_tree_modedata` from the `wme` structure.
    - Check if the `data` is NULL; if so, return immediately as there is nothing to free.
    - Set the `dead` flag of the `data` to 1, indicating that the data is no longer active.
    - Call [`mode_tree_free`](mode-tree.c.driver.md#mode_tree_free) to free the mode tree data associated with `data->data`.
    - Call [`window_tree_destroy`](#window_tree_destroy) to perform additional cleanup and free the `data` structure itself.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the provided `window_mode_entry`.
- **Functions called**:
    - [`mode_tree_free`](mode-tree.c.driver.md#mode_tree_free)
    - [`window_tree_destroy`](#window_tree_destroy)


---
### window_tree_resize<!-- {{#callable:window_tree_resize}} -->
The `window_tree_resize` function adjusts the size of a mode tree associated with a window mode entry to specified dimensions.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry`, which contains the mode data to be resized.
    - `sx`: An unsigned integer representing the new width for the mode tree.
    - `sy`: An unsigned integer representing the new height for the mode tree.
- **Control Flow**:
    - Retrieve the `window_tree_modedata` from the `wme` structure.
    - Call the [`mode_tree_resize`](mode-tree.c.driver.md#mode_tree_resize) function with the mode data and the new dimensions `sx` and `sy`.
- **Output**:
    - This function does not return a value; it modifies the mode tree's size in place.
- **Functions called**:
    - [`mode_tree_resize`](mode-tree.c.driver.md#mode_tree_resize)


---
### window_tree_update<!-- {{#callable:window_tree_update}} -->
The `window_tree_update` function updates the mode tree data structure and redraws the associated window pane.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure, which contains the mode data to be updated.
- **Control Flow**:
    - Retrieve the `window_tree_modedata` from the `window_mode_entry` structure.
    - Call [`mode_tree_build`](mode-tree.c.driver.md#mode_tree_build) to rebuild the mode tree data structure.
    - Call [`mode_tree_draw`](mode-tree.c.driver.md#mode_tree_draw) to redraw the mode tree based on the updated data.
    - Set the `PANE_REDRAW` flag on the associated window pane to indicate it needs to be redrawn.
- **Output**:
    - The function does not return a value; it updates the mode tree and redraws the window pane as a side effect.
- **Functions called**:
    - [`mode_tree_build`](mode-tree.c.driver.md#mode_tree_build)
    - [`mode_tree_draw`](mode-tree.c.driver.md#mode_tree_draw)


---
### window_tree_get_target<!-- {{#callable:window_tree_get_target}} -->
The `window_tree_get_target` function generates a target string based on the type of window tree item and updates the command find state accordingly.
- **Inputs**:
    - `item`: A pointer to a `window_tree_itemdata` structure representing the item for which the target is being generated.
    - `fs`: A pointer to a `cmd_find_state` structure that will be updated based on the target found.
- **Control Flow**:
    - Call [`window_tree_pull_item`](#window_tree_pull_item) to extract session, winlink, and window pane information from the item.
    - Initialize `target` to NULL.
    - Use a switch statement to determine the type of the item (`WINDOW_TREE_NONE`, `WINDOW_TREE_SESSION`, `WINDOW_TREE_WINDOW`, `WINDOW_TREE_PANE`).
    - For `WINDOW_TREE_SESSION`, if the session is not NULL, format the target as `=<session_name>:`.
    - For `WINDOW_TREE_WINDOW`, if both session and winlink are not NULL, format the target as `=<session_name>:<winlink_index>.`.
    - For `WINDOW_TREE_PANE`, if session, winlink, and window pane are not NULL, format the target as `=<session_name>:<winlink_index>.%<pane_id>`.
    - If `target` is still NULL, clear the command find state using [`cmd_find_clear_state`](cmd-find.c.driver.md#cmd_find_clear_state).
    - If `target` is not NULL, update the command find state using [`cmd_find_from_winlink_pane`](cmd-find.c.driver.md#cmd_find_from_winlink_pane).
    - Return the `target` string.
- **Output**:
    - Returns a dynamically allocated string representing the target, or NULL if no valid target could be generated.
- **Functions called**:
    - [`window_tree_pull_item`](#window_tree_pull_item)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`cmd_find_clear_state`](cmd-find.c.driver.md#cmd_find_clear_state)
    - [`cmd_find_from_winlink_pane`](cmd-find.c.driver.md#cmd_find_from_winlink_pane)


---
### window_tree_command_each<!-- {{#callable:window_tree_command_each}} -->
The `window_tree_command_each` function executes a command for each tagged item in a window tree, using the item's target information.
- **Inputs**:
    - `modedata`: A pointer to `window_tree_modedata`, which contains the mode data for the window tree.
    - `itemdata`: A pointer to `window_tree_itemdata`, which contains the data for the specific item in the window tree.
    - `c`: A pointer to a `client` structure, representing the client for which the command is being executed.
    - `key`: A `key_code` representing the key pressed, which is unused in this function.
- **Control Flow**:
    - Retrieve the target name for the item using [`window_tree_get_target`](#window_tree_get_target) and store it in `name`.
    - If `name` is not NULL, execute the command using [`mode_tree_run_command`](mode-tree.c.driver.md#mode_tree_run_command) with the client, command find state, entered command, and target name.
    - Free the memory allocated for `name`.
- **Output**:
    - The function does not return a value; it performs actions based on the item data and client.
- **Functions called**:
    - [`window_tree_get_target`](#window_tree_get_target)
    - [`mode_tree_run_command`](mode-tree.c.driver.md#mode_tree_run_command)


---
### window_tree_command_done<!-- {{#callable:window_tree_command_done}} -->
The `window_tree_command_done` function finalizes the window tree command execution by rebuilding and redrawing the mode tree if the data is not marked as dead, and then destroys the mode data.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which is unused in this function.
    - `modedata`: A pointer to `window_tree_modedata` structure, which contains the mode data for the window tree.
- **Control Flow**:
    - Check if the `data` is not marked as dead.
    - If not dead, call [`mode_tree_build`](mode-tree.c.driver.md#mode_tree_build) to rebuild the mode tree using `data->data`.
    - Call [`mode_tree_draw`](mode-tree.c.driver.md#mode_tree_draw) to redraw the mode tree using `data->data`.
    - Set the `PANE_REDRAW` flag on `data->wp` to indicate that the pane needs to be redrawn.
    - Call [`window_tree_destroy`](#window_tree_destroy) to clean up and free the resources associated with `data`.
    - Return `CMD_RETURN_NORMAL` to indicate normal command completion.
- **Output**:
    - The function returns `CMD_RETURN_NORMAL`, indicating successful completion of the command.
- **Functions called**:
    - [`mode_tree_build`](mode-tree.c.driver.md#mode_tree_build)
    - [`mode_tree_draw`](mode-tree.c.driver.md#mode_tree_draw)
    - [`window_tree_destroy`](#window_tree_destroy)


---
### window_tree_command_callback<!-- {{#callable:window_tree_command_callback}} -->
The `window_tree_command_callback` function processes a command string for a window tree mode, executing actions on tagged items and scheduling a callback for completion.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client that initiated the command.
    - `modedata`: A pointer to `window_tree_modedata` structure containing the mode data for the window tree.
    - `s`: A constant character pointer representing the command string to be processed.
    - `done`: An integer flag indicating whether the command processing is complete, but it is unused in this function.
- **Control Flow**:
    - Check if the command string `s` is NULL, empty, or if the mode data is marked as dead; if any of these conditions are true, return 0 immediately.
    - Assign the command string `s` to the `entered` field of the mode data structure.
    - Invoke [`mode_tree_each_tagged`](mode-tree.c.driver.md#mode_tree_each_tagged) to execute `window_tree_command_each` on each tagged item in the mode tree, passing the client `c` and a key code of `KEYC_NONE`.
    - Reset the `entered` field of the mode data structure to NULL after processing the command.
    - Increment the `references` count of the mode data structure to indicate an ongoing operation.
    - Append a command queue item to the client `c` using [`cmdq_append`](cmd-queue.c.driver.md#cmdq_append), with a callback to `window_tree_command_done` and the mode data as arguments.
    - Return 0 to indicate the function has completed its execution.
- **Output**:
    - The function returns an integer value of 0, indicating successful execution without errors.
- **Functions called**:
    - [`mode_tree_each_tagged`](mode-tree.c.driver.md#mode_tree_each_tagged)
    - [`cmdq_append`](cmd-queue.c.driver.md#cmdq_append)


---
### window_tree_command_free<!-- {{#callable:window_tree_command_free}} -->
The `window_tree_command_free` function is responsible for freeing the resources associated with a `window_tree_modedata` structure.
- **Inputs**:
    - `modedata`: A pointer to the `window_tree_modedata` structure that needs to be freed.
- **Control Flow**:
    - The function casts the `modedata` pointer to a `window_tree_modedata` pointer named `data`.
    - It calls the [`window_tree_destroy`](#window_tree_destroy) function, passing `data` as an argument to free the resources associated with the `window_tree_modedata` structure.
- **Output**:
    - This function does not return any value; it performs cleanup operations on the provided data structure.
- **Functions called**:
    - [`window_tree_destroy`](#window_tree_destroy)


---
### window_tree_kill_each<!-- {{#callable:window_tree_kill_each}} -->
The `window_tree_kill_each` function processes a window tree item and performs a kill operation based on the item's type, which can be a session, window, or pane.
- **Inputs**:
    - `modedata`: This is unused in the function.
    - `itemdata`: A pointer to a `window_tree_itemdata` structure representing the item to be processed.
    - `c`: This is unused in the function.
    - `key`: This is unused in the function.
- **Control Flow**:
    - The function begins by casting `itemdata` to a `window_tree_itemdata` structure pointer named `item`.
    - It then calls [`window_tree_pull_item`](#window_tree_pull_item) to extract the session, winlink, and window pane associated with the item.
    - A switch statement is used to determine the type of the item (`WINDOW_TREE_NONE`, `WINDOW_TREE_SESSION`, `WINDOW_TREE_WINDOW`, or `WINDOW_TREE_PANE`).
    - If the item type is `WINDOW_TREE_SESSION` and the session is not NULL, it calls [`server_destroy_session`](server-fn.c.driver.md#server_destroy_session) and [`session_destroy`](session.c.driver.md#session_destroy) to destroy the session.
    - If the item type is `WINDOW_TREE_WINDOW` and the winlink is not NULL, it calls [`server_kill_window`](server-fn.c.driver.md#server_kill_window) to kill the window.
    - If the item type is `WINDOW_TREE_PANE` and the window pane is not NULL, it calls [`server_kill_pane`](server-fn.c.driver.md#server_kill_pane) to kill the pane.
- **Output**:
    - The function does not return any value; it performs actions based on the type of the item.
- **Functions called**:
    - [`window_tree_pull_item`](#window_tree_pull_item)
    - [`server_destroy_session`](server-fn.c.driver.md#server_destroy_session)
    - [`session_destroy`](session.c.driver.md#session_destroy)
    - [`server_kill_window`](server-fn.c.driver.md#server_kill_window)
    - [`server_kill_pane`](server-fn.c.driver.md#server_kill_pane)


---
### window_tree_kill_current_callback<!-- {{#callable:window_tree_kill_current_callback}} -->
The `window_tree_kill_current_callback` function processes a confirmation input to kill the currently selected item in a window tree, and if confirmed, executes the kill operation and updates the server state.
- **Inputs**:
    - `c`: A pointer to the client structure, representing the client that initiated the callback.
    - `modedata`: A pointer to the window tree mode data structure, which contains the state and data for the window tree mode.
    - `s`: A string representing the user's input, typically a confirmation response.
    - `done`: An integer flag indicating whether the operation is complete, but it is unused in this function.
- **Control Flow**:
    - Check if the input string `s` is NULL, empty, or if the mode data is marked as dead; if any of these conditions are true, return 0 to indicate no action is taken.
    - Convert the first character of `s` to lowercase and check if it is 'y' and if `s` has no other characters; if not, return 0 to indicate no action is taken.
    - Call [`window_tree_kill_each`](#window_tree_kill_each) to perform the kill operation on the current item in the mode tree.
    - Call [`server_renumber_all`](server-fn.c.driver.md#server_renumber_all) to update the server's numbering of sessions, windows, or panes.
    - Increment the reference count of the mode data to manage its lifecycle.
    - Append a command queue item to execute `window_tree_command_done` with the mode data, ensuring the command is processed after the current operation.
- **Output**:
    - The function returns an integer 0, indicating the completion of the callback without errors.
- **Functions called**:
    - [`window_tree_kill_each`](#window_tree_kill_each)
    - [`mode_tree_get_current`](mode-tree.c.driver.md#mode_tree_get_current)
    - [`server_renumber_all`](server-fn.c.driver.md#server_renumber_all)
    - [`cmdq_append`](cmd-queue.c.driver.md#cmdq_append)


---
### window_tree_kill_tagged_callback<!-- {{#callable:window_tree_kill_tagged_callback}} -->
The `window_tree_kill_tagged_callback` function processes a confirmation input to kill all tagged items in a window tree and updates the server state accordingly.
- **Inputs**:
    - `c`: A pointer to the client structure, representing the client that initiated the callback.
    - `modedata`: A pointer to the window_tree_modedata structure, which contains the mode tree data and other related information.
    - `s`: A string representing the user's input, typically a confirmation response.
    - `done`: An integer flag indicating whether the operation is complete, but it is unused in this function.
- **Control Flow**:
    - Check if the input string `s` is NULL, empty, or if the `data->dead` flag is set; if any of these conditions are true, return 0 immediately.
    - Convert the first character of `s` to lowercase and check if it is 'y' and if `s` has no other characters; if not, return 0.
    - Call [`mode_tree_each_tagged`](mode-tree.c.driver.md#mode_tree_each_tagged) to apply the `window_tree_kill_each` function to each tagged item in the mode tree data, passing the client `c` and a flag `KEYC_NONE`.
    - Call [`server_renumber_all`](server-fn.c.driver.md#server_renumber_all) to update the numbering of all server entities.
    - Increment the `data->references` counter to indicate an additional reference to the mode data.
    - Append a command queue item to the client `c` using [`cmdq_append`](cmd-queue.c.driver.md#cmdq_append), with a callback to `window_tree_command_done` and the mode data `data`.
    - Return 0 to indicate the callback has completed its processing.
- **Output**:
    - The function returns an integer value, always 0, indicating the callback has completed its processing.
- **Functions called**:
    - [`mode_tree_each_tagged`](mode-tree.c.driver.md#mode_tree_each_tagged)
    - [`server_renumber_all`](server-fn.c.driver.md#server_renumber_all)
    - [`cmdq_append`](cmd-queue.c.driver.md#cmdq_append)


---
### window_tree_mouse<!-- {{#callable:window_tree_mouse}} -->
The `window_tree_mouse` function handles mouse click events within a window tree structure, determining the appropriate action based on the mouse position and the type of item clicked.
- **Inputs**:
    - `data`: A pointer to a `window_tree_modedata` structure containing the state and configuration of the window tree.
    - `key`: A `key_code` representing the key or mouse event that triggered the function, specifically checking for `KEYC_MOUSEDOWN1_PANE`.
    - `x`: An unsigned integer representing the x-coordinate of the mouse click within the window tree.
    - `item`: A pointer to a `window_tree_itemdata` structure representing the item in the window tree that was clicked.
- **Control Flow**:
    - Check if the key is `KEYC_MOUSEDOWN1_PANE`; if not, return `KEYC_NONE`.
    - Determine if the click is on the left or right navigation area based on `data->left` and `data->right`, returning '<' or '>' respectively if true.
    - Adjust the x-coordinate relative to `data->left` if applicable, and calculate the index `x` within the visible range of items.
    - Call [`window_tree_pull_item`](#window_tree_pull_item) to retrieve the session, winlink, and window pane associated with the clicked item.
    - If the item type is `WINDOW_TREE_SESSION`, expand the current mode tree and iterate through the session's windows to find the clicked window, setting it as current if found.
    - If the item type is `WINDOW_TREE_WINDOW`, expand the current mode tree and iterate through the window's panes to find the clicked pane, setting it as current if found.
    - Return `KEYC_NONE` if no valid action is determined.
- **Output**:
    - Returns a `key_code` indicating the action to be taken, such as '<', '>', '\r', or `KEYC_NONE` if no action is applicable.
- **Functions called**:
    - [`window_tree_pull_item`](#window_tree_pull_item)
    - [`mode_tree_expand_current`](mode-tree.c.driver.md#mode_tree_expand_current)
    - [`mode_tree_set_current`](mode-tree.c.driver.md#mode_tree_set_current)


---
### window_tree_key<!-- {{#callable:window_tree_key}} -->
The `window_tree_key` function handles key and mouse events for navigating and interacting with a tree-like structure in a window pane, allowing operations such as expanding nodes, marking items, and executing commands.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure representing the current mode entry for the window.
    - `c`: A pointer to a `client` structure representing the client interacting with the window.
    - `s`: An unused pointer to a `session` structure.
    - `wl`: An unused pointer to a `winlink` structure.
    - `key`: A `key_code` representing the key event to be processed.
    - `m`: A pointer to a `mouse_event` structure representing the mouse event to be processed.
- **Control Flow**:
    - Retrieve the current item from the mode tree using [`mode_tree_get_current`](mode-tree.c.driver.md#mode_tree_get_current) and check if the key event is finished using [`mode_tree_key`](mode-tree.c.driver.md#mode_tree_key).
    - If the current item changes, update the item and reset the offset.
    - If the key is a mouse event, process it using [`window_tree_mouse`](#window_tree_mouse) and repeat the check for item change.
    - Handle different key codes with a switch statement, performing actions such as adjusting offsets, expanding nodes, marking items, and setting prompts for commands or kills.
    - If the key is a carriage return, execute the command associated with the current item.
    - If the operation is finished, reset the window pane mode; otherwise, redraw the mode tree and set the pane redraw flag.
- **Output**:
    - The function does not return a value but modifies the state of the window pane and mode tree based on the key or mouse event.
- **Functions called**:
    - [`mode_tree_get_current`](mode-tree.c.driver.md#mode_tree_get_current)
    - [`mode_tree_key`](mode-tree.c.driver.md#mode_tree_key)
    - [`window_tree_mouse`](#window_tree_mouse)
    - [`mode_tree_expand`](mode-tree.c.driver.md#mode_tree_expand)
    - [`mode_tree_set_current`](mode-tree.c.driver.md#mode_tree_set_current)
    - [`window_tree_pull_item`](#window_tree_pull_item)
    - [`server_set_marked`](server.c.driver.md#server_set_marked)
    - [`mode_tree_build`](mode-tree.c.driver.md#mode_tree_build)
    - [`server_clear_marked`](server.c.driver.md#server_clear_marked)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`window_pane_index`](window.c.driver.md#window_pane_index)
    - [`status_prompt_set`](status.c.driver.md#status_prompt_set)
    - [`mode_tree_count_tagged`](mode-tree.c.driver.md#mode_tree_count_tagged)
    - [`window_tree_get_target`](#window_tree_get_target)
    - [`mode_tree_run_command`](mode-tree.c.driver.md#mode_tree_run_command)
    - [`window_pane_reset_mode`](window.c.driver.md#window_pane_reset_mode)
    - [`mode_tree_draw`](mode-tree.c.driver.md#mode_tree_draw)


