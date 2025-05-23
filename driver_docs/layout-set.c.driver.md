# Purpose
This C source code file is part of the tmux terminal multiplexer project and is responsible for managing window layouts within tmux sessions. The file defines several functions that arrange the panes within a tmux window according to predefined layout strategies. These strategies include even horizontal and vertical splits, main pane-focused layouts (both horizontal and vertical, with mirrored variants), and a tiled layout. The code provides a mechanism to set, select, and cycle through these layouts, allowing users to organize their terminal panes efficiently.

The file contains a collection of static functions that implement the logic for each layout type, as well as functions to look up and apply these layouts to a given window. The layout_set_lookup function identifies a layout by name, while layout_set_select, layout_set_next, and layout_set_previous functions manage the application and cycling of layouts. The layout_set_tiled function, for example, arranges panes in a grid-like structure, optimizing the use of available space. This file is integral to the tmux application, providing a modular and flexible approach to window management, enhancing user productivity by allowing customized pane arrangements.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### layout_sets
- **Type**: `static const struct`
- **Description**: The `layout_sets` variable is an array of structures, each containing a layout name and a corresponding function pointer for arranging windows. It defines a set of predefined window layout methods, such as 'even-horizontal', 'main-vertical', and 'tiled', each associated with a specific function to arrange windows in that layout.
- **Use**: This variable is used to map layout names to their respective arrangement functions, allowing the program to apply different window layouts based on user selection or program logic.


# Functions

---
### layout_set_lookup<!-- {{#callable:layout_set_lookup}} -->
The `layout_set_lookup` function searches for a layout by name in a predefined list and returns its index or an indication of ambiguity.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the layout to search for.
- **Control Flow**:
    - Initialize an integer `matched` to -1 to track the index of a partial match.
    - Iterate over the `layout_sets` array using a for loop to find an exact match with `strcmp`; return the index if found.
    - If no exact match is found, iterate again using `strncmp` to find a partial match; if a partial match is found and `matched` is not -1, return -1 to indicate ambiguity.
    - If a single partial match is found, store its index in `matched`.
    - Return the value of `matched`, which will be -1 if no match is found or the index of the partial match if one exists.
- **Output**:
    - Returns the index of the layout if an exact match is found, -1 if multiple partial matches are found, or the index of a single partial match if no exact match is found.


---
### layout_set_select<!-- {{#callable:layout_set_select}} -->
The `layout_set_select` function sets a specified layout for a window and returns the layout index.
- **Inputs**:
    - `w`: A pointer to a `struct window` representing the window for which the layout is to be set.
    - `layout`: An unsigned integer representing the index of the desired layout in the `layout_sets` array.
- **Control Flow**:
    - Check if the provided layout index is greater than the maximum index of the `layout_sets` array; if so, set it to the maximum index.
    - Check if the `arrange` function pointer for the specified layout is not NULL; if it is not, call the `arrange` function with the window `w` as an argument.
    - Set the `lastlayout` field of the window `w` to the specified layout index.
    - Return the layout index.
- **Output**:
    - The function returns the layout index that was set for the window.


---
### layout_set_next<!-- {{#callable:layout_set_next}} -->
The `layout_set_next` function updates a window's layout to the next predefined layout in a cyclic manner and applies it.
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
The `layout_set_previous` function sets the layout of a window to the previous layout in a predefined list of layouts.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window whose layout is to be set.
- **Control Flow**:
    - Check if the `lastlayout` of the window `w` is -1, indicating no previous layout; if so, set `layout` to the last index of `layout_sets`.
    - If `lastlayout` is not -1, set `layout` to `lastlayout`.
    - If `layout` is 0, set it to the last index of `layout_sets`; otherwise, decrement `layout` by 1.
    - Check if the `arrange` function pointer in `layout_sets[layout]` is not NULL; if so, call it with `w` as the argument to arrange the window.
    - Update `w->lastlayout` to the current `layout`.
    - Return the current `layout`.
- **Output**:
    - The function returns the index of the layout that was set, as an unsigned integer (`u_int`).


---
### layout_set_even<!-- {{#callable:layout_set_even}} -->
The `layout_set_even` function arranges the panes of a window evenly in either a horizontal or vertical layout based on the specified layout type.
- **Inputs**:
    - `w`: A pointer to the `window` structure representing the window whose layout is to be set.
    - `type`: An enumeration value of type `layout_type` indicating the desired layout direction, either `LAYOUT_LEFTRIGHT` for horizontal or `LAYOUT_TOPBOTTOM` for vertical.
- **Control Flow**:
    - Prints the current layout cell for debugging purposes.
    - Counts the number of panes in the window; if there is only one pane, the function returns early as no layout change is needed.
    - Frees the existing layout root and creates a new root layout cell.
    - Determines the size of the new layout based on the number of panes and the window's dimensions, adjusting for the minimum pane size.
    - Sets the size of the root layout cell and initializes it as a node of the specified layout type.
    - Iterates over each pane in the window, creating a new leaf layout cell for each pane and adding it to the root cell's list of children.
    - Spreads the layout cells evenly within the window's dimensions.
    - Fixes the offsets of the layout cells to ensure they are correctly positioned.
    - Adjusts the panes to fit the new layout and prints the updated layout cell for debugging.
    - Resizes the window to match the new layout dimensions, notifies that the window layout has changed, and triggers a redraw of the window.
- **Output**:
    - The function does not return a value; it modifies the layout of the window in place and triggers a redraw of the window.


---
### layout_set_even_h<!-- {{#callable:layout_set_even_h}} -->
The function `layout_set_even_h` arranges the panes of a given window in an even horizontal layout.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window whose layout is to be set.
- **Control Flow**:
    - The function calls [`layout_set_even`](#layout_set_even) with the window `w` and the layout type `LAYOUT_LEFTRIGHT` to arrange the panes evenly in a horizontal manner.
- **Output**:
    - This function does not return a value; it modifies the layout of the window `w` in place.
- **Functions called**:
    - [`layout_set_even`](#layout_set_even)


---
### layout_set_even_v<!-- {{#callable:layout_set_even_v}} -->
The `layout_set_even_v` function arranges the panes of a window in a vertical, evenly distributed layout.
- **Inputs**:
    - `w`: A pointer to a `struct window` object representing the window whose layout is to be set.
- **Control Flow**:
    - The function calls [`layout_set_even`](#layout_set_even) with the window `w` and the layout type `LAYOUT_TOPBOTTOM` to arrange the panes vertically.
- **Output**:
    - The function does not return a value; it modifies the layout of the given window in place.
- **Functions called**:
    - [`layout_set_even`](#layout_set_even)


---
### layout_set_main_h<!-- {{#callable:layout_set_main_h}} -->
The `layout_set_main_h` function arranges the panes in a window with a main pane at the top and other panes below it, adjusting their sizes based on specified options.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window whose layout is being set.
- **Control Flow**:
    - Prints the current layout cell for debugging purposes.
    - Counts the number of panes in the window and returns if there is only one pane.
    - Calculates the available height for panes by subtracting one for the border.
    - Retrieves the desired height for the main pane from options and calculates it as a percentage of the available height.
    - If the calculated main pane height is invalid, defaults to a height of 24.
    - Calculates the height for the other panes, ensuring it does not exceed available space and adjusts if necessary.
    - Calculates the required width for the layout based on the number of panes and ensures it fits within the window's width.
    - Frees the existing layout tree and creates a new root layout cell.
    - Creates and sizes the main pane cell, adding it to the layout tree.
    - Creates and sizes the other pane cell, adding it to the layout tree, and if there are multiple panes, arranges them horizontally within this cell.
    - Adjusts the layout to spread the panes evenly and fixes offsets and pane sizes.
    - Prints the updated layout cell for debugging purposes.
    - Resizes the window to fit the new layout and triggers a redraw and notification of the layout change.
- **Output**:
    - The function does not return a value; it modifies the layout of the given window in place.


---
### layout_set_main_h_mirrored<!-- {{#callable:layout_set_main_h_mirrored}} -->
The `layout_set_main_h_mirrored` function arranges the panes in a window with a main pane at the bottom and other panes above it, mirroring the main-horizontal layout.
- **Inputs**:
    - `w`: A pointer to the `window` structure representing the window whose layout is to be set.
- **Control Flow**:
    - Prints the current layout cell for debugging purposes.
    - Counts the number of panes in the window and returns if there is only one pane.
    - Calculates the available height for panes, subtracting one for the border.
    - Retrieves and calculates the height for the main pane using the 'main-pane-height' option, defaulting to 24 if an error occurs.
    - Determines the height for the other panes, ensuring it fits within the available space and adjusting if necessary.
    - Calculates the required width for the layout based on the number of panes and the minimum pane size.
    - Frees the existing layout tree and creates a new root layout cell.
    - Creates a layout node for the other panes, either as a single pane or a node containing multiple panes, and spreads them horizontally if needed.
    - Creates a layout cell for the main pane at the bottom of the layout.
    - Fixes the offsets of the layout cells and adjusts the panes to fit the new layout.
    - Prints the updated layout cell for debugging purposes.
    - Resizes the window to fit the new layout dimensions and triggers a redraw of the window.
- **Output**:
    - The function does not return a value; it modifies the layout of the given window in place.


---
### layout_set_main_v<!-- {{#callable:layout_set_main_v}} -->
The `layout_set_main_v` function arranges the panes of a window in a vertical layout with a main pane and other panes distributed alongside it.
- **Inputs**:
    - `w`: A pointer to a `struct window` which represents the window whose layout is to be set.
- **Control Flow**:
    - Prints the current layout cell for debugging purposes.
    - Counts the number of panes in the window; if there is only one pane, the function returns early.
    - Calculates the available width for the panes, subtracting one for the border.
    - Retrieves the main pane width from the window's options and calculates it as a percentage of the available width; defaults to 80 if an error occurs.
    - Determines the width of the other panes based on the remaining space after allocating the main pane width.
    - Calculates the required height for the layout based on the number of panes and the minimum pane size.
    - Frees the existing layout tree and creates a new root layout cell.
    - Sets the size of the root layout cell and configures it to arrange its children from left to right.
    - Creates and sizes the main pane as a child of the root layout cell.
    - Creates and sizes the other pane(s) as a child of the root layout cell; if there are multiple other panes, they are arranged top to bottom within their parent cell.
    - Adjusts the offsets of all layout cells to ensure they are correctly positioned.
    - Fixes the pane sizes to match the layout cell sizes.
    - Prints the updated layout cell for debugging purposes.
    - Resizes the window to match the new layout dimensions.
    - Notifies the system that the window layout has changed and requests a redraw of the window.
- **Output**:
    - The function does not return a value; it modifies the layout of the given window in place.


---
### layout_set_main_v_mirrored<!-- {{#callable:layout_set_main_v_mirrored}} -->
The `layout_set_main_v_mirrored` function arranges the panes of a window in a vertical layout with a main pane and mirrored secondary panes.
- **Inputs**:
    - `w`: A pointer to a `struct window` representing the window whose layout is to be set.
- **Control Flow**:
    - Prints the current layout cell for debugging purposes.
    - Counts the number of panes in the window and returns if there is only one pane.
    - Calculates the available width for the panes, subtracting one for the border.
    - Retrieves the main pane width from the window options and calculates it as a percentage of the available width.
    - Determines the width of the other panes, ensuring they fit within the available width and adjusting if necessary.
    - Calculates the required height for the layout based on the number of panes and the minimum pane size.
    - Frees the existing layout tree and creates a new root layout cell.
    - Sets the size of the root layout cell and configures it for a left-right layout.
    - Creates and sizes the secondary pane cell, adding it to the root layout cell.
    - If there is more than one secondary pane, creates a top-bottom layout for the secondary panes and adds each pane as a child cell.
    - Spreads the secondary pane cells to fit the layout.
    - Creates and sizes the main pane cell, adding it to the root layout cell.
    - Fixes the offsets of the layout cells and adjusts the panes to fit the new layout.
    - Prints the updated layout cell for debugging purposes.
    - Resizes the window to fit the new layout dimensions and triggers a redraw of the window.
- **Output**:
    - The function does not return a value; it modifies the layout of the given window in place.


---
### layout_set_tiled<!-- {{#callable:layout_set_tiled}} -->
The `layout_set_tiled` function arranges the panes of a window in a tiled layout, dividing the window into a grid of rows and columns to fit all panes.
- **Inputs**:
    - `w`: A pointer to a `struct window` representing the window whose panes are to be arranged in a tiled layout.
- **Control Flow**:
    - Prints the current layout cell for debugging purposes.
    - Counts the number of panes in the window; if there is only one pane, the function returns immediately.
    - Calculates the number of rows and columns needed to fit all panes, incrementing rows and columns until the product is greater than or equal to the number of panes.
    - Determines the width and height of each pane based on the window's dimensions and the number of columns and rows, ensuring they are not smaller than the minimum pane size.
    - Frees the existing layout tree and creates a new root layout cell with the calculated size.
    - Iterates over the rows, creating a new row cell for each row and setting its size.
    - If there is only one column or only one pane left, the row is used directly as a leaf node for the pane.
    - Otherwise, creates a node for columns within the row and iterates over the columns, creating a cell for each pane and adding it to the row.
    - Adjusts the last column in each row to fit the full width of the window if necessary.
    - Adjusts the height of the last row to fit the full height of the window if necessary.
    - Fixes the offsets of all cells and panes to ensure they are correctly positioned.
    - Prints the updated layout cell for debugging purposes.
    - Resizes the window to match the new layout dimensions and notifies the system of the layout change, triggering a redraw of the window.
- **Output**:
    - The function does not return a value; it modifies the layout of the window's panes in place.


# Function Declarations (Public API)

---
### layout_set_even_h<!-- {{#callable_declaration:layout_set_even_h}} -->
Arranges the panes in a window evenly in a horizontal layout.
- **Description**: This function arranges all panes within a given window in an even horizontal layout, distributing the available space equally among them. It is typically used when a uniform distribution of panes is desired across the width of the window. This function should be called when the window is already initialized and contains multiple panes. It does not handle cases where there is only one pane, as no layout adjustment is necessary in such scenarios.
- **Inputs**:
    - `w`: A pointer to a 'struct window' representing the window whose panes are to be arranged. Must not be null. The caller retains ownership of the window structure.
- **Output**: None
- **See also**: [`layout_set_even_h`](#layout_set_even_h)  (Implementation)


---
### layout_set_even_v<!-- {{#callable_declaration:layout_set_even_v}} -->
Arranges the panes in a window in an even vertical layout.
- **Description**: Use this function to arrange the panes within a given window in an even vertical layout, where panes are stacked from top to bottom. This function is typically used when you want to ensure that all panes within the window are evenly distributed vertically. It should be called when a window needs to be rearranged to this specific layout style. The function assumes that the window structure is properly initialized and contains multiple panes.
- **Inputs**:
    - `w`: A pointer to a 'struct window' that represents the window to be arranged. Must not be null. The caller retains ownership of the window structure.
- **Output**: None
- **See also**: [`layout_set_even_v`](#layout_set_even_v)  (Implementation)


---
### layout_set_main_h<!-- {{#callable_declaration:layout_set_main_h}} -->
Arrange window panes in a main-horizontal layout.
- **Description**: This function arranges the panes of a given window into a main-horizontal layout, where one pane is designated as the main pane and occupies a larger portion of the window's height, while the remaining panes are arranged horizontally below it. It should be used when a user wants to emphasize one pane over others in a horizontal layout. The function assumes that the window has more than one pane and will not perform any action if there is only one pane. It modifies the window's layout and triggers a redraw of the window.
- **Inputs**:
    - `w`: A pointer to a 'struct window' representing the window whose panes are to be arranged. Must not be null. The function assumes that the window has been properly initialized and contains at least two panes.
- **Output**: None
- **See also**: [`layout_set_main_h`](#layout_set_main_h)  (Implementation)


---
### layout_set_main_h_mirrored<!-- {{#callable_declaration:layout_set_main_h_mirrored}} -->
Arranges window panes in a mirrored horizontal layout with a main pane.
- **Description**: This function arranges the panes within a given window into a mirrored horizontal layout, where one pane is designated as the main pane and occupies a larger portion of the window's height, while the remaining panes are arranged horizontally in the remaining space. It should be used when a mirrored layout with a prominent main pane is desired. The function assumes that the window has more than one pane and will not perform any action if there is only one pane. It modifies the layout of the window and triggers a redraw of the window to reflect the new layout.
- **Inputs**:
    - `w`: A pointer to a 'struct window' representing the window whose panes are to be arranged. Must not be null. The function assumes that the window has been properly initialized and contains at least two panes.
- **Output**: None
- **See also**: [`layout_set_main_h_mirrored`](#layout_set_main_h_mirrored)  (Implementation)


---
### layout_set_main_v<!-- {{#callable_declaration:layout_set_main_v}} -->
Arrange window panes in a vertical layout with a main pane.
- **Description**: This function arranges the panes of a given window into a vertical layout where one pane is designated as the main pane, and the remaining panes are arranged vertically alongside it. It should be used when a vertical layout with a prominent main pane is desired. The function assumes that the window has more than one pane; otherwise, it returns without making changes. It calculates the width of the main and other panes based on the window's available width and user-defined options, ensuring that the layout fits within the window's dimensions. The function modifies the window's layout tree and triggers a redraw of the window.
- **Inputs**:
    - `w`: A pointer to a 'struct window' representing the window whose panes are to be arranged. Must not be null. The function assumes the window has more than one pane; otherwise, it returns immediately.
- **Output**: None
- **See also**: [`layout_set_main_v`](#layout_set_main_v)  (Implementation)


---
### layout_set_main_v_mirrored<!-- {{#callable_declaration:layout_set_main_v_mirrored}} -->
Arranges window panes in a mirrored vertical layout with a main pane.
- **Description**: This function arranges the panes within a given window into a mirrored vertical layout, where one pane is designated as the main pane and occupies a larger portion of the window's width. The remaining panes are arranged vertically in the remaining space. This function should be called when a specific layout is desired for better organization or visualization of panes. It requires the window to have more than one pane to perform the layout change. The function modifies the window's layout and triggers a redraw of the window to reflect the changes.
- **Inputs**:
    - `w`: A pointer to a 'struct window' representing the window whose panes are to be arranged. Must not be null. The function assumes the window has at least two panes; otherwise, it returns without making changes.
- **Output**: None
- **See also**: [`layout_set_main_v_mirrored`](#layout_set_main_v_mirrored)  (Implementation)


---
### layout_set_tiled<!-- {{#callable_declaration:layout_set_tiled}} -->
Arranges the panes of a window in a tiled layout.
- **Description**: This function arranges the panes within a given window in a tiled layout, distributing them evenly across the available space. It is typically used when a user wants to organize multiple panes in a grid-like structure within a window. The function calculates the optimal number of rows and columns based on the number of panes and adjusts their sizes to fit within the window's dimensions. It should be called when the window has more than one pane, as it does nothing if there is only one pane. The function modifies the layout of the window and triggers a redraw to reflect the changes.
- **Inputs**:
    - `w`: A pointer to a 'struct window' representing the window whose panes are to be arranged. Must not be null. The function assumes the window has been properly initialized and contains at least one pane.
- **Output**: None
- **See also**: [`layout_set_tiled`](#layout_set_tiled)  (Implementation)


