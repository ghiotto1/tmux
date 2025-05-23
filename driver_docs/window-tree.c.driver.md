# Purpose
This C source code file is part of the `tmux` project, specifically implementing a "tree mode" for displaying and interacting with sessions, windows, and panes in a hierarchical manner. The code defines a set of functions and data structures that manage the initialization, rendering, and interaction of this tree view within a `tmux` window. The primary components include functions for initializing the tree ([`window_tree_init`](#window_tree_init)), handling user input ([`window_tree_key`](#window_tree_key)), and rendering the tree structure ([`window_tree_draw`](#window_tree_draw)). The code also defines sorting and filtering mechanisms to organize the tree items based on criteria such as index, name, or time.

The file is structured to provide a cohesive set of functionalities for managing the tree view, including handling user commands, updating the display, and managing the lifecycle of the tree mode. It defines a public API through the `window_tree_mode` structure, which includes function pointers to the key operations like initialization, resizing, updating, and handling key events. This API allows the tree mode to be integrated into the broader `tmux` application, enabling users to navigate and manipulate sessions, windows, and panes efficiently. The code also includes a menu system for user interaction and supports custom commands and key bindings, enhancing the flexibility and usability of the tree mode within `tmux`.
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
- **Type**: `struct screen*`
- **Description**: The `window_tree_init` function is a static function that initializes a window tree mode entry. It returns a pointer to a `struct screen`, which represents the screen associated with the window tree mode.
- **Use**: This function is used to set up the window tree mode by allocating and initializing the necessary data structures and configurations.


---
### window_tree_menu_items
- **Type**: ``static const struct menu_item[]``
- **Description**: The `window_tree_menu_items` is a static constant array of `struct menu_item` that defines a set of menu items for a window tree interface. Each element in the array represents a menu option with a label, a key code for selection, and a placeholder for additional data (currently set to NULL).
- **Use**: This array is used to define the menu options available in the window tree interface, allowing users to interact with the interface using specific key bindings.


---
### window_tree_mode
- **Type**: ``const struct window_mode``
- **Description**: The `window_tree_mode` is a constant structure of type `window_mode` that defines a specific mode for a window, named "tree-mode". It includes function pointers for initialization, freeing resources, resizing, updating, and handling key events, as well as a default format for displaying the window tree.
- **Use**: This variable is used to define and manage the behavior and display format of the "tree-mode" in a window, providing a structured way to handle various operations related to this mode.


---
### window_tree_sort_list
- **Type**: ``const char *` array`
- **Description**: `window_tree_sort_list` is a static constant array of strings that defines the sorting criteria for a window tree structure. It contains three elements: "index", "name", and "time".
- **Use**: This array is used to specify the available sorting options for organizing window tree items.


---
### window_tree_sort
- **Type**: `struct mode_tree_sort_criteria*`
- **Description**: `window_tree_sort` is a static pointer to a `struct mode_tree_sort_criteria`, which is used to define the sorting criteria for the window tree structure in the tmux application. This structure likely contains fields that specify how the windows, sessions, or panes should be sorted, such as by index, name, or time, and whether the sorting should be reversed.
- **Use**: This variable is used to determine the sorting order of items in the window tree, affecting how sessions, windows, and panes are displayed and organized.


# Data Structures

---
### window_tree_sort_type
- **Type**: `enum`
- **Members**:
    - `WINDOW_TREE_BY_INDEX`: Sorts the window tree by index.
    - `WINDOW_TREE_BY_NAME`: Sorts the window tree by name.
    - `WINDOW_TREE_BY_TIME`: Sorts the window tree by time.
- **Description**: The `window_tree_sort_type` is an enumeration that defines the sorting criteria for a window tree structure. It provides three options for sorting: by index, by name, and by time. This enumeration is used to determine the order in which windows, sessions, or panes are displayed in a tree view, allowing users to organize and navigate through them based on their preferred sorting method.


---
### window_tree_type
- **Type**: `enum`
- **Members**:
    - `WINDOW_TREE_NONE`: Represents a state where no specific window tree type is selected.
    - `WINDOW_TREE_SESSION`: Represents a session in the window tree.
    - `WINDOW_TREE_WINDOW`: Represents a window in the window tree.
    - `WINDOW_TREE_PANE`: Represents a pane in the window tree.
- **Description**: The `window_tree_type` is an enumeration that defines different types of entities that can be represented in a window tree structure. It includes four possible values: `WINDOW_TREE_NONE`, `WINDOW_TREE_SESSION`, `WINDOW_TREE_WINDOW`, and `WINDOW_TREE_PANE`, each representing a different level or component within a window management system, such as a session, a window, or a pane. This enumeration is used to categorize and manage these components within the context of a window tree, allowing for operations and manipulations specific to each type.


---
### window_tree_itemdata
- **Type**: `struct`
- **Members**:
    - `type`: An enumeration of type `window_tree_type` that indicates the type of item (session, window, or pane).
    - `session`: An integer representing the session ID associated with the item.
    - `winlink`: An integer representing the window link index associated with the item.
    - `pane`: An integer representing the pane ID associated with the item.
- **Description**: The `window_tree_itemdata` structure is used to represent an item in a window tree, which can be a session, window, or pane. It contains information about the type of the item, as well as identifiers for the session, window link, and pane associated with the item. This structure is part of a larger system for managing and displaying a hierarchical view of sessions, windows, and panes in a terminal multiplexer environment.


---
### window_tree_modedata
- **Type**: `struct`
- **Members**:
    - `wp`: Pointer to a window pane associated with the mode data.
    - `dead`: Flag indicating if the mode data is no longer active.
    - `references`: Count of references to this mode data structure.
    - `data`: Pointer to mode tree data structure for managing tree items.
    - `format`: String format for displaying tree items.
    - `key_format`: String format for key bindings in the tree.
    - `command`: Command string to execute when an item is selected.
    - `squash_groups`: Flag to determine if session groups should be squashed.
    - `prompt_flags`: Flags for prompt behavior in the mode.
    - `item_list`: Array of pointers to window tree item data structures.
    - `item_size`: Number of items in the item_list array.
    - `entered`: String representing the last entered command or input.
    - `fs`: Command find state structure for locating sessions, windows, or panes.
    - `type`: Enumeration indicating the type of tree (session, window, or pane).
    - `offset`: Offset for scrolling or navigating within the tree.
    - `left`: Position of the left navigation indicator.
    - `right`: Position of the right navigation indicator.
    - `start`: Start index for visible items in the tree.
    - `end`: End index for visible items in the tree.
    - `each`: Width of each item in the tree display.
- **Description**: The `window_tree_modedata` structure is used to manage and display a hierarchical tree of sessions, windows, and panes within a terminal multiplexer environment. It holds pointers to associated window panes, mode tree data, and item lists, along with various configuration strings and flags for formatting, commands, and navigation. The structure also maintains state information such as offsets and indices for navigating and displaying the tree, as well as reference counts and flags for managing the lifecycle of the mode data.


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
    - Check if the window contains the pane; if not, set `wp` to NULL.
    - If `wp` is NULL, set `sp` and `wlp` to NULL and return.
- **Output**:
    - The function does not return a value but modifies the pointers `sp`, `wlp`, and `wp` to point to the appropriate session, winlink, and window pane based on the item data.


---
### window_tree_add_item<!-- {{#callable:window_tree_add_item}} -->
The `window_tree_add_item` function adds a new item to a list of window tree items, expanding the list if necessary, and returns the newly added item.
- **Inputs**:
    - `data`: A pointer to a `window_tree_modedata` structure, which contains the list of window tree items and its current size.
- **Control Flow**:
    - The function begins by reallocating memory for the `item_list` array in the `data` structure to accommodate one more item, using `xreallocarray`.
    - A new `window_tree_itemdata` structure is allocated and initialized to zero using `xcalloc`.
    - The newly allocated item is added to the `item_list` at the current `item_size` index, and the `item_size` is incremented.
    - The function returns a pointer to the newly added `window_tree_itemdata` item.
- **Output**:
    - A pointer to the newly added `window_tree_itemdata` structure.


---
### window_tree_free_item<!-- {{#callable:window_tree_free_item}} -->
The `window_tree_free_item` function deallocates memory for a `window_tree_itemdata` structure.
- **Inputs**:
    - `item`: A pointer to a `window_tree_itemdata` structure that needs to be freed.
- **Control Flow**:
    - The function takes a pointer to a `window_tree_itemdata` structure as an argument.
    - It calls the `free` function to deallocate the memory associated with the `item` pointer.
- **Output**:
    - The function does not return any value; it performs a side effect by freeing the memory of the given `window_tree_itemdata` structure.


---
### window_tree_cmp_session<!-- {{#callable:window_tree_cmp_session}} -->
The `window_tree_cmp_session` function compares two session objects based on a specified sorting criterion and returns an integer indicating their relative order.
- **Inputs**:
    - `a0`: A pointer to the first session object to be compared.
    - `b0`: A pointer to the second session object to be compared.
- **Control Flow**:
    - The function casts the input pointers to session pointers and dereferences them to obtain the session objects `sa` and `sb`.
    - It initializes a result variable to 0.
    - A switch statement checks the sorting field specified in `window_tree_sort->field`.
    - If the field is `WINDOW_TREE_BY_INDEX`, it compares the session IDs and assigns the difference to `result`.
    - If the field is `WINDOW_TREE_BY_TIME`, it uses `timercmp` to compare the activity times of the sessions and sets `result` to -1, 1, or falls through if they are equal.
    - If the field is `WINDOW_TREE_BY_NAME`, it uses `strcmp` to compare the session names and assigns the result to `result`.
    - If `window_tree_sort->reversed` is true, it negates the `result`.
    - The function returns the `result`.
- **Output**:
    - An integer indicating the relative order of the two sessions: negative if the first is less than the second, zero if they are equal, and positive if the first is greater than the second.


---
### window_tree_cmp_window<!-- {{#callable:window_tree_cmp_window}} -->
The `window_tree_cmp_window` function compares two `winlink` structures based on a specified sorting criterion and returns an integer indicating their relative order.
- **Inputs**:
    - `a0`: A pointer to the first `winlink` structure to be compared.
    - `b0`: A pointer to the second `winlink` structure to be compared.
- **Control Flow**:
    - The function casts the input pointers `a0` and `b0` to pointers to `const struct winlink *` and dereferences them to obtain `wla` and `wlb`, respectively.
    - It retrieves the `window` structures `wa` and `wb` from `wla` and `wlb`.
    - The function initializes a variable `result` to 0 to store the comparison result.
    - A switch statement is used to determine the sorting criterion based on `window_tree_sort->field`.
    - If the sorting criterion is `WINDOW_TREE_BY_INDEX`, it calculates the difference between the indices of `wla` and `wlb` and assigns it to `result`.
    - If the sorting criterion is `WINDOW_TREE_BY_TIME`, it compares the `activity_time` of `wa` and `wb` using `timercmp` and sets `result` to -1, 1, or 0 based on the comparison.
    - If the sorting criterion is `WINDOW_TREE_BY_NAME`, it uses `strcmp` to compare the names of `wa` and `wb` and assigns the result to `result`.
    - If `window_tree_sort->reversed` is true, the function negates `result` to reverse the order.
    - The function returns the `result` value.
- **Output**:
    - An integer indicating the relative order of the two `winlink` structures: negative if the first is less than the second, positive if greater, and zero if they are equal.


---
### window_tree_cmp_pane<!-- {{#callable:window_tree_cmp_pane}} -->
The `window_tree_cmp_pane` function compares two window panes based on a specified sorting criterion, either by active time or by index, and returns an integer indicating their relative order.
- **Inputs**:
    - `a0`: A pointer to the first window pane to be compared.
    - `b0`: A pointer to the second window pane to be compared.
- **Control Flow**:
    - The function casts the input pointers `a0` and `b0` to pointers to pointers of `struct window_pane`.
    - It checks if the sorting field is `WINDOW_TREE_BY_TIME`.
    - If sorting by time, it calculates the difference between the `active_point` of the two panes.
    - If not sorting by time, it retrieves the index of each pane using `window_pane_index` and calculates the difference between these indices.
    - If the sorting order is reversed, it negates the result.
    - Finally, it returns the result indicating the order of the two panes.
- **Output**:
    - An integer that is negative, zero, or positive if the first pane is considered less than, equal to, or greater than the second pane, respectively.


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
    - Retrieve the index of the window pane using `window_pane_index` and store it in `idx`.
    - Create a new item in the mode tree using [`window_tree_add_item`](#window_tree_add_item) and set its type to `WINDOW_TREE_PANE`.
    - Assign the session ID, winlink index, and pane ID to the new item.
    - Create a format tree with `format_create` and set default values using `format_defaults`.
    - Expand the format tree to a string using `format_expand` and store it in `text`.
    - Format the pane index into a string `name` using `xasprintf`.
    - Free the format tree using `format_free`.
    - Add the new item to the mode tree with `mode_tree_add`, using the parent item, pane ID, name, and text.
    - Free the allocated `text` and `name` strings.
    - Align the mode tree item using `mode_tree_align`.
- **Output**:
    - The function does not return a value; it modifies the mode tree structure by adding a new pane item.
- **Functions called**:
    - [`window_tree_add_item`](#window_tree_add_item)


---
### window_tree_filter_pane<!-- {{#callable:window_tree_filter_pane}} -->
The `window_tree_filter_pane` function evaluates a filter string against a specific window pane and returns whether the filter condition is met.
- **Inputs**:
    - `s`: A pointer to a `session` structure representing the current session.
    - `wl`: A pointer to a `winlink` structure representing the window link within the session.
    - `wp`: A pointer to a `window_pane` structure representing the specific pane to be filtered.
    - `filter`: A constant character pointer to the filter string to be applied to the window pane.
- **Control Flow**:
    - Check if the `filter` is `NULL`; if so, return `1` indicating the filter condition is met by default.
    - Use `format_single` to format the `filter` string with the session, winlink, and window pane data, storing the result in `cp`.
    - Evaluate the formatted string `cp` using `format_true` to determine if the filter condition is true, storing the result in `result`.
    - Free the memory allocated for `cp`.
    - Return the value of `result`, indicating whether the filter condition is met.
- **Output**:
    - Returns an integer `1` if the filter condition is met, otherwise `0`.


---
### window_tree_build_window<!-- {{#callable:window_tree_build_window}} -->
The `window_tree_build_window` function constructs a tree representation of a window and its panes, applying filters and sorting criteria, and returns whether any panes were successfully added.
- **Inputs**:
    - `s`: A pointer to the session structure representing the current session.
    - `wl`: A pointer to the winlink structure representing the window link within the session.
    - `modedata`: A pointer to the window_tree_modedata structure containing mode-specific data.
    - `sort_crit`: A pointer to the mode_tree_sort_criteria structure specifying the sorting criteria for the panes.
    - `parent`: A pointer to the mode_tree_item structure representing the parent item in the tree.
    - `filter`: A string representing the filter criteria to apply to the panes.
- **Control Flow**:
    - Initialize a new window_tree_itemdata structure and set its type, session, and winlink properties.
    - Create a format tree and expand it to generate a text representation of the window, then free the format tree.
    - Determine whether the window should be expanded based on the type of data.
    - Add the window item to the mode tree and align it.
    - Check if the window has any panes; if not, clean up and return 0.
    - Iterate over the panes in the window, applying the filter to each pane.
    - If any panes pass the filter, allocate memory for them and store them in an array.
    - Sort the array of panes using the provided sorting criteria.
    - For each pane in the sorted array, call [`window_tree_build_pane`](#window_tree_build_pane) to add it to the tree.
    - Free the array of panes and return 1 if any panes were added, otherwise clean up and return 0.
- **Output**:
    - Returns an integer indicating success (1) if any panes were added to the tree, or failure (0) if no panes were added.
- **Functions called**:
    - [`window_tree_add_item`](#window_tree_add_item)
    - [`window_tree_filter_pane`](#window_tree_filter_pane)
    - [`window_tree_build_pane`](#window_tree_build_pane)
    - [`window_tree_free_item`](#window_tree_free_item)


---
### window_tree_build_session<!-- {{#callable:window_tree_build_session}} -->
The `window_tree_build_session` function constructs a tree representation of a session's windows and panes, applying sorting and filtering criteria.
- **Inputs**:
    - `s`: A pointer to the session structure representing the session to be processed.
    - `modedata`: A pointer to the window tree mode data structure, which holds information about the current mode and items.
    - `sort_crit`: A pointer to the mode tree sort criteria structure, which specifies how the windows should be sorted.
    - `filter`: A string representing a filter to apply to the windows and panes.
- **Control Flow**:
    - Initialize a new item for the session and set its type, session ID, winlink, and pane to default values.
    - Create a format tree and expand it to generate a text representation of the session using the provided format.
    - Determine whether the session should be expanded based on the type of data and add it to the mode tree.
    - Iterate over all windows in the session, reallocating memory for the window list and sorting it using the provided sort criteria.
    - For each window, call [`window_tree_build_window`](#window_tree_build_window) to build its representation, incrementing the empty counter if the window is filtered out.
    - If all windows are filtered out (empty equals the number of windows), remove the session item from the mode tree and adjust the item size.
    - Free the allocated memory for the window list.
- **Output**:
    - The function does not return a value; it modifies the mode tree data structure to include the session and its windows.
- **Functions called**:
    - [`window_tree_add_item`](#window_tree_add_item)
    - [`window_tree_build_window`](#window_tree_build_window)
    - [`window_tree_free_item`](#window_tree_free_item)


---
### window_tree_build<!-- {{#callable:window_tree_build}} -->
The `window_tree_build` function constructs a tree structure of sessions, windows, and panes based on specified sorting criteria and filters, and updates a tag with the current focus.
- **Inputs**:
    - `modedata`: A pointer to `window_tree_modedata` structure containing the current state and configuration for the window tree.
    - `sort_crit`: A pointer to `mode_tree_sort_criteria` structure that defines the sorting criteria for the sessions, windows, and panes.
    - `tag`: A pointer to a `uint64_t` where the function will store the identifier of the current session, window, or pane.
    - `filter`: A string used to filter the sessions, windows, or panes to be included in the tree.
- **Control Flow**:
    - Initialize the current session group from the `modedata` structure.
    - Free and reset the existing item list in `modedata`.
    - Iterate over all sessions using `RB_FOREACH` to build a list of sessions, applying group squashing logic if necessary.
    - Sort the session list using the provided sorting criteria.
    - Iterate over the sorted session list and call [`window_tree_build_session`](#window_tree_build_session) for each session to build the tree structure.
    - Free the session list after processing.
    - Set the `tag` based on the type of the current focus (session, window, or pane) in `modedata`.
- **Output**:
    - The function updates the `tag` with the identifier of the current session, window, or pane, and modifies the `modedata` structure to reflect the newly built tree.
- **Functions called**:
    - [`window_tree_free_item`](#window_tree_free_item)
    - [`window_tree_build_session`](#window_tree_build_session)


---
### window_tree_draw_label<!-- {{#callable:window_tree_draw_label}} -->
The `window_tree_draw_label` function draws a centered label on a screen context, optionally surrounded by a box, based on the provided dimensions and grid cell properties.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure, representing the screen context where the label will be drawn.
    - `px`: An unsigned integer representing the x-coordinate of the starting position for drawing the label.
    - `py`: An unsigned integer representing the y-coordinate of the starting position for drawing the label.
    - `sx`: An unsigned integer representing the width of the area available for drawing the label.
    - `sy`: An unsigned integer representing the height of the area available for drawing the label.
    - `gc`: A pointer to a `grid_cell` structure that defines the appearance (such as color and attributes) of the text to be drawn.
    - `label`: A constant character pointer to the string that represents the label to be drawn.
- **Control Flow**:
    - Calculate the length of the label string using `strlen`.
    - Check if the width (`sx`) is zero, the height (`sy`) is one, or the label length exceeds the width; if any condition is true, return immediately without drawing.
    - Calculate the horizontal offset (`ox`) to center the label within the width, and the vertical offset (`oy`) to center it within the height.
    - If the label can be surrounded by a box (based on offsets and dimensions), move the cursor to the top-left corner of the box and draw a box using `screen_write_box`.
    - Move the cursor to the position where the label should be drawn, using `screen_write_cursormove`.
    - Draw the label at the calculated position using `screen_write_puts`.
- **Output**:
    - The function does not return a value; it performs drawing operations on the provided screen context.


---
### window_tree_draw_session<!-- {{#callable:window_tree_draw_session}} -->
The `window_tree_draw_session` function is responsible for rendering a visual representation of a session's windows in a tree-like structure on the screen.
- **Inputs**:
    - `data`: A pointer to a `window_tree_modedata` structure containing session-specific data and drawing parameters.
    - `s`: A pointer to a `session` structure representing the session whose windows are to be drawn.
    - `ctx`: A pointer to a `screen_write_ctx` structure used for writing to the screen.
    - `sx`: An unsigned integer representing the width of the screen area available for drawing.
    - `sy`: An unsigned integer representing the height of the screen area available for drawing.
- **Control Flow**:
    - Calculate the total number of windows in the session using `winlink_count`.
    - Determine the number of windows that can be displayed based on the available screen width (`sx`).
    - Identify the current window index and calculate the start and end indices for visible windows.
    - Adjust the start and end indices based on the `data->offset` to handle scrolling.
    - Determine if left and right navigation indicators should be displayed based on the start and end indices.
    - Calculate the width allocated for each window and handle any remaining space.
    - Draw left and right navigation indicators if applicable.
    - Iterate over the windows in the session, drawing each window's preview and label within the calculated width.
    - Adjust the drawing position for each window and handle the drawing of vertical lines between windows.
- **Output**:
    - The function does not return a value; it directly modifies the screen context to display the session's windows.
- **Functions called**:
    - [`window_tree_draw_label`](#window_tree_draw_label)


---
### window_tree_draw_window<!-- {{#callable:window_tree_draw_window}} -->
The `window_tree_draw_window` function is responsible for rendering a visual representation of a window's panes within a given screen context, adjusting for visibility and active pane highlighting.
- **Inputs**:
    - `data`: A pointer to a `window_tree_modedata` structure containing mode-specific data, including offsets and pane visibility settings.
    - `s`: A pointer to a `session` structure representing the current session.
    - `w`: A pointer to a `window` structure representing the window whose panes are to be drawn.
    - `ctx`: A pointer to a `screen_write_ctx` structure used for writing to the screen.
    - `sx`: An unsigned integer representing the width of the screen area available for drawing.
    - `sy`: An unsigned integer representing the height of the screen area available for drawing.
- **Control Flow**:
    - Calculate the total number of panes in the window using `window_count_panes`.
    - Determine the color settings for inactive and active panes from session options.
    - Calculate the number of visible panes based on the screen width and total panes.
    - Identify the current active pane and calculate the start and end indices for visible panes.
    - Adjust the start and end indices based on the data offset to ensure proper scrolling.
    - Determine if left and right navigation indicators should be displayed based on visibility and screen width.
    - Calculate the width of each pane to be displayed and handle any remaining space.
    - Draw left and right navigation indicators if applicable.
    - Iterate over the panes within the visible range, setting the foreground color based on active status.
    - For each visible pane, calculate its offset and width, then draw the pane preview and label.
    - Draw vertical lines between panes if necessary.
- **Output**:
    - The function does not return a value; it modifies the screen context to visually represent the window's panes.
- **Functions called**:
    - [`window_tree_draw_label`](#window_tree_draw_label)


---
### window_tree_draw<!-- {{#callable:window_tree_draw}} -->
The `window_tree_draw` function renders a visual representation of a session, window, or pane in a terminal screen context based on the type of item data provided.
- **Inputs**:
    - `modedata`: A pointer to mode-specific data, which may contain configuration or state information for the drawing operation.
    - `itemdata`: A pointer to a `window_tree_itemdata` structure that specifies the type of item (session, window, or pane) to be drawn.
    - `ctx`: A pointer to a `screen_write_ctx` structure that represents the context in which the drawing operations will be performed.
    - `sx`: An unsigned integer representing the width of the area in which to draw.
    - `sy`: An unsigned integer representing the height of the area in which to draw.
- **Control Flow**:
    - The function begins by extracting session, winlink, and window pane information from the `itemdata` using [`window_tree_pull_item`](#window_tree_pull_item).
    - If the window pane (`wp`) is `NULL`, the function returns immediately without drawing.
    - A switch statement is used to determine the type of item (`WINDOW_TREE_NONE`, `WINDOW_TREE_SESSION`, `WINDOW_TREE_WINDOW`, or `WINDOW_TREE_PANE`) and calls the appropriate drawing function for each type.
    - For `WINDOW_TREE_SESSION`, it calls [`window_tree_draw_session`](#window_tree_draw_session) to draw the session.
    - For `WINDOW_TREE_WINDOW`, it calls [`window_tree_draw_window`](#window_tree_draw_window) to draw the window.
    - For `WINDOW_TREE_PANE`, it uses `screen_write_preview` to draw a preview of the pane.
- **Output**:
    - The function does not return a value; it performs drawing operations directly on the provided screen context.
- **Functions called**:
    - [`window_tree_pull_item`](#window_tree_pull_item)
    - [`window_tree_draw_session`](#window_tree_draw_session)
    - [`window_tree_draw_window`](#window_tree_draw_window)


---
### window_tree_search<!-- {{#callable:window_tree_search}} -->
The `window_tree_search` function searches for a substring within the names of sessions, windows, or panes in a window tree structure.
- **Inputs**:
    - `modedata`: Unused parameter, typically used for passing mode-specific data.
    - `itemdata`: Pointer to a `window_tree_itemdata` structure representing the item to be searched.
    - `ss`: The substring to search for within the item names.
- **Control Flow**:
    - The function begins by extracting session, window link, and window pane information from the `itemdata` using [`window_tree_pull_item`](#window_tree_pull_item).
    - It then uses a switch statement to determine the type of item (session, window, or pane) and performs the search accordingly.
    - For `WINDOW_TREE_NONE`, it returns 0 immediately as there is nothing to search.
    - For `WINDOW_TREE_SESSION`, it checks if the session exists and searches for the substring in the session's name.
    - For `WINDOW_TREE_WINDOW`, it checks if both session and window link exist and searches for the substring in the window's name.
    - For `WINDOW_TREE_PANE`, it checks if session, window link, and pane exist, retrieves the pane's command name, and searches for the substring in the command name.
    - If the command name is found, it frees the allocated memory for the command name and returns the result of the search.
    - If none of the conditions are met, it returns 0.
- **Output**:
    - Returns an integer indicating whether the substring `ss` is found in the relevant name (1 if found, 0 otherwise).
- **Functions called**:
    - [`window_tree_pull_item`](#window_tree_pull_item)


---
### window_tree_menu<!-- {{#callable:window_tree_menu}} -->
The `window_tree_menu` function processes a key event for a window tree mode entry in a client, ensuring the mode entry is valid before delegating the key handling to the [`window_tree_key`](#window_tree_key) function.
- **Inputs**:
    - `modedata`: A pointer to the window tree mode data, which contains information about the current window pane and other mode-specific data.
    - `c`: A pointer to the client structure, representing the client that triggered the key event.
    - `key`: The key code representing the key event that needs to be processed.
- **Control Flow**:
    - Retrieve the window pane from the mode data.
    - Get the first window mode entry from the window pane's modes list.
    - Check if the mode entry is null or if its data does not match the provided mode data; if so, return immediately.
    - Call the [`window_tree_key`](#window_tree_key) function with the mode entry, client, and key to handle the key event.
- **Output**:
    - The function does not return any value; it performs actions based on the key event and the current state of the window tree mode.
- **Functions called**:
    - [`window_tree_key`](#window_tree_key)


---
### window_tree_get_key<!-- {{#callable:window_tree_get_key}} -->
The `window_tree_get_key` function retrieves a key code based on a formatted string derived from the provided item data and line number.
- **Inputs**:
    - `modedata`: A pointer to `window_tree_modedata` structure containing mode-specific data, including the key format string.
    - `itemdata`: A pointer to `window_tree_itemdata` structure representing the item for which the key code is being retrieved.
    - `line`: An unsigned integer representing the line number used in formatting.
- **Control Flow**:
    - Create a format tree `ft` with no specific format context.
    - Call [`window_tree_pull_item`](#window_tree_pull_item) to extract session, winlink, and window pane information from `itemdata`.
    - Set default format values in `ft` based on the type of item (session, window, or pane).
    - Add the line number to the format tree `ft`.
    - Expand the format tree using the key format string from `modedata` to produce a formatted string `expanded`.
    - Look up the key code corresponding to the `expanded` string using `key_string_lookup_string`.
    - Free the memory allocated for `expanded` and the format tree `ft`.
    - Return the retrieved key code.
- **Output**:
    - The function returns a `key_code`, which is a code representing a key derived from the formatted string.
- **Functions called**:
    - [`window_tree_pull_item`](#window_tree_pull_item)


---
### window_tree_swap<!-- {{#callable:window_tree_swap}} -->
The `window_tree_swap` function swaps two window items within the same session if they are of the same type and meet certain conditions.
- **Inputs**:
    - `cur_itemdata`: A pointer to the current window tree item data structure.
    - `other_itemdata`: A pointer to the other window tree item data structure to be swapped with the current one.
- **Control Flow**:
    - Check if the types of the current and other items are the same; if not, return 0.
    - Check if the type of the items is `WINDOW_TREE_WINDOW`; if not, return 0.
    - Retrieve session, winlink, and pane information for both current and other items using [`window_tree_pull_item`](#window_tree_pull_item).
    - Check if both items belong to the same session; if not, return 0.
    - If the sorting field is not `WINDOW_TREE_BY_INDEX` and the windows are not comparable, return 0 to prevent swapping.
    - Remove the winlinks from their respective window's winlink list.
    - Swap the windows between the winlinks and reinsert them into the winlink lists.
    - Update the current window in the session if it was one of the swapped winlinks.
    - Synchronize the session group and redraw the session group.
    - Recalculate the sizes of the windows.
    - Return 1 to indicate a successful swap.
- **Output**:
    - Returns an integer value: 1 if the swap was successful, or 0 if it was not.
- **Functions called**:
    - [`window_tree_pull_item`](#window_tree_pull_item)
    - [`window_tree_cmp_window`](#window_tree_cmp_window)


---
### window_tree_init<!-- {{#callable:window_tree_init}} -->
The `window_tree_init` function initializes a window tree mode for a given window mode entry, setting up the necessary data structures and configurations based on provided arguments.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` representing the window mode entry to be initialized.
    - `fs`: A pointer to a `struct cmd_find_state` containing the command find state information.
    - `args`: A pointer to a `struct args` containing the arguments for configuring the window tree mode.
- **Control Flow**:
    - Allocate memory for `window_tree_modedata` and assign it to `wme->data` and `data`.
    - Set `data->wp` to the window pane from `wme` and initialize `data->references` to 1.
    - Determine the type of window tree (session, window, or pane) based on the presence of 's', 'w', or default to pane.
    - Copy the command find state from `fs` to `data->fs`.
    - Set `data->format`, `data->key_format`, and `data->command` based on the presence of 'F', 'K', and the first argument in `args`, respectively, or use default values.
    - Set `data->squash_groups` to the inverse of the presence of 'G' in `args`.
    - Set `data->prompt_flags` to `PROMPT_ACCEPT` if 'y' is present in `args`.
    - Initialize `data->data` using `mode_tree_start` with various callbacks and configurations.
    - Call `mode_tree_zoom` with `data->data` and `args`.
    - Build and draw the mode tree using `mode_tree_build` and `mode_tree_draw`.
    - Set `data->type` to `WINDOW_TREE_NONE`.
    - Return the screen pointer `s`.
- **Output**:
    - Returns a pointer to a `struct screen` which represents the initialized screen for the window tree mode.


---
### window_tree_destroy<!-- {{#callable:window_tree_destroy}} -->
The `window_tree_destroy` function deallocates memory and resources associated with a `window_tree_modedata` structure when its reference count reaches zero.
- **Inputs**:
    - `data`: A pointer to a `window_tree_modedata` structure that holds the data and resources to be freed.
- **Control Flow**:
    - Decrement the reference count of the `data` structure.
    - If the reference count is not zero, exit the function.
    - Iterate over each item in the `item_list` and free it using [`window_tree_free_item`](#window_tree_free_item).
    - Free the `item_list` array itself.
    - Free the `format`, `key_format`, and `command` strings.
    - Free the `data` structure itself.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the provided data structure.
- **Functions called**:
    - [`window_tree_free_item`](#window_tree_free_item)


---
### window_tree_free<!-- {{#callable:window_tree_free}} -->
The `window_tree_free` function releases resources associated with a window mode entry by marking its data as dead, freeing its mode tree data, and destroying the window tree mode data structure.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` which contains the data to be freed.
- **Control Flow**:
    - Retrieve the `window_tree_modedata` from the `wme` structure.
    - Check if the `data` is `NULL`; if so, return immediately as there is nothing to free.
    - Set the `dead` flag of the `data` to 1, indicating that the data is no longer active.
    - Call `mode_tree_free` to free the mode tree data associated with `data->data`.
    - Call [`window_tree_destroy`](#window_tree_destroy) to clean up and free the `window_tree_modedata` structure.
- **Output**:
    - This function does not return any value; it performs cleanup operations on the provided data structure.
- **Functions called**:
    - [`window_tree_destroy`](#window_tree_destroy)


---
### window_tree_resize<!-- {{#callable:window_tree_resize}} -->
The `window_tree_resize` function resizes the mode tree data associated with a window mode entry to the specified dimensions.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry`, which contains the mode data to be resized.
    - `sx`: An unsigned integer representing the new width for the mode tree.
    - `sy`: An unsigned integer representing the new height for the mode tree.
- **Control Flow**:
    - Retrieve the `window_tree_modedata` from the `wme` structure.
    - Call `mode_tree_resize` with the mode data, `sx`, and `sy` to resize the mode tree.
- **Output**:
    - This function does not return a value; it modifies the mode tree data in place.


---
### window_tree_update<!-- {{#callable:window_tree_update}} -->
The `window_tree_update` function updates the display of a window tree by rebuilding and redrawing the mode tree data and marking the window pane for redraw.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` which contains the mode data for the window.
- **Control Flow**:
    - Retrieve the `window_tree_modedata` from the `wme` structure.
    - Call `mode_tree_build` with the mode data to rebuild the tree structure.
    - Call `mode_tree_draw` with the mode data to redraw the tree structure.
    - Set the `PANE_REDRAW` flag on the window pane to indicate it needs to be redrawn.
- **Output**:
    - The function does not return a value; it updates the state of the window pane and mode tree data.


---
### window_tree_get_target<!-- {{#callable:window_tree_get_target}} -->
The `window_tree_get_target` function generates a target string based on the type of a window tree item and updates the command find state accordingly.
- **Inputs**:
    - `item`: A pointer to a `window_tree_itemdata` structure representing the item for which the target string is to be generated.
    - `fs`: A pointer to a `cmd_find_state` structure that will be updated based on the generated target.
- **Control Flow**:
    - The function begins by calling [`window_tree_pull_item`](#window_tree_pull_item) to extract session, winlink, and window pane information from the `item` structure.
    - It initializes the `target` variable to `NULL`.
    - A switch statement is used to determine the type of the `item` and generate the appropriate target string:
    - - If the type is `WINDOW_TREE_NONE`, no action is taken.
    - - If the type is `WINDOW_TREE_SESSION`, it checks if the session is not `NULL` and formats the target as `=<session_name>:`.
    - - If the type is `WINDOW_TREE_WINDOW`, it checks if both session and winlink are not `NULL` and formats the target as `=<session_name>:<winlink_index>.`.
    - - If the type is `WINDOW_TREE_PANE`, it checks if session, winlink, and window pane are not `NULL` and formats the target as `=<session_name>:<winlink_index>.%<pane_id>`.
    - If the `target` is still `NULL` after the switch, it calls `cmd_find_clear_state` to clear the command find state.
    - If a target was generated, it calls `cmd_find_from_winlink_pane` to update the command find state with the winlink and window pane information.
- **Output**:
    - Returns a dynamically allocated string representing the target, or `NULL` if no valid target could be generated.
- **Functions called**:
    - [`window_tree_pull_item`](#window_tree_pull_item)


---
### window_tree_command_each<!-- {{#callable:window_tree_command_each}} -->
The `window_tree_command_each` function executes a command for each tagged item in a window tree, using the item's target information.
- **Inputs**:
    - `modedata`: A pointer to `window_tree_modedata`, which contains the mode data for the window tree.
    - `itemdata`: A pointer to `window_tree_itemdata`, which contains the data for the specific item in the window tree.
    - `c`: A pointer to a `client` structure, representing the client for which the command is being executed.
    - `key`: A `key_code` value, which is unused in this function.
- **Control Flow**:
    - Retrieve the target name for the item using [`window_tree_get_target`](#window_tree_get_target) and store it in `name`.
    - If `name` is not NULL, execute the command using `mode_tree_run_command` with the client, find state, entered command, and target name.
    - Free the memory allocated for `name`.
- **Output**:
    - The function does not return a value; it performs actions based on the item's target and executes a command if applicable.
- **Functions called**:
    - [`window_tree_get_target`](#window_tree_get_target)


---
### window_tree_command_done<!-- {{#callable:window_tree_command_done}} -->
The `window_tree_command_done` function finalizes the window tree command by rebuilding and redrawing the mode tree if it is not marked as dead, and then destroys the mode data.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which is unused in this function.
    - `modedata`: A pointer to `window_tree_modedata` structure, which contains the mode data for the window tree.
- **Control Flow**:
    - Check if the `data` is not marked as dead.
    - If not dead, call `mode_tree_build` to rebuild the mode tree using `data->data`.
    - Call `mode_tree_draw` to redraw the mode tree using `data->data`.
    - Set the `PANE_REDRAW` flag on the window pane associated with `data`.
    - Call [`window_tree_destroy`](#window_tree_destroy) to clean up and free the resources associated with `data`.
    - Return `CMD_RETURN_NORMAL` to indicate normal command completion.
- **Output**:
    - The function returns an `enum cmd_retval` value, specifically `CMD_RETURN_NORMAL`, indicating the command completed normally.
- **Functions called**:
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
    - Assign the command string `s` to the `entered` field of the mode data.
    - Invoke `mode_tree_each_tagged` to execute `window_tree_command_each` on each tagged item in the mode tree, passing the client `c` and a key code of `KEYC_NONE`.
    - Set the `entered` field of the mode data back to NULL after processing the command.
    - Increment the `references` count in the mode data to indicate an ongoing operation.
    - Append a command queue item to the client `c` using `cmdq_append`, with a callback to `window_tree_command_done` to handle completion of the command processing.
    - Return 0 to indicate the function has completed its execution.
- **Output**:
    - The function returns an integer value, always 0, indicating the function's execution status.


---
### window_tree_command_free<!-- {{#callable:window_tree_command_free}} -->
The `window_tree_command_free` function is responsible for freeing the resources associated with a `window_tree_modedata` structure.
- **Inputs**:
    - `modedata`: A pointer to the `window_tree_modedata` structure that needs to be freed.
- **Control Flow**:
    - The function casts the `modedata` pointer to a `window_tree_modedata` pointer named `data`.
    - It calls the [`window_tree_destroy`](#window_tree_destroy) function, passing `data` as an argument to free the resources associated with the `window_tree_modedata` structure.
- **Output**:
    - This function does not return any value.
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
    - The function begins by casting `itemdata` to a `window_tree_itemdata` pointer named `item`.
    - It then calls [`window_tree_pull_item`](#window_tree_pull_item) to extract the session, winlink, and window pane associated with the item.
    - A switch statement is used to determine the type of the item (`WINDOW_TREE_NONE`, `WINDOW_TREE_SESSION`, `WINDOW_TREE_WINDOW`, or `WINDOW_TREE_PANE`).
    - If the item type is `WINDOW_TREE_SESSION` and the session is not NULL, it calls `server_destroy_session` and `session_destroy` to terminate the session.
    - If the item type is `WINDOW_TREE_WINDOW` and the winlink is not NULL, it calls `server_kill_window` to terminate the window.
    - If the item type is `WINDOW_TREE_PANE` and the window pane is not NULL, it calls `server_kill_pane` to terminate the pane.
    - The function does nothing if the item type is `WINDOW_TREE_NONE`.
- **Output**:
    - The function does not return any value; it performs actions based on the type of the item.
- **Functions called**:
    - [`window_tree_pull_item`](#window_tree_pull_item)


---
### window_tree_kill_current_callback<!-- {{#callable:window_tree_kill_current_callback}} -->
The `window_tree_kill_current_callback` function processes a confirmation input to kill the currently selected item in a window tree, and if confirmed, executes the kill operation and updates the server state.
- **Inputs**:
    - `c`: A pointer to the client structure, representing the client that initiated the callback.
    - `modedata`: A pointer to the window tree mode data structure, which contains the state and data for the window tree mode.
    - `s`: A string representing the user's input, typically a confirmation response.
    - `done`: An integer flag indicating whether the operation is complete, but it is unused in this function.
- **Control Flow**:
    - Check if the input string `s` is NULL, empty, or if the mode data is marked as dead; if any of these conditions are true, return 0 immediately.
    - Convert the first character of `s` to lowercase and check if it is 'y' and if `s` has no other characters; if not, return 0.
    - Call [`window_tree_kill_each`](#window_tree_kill_each) to perform the kill operation on the current item in the mode tree.
    - Call `server_renumber_all` to update the server's numbering of sessions, windows, or panes.
    - Increment the reference count of the mode data to indicate ongoing operations.
    - Append a command queue item to the client to handle the completion of the command using `cmdq_append` and `cmdq_get_callback`.
    - Return 0 to indicate the callback has completed its processing.
- **Output**:
    - The function returns an integer value, always 0, indicating the callback has completed its processing.
- **Functions called**:
    - [`window_tree_kill_each`](#window_tree_kill_each)


---
### window_tree_kill_tagged_callback<!-- {{#callable:window_tree_kill_tagged_callback}} -->
The `window_tree_kill_tagged_callback` function processes a confirmation input to kill all tagged items in a window tree and updates the server state accordingly.
- **Inputs**:
    - `c`: A pointer to the client structure, representing the client that initiated the callback.
    - `modedata`: A pointer to the window_tree_modedata structure, which contains the mode tree data and other related information.
    - `s`: A string representing the user's input, typically a confirmation response.
    - `done`: An integer flag indicating whether the operation is complete, but it is unused in this function.
- **Control Flow**:
    - Check if the input string `s` is NULL, empty, or if the `data->dead` flag is set; if any of these conditions are true, return 0 to exit early.
    - Convert the first character of `s` to lowercase and check if it is 'y' and if `s` has no other characters; if not, return 0 to exit early.
    - Call `mode_tree_each_tagged` to apply the `window_tree_kill_each` function to each tagged item in the mode tree data, passing the client `c` and a key code of `KEYC_NONE`.
    - Call `server_renumber_all` to update the numbering of all server sessions and windows.
    - Increment the `data->references` counter to indicate an ongoing operation.
    - Append a command queue item to the client's command queue using `cmdq_append`, with a callback to `window_tree_command_done` and the `data` structure.
    - Return 0 to indicate the function has completed its operations.
- **Output**:
    - The function returns an integer value, always 0, indicating the completion of its operations.


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
    - Determine if the click is on the left or right navigation arrows based on `data->left` and `data->right`, returning '<' or '>' respectively if true.
    - Adjust the x-coordinate relative to `data->left` and ensure it is within bounds of the window tree items.
    - Call [`window_tree_pull_item`](#window_tree_pull_item) to retrieve the session, winlink, and window pane associated with the clicked item.
    - If the item type is `WINDOW_TREE_SESSION`, expand the current mode tree and iterate through the session's windows to find the clicked window, setting it as current if found.
    - If the item type is `WINDOW_TREE_WINDOW`, expand the current mode tree and iterate through the window's panes to find the clicked pane, setting it as current if found.
    - Return `KEYC_NONE` if no valid action is determined.
- **Output**:
    - The function returns a `key_code` indicating the action to be taken, such as navigation ('<' or '>'), selection ('\r'), or no action (`KEYC_NONE`).
- **Functions called**:
    - [`window_tree_pull_item`](#window_tree_pull_item)


---
### window_tree_key<!-- {{#callable:window_tree_key}} -->
The `window_tree_key` function processes key and mouse events to navigate and manipulate a tree structure representing sessions, windows, and panes in a terminal multiplexer.
- **Inputs**:
    - `wme`: A pointer to a `window_mode_entry` structure representing the current mode entry.
    - `c`: A pointer to a `client` structure representing the client interacting with the window tree.
    - `s`: An unused pointer to a `session` structure.
    - `wl`: An unused pointer to a `winlink` structure.
    - `key`: A `key_code` representing the key event to be processed.
    - `m`: A pointer to a `mouse_event` structure representing the mouse event, if any.
- **Control Flow**:
    - Retrieve the current item from the mode tree data associated with the window mode entry.
    - Process the key event using `mode_tree_key`, updating the current item if it changes.
    - If the key event is a mouse event, process it with [`window_tree_mouse`](#window_tree_mouse) and repeat the key processing if necessary.
    - Handle specific key events such as '<', '>', 'H', 'm', 'M', 'x', 'X', ':', and '\r' to perform actions like navigating, marking, killing, or executing commands on sessions, windows, or panes.
    - If the operation is finished, reset the window pane mode; otherwise, redraw the mode tree and set the pane redraw flag.
- **Output**:
    - The function does not return a value but modifies the state of the window tree and potentially triggers actions like marking, killing, or executing commands on tree items.
- **Functions called**:
    - [`window_tree_mouse`](#window_tree_mouse)
    - [`window_tree_pull_item`](#window_tree_pull_item)
    - [`window_tree_get_target`](#window_tree_get_target)


# Function Declarations (Public API)

---
### window_tree_init<!-- {{#callable_declaration:window_tree_init}} -->
Initializes a screen for a window tree mode entry.
- **Description**: This function sets up a screen for a window tree mode entry, configuring it based on the provided arguments and command find state. It should be called when a new window tree mode is being initialized. The function determines the type of tree (session, window, or pane) and sets up formatting and command options. It is important to ensure that the `wme` and `fs` parameters are valid and properly initialized before calling this function. The function returns a pointer to the initialized screen, which can be used for further operations in the window tree mode.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` representing the mode entry to be initialized. Must not be null and should be properly initialized before calling this function.
    - `fs`: A pointer to a `struct cmd_find_state` that provides the current command find state. Must not be null and should be properly initialized.
    - `args`: A pointer to a `struct args` containing arguments that influence the initialization. Can be null, in which case default values are used for format, key format, and command.
- **Output**: Returns a pointer to a `struct screen` that represents the initialized screen for the window tree mode.
- **See also**: [`window_tree_init`](#window_tree_init)  (Implementation)


---
### window_tree_free<!-- {{#callable_declaration:window_tree_free}} -->
Releases resources associated with a window mode entry.
- **Description**: Use this function to clean up and release all resources associated with a given window mode entry when it is no longer needed. This function should be called to prevent memory leaks and ensure that all associated data structures are properly disposed of. It is important to ensure that the window mode entry is valid and has been initialized before calling this function.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that must not be null. The function will safely handle the case where the associated data is already null.
- **Output**: None
- **See also**: [`window_tree_free`](#window_tree_free)  (Implementation)


---
### window_tree_resize<!-- {{#callable_declaration:window_tree_resize}} -->
Resizes the window tree display.
- **Description**: This function adjusts the size of the window tree display based on the specified dimensions. It should be called whenever the display area for the window tree needs to be resized, such as when the terminal window is resized. The function requires a valid window mode entry and the new dimensions for the display.
- **Inputs**:
    - `wme`: A pointer to a struct window_mode_entry, which must not be null. It represents the window mode entry associated with the window tree being resized.
    - `sx`: An unsigned integer representing the new width of the window tree display. It should be a positive value.
    - `sy`: An unsigned integer representing the new height of the window tree display. It should be a positive value.
- **Output**: None
- **See also**: [`window_tree_resize`](#window_tree_resize)  (Implementation)


---
### window_tree_update<!-- {{#callable_declaration:window_tree_update}} -->
Updates the window tree display.
- **Description**: This function refreshes the display of a window tree, ensuring that the visual representation is up-to-date. It should be called whenever the underlying data of the window tree changes and a redraw is necessary. The function assumes that the window mode entry has been properly initialized and that the associated data is valid.
- **Inputs**:
    - `wme`: A pointer to a `struct window_mode_entry` that must not be null. It contains the data necessary for updating the window tree. The function expects this structure to be properly initialized and associated with a valid window tree mode data.
- **Output**: None
- **See also**: [`window_tree_update`](#window_tree_update)  (Implementation)


---
### window_tree_key<!-- {{#callable_declaration:window_tree_key}} -->
Handles key and mouse events for window tree navigation and actions.
- **Description**: This function processes key and mouse events to navigate and perform actions within a window tree interface. It should be called when a key or mouse event occurs while the window tree mode is active. The function updates the current selection, handles specific key commands for actions like expanding, marking, or killing items, and manages prompts for user confirmation. It requires a valid window mode entry and client, and it assumes the window tree mode is properly initialized.
- **Inputs**:
    - `wme`: A pointer to a struct window_mode_entry representing the current window mode entry. Must not be null.
    - `c`: A pointer to a struct client representing the client that triggered the event. Must not be null.
    - `s`: A pointer to a struct session, marked as unused in this function.
    - `wl`: A pointer to a struct winlink, marked as unused in this function.
    - `key`: A key_code representing the key event to handle. Valid key codes include navigation and action keys.
    - `m`: A pointer to a struct mouse_event representing the mouse event to handle. Can be null if the event is not mouse-related.
- **Output**: None
- **See also**: [`window_tree_key`](#window_tree_key)  (Implementation)


