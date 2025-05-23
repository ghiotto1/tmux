# Purpose
This C source code file is part of the `tmux` project, specifically handling the grid data structure that represents the content displayed on the screen. The grid is a fundamental component in `tmux`, as it manages the text and attributes that are shown in terminal windows. The code defines a grid as a collection of lines, each consisting of cells that store character data and associated attributes like foreground and background colors, and text styles. The grid is divided into two parts: history and viewable data, allowing for efficient management of terminal content, including scrolling and history tracking.

The file provides a comprehensive set of functions to manipulate the grid, including creating and destroying grids, adjusting line numbers, storing and retrieving cell data, and handling extended cells for complex character attributes. It also includes functions for managing the grid's history, such as scrolling, trimming, and collecting history lines. Additionally, the code supports operations like clearing, moving, and comparing grid lines, as well as converting grid content into strings with ANSI escape sequences for rendering. The file is integral to `tmux`'s functionality, enabling dynamic and flexible terminal window management.
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
- **Description**: The `grid_default_cell` is a constant structure of type `grid_cell` that defines the default properties of a cell in a grid. It is initialized with specific attributes such as a space character, default colors, and no special flags. This structure is used to represent the default state of a cell in the grid data structure.
- **Use**: This variable is used to initialize or reset grid cells to a default state when creating or clearing grid lines.


---
### grid_padding_cell
- **Type**: `struct grid_cell`
- **Description**: The `grid_padding_cell` is a static constant instance of the `struct grid_cell` structure, which represents a padding cell in a grid. Padding cells are unique in that they are zero-width and are always considered extended cells.
- **Use**: This variable is used to define and represent padding cells within a grid, which are necessary for handling zero-width cells in the grid structure.


---
### grid_cleared_cell
- **Type**: ``struct grid_cell``
- **Description**: The `grid_cleared_cell` is a static constant instance of the `struct grid_cell` data structure. It represents a cleared cell in a grid, initialized with a space character, default attributes, and a specific flag indicating it is cleared. The cell has default color values for foreground, background, and underscore.
- **Use**: This variable is used to represent and initialize cleared cells within a grid structure, typically when resetting or clearing parts of the grid.


---
### grid_cleared_entry
- **Type**: `struct grid_cell_entry`
- **Description**: The `grid_cleared_entry` is a static constant instance of the `struct grid_cell_entry` structure. It is initialized with a data array containing values `{0, 8, 8, ' '}` and a flag `GRID_FLAG_CLEARED`. This structure represents a cleared grid cell entry with default attributes.
- **Use**: This variable is used to represent a cleared cell in the grid, typically for resetting or initializing grid cells to a default cleared state.


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
    - `gce`: A pointer to a `struct grid_cell_entry` representing the cell entry to be checked.
    - `gc`: A pointer to a `struct grid_cell` representing the cell attributes to be evaluated.
- **Control Flow**:
    - Checks if the `flags` of `gce` indicate it is already extended.
    - Evaluates if the `attr` of `gc` exceeds 0xff.
    - Checks if the `data.size` or `data.width` of `gc` is greater than 1.
    - Determines if the foreground or background color flags indicate RGB usage.
    - Validates if the `us` attribute of `gc` is not equal to 8.
    - Checks if the `link` attribute of `gc` is non-zero.
    - Checks if the `flags` of `gc` indicate it is a tab.
- **Output**:
    - Returns 1 if the cell should be treated as extended, otherwise returns 0.


---
### grid_get_extended_cell<!-- {{#callable:grid_get_extended_cell}} -->
The `grid_get_extended_cell` function allocates and initializes an extended cell in a grid line.
- **Inputs**:
    - `gl`: A pointer to a `struct grid_line` representing the grid line where the extended cell will be added.
    - `gce`: A pointer to a `struct grid_cell_entry` that will hold the information about the newly created extended cell.
    - `flags`: An integer representing flags that will be combined with the `GRID_FLAG_EXTENDED` flag for the new cell.
- **Control Flow**:
    - Calculates the new size for the extended data array by incrementing the current size (`gl->extdsize`) by one.
    - Resizes the `extddata` array in the `grid_line` structure using `xreallocarray` to accommodate the new extended cell.
    - Updates the `extdsize` field of the `grid_line` structure to reflect the new size.
    - Sets the `offset` field of the `grid_cell_entry` to the index of the newly added extended cell (which is the previous size of the array).
    - Combines the provided `flags` with `GRID_FLAG_EXTENDED` and assigns it to the `flags` field of the `grid_cell_entry`.
- **Output**:
    - The function does not return a value; it modifies the `grid_cell_entry` to reflect the new extended cell's offset and flags.


---
### grid_extended_cell<!-- {{#callable:grid_extended_cell}} -->
The `grid_extended_cell` function retrieves or creates an extended grid cell entry based on the provided grid line and cell entry.
- **Inputs**:
    - `gl`: A pointer to a `struct grid_line` representing the grid line where the cell is located.
    - `gce`: A pointer to a `struct grid_cell_entry` that contains information about the cell entry.
    - `gc`: A pointer to a `struct grid_cell` that holds the cell's data and attributes.
- **Control Flow**:
    - Checks if the `gce` flags indicate that it is not an extended cell; if so, it calls [`grid_get_extended_cell`](#grid_get_extended_cell) to create one.
    - If the `gce` offset is out of bounds, it triggers a fatal error.
    - Sets the `GRID_LINE_EXTENDED` flag on the grid line.
    - Determines the UTF-8 character to store based on the flags of the `gc` structure.
    - Populates the extended entry with data from the `gc` structure and returns a pointer to the extended entry.
- **Output**:
    - Returns a pointer to the `struct grid_extd_entry` that contains the extended cell data.
- **Functions called**:
    - [`grid_get_extended_cell`](#grid_get_extended_cell)


---
### grid_compact_line<!-- {{#callable:grid_compact_line}} -->
The `grid_compact_line` function reduces the size of the extended data array in a grid line by removing unused entries.
- **Inputs**:
    - `gl`: A pointer to a `struct grid_line` which contains the cell data and extended data to be compacted.
- **Control Flow**:
    - Checks if the `extdsize` of the grid line is zero; if so, it returns immediately as there is nothing to compact.
    - Iterates through each cell in the grid line to count how many cells have the `GRID_FLAG_EXTENDED` flag set, updating `new_extdsize` accordingly.
    - If no extended cells are found, it frees the existing extended data and sets `extddata` to NULL and `extdsize` to zero before returning.
    - Allocates a new array for the extended data of size `new_extdsize`.
    - Copies the data of the extended cells into the new array, updating their offsets in the process.
    - Frees the old extended data array and updates the grid line's `extddata` and `extdsize` to the new values.
- **Output**:
    - The function does not return a value; it modifies the `extddata` and `extdsize` fields of the provided `grid_line` structure in place.


---
### grid_get_line<!-- {{#callable:grid_get_line}} -->
The `grid_get_line` function retrieves a pointer to a specific line of grid data.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` which contains the grid data.
    - `line`: An unsigned integer representing the index of the line to retrieve.
- **Control Flow**:
    - The function directly accesses the `linedata` array of the `grid` structure using the provided line index.
    - It returns a pointer to the `grid_line` structure located at the specified index.
- **Output**:
    - Returns a pointer to the `grid_line` structure corresponding to the specified line index in the grid.


---
### grid_adjust_lines<!-- {{#callable:grid_adjust_lines}} -->
The `grid_adjust_lines` function reallocates the memory for the line data in a grid structure to accommodate a specified number of lines.
- **Inputs**:
    - `gd`: A pointer to a `grid` structure that contains the grid's line data.
    - `lines`: An unsigned integer representing the new number of lines to allocate for the grid.
- **Control Flow**:
    - The function calls `xreallocarray` to resize the `linedata` array in the `grid` structure.
    - The new size is determined by the `lines` parameter and the size of each line data structure.
- **Output**:
    - The function does not return a value; it modifies the `linedata` pointer in the `grid` structure to point to the newly allocated memory.


---
### grid_clear_cell<!-- {{#callable:grid_clear_cell}} -->
Clears a specific cell in a grid and optionally sets its background color.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the grid.
    - `px`: An unsigned integer representing the x-coordinate (column) of the cell to clear.
    - `py`: An unsigned integer representing the y-coordinate (row) of the cell to clear.
    - `bg`: An unsigned integer representing the background color to set, or 8 to leave it unchanged.
- **Control Flow**:
    - Accesses the line data for the specified row `py` in the grid.
    - Retrieves the cell entry for the specified column `px` in that line.
    - Copies the default cleared cell data into the specified cell entry.
    - Checks if the provided background color `bg` is not equal to 8 (default).
    - If `bg` indicates an RGB color, retrieves the extended cell and sets its background color.
    - If `bg` indicates a 256-color, sets the appropriate flag and updates the background color of the cell entry.
- **Output**:
    - The function does not return a value; it modifies the specified cell in the grid directly.
- **Functions called**:
    - [`grid_get_extended_cell`](#grid_get_extended_cell)
    - [`grid_extended_cell`](#grid_extended_cell)


---
### grid_check_y<!-- {{#callable:grid_check_y}} -->
Checks if a given y-coordinate is within the valid range of a grid.
- **Inputs**:
    - `gd`: A pointer to a `grid` structure that contains the grid's dimensions and state.
    - `from`: A string representing the source of the call, used for logging purposes.
    - `py`: An unsigned integer representing the y-coordinate to be checked.
- **Control Flow**:
    - The function first checks if the provided y-coordinate `py` is greater than or equal to the sum of `gd->hsize` and `gd->sy`.
    - If the condition is true, it logs a debug message indicating that the y-coordinate is out of range and returns -1.
    - If the condition is false, it returns 0, indicating that the y-coordinate is valid.
- **Output**:
    - Returns -1 if the y-coordinate is out of range; otherwise, returns 0.


---
### grid_cells_look_equal<!-- {{#callable:grid_cells_look_equal}} -->
The `grid_cells_look_equal` function checks if two grid cells visually appear the same.
- **Inputs**:
    - `gc1`: A pointer to the first `grid_cell` structure to compare.
    - `gc2`: A pointer to the second `grid_cell` structure to compare.
- **Control Flow**:
    - The function retrieves the `flags` of both grid cells.
    - It first checks if the foreground (`fg`) and background (`bg`) colors of the two cells are different; if so, it returns 0 (not equal).
    - Next, it checks if the attributes (`attr`) of the two cells are different; if so, it returns 0.
    - Then, it compares the flags of both cells, ignoring the `GRID_FLAG_CLEARED` flag; if they differ, it returns 0.
    - Finally, it checks if the `link` values of both cells are different; if so, it returns 0.
    - If all checks pass, it returns 1 (the cells look equal).
- **Output**:
    - Returns 1 if the two grid cells look equal, otherwise returns 0.


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
    - Sets the `flags` of the `grid_cell` to include the `GRID_FLAG_TAB` flag, indicating that this cell is a tab.
    - Assigns the specified `width` to the `width`, `size`, and `have` fields of the `data` structure within the `grid_cell`.
    - Fills the `data` field of the `grid_cell` with spaces (' ') up to the specified `size`.
- **Output**:
    - The function does not return a value; it modifies the `grid_cell` pointed to by `gc` in place to represent a tab with the specified width.


---
### grid_free_line<!-- {{#callable:grid_free_line}} -->
The `grid_free_line` function frees the memory allocated for the cell data and extended data of a specified line in a grid.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` that represents the grid containing the line to be freed.
    - `py`: An unsigned integer representing the index of the line in the grid that is to be freed.
- **Control Flow**:
    - The function first calls `free` on the `celldata` of the specified line, effectively deallocating the memory used for the cell data.
    - It then sets the `celldata` pointer to NULL to avoid dangling references.
    - Next, it calls `free` on the `extddata` of the specified line to deallocate the memory used for any extended data.
    - Finally, it sets the `extddata` pointer to NULL.
- **Output**:
    - The function does not return a value; it modifies the grid by freeing memory associated with the specified line.


---
### grid_free_lines<!-- {{#callable:grid_free_lines}} -->
The `grid_free_lines` function frees a specified number of lines in a grid starting from a given line index.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the grid from which lines will be freed.
    - `py`: An unsigned integer representing the starting line index from which to begin freeing lines.
    - `ny`: An unsigned integer representing the number of lines to free starting from the line index `py`.
- **Control Flow**:
    - A loop iterates from the starting line index `py` to `py + ny`, effectively covering the range of lines to be freed.
    - Within the loop, the [`grid_free_line`](#grid_free_line) function is called for each line index to free the resources associated with that line.
- **Output**:
    - The function does not return a value; it performs the operation of freeing memory associated with the specified lines in the grid.
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
    - Allocates memory for a new `grid` structure using `xmalloc`.
    - Sets the `sx` and `sy` fields of the grid to the provided dimensions.
    - Checks if `hlimit` is non-zero to set the `flags` field to indicate history is enabled.
    - Initializes `hscrolled`, `hsize`, and `hlimit` fields of the grid.
    - If `sy` is non-zero, allocates memory for `linedata` to hold the grid lines; otherwise, sets it to NULL.
    - Returns the pointer to the newly created grid.
- **Output**:
    - Returns a pointer to the newly created `grid` structure.


---
### grid_destroy<!-- {{#callable:grid_destroy}} -->
The `grid_destroy` function deallocates memory associated with a `grid` structure.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` that represents the grid to be destroyed.
- **Control Flow**:
    - Calls [`grid_free_lines`](#grid_free_lines) to free all lines in the grid from the first line up to the total height of the grid plus the screen height.
    - Frees the memory allocated for the `linedata` array that holds the grid lines.
    - Finally, frees the memory allocated for the `grid` structure itself.
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
    - Checks if the dimensions (sx and sy) of both grids are equal; if not, returns 1 indicating they are different.
    - Iterates over each line of the grids, comparing the cell sizes of corresponding lines; if they differ, returns 1.
    - For each cell in the lines, retrieves the cell data from both grids and compares them using [`grid_cells_equal`](#grid_cells_equal); if any cells differ, returns 1.
    - If all checks pass, returns 0 indicating the grids are equal.
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
    - `ny`: An unsigned integer representing the number of lines to be trimmed from the history.
- **Control Flow**:
    - Calls [`grid_free_lines`](#grid_free_lines) to free the specified number of lines from the top of the grid's history.
    - Uses `memmove` to shift the remaining lines in the grid up by `ny` positions, effectively removing the freed lines from the history.
- **Output**:
    - The function does not return a value; it modifies the grid in place by adjusting its history.
- **Functions called**:
    - [`grid_free_lines`](#grid_free_lines)


---
### grid_collect_history<!-- {{#callable:grid_collect_history}} -->
The `grid_collect_history` function manages the history of a grid by freeing the oldest lines when the history size exceeds a specified limit.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` that represents the grid whose history is being managed.
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
Removes a specified number of lines from the bottom of the grid's history.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the grid from which lines will be removed.
    - `ny`: An unsigned integer representing the number of lines to remove from the history.
- **Control Flow**:
    - Checks if the number of lines to remove (`ny`) exceeds the current history size (`gd->hsize`); if so, the function returns immediately without making any changes.
    - Iterates from 0 to `ny`, calling [`grid_free_line`](#grid_free_line) for each line to be removed, specifically targeting the lines at the bottom of the history.
    - Decreases the history size (`gd->hsize`) by the number of lines removed.
- **Output**:
    - The function does not return a value; it modifies the `grid` structure in place by freeing the specified lines from the history.
- **Functions called**:
    - [`grid_free_line`](#grid_free_line)


---
### grid_scroll_history<!-- {{#callable:grid_scroll_history}} -->
The `grid_scroll_history` function scrolls the visible grid history by adding a new line to the history.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` representing the grid data structure.
    - `bg`: An unsigned integer representing the background color to be used for the new line.
- **Control Flow**:
    - Calculates the new line index `yy` as the sum of the current history size `hsize` and the visible screen size `sy`.
    - Resizes the `linedata` array to accommodate the new line by using `xreallocarray`.
    - Calls [`grid_empty_line`](#grid_empty_line) to initialize the new line at index `yy` with the specified background color.
    - Increments the `hscrolled` counter to indicate that the history has been scrolled.
    - Calls [`grid_compact_line`](#grid_compact_line) to compact the current last line in the history.
    - Sets the `time` field of the new line to the current time and increments the `hsize` to reflect the addition of the new line.
- **Output**:
    - The function does not return a value; it modifies the grid's internal state by adding a new line to the history and updating related properties.
- **Functions called**:
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
    - Sets the `hsize` member of the grid to 0, indicating that the history size is now empty.
    - Reallocates the `linedata` array to match the current screen height (`sy`), effectively clearing the history.
- **Output**:
    - The function does not return a value; it modifies the grid structure in place to clear its history.
- **Functions called**:
    - [`grid_trim_history`](#grid_trim_history)


---
### grid_scroll_history_region<!-- {{#callable:grid_scroll_history_region}} -->
Scrolls a specified region of the grid up by one line, moving the top line into history.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the current grid.
    - `upper`: The upper boundary of the region to scroll, specified as a line index.
    - `lower`: The lower boundary of the region to scroll, specified as a line index.
    - `bg`: The background color to use when clearing the bottom line after scrolling.
- **Control Flow**:
    - Calculates the new size for the `linedata` array to accommodate an additional line.
    - Moves the existing lines down to create space for the new line at the bottom of the grid.
    - Adjusts the `upper` and `lower` indices to account for the new line being added.
    - Copies the line at the `upper` index into the newly created space for history.
    - Clears the line at the `lower` index using the specified background color.
    - Increments the history scroll count and the size of the history.
- **Output**:
    - The function does not return a value; it modifies the grid's internal state to reflect the new history and updated lines.
- **Functions called**:
    - [`grid_empty_line`](#grid_empty_line)


---
### grid_expand_line<!-- {{#callable:grid_expand_line}} -->
Expands a grid line to accommodate a specified number of cells.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the grid.
    - `py`: An unsigned integer representing the line index in the grid to be expanded.
    - `sx`: An unsigned integer specifying the new size of the line in terms of cells.
    - `bg`: An unsigned integer representing the background color to be used for newly added cells.
- **Control Flow**:
    - Retrieve the line data from the grid using the provided line index `py`.
    - Check if the new size `sx` is less than or equal to the current cell size; if so, exit the function.
    - Adjust `sx` to ensure it is at least one-fourth of the grid's width or half of it, or the grid's width itself, whichever is applicable.
    - Reallocate the `celldata` array of the line to accommodate the new size `sx`.
    - Clear the newly added cells from the current size to the new size using the [`grid_clear_cell`](#grid_clear_cell) function.
- **Output**:
    - The function does not return a value; it modifies the grid line in place by expanding its size and clearing newly added cells.
- **Functions called**:
    - [`grid_clear_cell`](#grid_clear_cell)


---
### grid_empty_line<!-- {{#callable:grid_empty_line}} -->
The `grid_empty_line` function clears a specified line in a grid and optionally expands it to a specified background color.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` representing the grid data structure.
    - `py`: An unsigned integer representing the y-coordinate (line number) of the line to be cleared.
    - `bg`: An unsigned integer representing the background color to be set for the line if it is not the default.
- **Control Flow**:
    - The function first clears the specified line in the grid by setting all its data to zero using `memset`.
    - It then checks if the provided background color is not the default using the `COLOUR_DEFAULT` macro.
    - If the background color is not the default, it calls the [`grid_expand_line`](#grid_expand_line) function to expand the line to fit the grid's width and set the background color.
- **Output**:
    - The function does not return a value; it modifies the grid's line data in place.
- **Functions called**:
    - [`grid_expand_line`](#grid_expand_line)


---
### grid_peek_line<!-- {{#callable:grid_peek_line}} -->
The `grid_peek_line` function retrieves a pointer to a specific line in a grid structure.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure from which the line is to be retrieved.
    - `py`: An unsigned integer representing the y-coordinate (line index) of the line to peek.
- **Control Flow**:
    - The function first calls [`grid_check_y`](#grid_check_y) to validate the y-coordinate `py` against the grid's dimensions.
    - If [`grid_check_y`](#grid_check_y) returns a non-zero value, indicating an out-of-bounds error, the function returns NULL.
    - If the y-coordinate is valid, the function returns a pointer to the line data at the specified index in the grid.
- **Output**:
    - Returns a pointer to the `grid_line` structure at the specified y-coordinate, or NULL if the coordinate is invalid.
- **Functions called**:
    - [`grid_check_y`](#grid_check_y)


---
### grid_get_cell1<!-- {{#callable:grid_get_cell1}} -->
The `grid_get_cell1` function retrieves the properties of a specific cell in a grid line based on its position.
- **Inputs**:
    - `gl`: A pointer to a `struct grid_line` representing the line from which the cell is to be retrieved.
    - `px`: An unsigned integer representing the horizontal position (index) of the cell within the grid line.
    - `gc`: A pointer to a `struct grid_cell` where the retrieved cell's properties will be stored.
- **Control Flow**:
    - The function first retrieves the cell entry from the grid line using the provided position `px`.
    - It checks if the cell entry has the `GRID_FLAG_EXTENDED` flag set, indicating that it is an extended cell.
    - If it is an extended cell, it further checks if the offset is valid; if not, it copies default cell data into `gc`.
    - If the offset is valid, it retrieves the extended entry and populates the `gc` structure with its properties.
    - If the cell is not extended, it copies the basic properties from the cell entry to `gc`, adjusting flags for foreground and background colors.
- **Output**:
    - The function does not return a value but populates the `gc` structure with the properties of the specified cell, including flags, attributes, foreground and background colors, and data.
- **Functions called**:
    - [`grid_set_tab`](#grid_set_tab)


---
### grid_get_cell<!-- {{#callable:grid_get_cell}} -->
Retrieves a cell from a grid at specified coordinates.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the grid from which to retrieve the cell.
    - `px`: An unsigned integer representing the x-coordinate (column index) of the cell to retrieve.
    - `py`: An unsigned integer representing the y-coordinate (row index) of the cell to retrieve.
    - `gc`: A pointer to a `grid_cell` structure where the retrieved cell data will be stored.
- **Control Flow**:
    - The function first checks if the y-coordinate `py` is valid by calling [`grid_check_y`](#grid_check_y). If it is invalid, it copies the default cell data into `gc`.
    - Next, it checks if the x-coordinate `px` is within the bounds of the cell size for the specified row.
    - If both coordinates are valid, it calls [`grid_get_cell1`](#grid_get_cell1) to retrieve the cell data from the specified row and column.
- **Output**:
    - The function does not return a value; instead, it populates the `gc` structure with the cell data from the grid or sets it to the default cell if the coordinates are out of bounds.
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
    - The function first checks if the y-coordinate `py` is valid by calling [`grid_check_y`](#grid_check_y). If it is invalid, the function returns immediately.
    - Next, it expands the line at the y-coordinate `py` to ensure it can accommodate the new cell at position `px` by calling [`grid_expand_line`](#grid_expand_line).
    - It retrieves the line data for the specified y-coordinate and updates the `cellused` count if necessary.
    - Then, it checks if the cell at position `px` needs to be an extended cell by calling [`grid_need_extended_cell`](#grid_need_extended_cell).
    - If an extended cell is needed, it calls [`grid_extended_cell`](#grid_extended_cell) to handle the storage of the new cell data; otherwise, it calls [`grid_store_cell`](#grid_store_cell) to store the cell data directly.
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
    - The function does not return a value; it modifies the grid in place by setting a padding cell.
- **Functions called**:
    - [`grid_set_cell`](#grid_set_cell)


---
### grid_set_cells<!-- {{#callable:grid_set_cells}} -->
Sets a range of cells in a grid with specified character data and attributes.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the grid where cells will be set.
    - `px`: The x-coordinate (column index) in the grid where the cell setting starts.
    - `py`: The y-coordinate (row index) in the grid where the cell setting starts.
    - `gc`: A pointer to a `grid_cell` structure that contains the attributes (like foreground and background color) to apply to the cells.
    - `s`: A pointer to a string containing the character data to be set in the grid cells.
    - `slen`: The length of the string `s`, indicating how many cells will be set.
- **Control Flow**:
    - The function first checks if the y-coordinate `py` is valid using [`grid_check_y`](#grid_check_y). If invalid, it returns immediately.
    - It then expands the line at `py` to ensure it can accommodate the new cells being set, based on the length of the string `s`.
    - The function updates the `cellused` count of the line if the new end position exceeds the current used cells.
    - A loop iterates over each character in the string `s`, setting each corresponding cell in the grid.
    - For each character, it checks if the cell needs to be an extended cell using [`grid_need_extended_cell`](#grid_need_extended_cell). If so, it creates an extended cell and sets its data; otherwise, it stores the character directly in the cell.
- **Output**:
    - The function does not return a value; it modifies the grid in place by setting the specified cells with the provided character data and attributes.
- **Functions called**:
    - [`grid_check_y`](#grid_check_y)
    - [`grid_expand_line`](#grid_expand_line)
    - [`grid_need_extended_cell`](#grid_need_extended_cell)
    - [`grid_extended_cell`](#grid_extended_cell)
    - [`grid_store_cell`](#grid_store_cell)


---
### grid_clear<!-- {{#callable:grid_clear}} -->
The `grid_clear` function clears a specified rectangular area of a grid, setting the cells to a specified background color.
- **Inputs**:
    - `struct grid *gd`: A pointer to the `grid` structure that represents the grid to be modified.
    - `u_int px`: The x-coordinate of the starting position in the grid from where the clearing begins.
    - `u_int py`: The y-coordinate of the starting position in the grid from where the clearing begins.
    - `u_int nx`: The width of the area to be cleared, in number of cells.
    - `u_int ny`: The height of the area to be cleared, in number of cells.
    - `u_int bg`: The background color to set for the cleared cells.
- **Control Flow**:
    - The function first checks if the width (nx) or height (ny) of the area to clear is zero; if so, it returns immediately.
    - If the area to clear spans the entire width of the grid at the specified y-coordinate, it calls [`grid_clear_lines`](#grid_clear_lines) to clear the lines directly.
    - It checks if the starting y-coordinate (py) and the ending y-coordinate (py + ny - 1) are within valid bounds using [`grid_check_y`](#grid_check_y).
    - For each row in the specified area (from py to py + ny), it retrieves the corresponding grid line and determines the effective width to clear.
    - If the background color is the default, it adjusts the width to clear based on the grid line's current size.
    - It expands the line to accommodate the clearing operation and then iterates through the specified width, calling [`grid_clear_cell`](#grid_clear_cell) to clear each cell.
- **Output**:
    - The function does not return a value; it modifies the grid in place by clearing the specified area and setting the cells to the specified background color.
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
    - It then verifies that the starting line index `py` and the ending line index (`py + ny - 1`) are within valid bounds using the [`grid_check_y`](#grid_check_y) function.
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
    - `gd`: A pointer to the `grid` structure representing the grid containing the lines to be moved.
    - `dy`: The destination y-coordinate where the lines will be moved.
    - `py`: The starting y-coordinate of the lines to be moved.
    - `ny`: The number of lines to be moved.
    - `bg`: The background color to be used when clearing lines that are moved.
- **Control Flow**:
    - Checks if the number of lines to move (`ny`) is zero or if the source and destination coordinates are the same, in which case it returns immediately.
    - Validates the y-coordinates (`py`, `dy`, and the range for `ny`) using the [`grid_check_y`](#grid_check_y) function to ensure they are within bounds.
    - Frees any lines in the destination range that will be replaced by the moved lines, skipping lines that are part of the original range being moved.
    - Clears the flags for the line above the destination if it is not the top of the grid.
    - Moves the specified lines from the source position to the destination using `memmove`.
    - Clears the contents of the lines that have been moved without freeing them, using the [`grid_empty_line`](#grid_empty_line) function.
    - Clears the flags for the line above the original starting position if it is not part of the moved range.
- **Output**:
    - The function does not return a value; it modifies the grid in place by moving lines and clearing their contents as necessary.
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
    - `bg`: The background color to use when clearing cells that have been moved.
- **Control Flow**:
    - Checks if the number of cells to move (`nx`) is zero or if the source and destination positions are the same (`px == dx`), in which case the function returns immediately.
    - Validates the y-coordinate (`py`) to ensure it is within the grid's bounds using [`grid_check_y`](#grid_check_y).
    - Retrieves the line data for the specified y-coordinate from the grid.
    - Expands the grid line to accommodate the new destination and source positions using [`grid_expand_line`](#grid_expand_line).
    - Moves the specified cells from the source position to the destination position using `memmove`.
    - Updates the `cellused` count if the new destination exceeds the current used cells.
    - Iterates over the moved cells to clear any cells that were moved but are not part of the new destination range, using [`grid_clear_cell`](#grid_clear_cell).
- **Output**:
    - The function does not return a value; it modifies the grid in place by moving cells and clearing the appropriate cells.
- **Functions called**:
    - [`grid_check_y`](#grid_check_y)
    - [`grid_expand_line`](#grid_expand_line)
    - [`grid_clear_cell`](#grid_clear_cell)


---
### grid_string_cells_fg<!-- {{#callable:grid_string_cells_fg}} -->
The `grid_string_cells_fg` function generates ANSI escape codes for setting the foreground color of a grid cell.
- **Inputs**:
    - `gc`: A pointer to a `struct grid_cell` that contains the foreground color information.
    - `values`: An integer array where the generated ANSI escape codes will be stored.
- **Control Flow**:
    - Initializes a counter `n` to zero to track the number of values added to the `values` array.
    - Checks if the foreground color in `gc` uses 256 colors; if so, adds the appropriate ANSI codes for 256 color mode.
    - If the foreground color uses RGB, it splits the RGB values and adds the corresponding ANSI codes.
    - If the foreground color is a standard color (0-7 or 90-97), it adds the corresponding ANSI code based on the color value.
    - Returns the count of values added to the `values` array.
- **Output**:
    - Returns the number of ANSI escape codes added to the `values` array, which corresponds to the foreground color settings.


---
### grid_string_cells_bg<!-- {{#callable:grid_string_cells_bg}} -->
Converts the background color of a grid cell into ANSI escape codes.
- **Inputs**:
    - `gc`: A pointer to a `struct grid_cell` that contains the background color information.
    - `values`: An integer array where the ANSI escape codes will be stored.
- **Control Flow**:
    - Initializes a counter `n` to zero to track the number of escape codes generated.
    - Checks if the background color in `gc` uses 256 colors; if so, it adds the appropriate ANSI codes for 256 colors to `values`.
    - If the background color uses RGB, it splits the RGB values and adds the corresponding ANSI codes to `values`.
    - If the background color is a standard color (0-7 or 90-97), it adds the corresponding ANSI codes based on the color value.
- **Output**:
    - Returns the number of ANSI escape codes generated and stored in the `values` array.


---
### grid_string_cells_us<!-- {{#callable:grid_string_cells_us}} -->
The `grid_string_cells_us` function generates ANSI escape codes for the underline color of a grid cell.
- **Inputs**:
    - `gc`: A pointer to a `struct grid_cell` that contains color information for the underline.
    - `values`: An integer array where the generated ANSI escape codes will be stored.
- **Control Flow**:
    - The function initializes a counter `n` to zero.
    - It checks if the `us` field of the `grid_cell` structure has the `COLOUR_FLAG_256` flag set.
    - If the `COLOUR_FLAG_256` is set, it stores the ANSI code for 256 colors in the `values` array.
    - If the `COLOUR_FLAG_RGB` is set instead, it splits the RGB color values and stores them in the `values` array.
    - The function returns the count of values stored in the `values` array.
- **Output**:
    - The function returns the number of ANSI escape code values stored in the `values` array.


---
### grid_string_cells_add_code<!-- {{#callable:grid_string_cells_add_code}} -->
Adds ANSI SGR codes to a buffer based on the provided color and attribute changes.
- **Inputs**:
    - `buf`: A pointer to a character buffer where the ANSI codes will be appended.
    - `len`: The maximum length of the buffer to prevent overflow.
    - `n`: The number of attributes or colors to process.
    - `s`: An array of integers representing the current state of attributes.
    - `newc`: An array of integers representing the new color attributes to be applied.
    - `oldc`: An array of integers representing the old color attributes for comparison.
    - `nnewc`: The number of new color attributes.
    - `noldc`: The number of old color attributes.
    - `flags`: Flags that determine how the function behaves, such as whether to use escape sequences.
- **Control Flow**:
    - Check if there are new colors to add; if not, return immediately.
    - Determine if a reset is needed based on the current state and the new colors.
    - If no reset is needed and the new colors are the same as the old colors, return.
    - If a reset is needed and the new color is a default color, return.
    - Append the appropriate escape sequence to the buffer based on the flags.
    - Iterate through the new color attributes and format them into the buffer.
    - Append the final 'm' to indicate the end of the SGR sequence.
- **Output**:
    - The function modifies the provided buffer to include ANSI SGR codes that set the specified text attributes and colors.


---
### grid_string_cells_add_hyperlink<!-- {{#callable:grid_string_cells_add_hyperlink}} -->
Adds a hyperlink to a string buffer for terminal display.
- **Inputs**:
    - `buf`: A pointer to the character buffer where the hyperlink will be added.
    - `len`: The maximum length of the buffer to prevent overflow.
    - `id`: An optional identifier for the hyperlink, can be an empty string.
    - `uri`: The URI that the hyperlink points to.
    - `flags`: Flags that determine how the hyperlink is formatted, such as whether to escape sequences.
- **Control Flow**:
    - Checks if the combined length of the URI and ID exceeds the buffer length; if so, returns 0.
    - Appends the appropriate escape sequence for starting a hyperlink based on the flags.
    - If the ID is not empty, formats it and appends it to the buffer.
    - Appends the URI to the buffer.
    - Appends the appropriate escape sequence for ending a hyperlink based on the flags.
    - Returns 1 to indicate success.
- **Output**:
    - Returns 1 on success, or 0 if the buffer length is exceeded.


---
### grid_string_cells_code<!-- {{#callable:grid_string_cells_code}} -->
The `grid_string_cells_code` function generates ANSI escape codes for terminal cell attributes and colors based on the differences between the current and previous grid cell states.
- **Inputs**:
    - `lastgc`: Pointer to the previous `grid_cell` structure representing the last state of the cell.
    - `gc`: Pointer to the current `grid_cell` structure representing the current state of the cell.
    - `buf`: Character buffer where the generated ANSI escape codes will be stored.
    - `len`: Size of the buffer to prevent overflow.
    - `flags`: Integer flags that modify the behavior of the function, such as whether to escape sequences.
    - `sc`: Pointer to a `screen` structure that may contain hyperlink information.
    - `has_link`: Pointer to an integer that indicates whether a hyperlink is present.
- **Control Flow**:
    - Initializes arrays to hold old and new color values and a temporary array for attribute codes.
    - Checks for removed attributes and resets the attribute state if necessary.
    - Identifies newly set attributes and appends their corresponding codes to the output buffer.
    - Generates ANSI codes for foreground and background colors by calling helper functions.
    - Appends shift in/shift out codes if the character set attribute changes.
    - Checks for changes in hyperlinks and updates the output buffer accordingly.
- **Output**:
    - The function does not return a value but modifies the provided buffer to contain the generated ANSI escape codes, which can be used to format text in a terminal.
- **Functions called**:
    - [`grid_string_cells_fg`](#grid_string_cells_fg)
    - [`grid_string_cells_add_code`](#grid_string_cells_add_code)
    - [`grid_string_cells_bg`](#grid_string_cells_bg)
    - [`grid_string_cells_us`](#grid_string_cells_us)
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
    - `flags`: Integer flags that modify the behavior of the function (e.g., escape sequences, trimming spaces).
    - `s`: Pointer to a `screen` structure used for hyperlink management.
- **Control Flow**:
    - If `lastgc` is NULL or points to NULL, initialize it with a default grid cell.
    - Allocate an initial buffer for the output string.
    - Peek at the grid line at the specified y-coordinate (`py`).
    - Determine the end of the cells to process based on the flags provided.
    - Iterate over the specified range of cells, retrieving each cell's data.
    - If the cell is a padding cell, skip it.
    - If the `GRID_STRING_WITH_SEQUENCES` flag is set, generate ANSI codes for the cell's attributes.
    - Handle tab characters and escape sequences based on the flags.
    - Resize the output buffer as needed to accommodate the data being added.
    - Append the cell's data and any generated codes to the output buffer.
    - If hyperlinks are present, add them to the output string.
    - Trim trailing spaces from the output if the `GRID_STRING_TRIM_SPACES` flag is set.
    - Null-terminate the output string before returning it.
- **Output**:
    - Returns a dynamically allocated string containing the formatted representation of the specified grid cells.
- **Functions called**:
    - [`grid_peek_line`](#grid_peek_line)
    - [`grid_get_cell`](#grid_get_cell)
    - [`grid_string_cells_code`](#grid_string_cells_code)
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


---
### grid_reflow_dead<!-- {{#callable:grid_reflow_dead}} -->
The `grid_reflow_dead` function marks a grid line as dead by zeroing its contents and setting its flags.
- **Inputs**:
    - `gl`: A pointer to a `struct grid_line` that represents the grid line to be marked as dead.
- **Control Flow**:
    - The function uses `memset` to clear the memory of the grid line pointed to by `gl`, setting all its bytes to zero.
    - It then sets the `flags` member of the grid line to `GRID_LINE_DEAD`, indicating that this line is no longer active.
- **Output**:
    - The function does not return a value; it modifies the input grid line in place to mark it as dead.


---
### grid_reflow_add<!-- {{#callable:grid_reflow_add}} -->
The `grid_reflow_add` function adds a specified number of new lines to a grid and returns a pointer to the first new line.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure that represents the grid to which new lines will be added.
    - `n`: An unsigned integer representing the number of new lines to add to the grid.
- **Control Flow**:
    - Calculates the new total number of lines in the grid by adding `n` to the current number of lines (`gd->sy`).
    - Resizes the `linedata` array of the grid using `xreallocarray` to accommodate the new total number of lines.
    - Obtains a pointer to the first new line by referencing the newly allocated space in `linedata`.
    - Initializes the newly added lines to zero using `memset`.
- **Output**:
    - Returns a pointer to the first newly added `grid_line` structure in the grid.


---
### grid_reflow_move<!-- {{#callable:grid_reflow_move}} -->
Moves a `grid_line` from one position to another in a `grid`.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure that contains the grid lines.
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
    - If `already` is false, a new line is added to the target grid by moving a line from the source grid.
    - A loop is initiated to consume lines from the source grid until either the target line is full or there are no more lines to consume.
    - Within the loop, if the next line in the source grid is empty, it is skipped unless it is the last line.
    - If the next line is not empty, the first cell is copied to the target grid, and additional cells are joined until the target line reaches its width limit.
    - If the entire line is consumed, the remaining cells in the source line are adjusted accordingly.
    - Finally, any completely consumed lines are freed, and the scroll position of the source grid is adjusted.
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
    - `u_int at`: The index in the line from which to start splitting.
- **Control Flow**:
    - Checks if the line is extended or not to determine how many new lines are needed.
    - Calculates the number of lines to insert based on the width of the cells and the specified maximum width.
    - Inserts new lines into the target grid.
    - Copies the cells from the original line to the new lines, wrapping cells as necessary based on the maximum width.
    - Updates the original line to reflect the split and marks it as wrapped if necessary.
    - Adjusts the scroll position of the grid if the split line is above the current scroll position.
    - Attempts to join the last new line with the next line if there is space and the original line was wrapped.
- **Output**:
    - The function does not return a value; it modifies the target grid in place by adding new lines.
- **Functions called**:
    - [`grid_get_cell1`](#grid_get_cell1)
    - [`grid_reflow_add`](#grid_reflow_add)
    - [`grid_set_cell`](#grid_set_cell)
    - [`grid_reflow_dead`](#grid_reflow_dead)
    - [`grid_reflow_join`](#grid_reflow_join)


---
### grid_reflow<!-- {{#callable:grid_reflow}} -->
The `grid_reflow` function adjusts the layout of a grid to fit a new specified width.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure that contains the current grid data.
    - `sx`: An unsigned integer representing the new width to which the grid should be reflowed.
- **Control Flow**:
    - Creates a new target grid to hold the reflowed data.
    - Iterates over each line in the source grid.
    - Checks if the current line is marked as dead and skips it if so.
    - Calculates the width of the current line and determines how many cells can fit within the new width.
    - If the line's width matches the new width, it moves the line to the target grid unchanged.
    - If the line's width exceeds the new width, it splits the line into multiple lines.
    - If the line is shorter than the new width and was previously wrapped, it attempts to join it with the next line.
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
    - Initializes two variables `ax` and `ay` to track the wrapped x and y positions respectively.
    - Iterates through each line in the grid up to the specified y-coordinate (`py`).
    - For each line, checks if it is wrapped using the `GRID_LINE_WRAPPED` flag.
    - If the line is wrapped, adds the number of used cells in that line to `ax`.
    - If the line is not wrapped, resets `ax` to 0 and increments `ay` to move to the next line.
    - After the loop, checks if the specified x-coordinate (`px`) exceeds the number of used cells in the current line.
    - If it does, sets `ax` to `UINT_MAX`, otherwise adds `px` to `ax`.
    - Finally, assigns the calculated `ax` and `ay` values to the provided pointers `wx` and `wy`.
- **Output**:
    - The function does not return a value but updates the values pointed to by `wx` and `wy` with the calculated wrapped x and y coordinates.


---
### grid_unwrap_position<!-- {{#callable:grid_unwrap_position}} -->
The `grid_unwrap_position` function converts wrapped grid coordinates back to their original unwrapped positions.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the grid data.
    - `px`: A pointer to an unsigned integer where the unwrapped x-coordinate will be stored.
    - `py`: A pointer to an unsigned integer where the unwrapped y-coordinate will be stored.
    - `wx`: The wrapped x-coordinate to be converted.
    - `wy`: The wrapped y-coordinate to be converted.
- **Control Flow**:
    - The function initializes a loop to iterate through the grid lines up to the height of the grid plus the screen size.
    - It counts the number of unwrapped lines until it reaches the specified wrapped y-coordinate (`wy`).
    - If the wrapped x-coordinate (`wx`) is `UINT_MAX`, it finds the end of the wrapped lines and sets `wx` to the cell used in the next unwrapped line.
    - If `wx` is not `UINT_MAX`, it decrements `wx` by the number of cells used in each wrapped line until it finds the line containing the unwrapped x-coordinate.
    - Finally, it assigns the calculated unwrapped x and y coordinates to the provided pointers.
- **Output**:
    - The function does not return a value but updates the values pointed to by `px` and `py` with the unwrapped coordinates.


---
### grid_line_length<!-- {{#callable:grid_line_length}} -->
Calculates the length of a line in a grid, trimming trailing padding and spaces.
- **Inputs**:
    - `gd`: A pointer to a `struct grid` representing the grid from which the line length is to be calculated.
    - `py`: An unsigned integer representing the y-coordinate (line number) in the grid for which the length is to be determined.
- **Control Flow**:
    - Retrieve the cell size of the specified line using [`grid_get_line`](#grid_get_line).
    - If the cell size exceeds the grid's width (`gd->sx`), set it to the grid's width.
    - Iterate backwards through the cells of the line, checking each cell's flags and data.
    - If a cell is found that is either a padding cell, has a size not equal to 1, or does not contain a space character, break the loop.
    - Return the calculated length of the line.
- **Output**:
    - Returns an unsigned integer representing the length of the line, excluding any trailing padding and spaces.
- **Functions called**:
    - [`grid_get_line`](#grid_get_line)
    - [`grid_get_cell`](#grid_get_cell)


---
### grid_in_set<!-- {{#callable:grid_in_set}} -->
Checks if a grid cell at specified coordinates contains a character from a given set.
- **Inputs**:
    - `gd`: A pointer to the `grid` structure representing the grid.
    - `px`: An unsigned integer representing the x-coordinate of the cell in the grid.
    - `py`: An unsigned integer representing the y-coordinate of the cell in the grid.
    - `set`: A pointer to a string containing the set of characters to check against.
- **Control Flow**:
    - Retrieve the grid cell at coordinates (px, py) into the `gc` structure.
    - Check if the `set` contains a tab character ('\t').
    - If the cell is a padding cell, decrement `px` and check adjacent cells until a non-padding cell is found.
    - If the found cell is a tab, return the width of the tab adjusted by the difference in x-coordinates.
    - If the current cell is a tab, return its width.
    - If the current cell is a padding cell, return 0.
    - Finally, check if the character in the current cell is present in the `set` using `utf8_cstrhas`.
- **Output**:
    - Returns an integer indicating the width of the tab if found, 0 if the cell is padding, or 1 if the character in the cell is in the set, otherwise returns 0.
- **Functions called**:
    - [`grid_get_cell`](#grid_get_cell)


