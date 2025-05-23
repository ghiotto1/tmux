# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file is responsible for managing the layout of windows and panes within tmux. It defines a tree structure to represent the layout, where each node (or "cell") can be a container for other cells or a window pane. The layout can be organized in a left-right or top-bottom manner, allowing for flexible arrangement of panes.

The file includes functions for creating, resizing, and destroying layout cells, as well as adjusting the layout when the window size changes. It provides functionality to split panes, assign panes to cells, and spread out panes evenly within a window. The code handles the complexities of resizing panes while maintaining the overall layout structure, ensuring that minimum size constraints are respected. The file does not define a public API but is likely used internally within the tmux project to manage window layouts.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `tmux.h`


# Functions

---
### layout_create_cell<!-- {{#callable:layout_create_cell}} -->
The `layout_create_cell` function allocates and initializes a new layout cell structure, setting its type to `LAYOUT_WINDOWPANE` and linking it to a parent cell if provided.
- **Inputs**:
    - `lcparent`: A pointer to the parent layout cell to which the new cell will be linked; it can be NULL if the new cell is a root cell.
- **Control Flow**:
    - Allocate memory for a new layout cell using [`xmalloc`](xmalloc.c.driver.md#xmalloc).
    - Set the type of the new cell to `LAYOUT_WINDOWPANE`.
    - Assign the provided parent cell pointer to the new cell's parent field.
    - Initialize the cell's internal queue of child cells using `TAILQ_INIT`.
    - Set the cell's size (`sx` and `sy`) and offset (`xoff` and `yoff`) to `UINT_MAX`.
    - Set the window pane pointer (`wp`) of the cell to NULL.
    - Return the pointer to the newly created layout cell.
- **Output**:
    - A pointer to the newly created and initialized `layout_cell` structure.
- **Functions called**:
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)


---
### layout_free_cell<!-- {{#callable:layout_free_cell}} -->
The [`layout_free_cell`](#layout_free_cell) function recursively frees a layout cell and its children, ensuring proper cleanup of associated resources.
- **Inputs**:
    - `lc`: A pointer to the `layout_cell` structure that represents the cell to be freed.
- **Control Flow**:
    - The function checks the type of the layout cell `lc`.
    - If the type is `LAYOUT_LEFTRIGHT` or `LAYOUT_TOPBOTTOM`, it enters a loop to free all child cells recursively.
    - Within the loop, it retrieves the first child cell, removes it from the list, and calls [`layout_free_cell`](#layout_free_cell) recursively on it.
    - If the type is `LAYOUT_WINDOWPANE`, it checks if the window pane pointer `wp` is not NULL and sets its `layout_cell` to NULL.
    - Finally, it frees the memory allocated for the layout cell `lc`.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the layout cell and its children.
- **Functions called**:
    - [`layout_free_cell`](#layout_free_cell)


---
### layout_print_cell<!-- {{#callable:layout_print_cell}} -->
The [`layout_print_cell`](#layout_print_cell) function logs the details of a layout cell and recursively prints its child cells if applicable.
- **Inputs**:
    - `lc`: A pointer to a `layout_cell` structure representing the cell to be printed.
    - `hdr`: A constant character pointer representing a header string to be included in the log message.
    - `n`: An unsigned integer representing the indentation level for the log message.
- **Control Flow**:
    - Determine the type of the layout cell (`LEFTRIGHT`, `TOPBOTTOM`, `WINDOWPANE`, or `UNKNOWN`) based on the `type` field of the `layout_cell` structure.
    - Log the details of the layout cell, including its type, parent, window pane, and dimensions, using the `log_debug` function.
    - If the cell type is `LAYOUT_LEFTRIGHT` or `LAYOUT_TOPBOTTOM`, iterate over its child cells and recursively call [`layout_print_cell`](#layout_print_cell) for each child, increasing the indentation level by 1.
    - If the cell type is `LAYOUT_WINDOWPANE`, do not recurse further.
- **Output**:
    - The function does not return a value; it outputs log messages detailing the layout cell and its children.
- **Functions called**:
    - [`layout_print_cell`](#layout_print_cell)


---
### layout_search_by_border<!-- {{#callable:layout_search_by_border}} -->
The function [`layout_search_by_border`](#layout_search_by_border) recursively searches a layout tree to find a cell that borders a given point (x, y) within a layout structure.
- **Inputs**:
    - `lc`: A pointer to the root layout cell from which the search begins.
    - `x`: The x-coordinate of the point to search for within the layout.
    - `y`: The y-coordinate of the point to search for within the layout.
- **Control Flow**:
    - Initialize `last` to NULL and iterate over each child cell `lcchild` of the given layout cell `lc` using `TAILQ_FOREACH`.
    - For each child cell, check if the point (x, y) is within the bounds of the cell. If so, recursively call [`layout_search_by_border`](#layout_search_by_border) on this child cell.
    - If the point is not within the current child cell, check if `last` is NULL. If it is, set `last` to the current child cell and continue to the next iteration.
    - Depending on the type of the layout (`LAYOUT_LEFTRIGHT` or `LAYOUT_TOPBOTTOM`), check if the point (x, y) is between the current child cell and the last child cell. If so, return the `last` cell.
    - Update `last` to the current child cell and continue the iteration.
    - If no cell is found that borders the point, return NULL.
- **Output**:
    - Returns a pointer to the layout cell that borders the given point (x, y), or NULL if no such cell is found.
- **Functions called**:
    - [`layout_search_by_border`](#layout_search_by_border)


---
### layout_set_size<!-- {{#callable:layout_set_size}} -->
The `layout_set_size` function sets the size and offset properties of a layout cell.
- **Inputs**:
    - `lc`: A pointer to a `layout_cell` structure whose size and offset properties are to be set.
    - `sx`: An unsigned integer representing the new width of the layout cell.
    - `sy`: An unsigned integer representing the new height of the layout cell.
    - `xoff`: An unsigned integer representing the new horizontal offset of the layout cell.
    - `yoff`: An unsigned integer representing the new vertical offset of the layout cell.
- **Control Flow**:
    - Assigns the value of `sx` to the `sx` field of the `layout_cell` structure pointed to by `lc`.
    - Assigns the value of `sy` to the `sy` field of the `layout_cell` structure pointed to by `lc`.
    - Assigns the value of `xoff` to the `xoff` field of the `layout_cell` structure pointed to by `lc`.
    - Assigns the value of `yoff` to the `yoff` field of the `layout_cell` structure pointed to by `lc`.
- **Output**:
    - The function does not return a value; it modifies the `layout_cell` structure in place.


---
### layout_make_leaf<!-- {{#callable:layout_make_leaf}} -->
The `layout_make_leaf` function converts a layout cell into a leaf node that directly contains a window pane.
- **Inputs**:
    - `lc`: A pointer to a `layout_cell` structure that represents the layout cell to be converted into a leaf node.
    - `wp`: A pointer to a `window_pane` structure that represents the window pane to be associated with the layout cell.
- **Control Flow**:
    - Set the type of the layout cell `lc` to `LAYOUT_WINDOWPANE`, indicating it is a leaf node containing a window pane.
    - Initialize the `cells` queue of the layout cell `lc` to an empty state using `TAILQ_INIT`.
    - Assign the layout cell `lc` to the `layout_cell` member of the window pane `wp`.
    - Set the `wp` member of the layout cell `lc` to point to the window pane `wp`.
- **Output**:
    - The function does not return a value; it modifies the input `layout_cell` and `window_pane` structures to establish a relationship between them.


---
### layout_make_node<!-- {{#callable:layout_make_node}} -->
The `layout_make_node` function initializes a layout cell as a non-windowpane node and resets its associated window pane pointer.
- **Inputs**:
    - `lc`: A pointer to a `layout_cell` structure that represents the layout cell to be initialized.
    - `type`: An enumeration value of type `layout_type` that specifies the type of layout node to be created, which must not be `LAYOUT_WINDOWPANE`.
- **Control Flow**:
    - Check if the `type` is `LAYOUT_WINDOWPANE` and call `fatalx` to terminate the program if true, as this is an invalid operation.
    - Set the `type` of the layout cell `lc` to the specified `type`.
    - Initialize the `cells` queue of the layout cell `lc` using `TAILQ_INIT`.
    - If the layout cell `lc` has an associated window pane (`lc->wp` is not NULL), set the `layout_cell` pointer of the window pane to NULL.
    - Set the `wp` pointer of the layout cell `lc` to NULL, indicating it is not associated with any window pane.
- **Output**:
    - The function does not return a value; it modifies the `layout_cell` structure pointed to by `lc`.


---
### layout_fix_offsets1<!-- {{#callable:layout_fix_offsets1}} -->
The function [`layout_fix_offsets1`](#layout_fix_offsets1) recursively adjusts the x and y offsets of layout cells within a layout tree based on their type and size.
- **Inputs**:
    - `lc`: A pointer to a `layout_cell` structure representing the root cell of the layout tree or subtree whose offsets need to be fixed.
- **Control Flow**:
    - Check if the type of the current cell `lc` is `LAYOUT_LEFTRIGHT` or not.
    - If `LAYOUT_LEFTRIGHT`, initialize `xoff` with the x-offset of the current cell `lc`.
    - Iterate over each child cell `lcchild` in the `lc->cells` list.
    - Set the x-offset of `lcchild` to `xoff` and y-offset to the y-offset of `lc`.
    - If `lcchild` is not of type `LAYOUT_WINDOWPANE`, recursively call [`layout_fix_offsets1`](#layout_fix_offsets1) on `lcchild`.
    - Increment `xoff` by the width of `lcchild` plus one.
    - If not `LAYOUT_LEFTRIGHT`, initialize `yoff` with the y-offset of the current cell `lc`.
    - Iterate over each child cell `lcchild` in the `lc->cells` list.
    - Set the y-offset of `lcchild` to `yoff` and x-offset to the x-offset of `lc`.
    - If `lcchild` is not of type `LAYOUT_WINDOWPANE`, recursively call [`layout_fix_offsets1`](#layout_fix_offsets1) on `lcchild`.
    - Increment `yoff` by the height of `lcchild` plus one.
- **Output**:
    - The function does not return a value; it modifies the x and y offsets of the cells in the layout tree in place.
- **Functions called**:
    - [`layout_fix_offsets1`](#layout_fix_offsets1)


---
### layout_fix_offsets<!-- {{#callable:layout_fix_offsets}} -->
The `layout_fix_offsets` function initializes the offsets of the root layout cell of a window to zero and recursively updates the offsets of all child cells based on their sizes and positions.
- **Inputs**:
    - `w`: A pointer to a `struct window` which contains the layout tree with the root cell whose offsets need to be fixed.
- **Control Flow**:
    - Retrieve the root layout cell from the window's layout tree.
    - Set the x and y offsets of the root layout cell to zero.
    - Call the helper function [`layout_fix_offsets1`](#layout_fix_offsets1) to recursively adjust the offsets of all child cells.
- **Output**:
    - The function does not return a value; it modifies the layout tree in place by updating the offsets of all cells.
- **Functions called**:
    - [`layout_fix_offsets1`](#layout_fix_offsets1)


---
### layout_cell_is_top<!-- {{#callable:layout_cell_is_top}} -->
The function `layout_cell_is_top` checks if a given layout cell is the topmost cell in a top-bottom layout within a window's layout tree.
- **Inputs**:
    - `w`: A pointer to a `struct window`, representing the window containing the layout tree.
    - `lc`: A pointer to a `struct layout_cell`, representing the cell to be checked if it is the topmost cell.
- **Control Flow**:
    - Initialize a pointer `next` to hold the parent of the current cell `lc`.
    - Enter a loop that continues until `lc` is the root of the window's layout tree (`w->layout_root`).
    - In each iteration, set `next` to the parent of `lc`.
    - Check if `next` is of type `LAYOUT_TOPBOTTOM` and if `lc` is not the first cell in `next`'s list of cells.
    - If both conditions are true, return 0, indicating `lc` is not the topmost cell.
    - Update `lc` to `next` and continue the loop.
    - If the loop completes without returning 0, return 1, indicating `lc` is the topmost cell.
- **Output**:
    - Returns an integer: 1 if the cell is the topmost cell in a top-bottom layout, otherwise 0.


---
### layout_cell_is_bottom<!-- {{#callable:layout_cell_is_bottom}} -->
The function `layout_cell_is_bottom` checks if a given layout cell is the bottom-most cell in a top-bottom layout within a window's layout tree.
- **Inputs**:
    - `w`: A pointer to the `window` structure, representing the window containing the layout tree.
    - `lc`: A pointer to the `layout_cell` structure, representing the cell to be checked if it is the bottom-most cell.
- **Control Flow**:
    - Initialize a pointer `next` to hold the parent of the current cell `lc`.
    - Enter a loop that continues until `lc` is the root of the layout tree (`w->layout_root`).
    - In each iteration, set `next` to the parent of `lc`.
    - Check if `next` is of type `LAYOUT_TOPBOTTOM` and if `lc` is not the last cell in `next`'s list of cells.
    - If both conditions are true, return 0, indicating `lc` is not the bottom-most cell.
    - Update `lc` to `next` and continue the loop.
    - If the loop completes without returning 0, return 1, indicating `lc` is the bottom-most cell.
- **Output**:
    - Returns an integer: 1 if the cell is the bottom-most cell in a top-bottom layout, 0 otherwise.


---
### layout_add_horizontal_border<!-- {{#callable:layout_add_horizontal_border}} -->
The function `layout_add_horizontal_border` determines if an extra line is needed for the pane status line based on the position of the pane within the window layout.
- **Inputs**:
    - `w`: A pointer to the `window` structure representing the window containing the layout.
    - `lc`: A pointer to the `layout_cell` structure representing the specific cell within the window layout.
    - `status`: An integer indicating the status of the pane, which can be `PANE_STATUS_TOP` or `PANE_STATUS_BOTTOM`.
- **Control Flow**:
    - Check if the `status` is `PANE_STATUS_TOP`; if true, call [`layout_cell_is_top`](#layout_cell_is_top) to determine if the cell is at the top of the layout and return its result.
    - Check if the `status` is `PANE_STATUS_BOTTOM`; if true, call [`layout_cell_is_bottom`](#layout_cell_is_bottom) to determine if the cell is at the bottom of the layout and return its result.
    - If neither condition is met, return 0, indicating no extra line is needed.
- **Output**:
    - Returns 1 if an extra line is needed for the pane status line (i.e., if the cell is at the top or bottom of the layout), otherwise returns 0.
- **Functions called**:
    - [`layout_cell_is_top`](#layout_cell_is_top)
    - [`layout_cell_is_bottom`](#layout_cell_is_bottom)


---
### layout_fix_panes<!-- {{#callable:layout_fix_panes}} -->
The `layout_fix_panes` function adjusts the offsets and sizes of window panes within a window, taking into account pane borders and scrollbars, while skipping a specified pane.
- **Inputs**:
    - `w`: A pointer to the `window` structure containing the panes to be adjusted.
    - `skip`: A pointer to a `window_pane` structure that should be skipped during the adjustment process.
- **Control Flow**:
    - Retrieve pane border status, scrollbar visibility, and scrollbar position from the window's options.
    - Iterate over each pane in the window, skipping the specified pane and any pane without a layout cell.
    - Set the pane's offset to match its layout cell's offset and retrieve the cell's size.
    - If a horizontal border is needed, adjust the pane's vertical offset and size accordingly.
    - If scrollbars are enabled and visible for the pane, adjust the pane's width and offset based on scrollbar width, padding, and position.
    - Mark the pane for scrollbar redraw if scrollbars are shown.
    - Resize the pane to the calculated size.
- **Output**:
    - The function does not return a value; it modifies the offsets and sizes of the panes within the window directly.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`layout_add_horizontal_border`](#layout_add_horizontal_border)
    - [`window_pane_show_scrollbar`](window.c.driver.md#window_pane_show_scrollbar)
    - [`window_pane_resize`](tmux.h.driver.md#window_pane_resize)


---
### layout_count_cells<!-- {{#callable:layout_count_cells}} -->
The [`layout_count_cells`](#layout_count_cells) function recursively counts the number of cells in a layout tree, returning the total number of windowpane cells.
- **Inputs**:
    - `lc`: A pointer to the root layout cell from which the counting starts.
- **Control Flow**:
    - The function checks the type of the current layout cell `lc`.
    - If the type is `LAYOUT_WINDOWPANE`, it returns 1, indicating a single windowpane cell.
    - If the type is `LAYOUT_LEFTRIGHT` or `LAYOUT_TOPBOTTOM`, it initializes a counter `count` to 0 and iterates over each child cell in the `lc->cells` list, recursively calling [`layout_count_cells`](#layout_count_cells) on each child and adding the result to `count`.
    - If the type is not recognized, it calls `fatalx` with an error message indicating a bad layout type.
- **Output**:
    - The function returns an unsigned integer representing the total number of windowpane cells in the layout tree.
- **Functions called**:
    - [`layout_count_cells`](#layout_count_cells)


---
### layout_resize_check<!-- {{#callable:layout_resize_check}} -->
The [`layout_resize_check`](#layout_resize_check) function calculates the available space that can be removed from a layout cell in a window, based on its type and configuration.
- **Inputs**:
    - `w`: A pointer to the `window` structure, representing the window containing the layout.
    - `lc`: A pointer to the `layout_cell` structure, representing the cell within the layout to check for available space.
    - `type`: An `enum layout_type` value indicating the type of layout (e.g., `LAYOUT_LEFTRIGHT` or `LAYOUT_TOPBOTTOM`) to consider for resizing.
- **Control Flow**:
    - Retrieve the pane border status and scrollbar options from the window's options.
    - Check if the current cell is a window pane (`LAYOUT_WINDOWPANE`).
    - If the cell is a window pane, determine the available space based on the layout type and adjust for scrollbars or horizontal borders if necessary.
    - If the cell type matches the specified type, recursively calculate the total available space from all child cells.
    - If the cell type differs from the specified type, recursively find the minimum available space among all child cells.
    - Return the calculated available space.
- **Output**:
    - The function returns an unsigned integer (`u_int`) representing the amount of space available to be removed from the specified layout cell.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`layout_add_horizontal_border`](#layout_add_horizontal_border)
    - [`layout_resize_check`](#layout_resize_check)


---
### layout_resize_adjust<!-- {{#callable:layout_resize_adjust}} -->
The [`layout_resize_adjust`](#layout_resize_adjust) function adjusts the size of a layout cell and its children based on a specified change in size, ensuring that the layout remains consistent with the specified layout type.
- **Inputs**:
    - `w`: A pointer to the `window` structure, representing the window whose layout is being adjusted.
    - `lc`: A pointer to the `layout_cell` structure, representing the cell within the layout tree that is being resized.
    - `type`: An `enum layout_type` value indicating the type of layout adjustment (e.g., `LAYOUT_LEFTRIGHT` or `LAYOUT_TOPBOTTOM`).
    - `change`: An integer representing the amount of size change to apply to the layout cell.
- **Control Flow**:
    - The function first adjusts the size of the cell `lc` by adding the `change` value to either its `sx` or `sy` attribute, depending on the `type` (horizontal or vertical).
    - If the `type` is `LAYOUT_WINDOWPANE`, the function returns immediately as no further adjustment is needed for leaf cells.
    - If the cell's type is different from the `type`, the function recursively calls itself for each child cell to propagate the size change.
    - If the cell's type matches the `type`, the function enters a loop to distribute the size change evenly among child cells, adjusting each child cell by 1 unit at a time until the `change` is exhausted.
    - Within the loop, if `change` is positive, it decreases by 1 after adjusting a child cell; if `change` is negative, it checks if the child cell can be reduced further using [`layout_resize_check`](#layout_resize_check) before adjusting and increasing `change` by 1.
- **Output**:
    - The function does not return a value; it modifies the layout cell sizes in place.
- **Functions called**:
    - [`layout_resize_adjust`](#layout_resize_adjust)
    - [`layout_resize_check`](#layout_resize_check)


---
### layout_destroy_cell<!-- {{#callable:layout_destroy_cell}} -->
The `layout_destroy_cell` function removes a specified layout cell from a window's layout tree, redistributes its space to adjacent cells, and adjusts the layout tree structure accordingly.
- **Inputs**:
    - `w`: A pointer to the `window` structure that contains the layout tree.
    - `lc`: A pointer to the `layout_cell` structure that is to be destroyed.
    - `lcroot`: A pointer to a pointer to the root `layout_cell` of the layout tree, which may be updated if the root cell changes.
- **Control Flow**:
    - Check if the cell `lc` has no parent, indicating it is the last pane; if so, free the cell and set `lcroot` to NULL.
    - Determine the adjacent cell (`lcother`) to merge space with, either the next or previous cell in the parent's list, and adjust its size accordingly.
    - Remove the cell `lc` from its parent's list and free it.
    - Check if the parent cell now has only one child; if so, remove the parent from the tree and replace it with the remaining child, updating the root if necessary.
- **Output**:
    - The function does not return a value, but it modifies the layout tree by removing a cell and potentially updating the root cell pointer.
- **Functions called**:
    - [`layout_free_cell`](#layout_free_cell)
    - [`layout_resize_adjust`](#layout_resize_adjust)


---
### layout_init<!-- {{#callable:layout_init}} -->
The `layout_init` function initializes the layout of a window by creating a root layout cell, setting its size, making it a leaf with a specified window pane, and fixing the pane offsets and sizes.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window whose layout is being initialized.
    - `wp`: A pointer to a `struct window_pane` which represents the window pane to be associated with the newly created layout cell.
- **Control Flow**:
    - Create a new layout cell by calling [`layout_create_cell`](#layout_create_cell) with a `NULL` parent and assign it to the window's `layout_root`.
    - Set the size of the newly created layout cell using [`layout_set_size`](#layout_set_size), with the window's dimensions and offsets set to zero.
    - Make the layout cell a leaf node by associating it with the provided window pane using [`layout_make_leaf`](#layout_make_leaf).
    - Call [`layout_fix_panes`](#layout_fix_panes) to update the pane offsets and sizes based on the new layout.
- **Output**:
    - The function does not return a value; it modifies the layout of the given window by setting up its root layout cell and associating it with a window pane.
- **Functions called**:
    - [`layout_create_cell`](#layout_create_cell)
    - [`layout_set_size`](#layout_set_size)
    - [`layout_make_leaf`](#layout_make_leaf)
    - [`layout_fix_panes`](#layout_fix_panes)


---
### layout_free<!-- {{#callable:layout_free}} -->
The `layout_free` function deallocates the layout tree of a given window by freeing its root layout cell.
- **Inputs**:
    - `w`: A pointer to a `struct window` whose layout tree is to be freed.
- **Control Flow**:
    - The function calls [`layout_free_cell`](#layout_free_cell) with the root layout cell of the window `w`.
- **Output**:
    - The function does not return any value; it performs a cleanup operation by freeing memory associated with the window's layout.
- **Functions called**:
    - [`layout_free_cell`](#layout_free_cell)


---
### layout_resize<!-- {{#callable:layout_resize}} -->
The `layout_resize` function adjusts the layout of a window to fit new dimensions by resizing its layout cells proportionally, ensuring the layout does not shrink below a minimum size, and then fixes the offsets and pane sizes accordingly.
- **Inputs**:
    - `w`: A pointer to the `window` structure representing the window whose layout is to be resized.
    - `sx`: An unsigned integer representing the new width of the window.
    - `sy`: An unsigned integer representing the new height of the window.
- **Control Flow**:
    - Retrieve the root layout cell of the window from `w->layout_root`.
    - Calculate the horizontal change (`xchange`) as the difference between the new width `sx` and the current width of the layout cell `lc->sx`.
    - Determine the maximum allowable horizontal reduction (`xlimit`) using [`layout_resize_check`](#layout_resize_check) with `LAYOUT_LEFTRIGHT`.
    - Adjust `xchange` to not exceed `xlimit` if reducing size, or set it to zero if no reduction is possible.
    - If `xchange` is non-zero, call [`layout_resize_adjust`](#layout_resize_adjust) to apply the horizontal change to the layout.
    - Calculate the vertical change (`ychange`) as the difference between the new height `sy` and the current height of the layout cell `lc->sy`.
    - Determine the maximum allowable vertical reduction (`ylimit`) using [`layout_resize_check`](#layout_resize_check) with `LAYOUT_TOPBOTTOM`.
    - Adjust `ychange` to not exceed `ylimit` if reducing size, or set it to zero if no reduction is possible.
    - If `ychange` is non-zero, call [`layout_resize_adjust`](#layout_resize_adjust) to apply the vertical change to the layout.
    - Call [`layout_fix_offsets`](#layout_fix_offsets) to update the offsets of all cells in the layout.
    - Call [`layout_fix_panes`](#layout_fix_panes) to update the sizes and offsets of all panes in the window.
- **Output**:
    - The function does not return a value; it modifies the layout of the window in place.
- **Functions called**:
    - [`layout_resize_check`](#layout_resize_check)
    - [`layout_resize_adjust`](#layout_resize_adjust)
    - [`layout_fix_offsets`](#layout_fix_offsets)
    - [`layout_fix_panes`](#layout_fix_panes)


---
### layout_resize_pane_to<!-- {{#callable:layout_resize_pane_to}} -->
The `layout_resize_pane_to` function resizes a window pane to a specified size by adjusting its layout cell and its parent's layout cell hierarchy.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane to be resized.
    - `type`: An enumeration value of `layout_type` indicating the layout direction (either `LAYOUT_LEFTRIGHT` or `LAYOUT_TOPBOTTOM`).
    - `new_size`: An unsigned integer representing the new size to which the pane should be resized.
- **Control Flow**:
    - Retrieve the layout cell associated with the given window pane `wp`.
    - Traverse up the layout cell hierarchy to find the nearest parent cell of the specified `type`.
    - If no such parent cell is found, exit the function without making any changes.
    - Determine the current size of the layout cell in the specified direction (`sx` for `LAYOUT_LEFTRIGHT` and `sy` for `LAYOUT_TOPBOTTOM`).
    - Calculate the size change needed to achieve the `new_size` for the pane.
    - Call [`layout_resize_pane`](#layout_resize_pane) to apply the calculated size change to the pane.
- **Output**:
    - The function does not return a value; it modifies the layout of the specified pane in place.
- **Functions called**:
    - [`layout_resize_pane`](#layout_resize_pane)


---
### layout_resize_layout<!-- {{#callable:layout_resize_layout}} -->
The `layout_resize_layout` function adjusts the size of a layout cell within a window, either growing or shrinking it, and then updates the layout offsets and notifies of the change.
- **Inputs**:
    - `w`: A pointer to the `window` structure that contains the layout to be resized.
    - `lc`: A pointer to the `layout_cell` structure representing the cell to be resized.
    - `type`: An `enum layout_type` indicating the type of layout (e.g., left-right or top-bottom) to be resized.
    - `change`: An integer representing the amount of size change to apply to the layout cell.
    - `opposite`: An integer flag indicating whether to consider resizing in the opposite direction if needed.
- **Control Flow**:
    - Initialize `needed` with the value of `change` to track the remaining size change needed.
    - Enter a loop that continues until `needed` is zero, indicating no more resizing is possible.
    - If `change` is positive, attempt to grow the pane using [`layout_resize_pane_grow`](#layout_resize_pane_grow), reducing `needed` by the amount actually resized.
    - If `change` is negative, attempt to shrink the pane using [`layout_resize_pane_shrink`](#layout_resize_pane_shrink), increasing `needed` by the amount actually resized.
    - Break the loop if no size change was possible in the current iteration (i.e., `size` is zero).
    - Call [`layout_fix_offsets`](#layout_fix_offsets) to update the offsets of all cells in the window layout.
    - Call [`layout_fix_panes`](#layout_fix_panes) to update the pane sizes and offsets based on the new layout.
    - Notify that the window layout has changed using [`notify_window`](notify.c.driver.md#notify_window).
- **Output**:
    - The function does not return a value; it modifies the layout of the window in place and triggers a notification of the layout change.
- **Functions called**:
    - [`layout_resize_pane_grow`](#layout_resize_pane_grow)
    - [`layout_resize_pane_shrink`](#layout_resize_pane_shrink)
    - [`layout_fix_offsets`](#layout_fix_offsets)
    - [`layout_fix_panes`](#layout_fix_panes)
    - [`notify_window`](notify.c.driver.md#notify_window)


---
### layout_resize_pane<!-- {{#callable:layout_resize_pane}} -->
The `layout_resize_pane` function adjusts the size of a specific pane within a window layout by finding the appropriate parent cell and resizing the layout accordingly.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane to be resized.
    - `type`: An enumeration value of `layout_type` indicating the type of layout (e.g., left-right or top-bottom) to be resized.
    - `change`: An integer representing the amount of size change to apply to the pane.
    - `opposite`: An integer flag indicating whether to consider resizing in the opposite direction if needed.
- **Control Flow**:
    - Retrieve the layout cell associated with the given window pane `wp`.
    - Find the nearest parent cell of the same layout type as specified by `type`.
    - If no such parent cell is found, exit the function.
    - Check if the current cell is the last in its parent's list; if so, move to the previous cell.
    - Call [`layout_resize_layout`](#layout_resize_layout) to perform the actual resizing of the layout starting from the identified cell.
- **Output**:
    - The function does not return a value; it modifies the layout of the window pane in place.
- **Functions called**:
    - [`layout_resize_layout`](#layout_resize_layout)


---
### layout_resize_pane_grow<!-- {{#callable:layout_resize_pane_grow}} -->
The function `layout_resize_pane_grow` attempts to grow a specified layout cell by reducing the size of adjacent cells in a window layout.
- **Inputs**:
    - `w`: A pointer to the `window` structure representing the window containing the layout.
    - `lc`: A pointer to the `layout_cell` structure representing the cell to be grown.
    - `type`: An `enum layout_type` indicating the type of layout (e.g., left-right or top-bottom) to be adjusted.
    - `needed`: An integer specifying the amount of space needed to grow the cell.
    - `opposite`: An integer flag indicating whether to search for space in the opposite direction if needed.
- **Control Flow**:
    - Initialize `lcadd` to the current cell `lc` to be grown.
    - Search for a suitable adjacent cell `lcremove` towards the tail of the list that can be reduced in size.
    - If no suitable cell is found and `opposite` is true, search towards the head of the list for a suitable cell.
    - If no suitable cell is found in either direction, return 0 indicating no growth is possible.
    - Determine the actual size to adjust, which is the minimum of `size` and `needed`.
    - Adjust the size of `lcadd` by increasing it and `lcremove` by decreasing it by the determined size.
    - Return the size by which the cell was successfully grown.
- **Output**:
    - Returns the size by which the cell was successfully grown, or 0 if no growth was possible.
- **Functions called**:
    - [`layout_resize_check`](#layout_resize_check)
    - [`layout_resize_adjust`](#layout_resize_adjust)


---
### layout_resize_pane_shrink<!-- {{#callable:layout_resize_pane_shrink}} -->
The function `layout_resize_pane_shrink` attempts to shrink a pane within a window layout by redistributing space from adjacent cells.
- **Inputs**:
    - `w`: A pointer to the `window` structure representing the window containing the layout.
    - `lc`: A pointer to the `layout_cell` structure representing the cell to be shrunk.
    - `type`: An enumeration value of `layout_type` indicating the type of layout (e.g., horizontal or vertical).
    - `needed`: An integer representing the amount of space needed to shrink the pane.
- **Control Flow**:
    - Initialize `lcremove` to the current cell `lc` and attempt to find a cell to shrink by moving towards the head of the list.
    - Use [`layout_resize_check`](#layout_resize_check) to determine if the current cell `lcremove` can be shrunk; if not, move to the previous cell in the list.
    - If no suitable cell is found (`lcremove` becomes `NULL`), return 0 indicating no shrinkage is possible.
    - Set `lcadd` to the next cell in the list from the original cell `lc`; if `lcadd` is `NULL`, return 0 as no adjacent cell is available to expand.
    - Determine the amount of space to shrink, ensuring it does not exceed the `needed` amount, and adjust the sizes of `lcadd` and `lcremove` accordingly using [`layout_resize_adjust`](#layout_resize_adjust).
    - Return the actual size adjusted.
- **Output**:
    - Returns the actual size adjusted, which is the amount of space successfully shrunk from the pane.
- **Functions called**:
    - [`layout_resize_check`](#layout_resize_check)
    - [`layout_resize_adjust`](#layout_resize_adjust)


---
### layout_assign_pane<!-- {{#callable:layout_assign_pane}} -->
The `layout_assign_pane` function assigns a window pane to a layout cell and adjusts the layout based on whether resizing is allowed.
- **Inputs**:
    - `lc`: A pointer to the layout cell to which the window pane will be assigned.
    - `wp`: A pointer to the window pane that is being assigned to the layout cell.
    - `do_not_resize`: An integer flag indicating whether the layout should be adjusted to prevent resizing (non-zero value) or allow resizing (zero value).
- **Control Flow**:
    - Call [`layout_make_leaf`](#layout_make_leaf) to set the layout cell `lc` as a leaf node containing the window pane `wp`.
    - Check if `do_not_resize` is true (non-zero).
    - If `do_not_resize` is true, call [`layout_fix_panes`](#layout_fix_panes) with the window of `wp` and `wp` itself to adjust the layout without resizing.
    - If `do_not_resize` is false, call [`layout_fix_panes`](#layout_fix_panes) with the window of `wp` and `NULL` to allow resizing of the layout.
- **Output**:
    - The function does not return a value; it modifies the layout structure by assigning a pane to a cell and potentially adjusting the layout.
- **Functions called**:
    - [`layout_make_leaf`](#layout_make_leaf)
    - [`layout_fix_panes`](#layout_fix_panes)


---
### layout_new_pane_size<!-- {{#callable:layout_new_pane_size}} -->
The `layout_new_pane_size` function calculates the new size for a pane in a window layout based on the available space and the desired size distribution.
- **Inputs**:
    - `w`: A pointer to the `window` structure representing the window containing the layout.
    - `previous`: The previous total size of the layout before resizing.
    - `lc`: A pointer to the `layout_cell` structure representing the current cell in the layout.
    - `type`: An enum value of `layout_type` indicating the layout direction (either `LAYOUT_LEFTRIGHT` or `LAYOUT_TOPBOTTOM`).
    - `size`: The desired size for the new pane.
    - `count_left`: The number of panes left to be resized, including the current one.
    - `size_left`: The total size left to be distributed among the remaining panes.
- **Control Flow**:
    - Check if the current cell is the last one to be resized; if so, return the remaining size (`size_left`).
    - Calculate the available space in the parent cell using [`layout_resize_check`](#layout_resize_check).
    - Determine the minimum size required for the current cell based on the number of remaining cells and the available space.
    - Calculate the new size proportionate to the previous size of the layout.
    - Adjust the new size to ensure it does not exceed the maximum allowable size or fall below the minimum pane size.
    - Return the calculated new size for the pane.
- **Output**:
    - The function returns the calculated new size for the pane as an unsigned integer (`u_int`).
- **Functions called**:
    - [`layout_resize_check`](#layout_resize_check)


---
### layout_set_size_check<!-- {{#callable:layout_set_size_check}} -->
The [`layout_set_size_check`](#layout_set_size_check) function verifies if a layout cell and its children can be resized to a specified size while maintaining layout constraints.
- **Inputs**:
    - `w`: A pointer to the `window` structure representing the window containing the layout.
    - `lc`: A pointer to the `layout_cell` structure representing the cell to be checked for resizing.
    - `type`: An `enum layout_type` indicating the type of layout (e.g., LAYOUT_LEFTRIGHT or LAYOUT_TOPBOTTOM) to be checked.
    - `size`: An integer representing the new size to which the layout cell is to be resized.
- **Control Flow**:
    - If the cell type is `LAYOUT_WINDOWPANE`, return whether the size is greater than or equal to `PANE_MINIMUM`.
    - Initialize `available` with the given size and count the number of child cells.
    - If the cell type matches the specified `type`, check if the available size is sufficient for the children, considering the layout type and previous size.
    - Iterate over each child cell, calculate the new size using [`layout_new_pane_size`](#layout_new_pane_size), and recursively call [`layout_set_size_check`](#layout_set_size_check) for each child.
    - If the cell type does not match the specified `type`, recursively call [`layout_set_size_check`](#layout_set_size_check) for each non-`LAYOUT_WINDOWPANE` child cell.
    - Return 1 if all checks pass, otherwise return 0.
- **Output**:
    - Returns 1 if the cell and its children can be resized to the specified size, otherwise returns 0.
- **Functions called**:
    - [`layout_new_pane_size`](#layout_new_pane_size)
    - [`layout_set_size_check`](#layout_set_size_check)


---
### layout_resize_child_cells<!-- {{#callable:layout_resize_child_cells}} -->
The function [`layout_resize_child_cells`](#layout_resize_child_cells) resizes all child cells within a given layout cell to fit within the current cell's dimensions.
- **Inputs**:
    - `w`: A pointer to the `window` structure, representing the window containing the layout.
    - `lc`: A pointer to the `layout_cell` structure, representing the layout cell whose children are to be resized.
- **Control Flow**:
    - Check if the layout cell `lc` is of type `LAYOUT_WINDOWPANE`; if so, return immediately as it has no children to resize.
    - Initialize variables `count` and `previous` to zero to track the number of child cells and the total size used by them, respectively.
    - Iterate over each child cell in `lc->cells` to calculate the total size (`previous`) used by the children and count the number of children (`count`).
    - Determine the available size (`available`) based on the type of the layout cell (`LAYOUT_LEFTRIGHT` or `LAYOUT_TOPBOTTOM`).
    - Iterate over each child cell again to resize them according to the available space, adjusting their dimensions (`sx` and `sy`) and offsets (`xoff` and `yoff`) as necessary.
    - Recursively call [`layout_resize_child_cells`](#layout_resize_child_cells) for each child cell to ensure all nested children are resized appropriately.
- **Output**:
    - The function does not return a value; it modifies the dimensions and offsets of the child cells of the given layout cell in place.
- **Functions called**:
    - [`layout_new_pane_size`](#layout_new_pane_size)
    - [`layout_resize_child_cells`](#layout_resize_child_cells)


---
### layout_split_pane<!-- {{#callable:layout_split_pane}} -->
The `layout_split_pane` function splits a given window pane into two new panes, either horizontally or vertically, based on specified parameters.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane to be split.
    - `type`: An enum `layout_type` indicating the direction of the split, either `LAYOUT_LEFTRIGHT` or `LAYOUT_TOPBOTTOM`.
    - `size`: An integer specifying the target size for the new pane, or -1 for a default half/half split.
    - `flags`: An integer containing flags that modify the behavior of the split, such as `SPAWN_FULLSIZE` or `SPAWN_BEFORE`.
- **Control Flow**:
    - Determine if the split should be full size or based on the current pane, setting `lc` to the appropriate layout cell.
    - Retrieve the current pane's size and offset values.
    - Check if there is enough space to perform the split based on the layout type and scrollbar settings.
    - Calculate the sizes of the new panes (`size1` and `size2`) based on the provided `size` parameter and flags.
    - If the parent cell is of the same type as the split, insert a new cell after the current one; otherwise, create a new parent cell and adjust the layout tree accordingly.
    - Set the sizes and offsets for the new cells (`lc1` and `lc2`) based on the calculated sizes and the split type.
    - If the split is full size, resize child cells and fix offsets; otherwise, make the current cell a leaf with the window pane.
    - Return the newly created layout cell `lcnew`.
- **Output**:
    - Returns a pointer to the newly created `layout_cell` structure representing the new pane, or `NULL` if the split cannot be performed.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`layout_add_horizontal_border`](#layout_add_horizontal_border)
    - [`layout_set_size_check`](#layout_set_size_check)
    - [`layout_create_cell`](#layout_create_cell)
    - [`layout_resize_child_cells`](#layout_resize_child_cells)
    - [`layout_set_size`](#layout_set_size)
    - [`layout_make_node`](#layout_make_node)
    - [`layout_fix_offsets`](#layout_fix_offsets)
    - [`layout_make_leaf`](#layout_make_leaf)


---
### layout_close_pane<!-- {{#callable:layout_close_pane}} -->
The `layout_close_pane` function removes a specified window pane from its layout and adjusts the remaining layout accordingly.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane to be closed.
- **Control Flow**:
    - Retrieve the window associated with the given window pane `wp`.
    - Call [`layout_destroy_cell`](#layout_destroy_cell) to remove the layout cell associated with the pane from the window's layout tree.
    - Check if the window's layout root is not NULL, indicating there are remaining panes.
    - If there are remaining panes, call [`layout_fix_offsets`](#layout_fix_offsets) to adjust the offsets of the remaining layout cells.
    - Call [`layout_fix_panes`](#layout_fix_panes) to adjust the sizes of the remaining panes.
    - Notify that the window layout has changed by calling [`notify_window`](notify.c.driver.md#notify_window) with the event 'window-layout-changed'.
- **Output**:
    - The function does not return a value; it modifies the layout of the window by removing the specified pane and adjusting the remaining layout.
- **Functions called**:
    - [`layout_destroy_cell`](#layout_destroy_cell)
    - [`layout_fix_offsets`](#layout_fix_offsets)
    - [`layout_fix_panes`](#layout_fix_panes)
    - [`notify_window`](notify.c.driver.md#notify_window)


---
### layout_spread_cell<!-- {{#callable:layout_spread_cell}} -->
The `layout_spread_cell` function evenly distributes space among child cells of a parent layout cell in a window, adjusting their sizes accordingly.
- **Inputs**:
    - `w`: A pointer to the `window` structure, representing the window containing the layout.
    - `parent`: A pointer to the `layout_cell` structure, representing the parent layout cell whose child cells are to be spread out.
- **Control Flow**:
    - Initialize `number` to count the number of child cells in `parent` using a loop over `parent->cells`.
    - If `number` is less than or equal to 1, return 0 as no spreading is needed.
    - Retrieve `status` and `scrollbars` options from the window's options.
    - Determine the available `size` for spreading based on the `parent` type (either `LAYOUT_LEFTRIGHT` or `LAYOUT_TOPBOTTOM`) and adjust for scrollbars or borders if necessary.
    - If `size` is less than `number - 1`, return 0 as there is insufficient space to spread.
    - Calculate `each` as the evenly distributed size for each child cell and `remainder` as the leftover space after distribution.
    - Initialize `changed` to 0 to track if any changes are made to child cell sizes.
    - Iterate over each child cell `lc` in `parent->cells`, calculate `change` needed for each cell based on `each` and `remainder`, and adjust the cell size using [`layout_resize_adjust`](#layout_resize_adjust).
    - If any `change` is made, set `changed` to 1.
    - Return `changed` to indicate if any adjustments were made.
- **Output**:
    - Returns an integer indicating whether any child cell sizes were changed (1 if changed, 0 if not).
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`layout_add_horizontal_border`](#layout_add_horizontal_border)
    - [`layout_resize_adjust`](#layout_resize_adjust)


---
### layout_spread_out<!-- {{#callable:layout_spread_out}} -->
The `layout_spread_out` function attempts to evenly distribute the space among sibling cells of a window pane's parent layout cell, adjusting offsets and pane sizes accordingly.
- **Inputs**:
    - `wp`: A pointer to a `window_pane` structure, representing the window pane whose layout is to be adjusted.
- **Control Flow**:
    - Retrieve the parent layout cell of the given window pane's layout cell.
    - If the parent is NULL, return immediately as there is no layout to adjust.
    - Enter a loop to attempt spreading the layout starting from the parent cell upwards through its ancestors.
    - Call [`layout_spread_cell`](#layout_spread_cell) to try and spread the layout for the current parent cell.
    - If [`layout_spread_cell`](#layout_spread_cell) returns true (indicating a successful spread), call [`layout_fix_offsets`](#layout_fix_offsets) and [`layout_fix_panes`](#layout_fix_panes) to adjust the layout offsets and pane sizes, then break out of the loop.
    - Continue the loop with the next parent cell if the current spread was unsuccessful.
- **Output**:
    - The function does not return a value; it modifies the layout of the window panes in the given window.
- **Functions called**:
    - [`layout_spread_cell`](#layout_spread_cell)
    - [`layout_fix_offsets`](#layout_fix_offsets)
    - [`layout_fix_panes`](#layout_fix_panes)


