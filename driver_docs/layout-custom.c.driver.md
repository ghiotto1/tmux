# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file provides functionality for managing and manipulating window layouts within tmux. It includes functions to parse, construct, and validate layout structures, which define how terminal panes are arranged within a window. The code handles the serialization and deserialization of these layouts, allowing them to be saved as strings and later reconstructed. Key functions include [`layout_parse`](#layout_parse), which interprets a layout string and arranges the window panes accordingly, and [`layout_dump`](#layout_dump), which converts a layout structure into a string representation.

The file defines several static functions that operate on `layout_cell` structures, which represent individual panes or groups of panes within a layout. These functions include [`layout_find_bottomright`](#layout_find_bottomright), which locates the bottom-right cell in a layout, and [`layout_assign`](#layout_assign), which assigns window panes to layout cells. The code also includes mechanisms for checking the validity of layouts and ensuring that they fit within the specified window dimensions. The file is not intended to be an executable on its own but rather a component of the larger tmux codebase, providing essential layout management capabilities.
# Imports and Dependencies

---
- `sys/types.h`
- `ctype.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### layout_find_bottomright
- **Type**: `static struct layout_cell*`
- **Description**: The `layout_find_bottomright` is a static function that returns a pointer to a `layout_cell` structure. It is designed to find the bottom-right cell in a layout structure, which is typically a tree of cells representing a window layout.
- **Use**: This function is used to recursively traverse a layout structure to locate and return the bottom-right cell, which is often needed for operations like closing or resizing the layout.


---
### layout_construct
- **Type**: `static struct layout_cell*`
- **Description**: The `layout_construct` function is a static function that constructs a layout cell from a given layout string. It parses the layout string to extract dimensions and offsets, and recursively constructs a tree of layout cells representing the layout structure.
- **Use**: This function is used to build a layout cell tree from a layout string, which is then used to arrange window panes in a specified layout.


# Functions

---
### layout_find_bottomright<!-- {{#callable:layout_find_bottomright}} -->
The [`layout_find_bottomright`](#layout_find_bottomright) function recursively finds and returns the bottom-right cell in a layout tree.
- **Inputs**:
    - `lc`: A pointer to a `layout_cell` structure representing the root or current cell in the layout tree.
- **Control Flow**:
    - Check if the current cell `lc` is of type `LAYOUT_WINDOWPANE`; if so, return it as it is the bottom-right cell.
    - If not, retrieve the last cell in the list of child cells of `lc` using `TAILQ_LAST`.
    - Recursively call [`layout_find_bottomright`](#layout_find_bottomright) with the last child cell to continue searching for the bottom-right cell.
- **Output**:
    - Returns a pointer to the `layout_cell` structure that is the bottom-right cell in the layout tree.
- **Functions called**:
    - [`layout_find_bottomright`](#layout_find_bottomright)


---
### layout_checksum<!-- {{#callable:layout_checksum}} -->
The `layout_checksum` function calculates a checksum for a given layout string by iterating over each character and applying a specific bitwise operation.
- **Inputs**:
    - `layout`: A constant character pointer representing the layout string for which the checksum is to be calculated.
- **Control Flow**:
    - Initialize a variable `csum` of type `u_short` to 0 to store the checksum.
    - Iterate over each character in the `layout` string until the null terminator is reached.
    - For each character, perform a bitwise right shift on `csum` by 1, add the result of a bitwise AND operation between `csum` and 1 left-shifted by 15, and then add the ASCII value of the current character to `csum`.
    - Return the final value of `csum` as the checksum.
- **Output**:
    - The function returns a `u_short` value representing the calculated checksum of the input layout string.


---
### layout_dump<!-- {{#callable:layout_dump}} -->
The `layout_dump` function generates a string representation of a layout tree, including a checksum, from a given root layout cell.
- **Inputs**:
    - `root`: A pointer to the root `layout_cell` structure representing the layout tree to be dumped.
- **Control Flow**:
    - Initialize an empty string `layout` with a size of 8192 characters.
    - Call [`layout_append`](#layout_append) to populate `layout` with the string representation of the layout tree starting from `root`.
    - If [`layout_append`](#layout_append) returns a non-zero value, indicating an error, return `NULL`.
    - Calculate the checksum of the `layout` string using [`layout_checksum`](#layout_checksum).
    - Format the output string `out` to include the checksum and the layout string using [`xasprintf`](xmalloc.c.driver.md#xasprintf).
    - Return the formatted string `out`.
- **Output**:
    - A dynamically allocated string containing the checksum and the layout string, or `NULL` if an error occurs during layout appending.
- **Functions called**:
    - [`layout_append`](#layout_append)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`layout_checksum`](#layout_checksum)


---
### layout_append<!-- {{#callable:layout_append}} -->
The [`layout_append`](#layout_append) function appends a string representation of a layout cell and its children to a buffer, handling different layout types and ensuring the buffer does not overflow.
- **Inputs**:
    - `lc`: A pointer to a `layout_cell` structure representing the current layout cell to be processed.
    - `buf`: A character buffer where the layout information will be appended.
    - `len`: The size of the buffer `buf`, indicating the maximum number of characters it can hold.
- **Control Flow**:
    - Check if the buffer length `len` is zero and return -1 if true, indicating an error.
    - Format the layout cell's dimensions and offsets into a temporary string `tmp`, including the window pane ID if present.
    - Check if the formatted string `tmp` exceeds its buffer size and return -1 if true.
    - Append the formatted string `tmp` to the buffer `buf` and return -1 if it exceeds the buffer length `len`.
    - Determine the type of layout (left-right or top-bottom) and set the appropriate bracket characters.
    - If the layout type is left-right or top-bottom, append the opening bracket to the buffer and iterate over child cells.
    - Recursively call [`layout_append`](#layout_append) for each child cell and append a comma after each, returning -1 if any operation fails.
    - Replace the last comma with the closing bracket to complete the layout string for the current cell.
    - Return 0 to indicate successful completion.
- **Output**:
    - Returns 0 on success, or -1 if an error occurs, such as buffer overflow or invalid input.
- **Functions called**:
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)
    - [`strlcat`](compat/strlcat.c.driver.md#strlcat)
    - [`layout_append`](#layout_append)


---
### layout_check<!-- {{#callable:layout_check}} -->
The [`layout_check`](#layout_check) function verifies that a given layout cell and its children have consistent dimensions according to their layout type.
- **Inputs**:
    - `lc`: A pointer to a `layout_cell` structure representing the root of the layout tree to be checked.
- **Control Flow**:
    - The function begins by initializing a counter `n` to zero.
    - It checks the type of the layout cell `lc` using a switch statement.
    - If the type is `LAYOUT_WINDOWPANE`, the function does nothing and proceeds to return success.
    - For `LAYOUT_LEFTRIGHT`, it iterates over each child cell in `lc->cells` using `TAILQ_FOREACH`.
    - During iteration, it checks if each child cell's `sy` matches `lc->sy`; if not, it returns failure.
    - It recursively calls [`layout_check`](#layout_check) on each child cell, returning failure if any call fails.
    - It accumulates the width (`sx`) of each child cell plus one into `n`.
    - After the loop, it checks if `n - 1` equals `lc->sx`; if not, it returns failure.
    - For `LAYOUT_TOPBOTTOM`, it similarly iterates over each child cell, checking `sx` against `lc->sx` and accumulating `sy` into `n`.
    - It performs the same recursive check and final dimension check as in `LAYOUT_LEFTRIGHT`.
    - If all checks pass, the function returns success.
- **Output**:
    - The function returns an integer: 1 if the layout is valid and 0 if there is a size mismatch.
- **Functions called**:
    - [`layout_check`](#layout_check)


---
### layout_parse<!-- {{#callable:layout_parse}} -->
The `layout_parse` function parses a layout string to arrange a window's panes according to the specified layout, ensuring the layout is valid and fits the window's constraints.
- **Inputs**:
    - `w`: A pointer to a `struct window` representing the window to be arranged according to the layout.
    - `layout`: A constant character pointer to the layout string that describes how the window should be arranged.
    - `cause`: A pointer to a character pointer where an error message will be stored if the function fails.
- **Control Flow**:
    - The function begins by validating the layout string's checksum using `sscanf` and [`layout_checksum`](#layout_checksum); if invalid, it sets an error message and returns -1.
    - It constructs the layout using [`layout_construct`](#layout_construct); if construction fails or the layout string is not fully consumed, it sets an error message and jumps to the fail label.
    - The function checks if the window can fit into the layout by comparing the number of panes and cells; if there are more panes than cells, it sets an error message and jumps to the fail label.
    - If there are fewer panes than cells, it repeatedly closes the bottom-right cell using [`layout_find_bottomright`](#layout_find_bottomright) and [`layout_destroy_cell`](layout.c.driver.md#layout_destroy_cell) until the numbers match.
    - It adjusts the layout's top cell size if necessary to correct any discrepancies from older versions of tmux.
    - The function verifies the new layout with [`layout_check`](#layout_check); if it fails, it sets an error message and jumps to the fail label.
    - It resizes the window to match the layout's size using [`window_resize`](window.c.driver.md#window_resize).
    - The old layout is destroyed, and the new layout is assigned to the window's root layout cell.
    - Panes are assigned to the layout cells using [`layout_assign`](#layout_assign).
    - Pane offsets and sizes are updated with [`layout_fix_offsets`](layout.c.driver.md#layout_fix_offsets) and [`layout_fix_panes`](layout.c.driver.md#layout_fix_panes), and sizes are recalculated.
    - The layout is printed for debugging, and a notification is sent that the window layout has changed.
    - If any step fails, the function frees the constructed layout and returns -1.
- **Output**:
    - The function returns 0 on success, indicating the layout was successfully parsed and applied, or -1 on failure, with an error message stored in `cause`.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`layout_checksum`](#layout_checksum)
    - [`layout_construct`](#layout_construct)
    - [`window_count_panes`](window.c.driver.md#window_count_panes)
    - [`layout_count_cells`](layout.c.driver.md#layout_count_cells)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`layout_find_bottomright`](#layout_find_bottomright)
    - [`layout_destroy_cell`](layout.c.driver.md#layout_destroy_cell)
    - [`layout_print_cell`](layout.c.driver.md#layout_print_cell)
    - [`layout_check`](#layout_check)
    - [`window_resize`](window.c.driver.md#window_resize)
    - [`layout_free_cell`](layout.c.driver.md#layout_free_cell)
    - [`layout_assign`](#layout_assign)
    - [`layout_fix_offsets`](layout.c.driver.md#layout_fix_offsets)
    - [`layout_fix_panes`](layout.c.driver.md#layout_fix_panes)
    - [`recalculate_sizes`](resize.c.driver.md#recalculate_sizes)
    - [`notify_window`](notify.c.driver.md#notify_window)


---
### layout_assign<!-- {{#callable:layout_assign}} -->
The [`layout_assign`](#layout_assign) function assigns window panes to layout cells based on the cell type, recursively handling nested cells.
- **Inputs**:
    - `wp`: A pointer to a pointer to a `window_pane` structure, representing the current window pane to be assigned.
    - `lc`: A pointer to a `layout_cell` structure, representing the current layout cell to which a window pane may be assigned.
- **Control Flow**:
    - The function checks the type of the layout cell `lc`.
    - If the cell type is `LAYOUT_WINDOWPANE`, it calls [`layout_make_leaf`](layout.c.driver.md#layout_make_leaf) to assign the current window pane to the cell and advances the window pane pointer to the next pane in the queue.
    - If the cell type is `LAYOUT_LEFTRIGHT` or `LAYOUT_TOPBOTTOM`, it iterates over each child cell in the `lc->cells` list and recursively calls [`layout_assign`](#layout_assign) for each child cell.
- **Output**:
    - The function does not return a value; it modifies the layout cells and window pane pointers in place.
- **Functions called**:
    - [`layout_make_leaf`](layout.c.driver.md#layout_make_leaf)
    - [`layout_assign`](#layout_assign)


---
### layout_construct<!-- {{#callable:layout_construct}} -->
The [`layout_construct`](#layout_construct) function parses a layout string to construct a hierarchical layout cell structure representing window pane arrangements.
- **Inputs**:
    - `lcparent`: A pointer to the parent layout cell, which can be NULL if the current cell is the root.
    - `layout`: A pointer to a string representing the layout, which is parsed to extract dimensions and offsets for the layout cells.
- **Control Flow**:
    - Check if the layout string starts with a digit; if not, return NULL.
    - Parse the layout string for dimensions (sx, sy) and offsets (xoff, yoff) using sscanf; if parsing fails, return NULL.
    - Advance the layout pointer past the parsed numbers and check for expected delimiters ('x', ','); if any are missing, return NULL.
    - Create a new layout cell using [`layout_create_cell`](layout.c.driver.md#layout_create_cell) and assign parsed dimensions and offsets to it.
    - Determine the type of layout (LAYOUT_LEFTRIGHT or LAYOUT_TOPBOTTOM) based on the next character in the layout string; if invalid, go to fail.
    - Recursively construct child layout cells if the layout type is LAYOUT_LEFTRIGHT or LAYOUT_TOPBOTTOM, inserting them into the current cell's list of children.
    - Check for closing delimiters ('}', ']') based on the layout type; if missing, go to fail.
    - Return the constructed layout cell if successful, or free the cell and return NULL on failure.
- **Output**:
    - Returns a pointer to the constructed layout cell if successful, or NULL if there is an error in parsing the layout string.
- **Functions called**:
    - [`layout_create_cell`](layout.c.driver.md#layout_create_cell)
    - [`layout_construct`](#layout_construct)
    - [`layout_free_cell`](layout.c.driver.md#layout_free_cell)


