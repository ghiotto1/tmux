# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file is responsible for managing the layout of windows and panes within tmux. It defines a tree structure to represent the layout, where each node, called a "layout cell," can either be a container for other cells or a container for a window pane. The layout cells can be organized in a left-right or top-bottom manner, allowing for flexible arrangement of panes within a window.

The file provides a comprehensive set of functions to manipulate these layout cells, including creating, freeing, resizing, and splitting them. It also includes functions to adjust the layout when the window size changes, ensuring that the panes are resized proportionally and that their offsets are updated correctly. The code handles various layout operations, such as splitting a pane into two, resizing a pane to a specific size, and spreading out panes evenly within a window. The file is integral to the dynamic and flexible layout management in tmux, allowing users to customize their terminal environment efficiently.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `tmux.h`


# Functions

---
### layout_create_cell<!-- {{#callable:layout_create_cell}} -->
The `layout_create_cell` function allocates and initializes a new layout cell structure, setting its type to a window pane and linking it to a parent cell if provided.
- **Inputs**:
    - `lcparent`: A pointer to the parent layout cell to which the new cell will be linked; it can be NULL if the new cell has no parent.
- **Control Flow**:
    - Allocate memory for a new layout cell using `xmalloc`.
    - Set the type of the new cell to `LAYOUT_WINDOWPANE`.
    - Assign the provided parent cell pointer to the new cell's parent field.
    - Initialize the cell's internal queue of child cells using `TAILQ_INIT`.
    - Set the cell's size (`sx` and `sy`) and offset (`xoff` and `yoff`) to `UINT_MAX`.
    - Set the window pane pointer (`wp`) of the cell to NULL.
    - Return the pointer to the newly created layout cell.
- **Output**:
    - A pointer to the newly created and initialized `layout_cell` structure.


---
### layout_free_cell<!-- {{#callable:layout_free_cell}} -->
The [`layout_free_cell`](#layout_free_cell) function recursively frees a layout cell and its children, and cleans up associated resources.
- **Inputs**:
    - `lc`: A pointer to the `layout_cell` structure that represents the cell to be freed.
- **Control Flow**:
    - The function checks the type of the layout cell `lc`.
    - If the type is `LAYOUT_LEFTRIGHT` or `LAYOUT_TOPBOTTOM`, it enters a loop to free all child cells recursively.
    - Within the loop, it retrieves the first child cell, removes it from the list, and calls [`layout_free_cell`](#layout_free_cell) on it.
    - If the type is `LAYOUT_WINDOWPANE`, it checks if the cell has an associated window pane (`wp`) and sets its `layout_cell` pointer to `NULL`.
    - Finally, it frees the memory allocated for the layout cell `lc`.
- **Output**:
    - The function does not return any value; it performs cleanup and deallocation of the specified layout cell and its children.
- **Functions called**:
    - [`layout_free_cell`](#layout_free_cell)


---
### layout_print_cell<!-- {{#callable:layout_print_cell}} -->
The [`layout_print_cell`](#layout_print_cell) function recursively logs the details of a layout cell and its children, including their type and dimensions.
- **Inputs**:
    - `lc`: A pointer to a `layout_cell` structure representing the current cell to be logged.
    - `hdr`: A constant character pointer used as a header in the log message.
    - `n`: An unsigned integer representing the indentation level for the log message.
- **Control Flow**:
    - Determine the type of the current layout cell (`lc`) and assign a string representation to `type` based on the cell's type (e.g., 'LEFTRIGHT', 'TOPBOTTOM', 'WINDOWPANE', or 'UNKNOWN').
    - Log the details of the current cell using `log_debug`, including its type, parent, window pane, and dimensions.
    - If the cell type is `LAYOUT_LEFTRIGHT` or `LAYOUT_TOPBOTTOM`, iterate over its child cells using `TAILQ_FOREACH` and recursively call [`layout_print_cell`](#layout_print_cell) for each child, increasing the indentation level `n` by 1.
    - If the cell type is `LAYOUT_WINDOWPANE`, do nothing further as it has no children to process.
- **Output**:
    - The function does not return a value; it outputs log messages detailing the layout cell structure.
- **Functions called**:
    - [`layout_print_cell`](#layout_print_cell)


---
### layout_search_by_border<!-- {{#callable:layout_search_by_border}} -->
The [`layout_search_by_border`](#layout_search_by_border) function searches for a layout cell that borders a given point (x, y) within a layout tree.
- **Inputs**:
    - `lc`: A pointer to the root layout cell from which the search begins.
    - `x`: The x-coordinate of the point to search for within the layout.
    - `y`: The y-coordinate of the point to search for within the layout.
- **Control Flow**:
    - Initialize `last` to NULL and iterate over each child cell `lcchild` of the given layout cell `lc`.
    - Check if the point (x, y) is within the bounds of the current child cell `lcchild`. If so, recursively call [`layout_search_by_border`](#layout_search_by_border) on `lcchild`.
    - If `last` is NULL, set `last` to the current `lcchild` and continue to the next iteration.
    - Depending on the type of the layout cell `lc`, check if the point (x, y) is between the current `lcchild` and the `last` cell. If so, return `last`.
    - Update `last` to the current `lcchild` and continue the loop.
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
The `layout_make_leaf` function sets a layout cell to be a leaf node containing a specific window pane.
- **Inputs**:
    - `lc`: A pointer to a `layout_cell` structure that represents the layout cell to be set as a leaf.
    - `wp`: A pointer to a `window_pane` structure that represents the window pane to be associated with the layout cell.
- **Control Flow**:
    - Set the type of the layout cell `lc` to `LAYOUT_WINDOWPANE`, indicating it is a leaf node containing a window pane.
    - Initialize the `cells` queue of the layout cell `lc` to be empty using `TAILQ_INIT`.
    - Assign the layout cell `lc` to the `layout_cell` member of the window pane `wp`.
    - Set the `wp` member of the layout cell `lc` to point to the window pane `wp`.
- **Output**:
    - The function does not return a value; it modifies the input `layout_cell` and `window_pane` structures to establish their relationship.


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
    - Set the `wp` pointer of the layout cell `lc` to NULL, effectively disassociating any window pane from this layout cell.
- **Output**:
    - The function does not return a value; it modifies the layout cell `lc` in place.


---
### layout_fix_offsets1<!-- {{#callable:layout_fix_offsets1}} -->
The [`layout_fix_offsets1`](#layout_fix_offsets1) function recursively adjusts the x and y offsets of layout cells within a layout tree based on their type and position.
- **Inputs**:
    - `lc`: A pointer to a `layout_cell` structure representing the current cell whose offsets need to be fixed.
- **Control Flow**:
    - Check if the current cell `lc` is of type `LAYOUT_LEFTRIGHT` or not.
    - If `lc` is `LAYOUT_LEFTRIGHT`, initialize `xoff` with `lc->xoff` and iterate over each child cell `lcchild` in `lc->cells`.
    - For each child cell, set its `xoff` to the current `xoff` and `yoff` to `lc->yoff`.
    - If the child cell is not of type `LAYOUT_WINDOWPANE`, recursively call [`layout_fix_offsets1`](#layout_fix_offsets1) on the child cell.
    - Increment `xoff` by the width of the child cell (`lcchild->sx`) plus one.
    - If `lc` is not `LAYOUT_LEFTRIGHT`, initialize `yoff` with `lc->yoff` and iterate over each child cell `lcchild` in `lc->cells`.
    - For each child cell, set its `xoff` to `lc->xoff` and `yoff` to the current `yoff`.
    - If the child cell is not of type `LAYOUT_WINDOWPANE`, recursively call [`layout_fix_offsets1`](#layout_fix_offsets1) on the child cell.
    - Increment `yoff` by the height of the child cell (`lcchild->sy`) plus one.
- **Output**:
    - The function does not return a value; it modifies the `xoff` and `yoff` properties of the `layout_cell` structures in place.
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
    - Call the helper function [`layout_fix_offsets1`](#layout_fix_offsets1) to recursively adjust the offsets of all child cells in the layout tree.
- **Output**:
    - The function does not return a value; it modifies the layout tree in place by updating the offsets of all cells.
- **Functions called**:
    - [`layout_fix_offsets1`](#layout_fix_offsets1)


---
### layout_cell_is_top<!-- {{#callable:layout_cell_is_top}} -->
The function `layout_cell_is_top` checks if a given layout cell is the topmost cell in a top-bottom layout within a window's layout tree.
- **Inputs**:
    - `w`: A pointer to the `window` structure, representing the window containing the layout tree.
    - `lc`: A pointer to the `layout_cell` structure, representing the cell to be checked if it is the topmost cell.
- **Control Flow**:
    - Initialize a pointer `next` to hold the parent of the current cell `lc`.
    - Enter a loop that continues until `lc` is the root of the window's layout tree (`w->layout_root`).
    - In each iteration, set `next` to the parent of `lc`.
    - Check if `next` is of type `LAYOUT_TOPBOTTOM` and if `lc` is not the first cell in `next`'s list of cells.
    - If both conditions are true, return 0, indicating `lc` is not the topmost cell.
    - Update `lc` to `next` and continue the loop.
    - If the loop completes without returning 0, return 1, indicating `lc` is the topmost cell.
- **Output**:
    - Returns an integer: 1 if the cell is the topmost cell in a top-bottom layout, 0 otherwise.


---
### layout_cell_is_bottom<!-- {{#callable:layout_cell_is_bottom}} -->
The function `layout_cell_is_bottom` checks if a given layout cell is the bottom-most cell in a top-bottom layout within a window's layout tree.
- **Inputs**:
    - `w`: A pointer to a `struct window`, representing the window containing the layout tree.
    - `lc`: A pointer to a `struct layout_cell`, representing the cell to be checked if it is the bottom-most cell.
- **Control Flow**:
    - Initialize a pointer `next` to hold the parent of the current cell `lc`.
    - Enter a loop that continues until `lc` is the root of the layout tree (`w->layout_root`).
    - In each iteration, set `next` to the parent of `lc`.
    - Check if `next` is of type `LAYOUT_TOPBOTTOM` and if `lc` is not the last cell in `next`'s list of cells.
    - If the above condition is true, return 0, indicating `lc` is not the bottom-most cell.
    - Update `lc` to `next` and continue the loop.
    - If the loop completes without returning 0, return 1, indicating `lc` is the bottom-most cell.
- **Output**:
    - Returns an integer: 1 if the cell is the bottom-most cell in a top-bottom layout, 0 otherwise.


---
### layout_add_horizontal_border<!-- {{#callable:layout_add_horizontal_border}} -->
The `layout_add_horizontal_border` function determines if an extra line is needed for the pane status line based on the position of the pane within the window layout.
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
The `layout_fix_panes` function adjusts the offsets and sizes of window panes within a window layout, taking into account pane borders and scrollbars.
- **Inputs**:
    - `w`: A pointer to the `window` structure representing the window whose panes are to be adjusted.
    - `skip`: A pointer to a `window_pane` structure that should be skipped during the adjustment process.
- **Control Flow**:
    - Retrieve pane border status, scrollbar visibility, and scrollbar position from the window's options.
    - Iterate over each pane in the window, skipping the specified pane and those without a layout cell.
    - Set the pane's offset to match its layout cell's offset and retrieve its size.
    - If a horizontal border is needed, adjust the pane's vertical offset and size accordingly.
    - If scrollbars are enabled and visible for the pane, adjust the pane's size and offset based on scrollbar width and padding, and set a flag to redraw the scrollbar.
    - Resize the pane to the calculated size.
- **Output**:
    - The function does not return a value; it modifies the offsets and sizes of the panes in the window directly.
- **Functions called**:
    - [`layout_add_horizontal_border`](#layout_add_horizontal_border)
    - [`window_pane_resize`](tmux.h.driver.md#window_pane_resize)


---
### layout_count_cells<!-- {{#callable:layout_count_cells}} -->
The [`layout_count_cells`](#layout_count_cells) function recursively counts the number of cells in a layout tree, returning the total count of windowpane cells.
- **Inputs**:
    - `lc`: A pointer to the root layout cell from which the counting of cells begins.
- **Control Flow**:
    - The function checks the type of the current layout cell `lc`.
    - If the type is `LAYOUT_WINDOWPANE`, it returns 1, indicating a single windowpane cell.
    - If the type is `LAYOUT_LEFTRIGHT` or `LAYOUT_TOPBOTTOM`, it initializes a counter `count` to 0 and iterates over each child cell in the `lc->cells` list.
    - For each child cell, it recursively calls [`layout_count_cells`](#layout_count_cells) and adds the result to `count`.
    - After iterating through all child cells, it returns the total `count`.
    - If the cell type is not recognized, it calls `fatalx` with an error message indicating a bad layout type.
- **Output**:
    - The function returns an unsigned integer representing the total number of windowpane cells in the layout tree.
- **Functions called**:
    - [`layout_count_cells`](#layout_count_cells)


---
### layout_resize_check<!-- {{#callable:layout_resize_check}} -->
The [`layout_resize_check`](#layout_resize_check) function calculates the available space that can be removed from a layout cell in a window, based on its type and configuration.
- **Inputs**:
    - `w`: A pointer to the `window` structure, representing the window containing the layout.
    - `lc`: A pointer to the `layout_cell` structure, representing the cell within the layout to be checked for available space.
    - `type`: An `enum layout_type` value indicating the type of layout (e.g., `LAYOUT_LEFTRIGHT` or `LAYOUT_TOPBOTTOM`) to check against.
- **Control Flow**:
    - Retrieve the pane border status and scrollbar options from the window's options.
    - Check if the current cell type is `LAYOUT_WINDOWPANE`.
    - If `LAYOUT_WINDOWPANE`, calculate available space based on the layout type (`LAYOUT_LEFTRIGHT` or `LAYOUT_TOPBOTTOM`) and adjust for scrollbars or borders.
    - If the cell type matches the specified type, recursively calculate the total available space from all child cells.
    - If the cell type differs from the specified type, recursively find the minimum available space among all child cells.
    - Return the calculated available space.
- **Output**:
    - The function returns an unsigned integer representing the amount of space available to be removed from the specified layout cell.
- **Functions called**:
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
    - The function first adjusts the size of the current cell (`lc`) based on the `type` and `change` parameters.
    - If the cell is a leaf cell (`LAYOUT_WINDOWPANE`), the function returns immediately as no further adjustment is needed.
    - If the cell's type is different from the specified `type`, the function recursively adjusts each child cell in the opposite direction.
    - If the cell's type matches the specified `type`, the function enters a loop to adjust each child cell equally until the `change` is fully applied or no further adjustment is possible.
    - Within the loop, the function iterates over each child cell, adjusting their size by 1 unit at a time, either increasing or decreasing, depending on the remaining `change`.
    - The function checks if further reduction is possible using [`layout_resize_check`](#layout_resize_check) before decreasing a child's size.
- **Output**:
    - The function does not return a value; it modifies the layout cell sizes in place.
- **Functions called**:
    - [`layout_resize_adjust`](#layout_resize_adjust)
    - [`layout_resize_check`](#layout_resize_check)


---
### layout_destroy_cell<!-- {{#callable:layout_destroy_cell}} -->
The `layout_destroy_cell` function removes a specified layout cell from a window's layout tree, redistributing its space to adjacent cells and adjusting the layout tree structure accordingly.
- **Inputs**:
    - `w`: A pointer to the `window` structure that contains the layout tree.
    - `lc`: A pointer to the `layout_cell` structure that is to be destroyed.
    - `lcroot`: A pointer to a pointer to the root `layout_cell` of the layout tree, which may be updated if the root cell changes.
- **Control Flow**:
    - Check if the cell `lc` has no parent, indicating it is the last pane; if so, free the cell and set the root to NULL.
    - Determine the adjacent cell (`lcother`) to merge space with, either the next or previous cell in the parent's list, and adjust its size accordingly.
    - Remove the cell `lc` from its parent's list and free it.
    - If the parent cell now contains only one child, remove the parent from the tree and replace it with the remaining child, updating the root if necessary.
- **Output**:
    - The function does not return a value, but it modifies the layout tree by removing a cell and potentially updating the root cell pointer.
- **Functions called**:
    - [`layout_free_cell`](#layout_free_cell)
    - [`layout_resize_adjust`](#layout_resize_adjust)


---
### layout_init<!-- {{#callable:layout_init}} -->
The `layout_init` function initializes the layout of a window by creating a root layout cell, setting its size, making it a leaf with a window pane, and fixing pane offsets and sizes.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window whose layout is being initialized.
    - `wp`: A pointer to a `struct window_pane` which represents the window pane to be associated with the newly created layout cell.
- **Control Flow**:
    - Create a new layout cell by calling [`layout_create_cell`](#layout_create_cell) with a `NULL` parent and assign it to the window's `layout_root`.
    - Set the size of the newly created layout cell using [`layout_set_size`](#layout_set_size), passing the window's dimensions and offsets.
    - Make the layout cell a leaf by associating it with the provided window pane using [`layout_make_leaf`](#layout_make_leaf).
    - Call [`layout_fix_panes`](#layout_fix_panes) to update the pane offsets and sizes based on the new layout.
- **Output**:
    - The function does not return a value; it modifies the layout of the given window in place.
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
    - The function calls [`layout_free_cell`](#layout_free_cell) with the root layout cell of the window's layout tree (`w->layout_root`).
- **Output**:
    - The function does not return any value; it performs a cleanup operation by freeing memory associated with the window's layout tree.
- **Functions called**:
    - [`layout_free_cell`](#layout_free_cell)


---
### layout_resize<!-- {{#callable:layout_resize}} -->
The `layout_resize` function adjusts the layout of a window to fit new dimensions by resizing its layout cells proportionally, ensuring that the layout does not shrink below a minimum size and redistributing any excess space.
- **Inputs**:
    - `w`: A pointer to the `window` structure whose layout is to be resized.
    - `sx`: The new width (in pixels or units) for the window layout.
    - `sy`: The new height (in pixels or units) for the window layout.
- **Control Flow**:
    - Retrieve the root layout cell of the window.
    - Calculate the horizontal change (`xchange`) by subtracting the current width of the layout cell from the new width (`sx`).
    - Determine the maximum allowable horizontal reduction (`xlimit`) using [`layout_resize_check`](#layout_resize_check).
    - Adjust `xchange` to ensure it does not exceed the allowable reduction (`xlimit`).
    - If `xlimit` is zero, set `xchange` to zero if the new width is less than or equal to the current width, otherwise set it to the difference between the new and current width.
    - If `xchange` is non-zero, call [`layout_resize_adjust`](#layout_resize_adjust) to apply the horizontal change.
    - Calculate the vertical change (`ychange`) similarly to the horizontal change, using `sy` and [`layout_resize_check`](#layout_resize_check) for vertical adjustments.
    - Adjust `ychange` to ensure it does not exceed the allowable reduction (`ylimit`).
    - If `ylimit` is zero, set `ychange` to zero if the new height is less than or equal to the current height, otherwise set it to the difference between the new and current height.
    - If `ychange` is non-zero, call [`layout_resize_adjust`](#layout_resize_adjust) to apply the vertical change.
    - Call [`layout_fix_offsets`](#layout_fix_offsets) to update the offsets of the layout cells.
    - Call [`layout_fix_panes`](#layout_fix_panes) to update the pane sizes and offsets based on the new layout.
- **Output**:
    - The function does not return a value; it modifies the layout of the window in place.
- **Functions called**:
    - [`layout_resize_check`](#layout_resize_check)
    - [`layout_resize_adjust`](#layout_resize_adjust)
    - [`layout_fix_offsets`](#layout_fix_offsets)
    - [`layout_fix_panes`](#layout_fix_panes)


---
### layout_resize_pane_to<!-- {{#callable:layout_resize_pane_to}} -->
The `layout_resize_pane_to` function resizes a window pane to a specified size by adjusting its layout cell and its parent's layout cell.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane to be resized.
    - `type`: An `enum layout_type` indicating the layout direction (either `LAYOUT_LEFTRIGHT` or `LAYOUT_TOPBOTTOM`) for resizing.
    - `new_size`: An unsigned integer representing the new size to which the pane should be resized.
- **Control Flow**:
    - Retrieve the layout cell associated with the given window pane `wp`.
    - Find the nearest parent layout cell of the same type as specified by `type`.
    - If no such parent is found, exit the function without making changes.
    - Determine the current size of the layout cell in the specified direction (`sx` for `LAYOUT_LEFTRIGHT` and `sy` for `LAYOUT_TOPBOTTOM`).
    - Calculate the size change needed to achieve the `new_size`.
    - Call [`layout_resize_pane`](#layout_resize_pane) to apply the calculated size change to the pane.
- **Output**:
    - The function does not return a value; it modifies the layout of the specified pane in place.
- **Functions called**:
    - [`layout_resize_pane`](#layout_resize_pane)


---
### layout_resize_layout<!-- {{#callable:layout_resize_layout}} -->
The `layout_resize_layout` function adjusts the size of a layout cell within a window by either growing or shrinking it, and then updates the layout offsets and notifies of the change.
- **Inputs**:
    - `w`: A pointer to the `window` structure that contains the layout to be resized.
    - `lc`: A pointer to the `layout_cell` structure representing the cell to be resized.
    - `type`: An `enum layout_type` indicating the type of layout (e.g., left-right or top-bottom) to be resized.
    - `change`: An integer representing the amount of size change to apply to the layout cell.
    - `opposite`: An integer flag indicating whether to consider resizing in the opposite direction if needed.
- **Control Flow**:
    - Initialize `needed` with the value of `change` to track the remaining size change needed.
    - Enter a loop that continues until `needed` is zero, indicating no more resizing is required.
    - If `change` is positive, attempt to grow the pane using [`layout_resize_pane_grow`](#layout_resize_pane_grow), and subtract the returned size from `needed`.
    - If `change` is negative, attempt to shrink the pane using [`layout_resize_pane_shrink`](#layout_resize_pane_shrink), and add the returned size to `needed`.
    - Break the loop if no size change is possible (i.e., `size` is zero).
    - Call [`layout_fix_offsets`](#layout_fix_offsets) to update the offsets of the layout cells.
    - Call [`layout_fix_panes`](#layout_fix_panes) to update the pane sizes and offsets within the window.
    - Notify that the window layout has changed using `notify_window`.
- **Output**:
    - The function does not return a value; it modifies the layout of the window in place and triggers a notification of the layout change.
- **Functions called**:
    - [`layout_resize_pane_grow`](#layout_resize_pane_grow)
    - [`layout_resize_pane_shrink`](#layout_resize_pane_shrink)
    - [`layout_fix_offsets`](#layout_fix_offsets)
    - [`layout_fix_panes`](#layout_fix_panes)


---
### layout_resize_pane<!-- {{#callable:layout_resize_pane}} -->
The `layout_resize_pane` function adjusts the size of a specific pane within a window layout by finding the appropriate parent cell and resizing the layout accordingly.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane to be resized.
    - `type`: An enumeration value of `layout_type` indicating the type of layout (e.g., horizontal or vertical) to be resized.
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
The `layout_resize_pane_grow` function attempts to increase the size of a specified layout cell by reducing the size of adjacent cells in a window layout.
- **Inputs**:
    - `w`: A pointer to the `window` structure representing the window containing the layout.
    - `lc`: A pointer to the `layout_cell` structure representing the cell to be grown.
    - `type`: An `enum layout_type` indicating the type of layout (e.g., left-right or top-bottom).
    - `needed`: An integer specifying the amount of space needed to grow the cell.
    - `opposite`: An integer flag indicating whether to search in the opposite direction if no suitable cell is found initially.
- **Control Flow**:
    - Initialize `lcadd` to the current cell `lc` to be grown.
    - Search for a suitable adjacent cell `lcremove` towards the tail of the list to reduce its size.
    - If no suitable cell is found and `opposite` is true, search towards the head of the list.
    - If a suitable cell `lcremove` is found, calculate the size to adjust, ensuring it does not exceed `needed`.
    - Adjust the size of `lcadd` by increasing it and `lcremove` by decreasing it by the calculated size.
    - Return the size by which the cell was successfully grown.
- **Output**:
    - The function returns the actual size by which the cell was grown, or 0 if no suitable adjacent cell was found to reduce.
- **Functions called**:
    - [`layout_resize_check`](#layout_resize_check)
    - [`layout_resize_adjust`](#layout_resize_adjust)


---
### layout_resize_pane_shrink<!-- {{#callable:layout_resize_pane_shrink}} -->
The `layout_resize_pane_shrink` function attempts to shrink a pane within a window layout by redistributing space from adjacent cells.
- **Inputs**:
    - `w`: A pointer to the `window` structure representing the window containing the layout.
    - `lc`: A pointer to the `layout_cell` structure representing the cell to be shrunk.
    - `type`: An `enum layout_type` indicating the type of layout (e.g., left-right or top-bottom) to be adjusted.
    - `needed`: An integer representing the amount of space needed to shrink the pane.
- **Control Flow**:
    - Initialize `lcremove` to the current cell `lc` and attempt to find a cell to shrink by moving towards the head of the list of cells.
    - Use [`layout_resize_check`](#layout_resize_check) to determine if the current cell `lcremove` can be shrunk; if not, move to the previous cell in the list.
    - If no suitable cell is found (`lcremove` becomes `NULL`), return 0 indicating no shrinkage is possible.
    - Set `lcadd` to the next cell in the list from the original cell `lc`; if `lcadd` is `NULL`, return 0 as no adjacent cell is available to expand.
    - Determine the maximum shrink size by comparing `size` and `-needed`, and adjust `lcadd` and `lcremove` by this size using [`layout_resize_adjust`](#layout_resize_adjust).
    - Return the size of the adjustment made.
- **Output**:
    - The function returns an integer representing the actual size by which the pane was successfully shrunk.
- **Functions called**:
    - [`layout_resize_check`](#layout_resize_check)
    - [`layout_resize_adjust`](#layout_resize_adjust)


---
### layout_assign_pane<!-- {{#callable:layout_assign_pane}} -->
The `layout_assign_pane` function assigns a window pane to a layout cell and optionally resizes the panes in the window.
- **Inputs**:
    - `lc`: A pointer to the `layout_cell` structure where the window pane will be assigned.
    - `wp`: A pointer to the `window_pane` structure that is being assigned to the layout cell.
    - `do_not_resize`: An integer flag indicating whether the panes should be resized (0 for resizing, non-zero for no resizing).
- **Control Flow**:
    - Call [`layout_make_leaf`](#layout_make_leaf) to set the layout cell `lc` as a leaf node containing the window pane `wp`.
    - Check if `do_not_resize` is true; if so, call [`layout_fix_panes`](#layout_fix_panes) with the window of `wp` and `wp` itself to fix pane offsets and sizes without resizing.
    - If `do_not_resize` is false, call [`layout_fix_panes`](#layout_fix_panes) with the window of `wp` and `NULL` to fix pane offsets and sizes with resizing.
- **Output**:
    - The function does not return a value; it modifies the layout cell and window pane structures in place.
- **Functions called**:
    - [`layout_make_leaf`](#layout_make_leaf)
    - [`layout_fix_panes`](#layout_fix_panes)


---
### layout_new_pane_size<!-- {{#callable:layout_new_pane_size}} -->
The `layout_new_pane_size` function calculates the new size for a pane in a window layout based on available space, previous size, and layout type constraints.
- **Inputs**:
    - `w`: A pointer to the `window` structure representing the window containing the layout.
    - `previous`: An unsigned integer representing the previous size of the layout.
    - `lc`: A pointer to the `layout_cell` structure representing the current layout cell.
    - `type`: An enumeration value of `layout_type` indicating the type of layout (e.g., left-right or top-bottom).
    - `size`: An unsigned integer representing the desired size for the new pane.
    - `count_left`: An unsigned integer representing the number of panes left to be sized.
    - `size_left`: An unsigned integer representing the total size left to be distributed among the remaining panes.
- **Control Flow**:
    - Check if this is the last cell; if so, return the remaining size (`size_left`).
    - Calculate the available space in the parent layout using [`layout_resize_check`](#layout_resize_check).
    - Determine the minimum size required for the current cell based on the number of remaining panes and the layout type.
    - Calculate the new size proportionate to the previous size, adjusting for layout type (left-right or top-bottom).
    - Ensure the new size does not exceed the maximum allowable size (`size_left - min`) and is not less than the minimum pane size (`PANE_MINIMUM`).
    - Return the calculated new size for the pane.
- **Output**:
    - Returns an unsigned integer representing the calculated new size for the pane.
- **Functions called**:
    - [`layout_resize_check`](#layout_resize_check)


---
### layout_set_size_check<!-- {{#callable:layout_set_size_check}} -->
The [`layout_set_size_check`](#layout_set_size_check) function verifies if a layout cell and its children can be resized to a specified size while maintaining layout constraints.
- **Inputs**:
    - `w`: A pointer to the `window` structure representing the window containing the layout.
    - `lc`: A pointer to the `layout_cell` structure representing the layout cell to be checked.
    - `type`: An `enum layout_type` indicating the type of layout (e.g., LAYOUT_LEFTRIGHT or LAYOUT_TOPBOTTOM) to be checked.
    - `size`: An integer representing the desired size to which the layout cell should be resized.
- **Control Flow**:
    - If the layout cell `lc` is of type `LAYOUT_WINDOWPANE`, it checks if the size is greater than or equal to `PANE_MINIMUM` and returns the result.
    - If the layout cell `lc` has children, it calculates the number of children and the available size.
    - If the layout cell `lc` is of the same type as the specified `type`, it checks if the available size is sufficient for the children and iterates over each child to calculate and verify the new size using [`layout_new_pane_size`](#layout_new_pane_size).
    - For each child, it recursively calls [`layout_set_size_check`](#layout_set_size_check) to ensure the new size is valid for the child layout.
    - If the layout cell `lc` is not of the same type as `type`, it iterates over each child and recursively calls [`layout_set_size_check`](#layout_set_size_check) for non-`LAYOUT_WINDOWPANE` children.
    - Returns 1 if all checks pass, otherwise returns 0.
- **Output**:
    - Returns an integer: 1 if the layout cell and its children can be resized to the specified size, 0 otherwise.
- **Functions called**:
    - [`layout_new_pane_size`](#layout_new_pane_size)
    - [`layout_set_size_check`](#layout_set_size_check)


---
### layout_resize_child_cells<!-- {{#callable:layout_resize_child_cells}} -->
The function [`layout_resize_child_cells`](#layout_resize_child_cells) resizes all child cells within a given layout cell to fit within the current cell's dimensions.
- **Inputs**:
    - `w`: A pointer to the `window` structure, representing the window containing the layout.
    - `lc`: A pointer to the `layout_cell` structure, representing the current layout cell whose children need to be resized.
- **Control Flow**:
    - Check if the current layout cell `lc` is of type `LAYOUT_WINDOWPANE`; if so, return immediately as it has no children to resize.
    - Initialize variables `count` and `previous` to zero to track the number of child cells and the total size used by them, respectively.
    - Iterate over each child cell `lcchild` in the `lc->cells` list to calculate the total size `previous` used by the children based on the layout type (`LAYOUT_LEFTRIGHT` or `LAYOUT_TOPBOTTOM`).
    - Add the number of separators (count - 1) to `previous` to account for the space between child cells.
    - Determine the `available` size based on the layout type (`LAYOUT_LEFTRIGHT` or `LAYOUT_TOPBOTTOM`).
    - Iterate over each child cell `lcchild` again to resize them according to the available space and update their offsets.
    - For each child cell, calculate the new size using [`layout_new_pane_size`](#layout_new_pane_size) and adjust the `available` space accordingly.
    - Recursively call [`layout_resize_child_cells`](#layout_resize_child_cells) for each child cell to ensure all nested children are resized appropriately.
- **Output**:
    - The function does not return a value; it modifies the sizes and offsets of the child cells in place.
- **Functions called**:
    - [`layout_new_pane_size`](#layout_new_pane_size)
    - [`layout_resize_child_cells`](#layout_resize_child_cells)


---
### layout_split_pane<!-- {{#callable:layout_split_pane}} -->
The `layout_split_pane` function splits a given window pane into two new panes based on specified layout type and size constraints.
- **Inputs**:
    - `wp`: A pointer to the `window_pane` structure representing the pane to be split.
    - `type`: An enum value of `layout_type` indicating the direction of the split, either `LAYOUT_LEFTRIGHT` or `LAYOUT_TOPBOTTOM`.
    - `size`: An integer specifying the target size for the new pane, or -1 for a default half/half split.
    - `flags`: An integer containing flags that modify the behavior of the split, such as `SPAWN_FULLSIZE` or `SPAWN_BEFORE`.
- **Control Flow**:
    - Determine if the split should be full size or based on the current pane, setting `lc` accordingly.
    - Retrieve the current pane's size and offset values.
    - Check if there is enough space to perform the split based on the layout type and scrollbar settings.
    - Calculate the sizes of the new panes (`size1` and `size2`) based on the provided size or default to a middle split.
    - If the parent cell is of the same type as the split, insert a new cell adjacent to the current one; otherwise, create a new parent cell and adjust the layout tree accordingly.
    - Set the sizes and offsets for the new cells based on the calculated sizes and the split type.
    - If the split is full size, resize child cells and fix offsets; otherwise, make the current cell a leaf node.
    - Return the newly created layout cell representing the new pane.
- **Output**:
    - Returns a pointer to the newly created `layout_cell` structure representing the new pane, or `NULL` if the split cannot be performed.
- **Functions called**:
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
    - Check if the window's layout root is not NULL, indicating that there are still panes in the layout.
    - If there are remaining panes, call [`layout_fix_offsets`](#layout_fix_offsets) to adjust the offsets of the remaining layout cells.
    - Call [`layout_fix_panes`](#layout_fix_panes) to adjust the sizes of the remaining panes based on the updated layout.
    - Notify that the window layout has changed by calling `notify_window` with the event "window-layout-changed".
- **Output**:
    - The function does not return a value; it modifies the layout of the window by removing a pane and adjusting the remaining layout.
- **Functions called**:
    - [`layout_destroy_cell`](#layout_destroy_cell)
    - [`layout_fix_offsets`](#layout_fix_offsets)
    - [`layout_fix_panes`](#layout_fix_panes)


---
### layout_spread_cell<!-- {{#callable:layout_spread_cell}} -->
The `layout_spread_cell` function evenly distributes space among child cells of a parent layout cell in a window, adjusting their sizes accordingly.
- **Inputs**:
    - `w`: A pointer to the `window` structure representing the window containing the layout.
    - `parent`: A pointer to the `layout_cell` structure representing the parent cell whose child cells are to be spread out.
- **Control Flow**:
    - Initialize `number` to count the number of child cells in `parent` using a loop.
    - If `number` is less than or equal to 1, return 0 as no spreading is needed.
    - Retrieve `status` and `scrollbars` options from the window's options.
    - Determine the available `size` for spreading based on the `parent` cell's type and scrollbar settings.
    - If `size` is less than `number - 1`, return 0 as there is insufficient space to spread.
    - Calculate `each` as the evenly distributed size for each child cell and `remainder` as the leftover space.
    - Initialize `changed` to 0 to track if any changes are made to child cell sizes.
    - Iterate over each child cell `lc` in `parent` and calculate `change` based on the difference between `each` and the current size of `lc`.
    - Adjust the size of each child cell using [`layout_resize_adjust`](#layout_resize_adjust) and update `remainder` if necessary.
    - If any `change` is made, set `changed` to 1.
    - Return `changed` to indicate if any adjustments were made.
- **Output**:
    - Returns an integer indicating whether any child cell sizes were changed (1 if changed, 0 if not).
- **Functions called**:
    - [`layout_add_horizontal_border`](#layout_add_horizontal_border)
    - [`layout_resize_adjust`](#layout_resize_adjust)


---
### layout_spread_out<!-- {{#callable:layout_spread_out}} -->
The `layout_spread_out` function attempts to evenly distribute the space among sibling cells of a window pane's parent layout cell, adjusting offsets and pane sizes accordingly.
- **Inputs**:
    - `wp`: A pointer to a `window_pane` structure, representing the window pane whose layout is to be adjusted.
- **Control Flow**:
    - Retrieve the parent layout cell of the given window pane's layout cell.
    - If the parent layout cell is NULL, exit the function as there is no layout to adjust.
    - Enter a loop to traverse up the layout tree, starting from the parent cell.
    - In each iteration, call [`layout_spread_cell`](#layout_spread_cell) to attempt to spread out the cells within the current parent cell.
    - If [`layout_spread_cell`](#layout_spread_cell) returns true, indicating a successful adjustment, call [`layout_fix_offsets`](#layout_fix_offsets) and [`layout_fix_panes`](#layout_fix_panes) to update the layout offsets and pane sizes, then break out of the loop.
    - Continue traversing up the layout tree until a successful adjustment is made or the root is reached.
- **Output**:
    - The function does not return a value; it modifies the layout of the window panes in place.
- **Functions called**:
    - [`layout_spread_cell`](#layout_spread_cell)
    - [`layout_fix_offsets`](#layout_fix_offsets)
    - [`layout_fix_panes`](#layout_fix_panes)


# Function Declarations (Public API)

---
### layout_resize_check<!-- {{#callable_declaration:layout_resize_check}} -->
Calculate the available space for resizing a layout cell.
- **Description**: This function determines the amount of space that can be removed from a specified layout cell within a window, based on the layout type. It is used to check if a cell can be resized without violating minimum size constraints. The function should be called when planning to resize a layout cell, ensuring that the operation will not result in a layout that is too small. It considers the type of layout (horizontal or vertical) and adjusts for any additional space required by elements like scrollbars or borders.
- **Inputs**:
    - `w`: A pointer to the window structure containing the layout. Must not be null.
    - `lc`: A pointer to the layout cell to be checked. Must not be null and should be part of the window's layout.
    - `type`: The type of layout (either LAYOUT_LEFTRIGHT or LAYOUT_TOPBOTTOM) to check against. Determines the direction of resizing.
- **Output**: Returns the amount of space available for resizing the specified layout cell, in unsigned integer units.
- **See also**: [`layout_resize_check`](#layout_resize_check)  (Implementation)


---
### layout_resize_pane_grow<!-- {{#callable_declaration:layout_resize_pane_grow}} -->
Adjusts the size of a layout cell by growing it.
- **Description**: This function is used to increase the size of a specified layout cell within a window layout. It attempts to grow the cell by a specified amount, looking for adjacent cells from which space can be reallocated. The function should be called when a layout cell needs to be expanded, and it will adjust the sizes of the involved cells accordingly. It is important to ensure that the window and layout cell are valid and properly initialized before calling this function.
- **Inputs**:
    - `w`: A pointer to the window structure containing the layout. Must not be null.
    - `lc`: A pointer to the layout cell to be grown. Must not be null and should be part of the window's layout.
    - `type`: The type of layout (e.g., horizontal or vertical) that determines the direction of growth. Must be a valid enum layout_type value.
    - `needed`: The amount of space needed to grow the layout cell. Must be a non-negative integer.
    - `opposite`: A flag indicating whether to consider cells in the opposite direction for space reallocation. Non-zero to enable, zero to disable.
- **Output**: Returns the actual amount of space by which the layout cell was grown. Returns 0 if no growth was possible.
- **See also**: [`layout_resize_pane_grow`](#layout_resize_pane_grow)  (Implementation)


---
### layout_resize_pane_shrink<!-- {{#callable_declaration:layout_resize_pane_shrink}} -->
Shrink a pane within a window layout.
- **Description**: This function is used to shrink a pane within a window layout by redistributing space from one cell to another. It should be called when a reduction in pane size is needed, and it will attempt to find a suitable cell to remove space from and another to add space to. The function requires a valid window and layout cell, and the type of layout must be specified. It returns the amount of space successfully adjusted, or zero if no adjustment was possible.
- **Inputs**:
    - `w`: A pointer to the window structure containing the layout. Must not be null.
    - `lc`: A pointer to the layout cell that is the starting point for the shrink operation. Must not be null.
    - `type`: The type of layout (e.g., LAYOUT_LEFTRIGHT or LAYOUT_TOPBOTTOM) that determines the direction of the shrink operation.
    - `needed`: The amount of space to shrink, specified as a negative integer. The function will attempt to shrink by this amount.
- **Output**: Returns the amount of space successfully adjusted as an integer. If no adjustment was possible, returns zero.
- **See also**: [`layout_resize_pane_shrink`](#layout_resize_pane_shrink)  (Implementation)


---
### layout_new_pane_size<!-- {{#callable_declaration:layout_new_pane_size}} -->
Calculate the new size for a pane in a window layout.
- **Description**: This function determines the appropriate size for a new pane within a window layout, taking into account the available space, the type of layout, and the number of panes remaining to be sized. It is typically used during the process of resizing or splitting panes in a window layout. The function ensures that the new pane size does not exceed the available space and respects minimum size constraints. It should be called when adjusting pane sizes to maintain a balanced layout.
- **Inputs**:
    - `w`: A pointer to the window structure containing the layout. Must not be null.
    - `previous`: The previous total size of the layout before resizing. Must be a positive integer.
    - `lc`: A pointer to the layout cell for which the new size is being calculated. Must not be null.
    - `type`: The type of layout (e.g., LAYOUT_LEFTRIGHT or LAYOUT_TOPBOTTOM). Determines the direction of resizing.
    - `size`: The desired size for the new pane. Must be a positive integer.
    - `count_left`: The number of panes remaining to be sized, including the current one. Must be a positive integer.
    - `size_left`: The total size available for the remaining panes. Must be a positive integer.
- **Output**: Returns the calculated size for the new pane as an unsigned integer, ensuring it fits within the available space and respects minimum size constraints.
- **See also**: [`layout_new_pane_size`](#layout_new_pane_size)  (Implementation)


---
### layout_set_size_check<!-- {{#callable_declaration:layout_set_size_check}} -->
Checks if a layout cell and its children can be resized to a specified size.
- **Description**: This function determines whether a given layout cell and all its child cells can be resized to a specified size while maintaining the layout constraints. It is typically used to validate potential layout changes before applying them. The function must be called with a valid window and layout cell, and the size must be a non-negative integer. It returns a boolean indicating whether the resizing is feasible.
- **Inputs**:
    - `w`: A pointer to a 'struct window' representing the window containing the layout. Must not be null.
    - `lc`: A pointer to a 'struct layout_cell' representing the layout cell to be checked. Must not be null.
    - `type`: An 'enum layout_type' indicating the type of layout (e.g., LAYOUT_LEFTRIGHT or LAYOUT_TOPBOTTOM). Determines the direction of resizing.
    - `size`: An integer representing the desired size for the layout cell. Must be non-negative.
- **Output**: Returns 1 if the layout cell and its children can be resized to the specified size, otherwise returns 0.
- **See also**: [`layout_set_size_check`](#layout_set_size_check)  (Implementation)


---
### layout_resize_child_cells<!-- {{#callable_declaration:layout_resize_child_cells}} -->
Resize all child cells to fit within the current cell.
- **Description**: This function adjusts the sizes of all child cells within a given layout cell to ensure they fit within the available space of the parent cell. It should be used when the size of a parent cell changes, necessitating a recalibration of its children's sizes. The function operates recursively, resizing each child cell and its descendants. It is important to ensure that the parent cell is not of type LAYOUT_WINDOWPANE before calling this function, as it does not apply to window pane cells.
- **Inputs**:
    - `w`: A pointer to the window structure containing the layout. Must not be null.
    - `lc`: A pointer to the layout cell whose children are to be resized. Must not be null and must not be of type LAYOUT_WINDOWPANE.
- **Output**: None
- **See also**: [`layout_resize_child_cells`](#layout_resize_child_cells)  (Implementation)


