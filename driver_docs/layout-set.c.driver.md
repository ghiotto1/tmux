# Purpose
This C source code file is part of the tmux terminal multiplexer project and is responsible for managing window layouts within tmux sessions. The file defines several functions that arrange the panes within a tmux window according to predefined layout strategies. These strategies include even horizontal and vertical splits, main pane layouts with horizontal and vertical orientations (both standard and mirrored), and a tiled layout. The code provides a mechanism to set, select, and cycle through these layouts, allowing users to organize their terminal panes in a way that best suits their workflow.

The file is not an executable on its own but rather a component of the larger tmux application, likely intended to be compiled and linked with other parts of the tmux codebase. It defines internal functions and data structures that are used to manipulate the layout of window panes, but it does not expose a public API or external interfaces directly. The core functionality revolves around creating and managing a layout tree structure, which represents the arrangement of panes, and adjusting this structure based on user commands or configuration options. The code ensures that the layout changes are reflected visually by triggering redraws of the window when necessary.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### layout_sets
- **Type**: `array of structs`
- **Description**: The `layout_sets` variable is a static constant array of structures, where each structure contains a layout name and a corresponding function pointer. This array defines a set of predefined window layout methods, each associated with a specific arrangement function for windows.
- **Use**: This variable is used to map layout names to their respective arrangement functions, allowing the program to apply different window layouts based on user selection or program logic.


# Functions

---
### layout_set_lookup<!-- {{#callable:layout_set_lookup}} -->
The `layout_set_lookup` function searches for a layout by name in a predefined list and returns its index or indicates ambiguity.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the layout to search for in the `layout_sets` array.
- **Control Flow**:
    - Initialize an integer `matched` to -1 to track the index of a potential match.
    - Iterate over the `layout_sets` array using a for loop to find an exact match with `strcmp`; return the index if found.
    - If no exact match is found, iterate again using `strncmp` to find a partial match; if a match is found and `matched` is already set, return -1 to indicate ambiguity.
    - If a single partial match is found, set `matched` to the current index.
    - Return the value of `matched`, which is either the index of a single partial match or -1 if no match or ambiguous matches were found.
- **Output**:
    - The function returns an integer representing the index of the layout in the `layout_sets` array if found, -1 if no match is found, or -1 if multiple partial matches are found (indicating ambiguity).


---
### layout_set_select<!-- {{#callable:layout_set_select}} -->
The `layout_set_select` function sets a specified layout for a given window and returns the layout index.
- **Inputs**:
    - `w`: A pointer to a `struct window` representing the window for which the layout is to be set.
    - `layout`: An unsigned integer representing the index of the desired layout in the `layout_sets` array.
- **Control Flow**:
    - Check if the provided layout index is greater than the maximum index of the `layout_sets` array; if so, set it to the maximum index.
    - If the `arrange` function pointer for the specified layout is not NULL, call it with the window `w` as an argument to arrange the window according to the layout.
    - Set the `lastlayout` field of the window `w` to the specified layout index.
    - Return the layout index.
- **Output**:
    - Returns the layout index that was set for the window.


---
### layout_set_next<!-- {{#callable:layout_set_next}} -->
The `layout_set_next` function updates a window's layout to the next available predefined layout and applies it.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window whose layout is to be updated.
- **Control Flow**:
    - Check if the `lastlayout` of the window is -1; if so, set `layout` to 0.
    - Otherwise, increment `lastlayout` by 1 and assign it to `layout`.
    - Check if `layout` exceeds the number of available layouts; if so, reset `layout` to 0.
    - If the `arrange` function for the current `layout` is not NULL, call it with the window `w`.
    - Update the window's `lastlayout` to the current `layout`.
    - Return the current `layout`.
- **Output**:
    - The function returns the index of the next layout as an unsigned integer.


---
### layout_set_previous<!-- {{#callable:layout_set_previous}} -->
The `layout_set_previous` function sets the layout of a window to the previous layout in a predefined list of layouts, wrapping around if necessary.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window whose layout is to be set.
- **Control Flow**:
    - Check if the `lastlayout` of the window is -1, indicating no previous layout; if so, set `layout` to the last index of `layout_sets`.
    - If `lastlayout` is not -1, set `layout` to `lastlayout`.
    - If `layout` is 0, set it to the last index of `layout_sets`; otherwise, decrement `layout` by 1.
    - Check if the `arrange` function pointer in `layout_sets[layout]` is not NULL; if so, call it with the window `w`.
    - Update the `lastlayout` of the window `w` to the current `layout`.
- **Output**:
    - Returns the index of the layout that was set, as an unsigned integer (`u_int`).


---
### layout_set_even<!-- {{#callable:layout_set_even}} -->
The `layout_set_even` function arranges the panes of a window evenly either horizontally or vertically based on the specified layout type.
- **Inputs**:
    - `w`: A pointer to the `struct window` object representing the window whose layout is to be set.
    - `type`: An `enum layout_type` value indicating the layout direction, either `LAYOUT_LEFTRIGHT` for horizontal or `LAYOUT_TOPBOTTOM` for vertical arrangement.
- **Control Flow**:
    - Prints the current layout cell for debugging purposes.
    - Counts the number of panes in the window; if there is only one pane, the function returns early.
    - Frees the existing layout root and creates a new root layout cell.
    - Calculates the size of the new layout based on the number of panes and the window's dimensions, adjusting for the minimum pane size.
    - Sets the size of the root layout cell and initializes it as a node of the specified layout type.
    - Iterates over each pane in the window, creating a new layout cell for each pane, setting its size, and adding it to the root cell's list of child cells.
    - Spreads the layout cells evenly within the window.
    - Fixes the offsets of the layout cells to ensure they are correctly positioned.
    - Adjusts the panes to fit the new layout.
    - Prints the updated layout cell for debugging purposes.
    - Resizes the window to fit the new layout dimensions.
    - Notifies that the window layout has changed and requests a redraw of the window.
- **Output**:
    - The function does not return a value; it modifies the layout of the window in place.
- **Functions called**:
    - [`layout_print_cell`](layout.c.driver.md#layout_print_cell)
    - [`window_count_panes`](window.c.driver.md#window_count_panes)
    - [`layout_free`](layout.c.driver.md#layout_free)
    - [`layout_create_cell`](layout.c.driver.md#layout_create_cell)
    - [`layout_set_size`](layout.c.driver.md#layout_set_size)
    - [`layout_make_node`](layout.c.driver.md#layout_make_node)
    - [`layout_make_leaf`](layout.c.driver.md#layout_make_leaf)
    - [`layout_spread_cell`](layout.c.driver.md#layout_spread_cell)
    - [`layout_fix_offsets`](layout.c.driver.md#layout_fix_offsets)
    - [`layout_fix_panes`](layout.c.driver.md#layout_fix_panes)
    - [`window_resize`](window.c.driver.md#window_resize)
    - [`notify_window`](notify.c.driver.md#notify_window)
    - [`server_redraw_window`](server-fn.c.driver.md#server_redraw_window)


---
### layout_set_even_h<!-- {{#callable:layout_set_even_h}} -->
The `layout_set_even_h` function arranges the panes of a window in an even horizontal layout.
- **Inputs**:
    - `w`: A pointer to a `struct window` object representing the window whose layout is to be set.
- **Control Flow**:
    - The function calls [`layout_set_even`](#layout_set_even) with the window `w` and the layout type `LAYOUT_LEFTRIGHT` to arrange the panes horizontally.
- **Output**:
    - The function does not return a value; it modifies the layout of the given window `w` to an even horizontal arrangement.
- **Functions called**:
    - [`layout_set_even`](#layout_set_even)


---
### layout_set_even_v<!-- {{#callable:layout_set_even_v}} -->
The function `layout_set_even_v` arranges the panes of a window in a vertical, evenly distributed layout.
- **Inputs**:
    - `w`: A pointer to a `struct window` object, representing the window whose layout is to be set.
- **Control Flow**:
    - The function calls [`layout_set_even`](#layout_set_even) with the window `w` and the layout type `LAYOUT_TOPBOTTOM` to arrange the panes vertically.
- **Output**:
    - The function does not return a value; it modifies the layout of the given window in place.
- **Functions called**:
    - [`layout_set_even`](#layout_set_even)


---
### layout_set_main_h<!-- {{#callable:layout_set_main_h}} -->
The `layout_set_main_h` function arranges the panes of a window in a layout where the main pane is at the top and other panes are below it, distributed horizontally.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window whose layout is being set.
- **Control Flow**:
    - Prints the current layout cell for debugging purposes.
    - Counts the number of panes in the window and returns if there is only one pane.
    - Calculates the available height for panes, subtracting one for the border.
    - Retrieves the main pane height from options and calculates it as a percentage of available height, defaulting to 24 if an error occurs.
    - Determines the height of the other panes based on the main pane height and available space, ensuring minimum pane size constraints are met.
    - Calculates the required width for the layout based on the number of panes and minimum pane size, ensuring it does not exceed the window's width.
    - Frees the existing layout tree and creates a new root layout cell.
    - Creates the main pane cell with the calculated size and adds it to the layout tree.
    - Creates the other pane cell, either as a single pane or a node containing multiple child panes, and adds it to the layout tree.
    - Spreads the child panes within the other pane cell if there are multiple panes.
    - Fixes the offsets of the layout cells and adjusts the panes to fit the new layout.
    - Prints the updated layout cell for debugging purposes.
    - Resizes the window to fit the new layout dimensions and triggers a redraw of the window.
- **Output**:
    - The function does not return a value; it modifies the layout of the given window in place.
- **Functions called**:
    - [`layout_print_cell`](layout.c.driver.md#layout_print_cell)
    - [`window_count_panes`](window.c.driver.md#window_count_panes)
    - [`options_get_string`](options.c.driver.md#options_get_string)
    - [`args_string_percentage`](arguments.c.driver.md#args_string_percentage)
    - [`layout_free`](layout.c.driver.md#layout_free)
    - [`layout_create_cell`](layout.c.driver.md#layout_create_cell)
    - [`layout_set_size`](layout.c.driver.md#layout_set_size)
    - [`layout_make_node`](layout.c.driver.md#layout_make_node)
    - [`layout_make_leaf`](layout.c.driver.md#layout_make_leaf)
    - [`layout_spread_cell`](layout.c.driver.md#layout_spread_cell)
    - [`layout_fix_offsets`](layout.c.driver.md#layout_fix_offsets)
    - [`layout_fix_panes`](layout.c.driver.md#layout_fix_panes)
    - [`window_resize`](window.c.driver.md#window_resize)
    - [`notify_window`](notify.c.driver.md#notify_window)
    - [`server_redraw_window`](server-fn.c.driver.md#server_redraw_window)


---
### layout_set_main_h_mirrored<!-- {{#callable:layout_set_main_h_mirrored}} -->
The `layout_set_main_h_mirrored` function arranges the panes of a window in a mirrored horizontal layout with a main pane and other panes distributed horizontally.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window whose layout is to be set.
- **Control Flow**:
    - Prints the current layout cell for debugging purposes.
    - Counts the number of panes in the window and returns if there is only one pane.
    - Calculates the available height for panes, subtracting one for the border.
    - Retrieves the main pane height from options and calculates it as a percentage of available height, defaulting to 24 if an error occurs.
    - Determines the height of the other panes based on the main pane height and available space, ensuring minimum pane size constraints are met.
    - Calculates the required width for the layout based on the number of panes and minimum pane size, ensuring it does not exceed the window's width.
    - Frees the existing layout tree and creates a new root layout cell.
    - Creates a layout node for the other panes, either as a single pane or a horizontal node containing multiple panes, and distributes the panes accordingly.
    - Creates a layout cell for the main pane and inserts it into the layout tree.
    - Fixes the offsets of the layout cells and adjusts the panes to fit the new layout.
    - Prints the updated layout cell for debugging purposes.
    - Resizes the window to fit the new layout dimensions and triggers a redraw of the window.
- **Output**:
    - The function does not return a value; it modifies the layout of the given window in place.
- **Functions called**:
    - [`layout_print_cell`](layout.c.driver.md#layout_print_cell)
    - [`window_count_panes`](window.c.driver.md#window_count_panes)
    - [`options_get_string`](options.c.driver.md#options_get_string)
    - [`args_string_percentage`](arguments.c.driver.md#args_string_percentage)
    - [`layout_free`](layout.c.driver.md#layout_free)
    - [`layout_create_cell`](layout.c.driver.md#layout_create_cell)
    - [`layout_set_size`](layout.c.driver.md#layout_set_size)
    - [`layout_make_node`](layout.c.driver.md#layout_make_node)
    - [`layout_make_leaf`](layout.c.driver.md#layout_make_leaf)
    - [`layout_spread_cell`](layout.c.driver.md#layout_spread_cell)
    - [`layout_fix_offsets`](layout.c.driver.md#layout_fix_offsets)
    - [`layout_fix_panes`](layout.c.driver.md#layout_fix_panes)
    - [`window_resize`](window.c.driver.md#window_resize)
    - [`notify_window`](notify.c.driver.md#notify_window)
    - [`server_redraw_window`](server-fn.c.driver.md#server_redraw_window)


---
### layout_set_main_v<!-- {{#callable:layout_set_main_v}} -->
The `layout_set_main_v` function arranges the panes of a window in a vertical layout with a main pane and other panes distributed alongside it.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window whose layout is to be set.
- **Control Flow**:
    - Prints the current layout cell for debugging purposes.
    - Counts the number of panes in the window and returns if there is only one pane.
    - Calculates the available width for the layout, subtracting one for the border.
    - Retrieves the main pane width from the window options and calculates it as a percentage of the available width.
    - Determines the width of the other panes based on the remaining space after allocating the main pane width.
    - Calculates the required height for the layout based on the number of panes and the minimum pane size.
    - Frees the existing layout tree and creates a new root layout cell.
    - Sets the size of the root layout cell and configures it to arrange its children from left to right.
    - Creates and sizes the main pane as a child of the root layout cell.
    - Creates and sizes the other pane(s) as children of the root layout cell, arranging them top to bottom if there are multiple.
    - Adjusts the offsets of the layout cells to ensure they are correctly positioned.
    - Fixes the pane sizes to match the layout cell sizes.
    - Prints the updated layout cell for debugging purposes.
    - Resizes the window to match the new layout dimensions.
    - Notifies that the window layout has changed and requests a redraw of the window.
- **Output**:
    - The function does not return a value; it modifies the layout of the window `w` in place.
- **Functions called**:
    - [`layout_print_cell`](layout.c.driver.md#layout_print_cell)
    - [`window_count_panes`](window.c.driver.md#window_count_panes)
    - [`options_get_string`](options.c.driver.md#options_get_string)
    - [`args_string_percentage`](arguments.c.driver.md#args_string_percentage)
    - [`layout_free`](layout.c.driver.md#layout_free)
    - [`layout_create_cell`](layout.c.driver.md#layout_create_cell)
    - [`layout_set_size`](layout.c.driver.md#layout_set_size)
    - [`layout_make_node`](layout.c.driver.md#layout_make_node)
    - [`layout_make_leaf`](layout.c.driver.md#layout_make_leaf)
    - [`layout_spread_cell`](layout.c.driver.md#layout_spread_cell)
    - [`layout_fix_offsets`](layout.c.driver.md#layout_fix_offsets)
    - [`layout_fix_panes`](layout.c.driver.md#layout_fix_panes)
    - [`window_resize`](window.c.driver.md#window_resize)
    - [`notify_window`](notify.c.driver.md#notify_window)
    - [`server_redraw_window`](server-fn.c.driver.md#server_redraw_window)


---
### layout_set_main_v_mirrored<!-- {{#callable:layout_set_main_v_mirrored}} -->
The `layout_set_main_v_mirrored` function arranges the panes of a window in a vertically mirrored layout with a main pane and other panes distributed alongside it.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window whose layout is to be set.
- **Control Flow**:
    - Prints the current layout cell for debugging purposes.
    - Counts the number of panes in the window and returns if there is only one pane.
    - Calculates the available width for the panes, subtracting one for the border.
    - Retrieves the main pane width from the window options and calculates it as a percentage of the available width.
    - Determines the width of the other panes, ensuring they fit within the available width and adjusting if necessary.
    - Calculates the required height for the layout based on the number of panes and the minimum pane size.
    - Frees the existing layout tree and creates a new root layout cell.
    - Sets the size of the root layout cell and configures it to arrange its children left-to-right.
    - Creates a layout cell for the other panes, setting its size and arranging it top-to-bottom if there are multiple panes.
    - Iterates over the remaining panes, creating a layout cell for each and adding them as children to the other pane cell.
    - Spreads the other pane cell to distribute the panes evenly.
    - Creates a layout cell for the main pane, setting its size and adding it to the root layout cell.
    - Fixes the offsets of the layout cells and adjusts the panes to fit the new layout.
    - Prints the updated layout cell for debugging purposes.
    - Resizes the window to match the new layout dimensions and triggers a redraw of the window.
- **Output**:
    - The function does not return a value; it modifies the layout of the given window in place.
- **Functions called**:
    - [`layout_print_cell`](layout.c.driver.md#layout_print_cell)
    - [`window_count_panes`](window.c.driver.md#window_count_panes)
    - [`options_get_string`](options.c.driver.md#options_get_string)
    - [`args_string_percentage`](arguments.c.driver.md#args_string_percentage)
    - [`layout_free`](layout.c.driver.md#layout_free)
    - [`layout_create_cell`](layout.c.driver.md#layout_create_cell)
    - [`layout_set_size`](layout.c.driver.md#layout_set_size)
    - [`layout_make_node`](layout.c.driver.md#layout_make_node)
    - [`layout_make_leaf`](layout.c.driver.md#layout_make_leaf)
    - [`layout_spread_cell`](layout.c.driver.md#layout_spread_cell)
    - [`layout_fix_offsets`](layout.c.driver.md#layout_fix_offsets)
    - [`layout_fix_panes`](layout.c.driver.md#layout_fix_panes)
    - [`window_resize`](window.c.driver.md#window_resize)
    - [`notify_window`](notify.c.driver.md#notify_window)
    - [`server_redraw_window`](server-fn.c.driver.md#server_redraw_window)


---
### layout_set_tiled<!-- {{#callable:layout_set_tiled}} -->
The `layout_set_tiled` function arranges the panes of a window in a tiled grid layout, adjusting the number of rows and columns based on the number of panes and the window's dimensions.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window whose panes are to be arranged in a tiled layout.
- **Control Flow**:
    - The function starts by printing the current layout cell for debugging purposes.
    - It retrieves the number of panes in the window using `window_count_panes(w)`.
    - If there is only one pane, the function returns early as no layout change is needed.
    - The function calculates the number of rows and columns required to fit all panes, incrementing rows and columns until the product is greater than or equal to the number of panes.
    - It calculates the width and height for each pane based on the window's dimensions, ensuring they are not smaller than `PANE_MINIMUM`.
    - The existing layout tree is freed, and a new root layout cell is created with the calculated dimensions.
    - The function iterates over the rows, creating a new row cell for each row and setting its size.
    - If there is only one column or only one pane left in the row, it directly assigns the pane to the row cell.
    - Otherwise, it creates a node for columns and iterates over the columns, creating and sizing each pane cell, and linking it to the corresponding pane.
    - If the total width used by the columns is less than the window's width, it adjusts the last column to fit the remaining space.
    - After processing all rows, it adjusts the last row's height if the total height used is less than the window's height.
    - Finally, it fixes the offsets of all cells, adjusts the panes, prints the new layout cell for debugging, resizes the window, and triggers a redraw and notification for the layout change.
- **Output**:
    - The function does not return a value; it modifies the layout of the window's panes in place.
- **Functions called**:
    - [`layout_print_cell`](layout.c.driver.md#layout_print_cell)
    - [`window_count_panes`](window.c.driver.md#window_count_panes)
    - [`layout_free`](layout.c.driver.md#layout_free)
    - [`layout_create_cell`](layout.c.driver.md#layout_create_cell)
    - [`layout_set_size`](layout.c.driver.md#layout_set_size)
    - [`layout_make_node`](layout.c.driver.md#layout_make_node)
    - [`layout_make_leaf`](layout.c.driver.md#layout_make_leaf)
    - [`layout_resize_adjust`](layout.c.driver.md#layout_resize_adjust)
    - [`layout_fix_offsets`](layout.c.driver.md#layout_fix_offsets)
    - [`layout_fix_panes`](layout.c.driver.md#layout_fix_panes)
    - [`window_resize`](window.c.driver.md#window_resize)
    - [`notify_window`](notify.c.driver.md#notify_window)
    - [`server_redraw_window`](server-fn.c.driver.md#server_redraw_window)


