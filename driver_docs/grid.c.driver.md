# Purpose
The provided C source code file is part of the `tmux` terminal multiplexer project and is responsible for managing the grid data structure, which represents the content displayed on the terminal screen. This file defines the core functionality for handling grid cells, which are the fundamental units of display in `tmux`. The grid is divided into history and viewable data, allowing for efficient management of terminal content, including scrolling and resizing operations. The code includes functions for creating, destroying, and manipulating grids, as well as handling individual grid cells, such as setting, clearing, and comparing them.

Key technical components of this file include the `grid_cell` and `grid_line` structures, which store information about individual cells and lines within the grid, respectively. The file provides a comprehensive set of functions to manage these structures, including functions to store and retrieve cell data, handle extended cells, manage line history, and perform operations like scrolling and reflowing lines to fit different screen sizes. The code also includes functions for converting grid content to strings with ANSI escape sequences, which are used for rendering text with various attributes like color and style. Overall, this file provides essential functionality for managing the display content in `tmux`, ensuring that terminal output is handled efficiently and accurately.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### grid_default_cell
- **Type**: `const struct grid_cell`
- **Description**: The `grid_default_cell` is a constant structure of type `grid_cell` that defines the default properties for a cell in a grid. It is initialized with specific values for character data, attributes, and color settings, which represent a space character with default styling and colors.
- **Use**: This variable is used as a template for initializing or resetting grid cells to a default state.


---
### grid_padding_cell
- **Type**: `struct grid_cell`
- **Description**: The `grid_padding_cell` is a static constant instance of the `struct grid_cell` data structure. It is used to represent padding cells within a grid, which are unique in that they are zero-width cells and are always considered extended cells.
- **Use**: This variable is used to set padding at specific positions in the grid, ensuring that these positions are marked with a special flag indicating they are padding.


---
### grid_cleared_cell
- **Type**: `struct grid_cell`
- **Description**: The `grid_cleared_cell` is a static constant instance of the `struct grid_cell` data structure. It represents a cleared cell in a grid, initialized with specific attributes such as a space character, default colors, and a cleared flag.
- **Use**: This variable is used to define the default state of a cleared cell in the grid, which can be referenced when resetting or clearing grid cells.


---
### grid_cleared_entry
- **Type**: ``struct grid_cell_entry``
- **Description**: The `grid_cleared_entry` is a static constant instance of the `struct grid_cell_entry` structure. It is initialized with a data array containing values `{0, 8, 8, ' '}` and a flag `GRID_FLAG_CLEARED`. This structure represents a cleared grid cell entry with default attributes.
- **Use**: This variable is used to represent a cleared cell in the grid, typically for initializing or resetting grid cells to a default cleared state.


# Functions

---
### grid_store_cell<!-- {{#callable:grid_store_cell}} -->
Stores a character and its associated attributes in a grid cell entry.
- **Inputs**:
    - `gce`: A pointer to a `struct grid_cell_entry` where the cell data will be stored.
    - `gc`: A pointer to a `struct grid_cell` containing the attributes to be copied.
    - `c`: A character (`u_char`) that represents the data to be stored in the cell.
- **Control Flow**:
    - The function first clears the `GRID_FLAG_CLEARED` flag from the `gce` flags.
    - It then sets the foreground color in `gce` from `gc`, checking if it uses 256 colors.
    - Next, it sets the background color in `gce` from `gc`, also checking for 256 colors.
    - The attributes and the character data are then directly copied from `gc` to `gce`.
- **Output**:
    - The function does not return a value; it modifies the `gce` structure directly to store the character and its attributes.


---
### grid_need_extended_cell<!-- {{#callable:grid_need_extended_cell}} -->
Determines if a grid cell should be treated as an extended cell based on various attributes.
- **Inputs**:
    - `gce`: A pointer to a `struct grid_cell_entry` representing the cell entry to be evaluated.
    - `gc`: A pointer to a `struct grid_cell` containing the attributes of the cell.
- **Control Flow**:
    - Checks if the `flags` of `gce` indicate it is already extended.
    - Evaluates if the `attr` of `gc` exceeds 0xff.
    - Checks if the `data.size` or `data.width` of `gc` is greater than 1.
    - Determines if the foreground or background color flags indicate RGB values.
    - Validates if the `us` attribute of `gc` is not equal to 8.
    - Checks if the `link` attribute of `gc` is non-zero.
    - Checks if the `flags` of `gc` indicate it is a tab.
- **Output**:
    - Returns 1 if the cell should be treated as extended, otherwise returns 0.


---
### grid_get_extended_cell<!-- {{#callable:grid_get_extended_cell}} -->
The `grid_get_extended_cell` function allocates and initializes an extended cell entry in a grid line.
- **Inputs**:
    - `gl`: A pointer to a `struct grid_line` representing the grid line where the extended cell will be added.
    - `gce`: A pointer to a `struct grid_cell_entry` that will be updated to reference the new extended cell.
    - `flags`: An integer representing flags that will be combined with the `GRID_FLAG_EXTENDED` flag for the new cell.
- **Control Flow**:
    - Calculates the new size for the extended data array by incrementing the current size (`gl->extdsize`) by one.
    - Resizes the `extddata` array in the `grid_line` structure using [`xreallocarray`](xmalloc.c.driver.md#xreallocarray) to accommodate the new cell.
    - Updates the `extdsize` of the grid line to reflect the new size.
    - Sets the `offset` of the `gce` to point to the last position in the extended data array (newly added cell).
    - Combines the provided `flags` with `GRID_FLAG_EXTENDED` and assigns it to the `flags` field of the `gce`.
- **Output**:
    - The function does not return a value; it modifies the `gce` structure to reference the newly allocated extended cell and updates the grid line's extended data.
- **Functions called**:
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)


---
### grid_extended_cell<!-- {{#callable:grid_extended_cell}} -->
The `grid_extended_cell` function retrieves or creates an extended grid cell entry based on the provided grid line and cell entry.
- **Inputs**:
    - `gl`: A pointer to a `struct grid_line` representing the grid line where the cell is located.
    - `gce`: A pointer to a `struct grid_cell_entry` representing the cell entry that may need to be extended.
    - `gc`: A pointer to a `struct grid_cell` containing the attributes and data for the cell.
- **Control Flow**:
    - The function first checks if the `gce` (grid cell entry) is not marked as extended; if not, it calls [`grid_get_extended_cell`](#grid_get_extended_cell) to create an extended cell.
    - If the `gce` offset is greater than or equal to the extended size of the grid line, it triggers a fatal error.
    - The function sets the extended flag on the grid line.
    - It checks if the cell has a tab flag; if so, it assigns the width to `uc`, otherwise it converts the cell data to UTF-8 format.
    - Finally, it populates the extended entry with the cell's data, attributes, and flags, and returns a pointer to the extended entry.
- **Output**:
    - The function returns a pointer to a `struct grid_extd_entry` that contains the extended cell data.
- **Functions called**:
    - [`grid_get_extended_cell`](#grid_get_extended_cell)
    - [`utf8_from_data`](utf8.c.driver.md#utf8_from_data)


---
### grid_compact_line<!-- {{#callable:grid_compact_line}} -->
The `grid_compact_line` function reduces the size of the extended data array in a grid line by removing unused entries.
- **Inputs**:
    - `gl`: A pointer to a `struct grid_line` that contains the cell data and extended data to be compacted.
- **Control Flow**:
    - Checks if the `extdsize` of the grid line is zero; if so, it returns immediately.
    - Iterates through each cell in the grid line to count how many cells are marked as extended.
    - If no extended cells are found, it frees the extended data and sets its pointer and size to NULL and zero, respectively.
    - Allocates a new array for the extended data based on the count of extended cells.
    - Copies the data of the extended cells from the old array to the new array, updating their offsets.
    - Frees the old extended data array and updates the grid line's extended data pointer and size.
- **Output**:
    - The function does not return a value; it modifies the `grid_line` structure in place, specifically updating the `extddata` pointer and `extdsize` field.
- **Functions called**:
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)


---
### grid_get_line<!-- {{#callable:grid_get_line}} -->
The `grid_get_line` function retrieves a pointer to a specific line of grid data.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` that represents the grid from which the line is to be retrieved.
    - `line`: An unsigned integer representing the index of the line to be retrieved from the grid.
- **Control Flow**:
    - The function directly accesses the `linedata` array of the `grid` structure using the provided line index.
    - It returns a pointer to the `grid_line` structure located at the specified index.
- **Output**:
    - Returns a pointer to the `grid_line` structure corresponding to the specified line index in the grid.


---
### grid_adjust_lines<!-- {{#callable:grid_adjust_lines}} -->
The `grid_adjust_lines` function reallocates the memory for the line data in a grid structure to accommodate a specified number of lines.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` which contains the grid data including the line data.
    - `lines`: An unsigned integer representing the new number of lines to allocate for the grid.
- **Control Flow**:
    - The function calls [`xreallocarray`](xmalloc.c.driver.md#xreallocarray) to resize the `linedata` array in the `grid` structure.
    - The new size of the `linedata` array is determined by the `lines` parameter and the size of each line.
- **Output**:
    - The function does not return a value; it modifies the `linedata` pointer in the `grid` structure to point to the newly allocated memory.
- **Functions called**:
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)


---
### grid_clear_cell<!-- {{#callable:grid_clear_cell}} -->
Clears a specific cell in a grid and sets its background color if specified.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the grid.
    - `px`: An unsigned integer representing the x-coordinate (column) of the cell to clear.
    - `py`: An unsigned integer representing the y-coordinate (row) of the cell to clear.
    - `bg`: An unsigned integer representing the background color to set for the cleared cell.
- **Control Flow**:
    - Retrieve the line data from the grid at the specified y-coordinate `py`.
    - Access the cell entry at the specified x-coordinate `px` within the retrieved line.
    - Copy the default cleared cell data into the specified cell entry.
    - Check if the provided background color `bg` is not the default value (8).
    - If `bg` indicates an RGB color, retrieve the extended cell data and set the background color accordingly.
    - If `bg` indicates a 256-color, set the appropriate flags and assign the background color to the cell entry.
- **Output**:
    - The function does not return a value; it modifies the specified cell in the grid directly.
- **Functions called**:
    - [`grid_get_extended_cell`](#grid_get_extended_cell)
    - [`grid_extended_cell`](#grid_extended_cell)


---
### grid_check_y<!-- {{#callable:grid_check_y}} -->
Checks if the given y-coordinate is within the valid range of the grid.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure that contains the grid's dimensions and data.
    - `from`: A string representing the context or source of the call, used for logging.
    - `py`: An unsigned integer representing the y-coordinate to be checked.
- **Control Flow**:
    - The function first checks if the provided y-coordinate `py` is greater than or equal to the sum of `gd->hsize` and `gd->sy`.
    - If the condition is true, it logs a debug message indicating that the y-coordinate is out of range and returns -1.
    - If the condition is false, it returns 0, indicating that the y-coordinate is valid.
- **Output**:
    - Returns -1 if the y-coordinate is out of range, otherwise returns 0.


---
### grid_cells_look_equal<!-- {{#callable:grid_cells_look_equal}} -->
Compares two `grid_cell` structures to determine if they visually appear equal.
- **Inputs**:
    - `gc1`: A pointer to the first `grid_cell` structure to compare.
    - `gc2`: A pointer to the second `grid_cell` structure to compare.
- **Control Flow**:
    - Extracts the `flags` from both `grid_cell` structures.
    - Checks if the foreground (`fg`) colors of `gc1` and `gc2` are different; if so, returns 0 (not equal).
    - Checks if the background (`bg`) colors of `gc1` and `gc2` are different; if so, returns 0 (not equal).
    - Checks if the attributes (`attr`) of `gc1` and `gc2` are different; if so, returns 0 (not equal).
    - Compares the `flags` of both cells, ignoring the `GRID_FLAG_CLEARED` flag; if they differ, returns 0 (not equal).
    - Finally, checks if the `link` values of both cells are different; if so, returns 0 (not equal).
    - If all checks pass, returns 1 (equal).
- **Output**:
    - Returns 1 if the two `grid_cell` structures look equal, otherwise returns 0.


---
### grid_cells_equal<!-- {{#callable:grid_cells_equal}} -->
Compares two `grid_cell` structures for equality.
- **Inputs**:
    - `gc1`: Pointer to the first `grid_cell` structure to compare.
    - `gc2`: Pointer to the second `grid_cell` structure to compare.
- **Control Flow**:
    - First, it checks if the two cells look equal using the [`grid_cells_look_equal`](#grid_cells_look_equal) function.
    - If they do not look equal, it returns 0 (not equal).
    - Next, it compares the `width` of the `data` fields of both cells.
    - If the widths are not equal, it returns 0.
    - Then, it compares the `size` of the `data` fields of both cells.
    - If the sizes are not equal, it returns 0.
    - Finally, it uses `memcmp` to compare the actual data of both cells and returns 1 if they are equal, otherwise returns 0.
- **Output**:
    - Returns 1 if the two `grid_cell` structures are equal, otherwise returns 0.
- **Functions called**:
    - [`grid_cells_look_equal`](#grid_cells_look_equal)


---
### grid_set_tab<!-- {{#callable:grid_set_tab}} -->
Sets a `grid_cell` to represent a tab character with a specified width.
- **Inputs**:
    - `gc`: A pointer to a `struct grid_cell` that will be modified to represent a tab.
    - `width`: An unsigned integer representing the width of the tab.
- **Control Flow**:
    - Clears the existing data in the `data` field of the `grid_cell` by setting it to zero.
    - Sets the `flags` of the `grid_cell` to include the `GRID_FLAG_TAB` flag.
    - Assigns the `width`, `size`, and `have` fields of the `data` structure within the `grid_cell` to the specified width.
    - Fills the `data` field of the `grid_cell` with spaces, up to the specified width.
- **Output**:
    - The function does not return a value; it modifies the input `grid_cell` directly to represent a tab character.


---
### grid_free_line<!-- {{#callable:grid_free_line}} -->
The `grid_free_line` function frees the memory allocated for a specific line's cell data in a grid.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` which contains the grid data.
    - `py`: An unsigned integer representing the index of the line to be freed.
- **Control Flow**:
    - The function first frees the memory allocated for the `celldata` of the specified line.
    - It then sets the pointer to `celldata` to NULL to avoid dangling pointers.
    - Next, it frees the memory allocated for the `extddata` of the specified line.
    - Finally, it sets the pointer to `extddata` to NULL.
- **Output**:
    - The function does not return a value; it modifies the grid by freeing memory associated with the specified line.


---
### grid_free_lines<!-- {{#callable:grid_free_lines}} -->
The `grid_free_lines` function frees a specified number of lines in a grid starting from a given line index.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure that contains the lines to be freed.
    - `py`: The starting line index from which to begin freeing lines.
    - `ny`: The number of lines to free, starting from the line index `py`.
- **Control Flow**:
    - A loop iterates from the starting line index `py` to `py + ny`, calling [`grid_free_line`](#grid_free_line) for each line index.
    - The [`grid_free_line`](#grid_free_line) function is responsible for freeing the memory associated with each individual line.
- **Output**:
    - The function does not return a value; it modifies the grid by freeing the specified lines.
- **Functions called**:
    - [`grid_free_line`](#grid_free_line)


---
### grid_create<!-- {{#callable:grid_create}} -->
Creates a new `grid` structure with specified dimensions and history limit.
- **Inputs**:
    - `sx`: The width of the grid.
    - `sy`: The height of the grid.
    - `hlimit`: The maximum number of history lines; if zero, history is disabled.
- **Control Flow**:
    - Allocates memory for a new `grid` structure using [`xmalloc`](xmalloc.c.driver.md#xmalloc).
    - Sets the `sx` and `sy` fields of the grid to the provided width and height.
    - Checks if `hlimit` is non-zero to set the `flags` field to indicate history is enabled.
    - Initializes `hscrolled`, `hsize`, and `hlimit` fields of the grid.
    - If `sy` is non-zero, allocates memory for `linedata` to hold the grid lines; otherwise, sets it to NULL.
    - Returns the pointer to the newly created grid.
- **Output**:
    - Returns a pointer to the newly created `grid` structure.
- **Functions called**:
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)


---
### grid_destroy<!-- {{#callable:grid_destroy}} -->
The `grid_destroy` function deallocates memory associated with a `grid` structure.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` that represents the grid to be destroyed.
- **Control Flow**:
    - Calls [`grid_free_lines`](#grid_free_lines) to free all lines in the grid from the specified starting index to the total height of the grid.
    - Frees the memory allocated for the `linedata` array within the grid structure.
    - Finally, frees the memory allocated for the grid structure itself.
- **Output**:
    - This function does not return a value; it performs cleanup by freeing allocated memory.
- **Functions called**:
    - [`grid_free_lines`](#grid_free_lines)


---
### grid_compare<!-- {{#callable:grid_compare}} -->
Compares two `grid` structures for equality.
- **Inputs**:
    - `ga`: A pointer to the first `grid` structure to compare.
    - `gb`: A pointer to the second `grid` structure to compare.
- **Control Flow**:
    - First, it checks if the dimensions (width and height) of the two grids are the same; if not, it returns 1 indicating they are different.
    - Then, it iterates over each row of the grids, comparing the size of the cells in each row; if they differ, it returns 1.
    - For each cell in the rows, it retrieves the cell data from both grids and compares them using the [`grid_cells_equal`](#grid_cells_equal) function; if any cells are not equal, it returns 1.
    - If all checks pass, it returns 0 indicating the grids are equal.
- **Output**:
    - Returns 0 if the grids are equal, and 1 if they are different.
- **Functions called**:
    - [`grid_get_cell`](#grid_get_cell)
    - [`grid_cells_equal`](#grid_cells_equal)


---
### grid_trim_history<!-- {{#callable:grid_trim_history}} -->
Trims the grid's history by freeing a specified number of lines and shifting the remaining lines up.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the grid whose history is to be trimmed.
    - `ny`: An unsigned integer representing the number of lines to be trimmed from the top of the grid's history.
- **Control Flow**:
    - Calls [`grid_free_lines`](#grid_free_lines) to free the specified number of lines from the top of the grid's history.
    - Uses `memmove` to shift the remaining lines in the grid upwards, effectively removing the freed lines from the history.
- **Output**:
    - This function does not return a value; it modifies the grid in place by adjusting its history.
- **Functions called**:
    - [`grid_free_lines`](#grid_free_lines)


---
### grid_collect_history<!-- {{#callable:grid_collect_history}} -->
The `grid_collect_history` function manages the history of a grid by freeing the oldest lines when the history size exceeds a specified limit.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` that contains the grid's data, including its history size and limit.
- **Control Flow**:
    - Checks if the grid's history size (`hsize`) is zero or less than the history limit (`hlimit`), and returns early if true.
    - Calculates the number of lines (`ny`) to free, which is 10% of the `hlimit`, ensuring it is at least 1 and at most the current `hsize`.
    - Calls [`grid_trim_history`](#grid_trim_history) to free the specified number of lines from the history.
    - Decreases the `hsize` by the number of lines freed and adjusts `hscrolled` if it exceeds the new `hsize`.
- **Output**:
    - The function does not return a value; it modifies the grid's history by freeing lines and updating the history size.
- **Functions called**:
    - [`grid_trim_history`](#grid_trim_history)


---
### grid_remove_history<!-- {{#callable:grid_remove_history}} -->
The `grid_remove_history` function removes a specified number of lines from the history of a grid.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` representing the grid from which lines will be removed.
    - `ny`: An unsigned integer representing the number of lines to remove from the history.
- **Control Flow**:
    - Checks if the number of lines to remove (`ny`) is greater than the current history size (`gd->hsize`); if so, the function returns immediately without making any changes.
    - Iterates from 0 to `ny`, calling [`grid_free_line`](#grid_free_line) for each line to be removed, specifically targeting the last `ny` lines in the history.
    - Decreases the history size (`gd->hsize`) by the number of lines removed (`ny`).
- **Output**:
    - The function does not return a value; it modifies the grid's history by removing the specified number of lines.
- **Functions called**:
    - [`grid_free_line`](#grid_free_line)


---
### grid_scroll_history<!-- {{#callable:grid_scroll_history}} -->
Scrolls the grid history by adding a new line at the bottom.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the current grid.
    - `bg`: An unsigned integer representing the background color to be used for the new line.
- **Control Flow**:
    - Calculates the new line index `yy` as the sum of the current history size `hsize` and the visible screen size `sy`.
    - Resizes the `linedata` array to accommodate the new line.
    - Calls [`grid_empty_line`](#grid_empty_line) to initialize the new line at index `yy` with the specified background color.
    - Increments the `hscrolled` counter to indicate that the history has been scrolled.
    - Calls [`grid_compact_line`](#grid_compact_line) to compact the current last line in the history.
    - Sets the `time` of the new line to the current time and increments the `hsize` to reflect the addition of the new line.
- **Output**:
    - The function does not return a value; it modifies the grid's internal state to reflect the new line added to the history.
- **Functions called**:
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`grid_empty_line`](#grid_empty_line)
    - [`grid_compact_line`](#grid_compact_line)


---
### grid_clear_history<!-- {{#callable:grid_clear_history}} -->
Clears the history of a grid by resetting its size and reallocating its line data.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` which represents the grid whose history is to be cleared.
- **Control Flow**:
    - Calls [`grid_trim_history`](#grid_trim_history) to free the lines in the history based on the current history size.
    - Resets the `hscrolled` member of the grid to 0, indicating no lines are scrolled.
    - Sets the `hsize` member of the grid to 0, effectively clearing the history size.
    - Reallocates the `linedata` array to match the current screen height (`sy`), ensuring it only contains the viewable lines.
- **Output**:
    - The function does not return a value; it modifies the grid structure in place to clear its history.
- **Functions called**:
    - [`grid_trim_history`](#grid_trim_history)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)


---
### grid_scroll_history_region<!-- {{#callable:grid_scroll_history_region}} -->
Scrolls a specified region of a grid up by moving the top line into history.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the current grid.
    - `upper`: The upper boundary of the region to scroll, specified as a line index.
    - `lower`: The lower boundary of the region to scroll, specified as a line index.
    - `bg`: The background color to use when clearing the bottom line after scrolling.
- **Control Flow**:
    - Allocate space for a new line in the grid's line data.
    - Move the entire screen down by one line to create space for the new line.
    - Adjust the upper and lower boundaries to account for the scrolling operation.
    - Copy the line at the upper boundary into the newly created line at the bottom of the grid.
    - Update the timestamp of the newly added line to the current time.
    - Shift the lines in the specified region up by one line.
    - Clear the bottom line of the region with the specified background color.
    - Increment the history scroll count and the size of the history.
- **Output**:
    - This function does not return a value; it modifies the grid in place by scrolling the specified region and updating the history.
- **Functions called**:
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`grid_empty_line`](#grid_empty_line)


---
### grid_expand_line<!-- {{#callable:grid_expand_line}} -->
Expands a specified line in a grid to accommodate a new cell size.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the grid to be modified.
    - `py`: An unsigned integer representing the y-coordinate (line index) of the grid line to expand.
    - `sx`: An unsigned integer representing the new desired size for the line in terms of cells.
    - `bg`: An unsigned integer representing the background color to be used for newly added cells.
- **Control Flow**:
    - Retrieve the line data structure `gl` corresponding to the specified line index `py`.
    - Check if the new size `sx` is less than or equal to the current cell size of the line; if so, exit the function.
    - Adjust `sx` to ensure it meets minimum size requirements based on the grid's total width.
    - Reallocate the `celldata` array of the line to accommodate the new size `sx`.
    - Clear the newly added cells in the line by calling [`grid_clear_cell`](#grid_clear_cell) for each new cell from the old size to the new size.
    - Update the line's `cellsize` to the new size `sx`.
- **Output**:
    - The function does not return a value; it modifies the grid line in place to expand its size and clear any new cells.
- **Functions called**:
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`grid_clear_cell`](#grid_clear_cell)


---
### grid_empty_line<!-- {{#callable:grid_empty_line}} -->
The `grid_empty_line` function clears a specified line in a grid and optionally expands it to a specified background color.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the grid to be modified.
    - `py`: An unsigned integer representing the y-coordinate (line number) of the line to be cleared.
    - `bg`: An unsigned integer representing the background color to be set for the line if it is not the default.
- **Control Flow**:
    - The function first clears the specified line by setting all its data to zero using `memset`.
    - It then checks if the provided background color is not the default using the `COLOUR_DEFAULT` macro.
    - If the background color is not the default, it calls [`grid_expand_line`](#grid_expand_line) to expand the line to fit the grid's width and set the background color.
- **Output**:
    - The function does not return a value; it modifies the grid in place by clearing the specified line and potentially expanding it with a new background color.
- **Functions called**:
    - [`grid_expand_line`](#grid_expand_line)


---
### grid_peek_line<!-- {{#callable:grid_peek_line}} -->
The `grid_peek_line` function retrieves a pointer to a specific line in a grid structure.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure from which the line is to be retrieved.
    - `py`: An unsigned integer representing the y-coordinate (line index) of the line to be accessed.
- **Control Flow**:
    - The function first calls [`grid_check_y`](#grid_check_y) to validate the y-coordinate `py` against the grid's dimensions.
    - If [`grid_check_y`](#grid_check_y) returns a non-zero value, indicating an invalid y-coordinate, the function returns NULL.
    - If the y-coordinate is valid, the function returns a pointer to the line data at the specified index in the grid.
- **Output**:
    - Returns a pointer to the `grid_line` structure at the specified y-coordinate if valid, otherwise returns NULL.
- **Functions called**:
    - [`grid_check_y`](#grid_check_y)


---
### grid_get_cell1<!-- {{#callable:grid_get_cell1}} -->
Retrieves the cell data from a specified position in a grid line.
- **Inputs**:
    - `gl`: A pointer to the `grid_line` structure representing the line from which to retrieve the cell.
    - `px`: An unsigned integer representing the horizontal position (column index) of the cell within the grid line.
    - `gc`: A pointer to a `grid_cell` structure where the retrieved cell data will be stored.
- **Control Flow**:
    - Accesses the cell entry at the specified position `px` in the grid line `gl`.
    - Checks if the cell entry has the `GRID_FLAG_EXTENDED` flag set.
    - If the flag is set, it checks if the offset is valid; if not, it copies the default cell data into `gc`.
    - If the offset is valid, it retrieves the extended cell data and populates the `gc` structure with the appropriate attributes and data.
    - If the `GRID_FLAG_EXTENDED` flag is not set, it populates `gc` with the standard attributes and data from the cell entry.
- **Output**:
    - The function does not return a value; instead, it populates the `gc` structure with the cell data retrieved from the specified position in the grid line.
- **Functions called**:
    - [`grid_set_tab`](#grid_set_tab)
    - [`utf8_to_data`](utf8.c.driver.md#utf8_to_data)
    - [`utf8_set`](utf8.c.driver.md#utf8_set)


---
### grid_get_cell<!-- {{#callable:grid_get_cell}} -->
Retrieves a cell from a grid at specified coordinates.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the grid from which to retrieve the cell.
    - `px`: An unsigned integer representing the x-coordinate (column index) of the cell to retrieve.
    - `py`: An unsigned integer representing the y-coordinate (row index) of the cell to retrieve.
    - `gc`: A pointer to a `grid_cell` structure where the retrieved cell data will be stored.
- **Control Flow**:
    - The function first checks if the y-coordinate `py` is valid by calling [`grid_check_y`](#grid_check_y). If it is invalid, it returns a default cell.
    - Next, it checks if the x-coordinate `px` is within the bounds of the cell size for the specified row.
    - If both coordinates are valid, it calls [`grid_get_cell1`](#grid_get_cell1) to retrieve the cell data from the specified row and column.
- **Output**:
    - The function does not return a value; instead, it populates the `gc` structure with the cell data from the grid or sets it to a default cell if the coordinates are out of bounds.
- **Functions called**:
    - [`grid_check_y`](#grid_check_y)
    - [`grid_get_cell1`](#grid_get_cell1)


---
### grid_set_cell<!-- {{#callable:grid_set_cell}} -->
Sets a specific cell in a grid to a given grid cell value.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the grid where the cell is to be set.
    - `px`: An unsigned integer representing the x-coordinate (column index) of the cell to be set.
    - `py`: An unsigned integer representing the y-coordinate (row index) of the cell to be set.
    - `gc`: A pointer to a `grid_cell` structure containing the data to be stored in the specified cell.
- **Control Flow**:
    - The function first checks if the y-coordinate `py` is within the valid range of the grid using [`grid_check_y`](#grid_check_y).
    - If the y-coordinate is valid, it expands the line at `py` to ensure it can accommodate the new cell at `px` using [`grid_expand_line`](#grid_expand_line).
    - It then retrieves the line data for the specified row `py` and updates the `cellused` count if necessary.
    - Next, it checks if the cell at `px` needs to be an extended cell by calling [`grid_need_extended_cell`](#grid_need_extended_cell).
    - If an extended cell is needed, it calls [`grid_extended_cell`](#grid_extended_cell) to handle the storage; otherwise, it stores the cell data directly using [`grid_store_cell`](#grid_store_cell).
- **Output**:
    - The function does not return a value; it modifies the grid in place by setting the specified cell to the provided grid cell data.
- **Functions called**:
    - [`grid_check_y`](#grid_check_y)
    - [`grid_expand_line`](#grid_expand_line)
    - [`grid_need_extended_cell`](#grid_need_extended_cell)
    - [`grid_extended_cell`](#grid_extended_cell)
    - [`grid_store_cell`](#grid_store_cell)


---
### grid_set_padding<!-- {{#callable:grid_set_padding}} -->
Sets a padding cell in the grid at specified coordinates.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure where the padding cell will be set.
    - `px`: The x-coordinate (column index) in the grid where the padding cell is to be placed.
    - `py`: The y-coordinate (row index) in the grid where the padding cell is to be placed.
- **Control Flow**:
    - Calls the [`grid_set_cell`](#grid_set_cell) function to set the cell at the specified coordinates.
    - Passes the `grid_padding_cell` structure to [`grid_set_cell`](#grid_set_cell) to define the properties of the padding cell.
- **Output**:
    - The function does not return a value; it modifies the grid by setting a padding cell at the specified coordinates.
- **Functions called**:
    - [`grid_set_cell`](#grid_set_cell)


---
### grid_set_cells<!-- {{#callable:grid_set_cells}} -->
Sets a range of cells in a grid with specified character data and attributes.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the grid where cells will be set.
    - `px`: The x-coordinate (column index) in the grid where the cell setting starts.
    - `py`: The y-coordinate (row index) in the grid where the cell setting starts.
    - `gc`: A pointer to a `grid_cell` structure that contains the attributes (like foreground and background color) for the cells.
    - `s`: A pointer to a string containing the characters to be set in the grid cells.
    - `slen`: The length of the string `s`, indicating how many cells will be set.
- **Control Flow**:
    - The function first checks if the y-coordinate `py` is valid using [`grid_check_y`](#grid_check_y) and returns if it is not.
    - It expands the line at `py` to accommodate the new cells if necessary using [`grid_expand_line`](#grid_expand_line).
    - The function updates the `cellused` count for the line if the new end exceeds the current used cells.
    - A loop iterates over each character in the string `s`, setting each corresponding cell in the grid.
    - For each character, it checks if an extended cell is needed using [`grid_need_extended_cell`](#grid_need_extended_cell).
    - If an extended cell is required, it creates one using [`grid_extended_cell`](#grid_extended_cell) and sets its data; otherwise, it stores the cell data directly using [`grid_store_cell`](#grid_store_cell).
- **Output**:
    - The function does not return a value; it modifies the grid in place by setting the specified cells with the provided character data and attributes.
- **Functions called**:
    - [`grid_check_y`](#grid_check_y)
    - [`grid_expand_line`](#grid_expand_line)
    - [`grid_need_extended_cell`](#grid_need_extended_cell)
    - [`grid_extended_cell`](#grid_extended_cell)
    - [`utf8_build_one`](utf8.c.driver.md#utf8_build_one)
    - [`grid_store_cell`](#grid_store_cell)


---
### grid_clear<!-- {{#callable:grid_clear}} -->
The `grid_clear` function clears a specified rectangular area of a grid, setting the cells to a specified background color.
- **Inputs**:
    - `struct grid *gd`: A pointer to the `grid` structure that represents the grid to be modified.
    - `u_int px`: The x-coordinate of the starting position in the grid from where the clearing begins.
    - `u_int py`: The y-coordinate of the starting position in the grid from where the clearing begins.
    - `u_int nx`: The width of the area to be cleared.
    - `u_int ny`: The height of the area to be cleared.
    - `u_int bg`: The background color to set for the cleared cells.
- **Control Flow**:
    - The function first checks if the width (`nx`) or height (`ny`) of the area to clear is zero, in which case it returns immediately.
    - If the clearing area spans the entire width of the grid at the specified y-coordinate, it calls [`grid_clear_lines`](#grid_clear_lines) to clear the lines directly.
    - It checks if the starting y-coordinate (`py`) and the ending y-coordinate (`py + ny - 1`) are within valid bounds using [`grid_check_y`](#grid_check_y).
    - For each line in the specified range, it retrieves the line data and determines the effective width of the line to clear.
    - If the background color is not the default, it adjusts the width of the area to clear based on the starting x-coordinate and the line's effective width.
    - It expands the line to accommodate the clearing operation and then iterates through the specified area, calling [`grid_clear_cell`](#grid_clear_cell) to clear each cell with the specified background color.
- **Output**:
    - The function does not return a value; it modifies the grid in place by clearing the specified area.
- **Functions called**:
    - [`grid_clear_lines`](#grid_clear_lines)
    - [`grid_check_y`](#grid_check_y)
    - [`grid_expand_line`](#grid_expand_line)
    - [`grid_clear_cell`](#grid_clear_cell)


---
### grid_clear_lines<!-- {{#callable:grid_clear_lines}} -->
The `grid_clear_lines` function clears a specified number of lines in a grid by freeing their resources and resetting their content.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the grid to be modified.
    - `py`: An unsigned integer representing the starting line index from which to clear lines.
    - `ny`: An unsigned integer indicating the number of lines to clear.
    - `bg`: An unsigned integer representing the background color to set for the cleared lines.
- **Control Flow**:
    - The function first checks if `ny` is zero; if so, it returns immediately without making any changes.
    - It then verifies that the starting line index `py` and the ending line index `py + ny - 1` are within valid bounds using the [`grid_check_y`](#grid_check_y) function.
    - If the checks pass, it enters a loop that iterates from `py` to `py + ny`, clearing each line by calling [`grid_free_line`](#grid_free_line) to free the line's resources and [`grid_empty_line`](#grid_empty_line) to reset the line's content with the specified background color.
    - Finally, if `py` is not zero, it updates the flags of the line above the cleared lines to indicate that it is no longer wrapped.
- **Output**:
    - The function does not return a value; it modifies the grid in place by clearing the specified lines and setting their background color.
- **Functions called**:
    - [`grid_check_y`](#grid_check_y)
    - [`grid_free_line`](#grid_free_line)
    - [`grid_empty_line`](#grid_empty_line)


---
### grid_move_lines<!-- {{#callable:grid_move_lines}} -->
Moves a specified number of lines within a grid to a new position.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the grid containing the lines.
    - `dy`: The destination y-coordinate where the lines should be moved.
    - `py`: The starting y-coordinate of the lines to be moved.
    - `ny`: The number of lines to move.
    - `bg`: The background color to use when clearing lines that are moved.
- **Control Flow**:
    - Checks if the number of lines to move (`ny`) is zero or if the source and destination coordinates are the same, in which case it returns immediately.
    - Validates the y-coordinates (`py`, `dy`, and the range for `ny`) using the [`grid_check_y`](#grid_check_y) function to ensure they are within the grid's bounds.
    - Frees any lines at the destination that will be replaced by the moved lines, skipping lines that are part of the move.
    - Clears the flags for the line above the destination if it exists.
    - Moves the specified lines from the source position to the destination using `memmove`.
    - Clears the content of the lines that have been moved without freeing them, using the specified background color.
    - Clears the flags for the line above the original starting position if it exists and is not part of the moved lines.
- **Output**:
    - The function does not return a value; it modifies the grid in place by moving lines and clearing content as necessary.
- **Functions called**:
    - [`grid_check_y`](#grid_check_y)
    - [`grid_free_line`](#grid_free_line)
    - [`grid_empty_line`](#grid_empty_line)


---
### grid_move_cells<!-- {{#callable:grid_move_cells}} -->
Moves a specified number of cells within a grid from one position to another.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the grid containing the cells.
    - `dx`: The destination x-coordinate where the cells will be moved.
    - `px`: The source x-coordinate from where the cells will be moved.
    - `py`: The y-coordinate of the row in the grid where the cells are located.
    - `nx`: The number of cells to move from the source position.
    - `bg`: The background color to use for clearing cells that have been moved.
- **Control Flow**:
    - Checks if the number of cells to move (`nx`) is zero or if the source and destination positions are the same (`px == dx`), in which case the function returns immediately.
    - Validates the y-coordinate (`py`) to ensure it is within the grid's bounds using [`grid_check_y`](#grid_check_y).
    - Retrieves the line data for the specified y-coordinate (`py`) from the grid.
    - Expands the grid line to accommodate the new positions for both the source and destination cells using [`grid_expand_line`](#grid_expand_line).
    - Moves the specified cells from the source position (`px`) to the destination position (`dx`) using `memmove`.
    - Updates the `cellused` count for the line if the new destination exceeds the current used cells.
    - Iterates over the moved cells to clear any cells that were moved but are not part of the new position, using [`grid_clear_cell`](#grid_clear_cell).
- **Output**:
    - The function does not return a value; it modifies the grid in place by moving the specified cells and clearing the appropriate cells.
- **Functions called**:
    - [`grid_check_y`](#grid_check_y)
    - [`grid_expand_line`](#grid_expand_line)
    - [`grid_clear_cell`](#grid_clear_cell)


---
### grid_string_cells_fg<!-- {{#callable:grid_string_cells_fg}} -->
The `grid_string_cells_fg` function generates ANSI escape codes for the foreground color of a grid cell.
- **Inputs**:
    - `gc`: A pointer to a `struct grid_cell` that contains the foreground color information.
    - `values`: An integer array where the generated ANSI escape codes will be stored.
- **Control Flow**:
    - Initializes a counter `n` to zero to track the number of values added to the `values` array.
    - Checks if the foreground color (`fg`) in `gc` uses 256 colors; if so, it adds the appropriate ANSI codes for 256 colors to `values`.
    - If the foreground color uses RGB, it splits the RGB value into its red, green, and blue components and adds the corresponding ANSI codes.
    - If the foreground color is a standard color (0-7 or 90-97), it adds the corresponding ANSI code based on the color value.
- **Output**:
    - Returns the number of ANSI codes added to the `values` array.
- **Functions called**:
    - [`colour_split_rgb`](colour.c.driver.md#colour_split_rgb)


---
### grid_string_cells_bg<!-- {{#callable:grid_string_cells_bg}} -->
Converts the background color of a grid cell into ANSI escape codes.
- **Inputs**:
    - `gc`: A pointer to a `struct grid_cell` that contains the background color information.
    - `values`: An integer array where the ANSI escape codes will be stored.
- **Control Flow**:
    - Initializes a counter `n` to zero to track the number of escape codes generated.
    - Checks if the background color (`gc->bg`) uses 256 colors by checking the `COLOUR_FLAG_256` flag.
    - If it does, it adds the appropriate ANSI codes for 256 colors to the `values` array.
    - If the background color uses RGB values (checked via `COLOUR_FLAG_RGB`), it splits the RGB values and adds the corresponding ANSI codes.
    - If the background color is a standard color (0-7 or 90-97), it adds the corresponding ANSI codes based on the color value.
    - Finally, it returns the count of escape codes generated.
- **Output**:
    - Returns the number of ANSI escape codes added to the `values` array.
- **Functions called**:
    - [`colour_split_rgb`](colour.c.driver.md#colour_split_rgb)


---
### grid_string_cells_us<!-- {{#callable:grid_string_cells_us}} -->
The `grid_string_cells_us` function generates ANSI escape codes for the underline color of a grid cell.
- **Inputs**:
    - `gc`: A pointer to a `struct grid_cell` that contains color information for the cell.
    - `values`: An integer array where the generated ANSI escape codes will be stored.
- **Control Flow**:
    - The function initializes a size_t variable `n` to 0 to keep track of the number of codes generated.
    - It checks if the `us` field of the `grid_cell` structure has the `COLOUR_FLAG_256` flag set.
    - If the `COLOUR_FLAG_256` is set, it adds the ANSI code for 256 colors to the `values` array.
    - If the `COLOUR_FLAG_RGB` is set instead, it adds the ANSI code for RGB colors and splits the RGB values into separate variables.
    - Finally, it returns the count of the number of codes added to the `values` array.
- **Output**:
    - The function returns the number of ANSI escape codes generated and stored in the `values` array.
- **Functions called**:
    - [`colour_split_rgb`](colour.c.driver.md#colour_split_rgb)


---
### grid_string_cells_add_code<!-- {{#callable:grid_string_cells_add_code}} -->
The `grid_string_cells_add_code` function appends ANSI escape codes for color and attributes to a buffer based on the provided parameters.
- **Inputs**:
    - `buf`: A pointer to a character buffer where the ANSI escape codes will be appended.
    - `len`: The maximum length of the buffer to prevent overflow.
    - `n`: The number of new attributes or colors to process.
    - `s`: An array of integers representing the current state of attributes or colors.
    - `newc`: An array of integers representing the new colors or attributes to be added.
    - `oldc`: An array of integers representing the old colors or attributes for comparison.
    - `nnewc`: The number of new colors or attributes in the `newc` array.
    - `noldc`: The number of old colors or attributes in the `oldc` array.
    - `flags`: Flags that determine how the function behaves, such as whether to use escape sequences.
- **Control Flow**:
    - The function first checks if there are new colors to add; if not, it returns immediately.
    - It checks if a reset is needed based on the current state and the new colors; if no changes are detected, it returns.
    - If a reset is required and the new color is a default color, it returns without making changes.
    - The function then appends the appropriate escape sequence prefix to the buffer based on the flags.
    - It iterates over the new colors, formatting them into the buffer as ANSI codes, separating them with semicolons.
    - Finally, it appends 'm' to the buffer to complete the ANSI sequence.
- **Output**:
    - The function does not return a value; instead, it modifies the provided buffer to include the ANSI escape codes for the specified colors and attributes.
- **Functions called**:
    - [`strlcat`](compat/strlcat.c.driver.md#strlcat)
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)


---
### grid_string_cells_add_hyperlink<!-- {{#callable:grid_string_cells_add_hyperlink}} -->
Adds a hyperlink to a string buffer in a specific format.
- **Inputs**:
    - `buf`: A pointer to the character buffer where the hyperlink will be added.
    - `len`: The maximum length of the buffer to prevent overflow.
    - `id`: An optional identifier for the hyperlink, can be an empty string.
    - `uri`: The URI that the hyperlink points to.
    - `flags`: Flags that determine how the hyperlink is formatted, such as whether to escape sequences.
- **Control Flow**:
    - Checks if the combined length of the URI, ID, and additional formatting exceeds the buffer length; if so, returns 0.
    - Appends the appropriate escape sequence for hyperlinks based on the flags provided.
    - If the ID is not empty, it formats and appends the ID to the buffer.
    - Appends the URI to the buffer.
    - Appends the closing escape sequence for the hyperlink based on the flags provided.
    - Returns 1 to indicate success.
- **Output**:
    - Returns 1 on success, indicating that the hyperlink was successfully added to the buffer; returns 0 if the buffer length would be exceeded.
- **Functions called**:
    - [`strlcat`](compat/strlcat.c.driver.md#strlcat)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### grid_string_cells_code<!-- {{#callable:grid_string_cells_code}} -->
The `grid_string_cells_code` function generates ANSI escape codes for grid cell attributes and colors based on the current and previous cell states.
- **Inputs**:
    - `lastgc`: Pointer to the previous `grid_cell` structure representing the last cell's attributes.
    - `gc`: Pointer to the current `grid_cell` structure representing the current cell's attributes.
    - `buf`: Pointer to a character buffer where the generated ANSI codes will be stored.
    - `len`: Size of the buffer `buf` to prevent overflow.
    - `flags`: Integer flags that modify the behavior of the function, such as whether to escape sequences.
    - `sc`: Pointer to a `screen` structure that may contain hyperlink information.
    - `has_link`: Pointer to an integer that indicates whether a hyperlink is present.
- **Control Flow**:
    - The function initializes arrays to hold attribute codes and counts for old and new colors.
    - It checks for removed attributes and resets the attribute state if necessary.
    - It iterates through defined attributes to determine which new attributes are set and adds their corresponding codes.
    - The function constructs the ANSI escape sequence for the attributes and appends it to the buffer.
    - It checks for changes in foreground, background, and underscore colors, appending their respective codes to the buffer.
    - It handles shift in and shift out sequences based on character set attributes.
    - Finally, it checks for hyperlink changes and adds the appropriate hyperlink code if necessary.
- **Output**:
    - The function does not return a value but modifies the `buf` parameter to contain the generated ANSI escape codes, which can be used to format text in a terminal.
- **Functions called**:
    - [`strlcat`](compat/strlcat.c.driver.md#strlcat)
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)
    - [`grid_string_cells_fg`](#grid_string_cells_fg)
    - [`grid_string_cells_add_code`](#grid_string_cells_add_code)
    - [`grid_string_cells_bg`](#grid_string_cells_bg)
    - [`grid_string_cells_us`](#grid_string_cells_us)
    - [`hyperlinks_get`](hyperlinks.c.driver.md#hyperlinks_get)
    - [`grid_string_cells_add_hyperlink`](#grid_string_cells_add_hyperlink)


---
### grid_string_cells<!-- {{#callable:grid_string_cells}} -->
Converts a range of grid cells into a formatted string representation.
- **Inputs**:
    - `gd`: Pointer to the `grid` structure representing the grid of cells.
    - `px`: The starting x-coordinate (column) of the cells to convert.
    - `py`: The y-coordinate (row) of the cells to convert.
    - `nx`: The number of cells to convert starting from (px, py).
    - `lastgc`: Pointer to a pointer of `grid_cell` to hold the last cell's state.
    - `flags`: Integer flags that modify the behavior of the function.
    - `s`: Pointer to a `screen` structure for handling screen-related operations.
- **Control Flow**:
    - Check if `lastgc` is NULL and initialize it with a default cell if necessary.
    - Allocate an initial buffer for the output string.
    - Peek the line at the specified y-coordinate to determine the range of cells to process.
    - Iterate over the specified range of cells, retrieving each cell's data.
    - Skip cells marked as padding and handle special cases for tabs and escape sequences.
    - Dynamically resize the output buffer as needed to accommodate the formatted string.
    - If hyperlinks are detected, append the hyperlink information to the output string.
    - Trim trailing spaces from the output string if the corresponding flag is set.
    - Null-terminate the output string and return it.
- **Output**:
    - Returns a dynamically allocated string containing the formatted representation of the specified grid cells.
- **Functions called**:
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`grid_peek_line`](#grid_peek_line)
    - [`grid_get_cell`](#grid_get_cell)
    - [`grid_string_cells_code`](#grid_string_cells_code)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`grid_string_cells_add_hyperlink`](#grid_string_cells_add_hyperlink)


---
### grid_duplicate_lines<!-- {{#callable:grid_duplicate_lines}} -->
Duplicates a specified number of lines from a source grid to a destination grid.
- **Inputs**:
    - `dst`: A pointer to the destination `grid` structure where lines will be copied.
    - `dy`: The starting line index in the destination grid where the lines will be duplicated.
    - `src`: A pointer to the source `grid` structure from which lines will be copied.
    - `sy`: The starting line index in the source grid from which lines will be copied.
    - `ny`: The number of lines to duplicate from the source grid.
- **Control Flow**:
    - Checks if the number of lines to duplicate exceeds the available space in the destination grid and adjusts `ny` accordingly.
    - Checks if the number of lines to duplicate exceeds the available lines in the source grid and adjusts `ny` accordingly.
    - Calls [`grid_free_lines`](#grid_free_lines) to free any existing lines in the destination grid at the specified starting index.
    - Iterates over the number of lines to duplicate, copying each line from the source grid to the destination grid.
    - Uses `memcpy` to copy the line data and reallocates memory for cell data and extended data if necessary.
- **Output**:
    - The function does not return a value; it modifies the destination grid in place by duplicating lines from the source grid.
- **Functions called**:
    - [`grid_free_lines`](#grid_free_lines)
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)


---
### grid_reflow_dead<!-- {{#callable:grid_reflow_dead}} -->
The `grid_reflow_dead` function marks a grid line as dead by zeroing its contents and setting its flags.
- **Inputs**:
    - `gl`: A pointer to a `struct grid_line` that represents the grid line to be marked as dead.
- **Control Flow**:
    - The function uses `memset` to set all bytes of the `grid_line` structure pointed to by `gl` to zero.
    - It then sets the `flags` member of the `grid_line` structure to `GRID_LINE_DEAD` to indicate that the line is no longer active.
- **Output**:
    - The function does not return a value; it modifies the input `grid_line` structure in place to indicate it is dead.


---
### grid_reflow_add<!-- {{#callable:grid_reflow_add}} -->
The `grid_reflow_add` function adds a specified number of new lines to a grid and returns a pointer to the first newly added line.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure that represents the grid to which new lines will be added.
    - `n`: An unsigned integer representing the number of new lines to add to the grid.
- **Control Flow**:
    - Calculates the new total number of lines in the grid by adding `n` to the current number of lines (`gd->sy`).
    - Resizes the `linedata` array of the grid using [`xreallocarray`](xmalloc.c.driver.md#xreallocarray) to accommodate the new total number of lines.
    - Obtains a pointer to the first new line by referencing the newly allocated space in `linedata`.
    - Initializes the newly added lines to zero using `memset`.
    - Updates the grid's line count (`gd->sy`) to reflect the addition of the new lines.
- **Output**:
    - Returns a pointer to the first newly added `grid_line` structure in the grid.
- **Functions called**:
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)


---
### grid_reflow_move<!-- {{#callable:grid_reflow_move}} -->
Moves a grid line from one location to another in the grid.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the grid containing the lines.
    - `from`: A pointer to the `grid_line` structure that is to be moved.
- **Control Flow**:
    - Calls [`grid_reflow_add`](#grid_reflow_add) to allocate a new line in the grid and returns a pointer to it.
    - Copies the contents of the `from` line to the newly allocated line using `memcpy`.
    - Calls [`grid_reflow_dead`](#grid_reflow_dead) to mark the `from` line as dead, effectively removing it from the grid.
- **Output**:
    - Returns a pointer to the newly allocated `grid_line` that contains the copied data from the `from` line.
- **Functions called**:
    - [`grid_reflow_add`](#grid_reflow_add)
    - [`grid_reflow_dead`](#grid_reflow_dead)


---
### grid_reflow_join<!-- {{#callable:grid_reflow_join}} -->
The `grid_reflow_join` function merges lines from a source grid into a target grid while respecting line width constraints.
- **Inputs**:
    - `target`: A pointer to the target `grid` structure where lines will be joined.
    - `gd`: A pointer to the source `grid` structure from which lines are being consumed.
    - `sx`: An unsigned integer representing the maximum width of a line in the target grid.
    - `yy`: An unsigned integer indicating the starting line index in the source grid.
    - `width`: An unsigned integer representing the current width of the line being constructed in the target grid.
    - `already`: An integer flag indicating whether the target line has already been initialized.
- **Control Flow**:
    - If `already` is false, a new line is added to the target grid by moving the specified line from the source grid.
    - The function enters a loop to consume lines from the source grid until either the target line is full or there are no more lines to consume.
    - Within the loop, it checks if the next line in the source grid is empty or if it has been wrapped, adjusting the flow accordingly.
    - If the next line is not empty and can fit within the target line's width, it copies cells from the source line to the target line.
    - If the target line becomes full or if the current line is not wrapped, the loop breaks.
    - After processing, if any cells remain in the last consumed line, they are moved back to the source grid, and the target line's wrap flag is adjusted if necessary.
    - Finally, any completely consumed lines from the source grid are freed, and the scroll position of the source grid is adjusted.
- **Output**:
    - The function does not return a value; it modifies the target grid in place by adding cells from the source grid.
- **Functions called**:
    - [`grid_reflow_move`](#grid_reflow_move)
    - [`grid_get_cell1`](#grid_get_cell1)
    - [`grid_set_cell`](#grid_set_cell)
    - [`grid_move_cells`](#grid_move_cells)
    - [`grid_reflow_dead`](#grid_reflow_dead)


---
### grid_reflow_split<!-- {{#callable:grid_reflow_split}} -->
Splits a specified line in a grid into multiple lines based on a given width.
- **Inputs**:
    - `struct grid *target`: Pointer to the target grid where the new lines will be added.
    - `struct grid *gd`: Pointer to the source grid from which the line will be split.
    - `u_int sx`: The maximum width of each new line after the split.
    - `u_int yy`: The index of the line in the source grid that is to be split.
    - `u_int at`: The index in the source line from which to start copying cells.
- **Control Flow**:
    - Checks if the line is extended or not to determine how many new lines are needed.
    - Calculates the number of lines required based on the width of the cells and the specified maximum width.
    - Inserts new lines into the target grid.
    - Copies cells from the original line to the new lines in the target grid, wrapping cells as necessary.
    - Updates the original line's properties to reflect the split and marks it as wrapped if applicable.
    - Adjusts the scroll position of the grid if necessary.
    - Attempts to join the last new line with the next line if there is space and the original line was wrapped.
- **Output**:
    - The function does not return a value; it modifies the target grid in place by adding new lines and adjusting the original line.
- **Functions called**:
    - [`grid_get_cell1`](#grid_get_cell1)
    - [`grid_reflow_add`](#grid_reflow_add)
    - [`grid_set_cell`](#grid_set_cell)
    - [`grid_reflow_dead`](#grid_reflow_dead)
    - [`grid_reflow_join`](#grid_reflow_join)


---
### grid_reflow<!-- {{#callable:grid_reflow}} -->
The `grid_reflow` function adjusts the layout of a grid's lines to fit a specified width.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure that contains the current grid data.
    - `sx`: An unsigned integer representing the new width to which the grid lines should be reflowed.
- **Control Flow**:
    - Creates a new target grid to hold the reflowed lines.
    - Iterates over each line in the source grid.
    - Checks if the line is marked as dead and skips it if so.
    - Calculates the width of the current line and determines if it needs to be moved, split, or joined with the next line based on the new width.
    - If the line's width matches the new width, it is moved directly to the target grid.
    - If the line's width exceeds the new width, it is split into multiple lines.
    - If the line is shorter than the new width and was previously wrapped, it attempts to join with the next line.
    - Finally, it replaces the old grid data with the new reflowed data.
- **Output**:
    - The function does not return a value but modifies the original grid structure to reflect the new layout based on the specified width.
- **Functions called**:
    - [`grid_create`](#grid_create)
    - [`grid_get_cell1`](#grid_get_cell1)
    - [`grid_reflow_move`](#grid_reflow_move)
    - [`grid_reflow_split`](#grid_reflow_split)
    - [`grid_reflow_join`](#grid_reflow_join)
    - [`grid_reflow_add`](#grid_reflow_add)


---
### grid_wrap_position<!-- {{#callable:grid_wrap_position}} -->
Calculates the wrapped grid position based on the given pixel coordinates.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the grid data.
    - `px`: An unsigned integer representing the pixel x-coordinate.
    - `py`: An unsigned integer representing the pixel y-coordinate.
    - `wx`: A pointer to an unsigned integer where the wrapped x-coordinate will be stored.
    - `wy`: A pointer to an unsigned integer where the wrapped y-coordinate will be stored.
- **Control Flow**:
    - Initializes two variables `ax` and `ay` to track the wrapped x and y positions.
    - Iterates through each line in the grid up to the specified y-coordinate (`py`).
    - For each line, checks if it is wrapped using the `GRID_LINE_WRAPPED` flag.
    - If the line is wrapped, adds the number of used cells in that line to `ax`.
    - If the line is not wrapped, resets `ax` to 0 and increments `ay` to move to the next line.
    - After the loop, checks if the pixel x-coordinate (`px`) exceeds the number of used cells in the current line.
    - If it does, sets `ax` to `UINT_MAX`, otherwise adds `px` to `ax`.
    - Finally, assigns the calculated `ax` and `ay` values to the output pointers `wx` and `wy`.
- **Output**:
    - The function does not return a value but updates the output parameters `wx` and `wy` with the calculated wrapped x and y coordinates.


---
### grid_unwrap_position<!-- {{#callable:grid_unwrap_position}} -->
The `grid_unwrap_position` function converts a wrapped grid position back to its original unwrapped coordinates.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the grid data.
    - `px`: A pointer to an unsigned integer where the unwrapped x-coordinate will be stored.
    - `py`: A pointer to an unsigned integer where the unwrapped y-coordinate will be stored.
    - `wx`: The wrapped x-coordinate to be converted.
    - `wy`: The wrapped y-coordinate to be converted.
- **Control Flow**:
    - The function initializes a loop to iterate through the grid lines until it finds the line corresponding to the provided wrapped y-coordinate (`wy`).
    - It increments a counter (`ay`) for each unwrapped line until it reaches the desired wrapped line.
    - If the wrapped x-coordinate (`wx`) is `UINT_MAX`, it finds the end of the wrapped lines and sets `wx` to the cell used in the next unwrapped line.
    - If `wx` is not `UINT_MAX`, it decrements `wx` by the number of cells used in each wrapped line until it finds the correct unwrapped line.
    - Finally, it assigns the calculated unwrapped x and y coordinates to the provided pointers.
- **Output**:
    - The function does not return a value but updates the pointers `px` and `py` with the unwrapped x and y coordinates, respectively.


---
### grid_line_length<!-- {{#callable:grid_line_length}} -->
Calculates the length of a line in a grid, trimming trailing padding.
- **Inputs**:
    - `gd`: A pointer to a `grid` structure representing the grid containing the line.
    - `py`: An unsigned integer representing the y-coordinate (line number) in the grid.
- **Control Flow**:
    - Retrieve the cell size of the specified line using [`grid_get_line`](#grid_get_line).
    - If the cell size exceeds the grid's width (`gd->sx`), set it to the grid's width.
    - Iterate backwards through the cells of the line, checking each cell for padding or non-space characters.
    - If a cell is found that is not padding or is not a single space, break the loop.
    - Return the adjusted length of the line.
- **Output**:
    - Returns an unsigned integer representing the length of the line, excluding any trailing padding.
- **Functions called**:
    - [`grid_get_line`](#grid_get_line)
    - [`grid_get_cell`](#grid_get_cell)


---
### grid_in_set<!-- {{#callable:grid_in_set}} -->
The `grid_in_set` function checks if a specific grid cell contains a character from a given set.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the grid.
    - `px`: An unsigned integer representing the x-coordinate of the cell in the grid.
    - `py`: An unsigned integer representing the y-coordinate of the cell in the grid.
    - `set`: A pointer to a string containing the set of characters to check against.
- **Control Flow**:
    - The function retrieves the grid cell at the specified coordinates (px, py) into the `gc` structure.
    - If the `set` contains a tab character, it checks if the cell is a padding cell.
    - If it is a padding cell, it moves left until it finds a non-padding cell, checking if that cell is a tab.
    - If the found cell is a tab, it calculates and returns the width of the tab based on the distance moved.
    - If the current cell is a tab, it returns its width directly.
    - If the current cell is a padding cell, it returns 0.
    - Finally, it checks if the character in the current cell is present in the `set` using [`utf8_cstrhas`](utf8.c.driver.md#utf8_cstrhas) and returns the result.
- **Output**:
    - The function returns an integer indicating the width of a tab if found, 0 if the cell is padding, or 1 if the character in the cell is found in the set, otherwise it returns 0.
- **Functions called**:
    - [`grid_get_cell`](#grid_get_cell)
    - [`utf8_cstrhas`](utf8.c.driver.md#utf8_cstrhas)


