# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to manage multiple terminal sessions within a single window. The file is focused on managing and manipulating the layout of window panes within tmux. It provides functions to parse, construct, and validate layout configurations, which define how different panes are arranged within a tmux window. The code includes functions to find specific layout cells, calculate checksums for layout strings, and convert layout structures into string representations. It also includes logic to ensure that the layout is valid and that the window panes fit within the specified layout dimensions.

The file defines several static functions, indicating that these functions are intended for internal use within this file and are not part of a public API. The functions handle tasks such as finding the bottom-right cell in a layout, appending layout information to a buffer, and constructing layout cells from a layout string. The [`layout_parse`](#layout_parse) function is a key component, as it parses a layout string, constructs the layout, checks its validity, and assigns window panes to the layout cells. This file is integral to the layout management subsystem of tmux, ensuring that window panes are organized and displayed according to user-defined or default layouts.
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
- **Description**: The `layout_find_bottomright` is a static function that returns a pointer to a `layout_cell` structure. It is designed to find the bottom-right cell in a layout structure, which is a hierarchical representation of window panes in a layout.
- **Use**: This function is used to recursively traverse a layout structure to locate and return the bottom-right cell, which is typically used for operations like closing or resizing the layout.


---
### layout_construct
- **Type**: `function pointer`
- **Description**: `layout_construct` is a static function that constructs a layout cell from a given layout string. It takes a pointer to a parent layout cell and a pointer to a layout string as arguments, and returns a pointer to the newly constructed layout cell.
- **Use**: This function is used to parse a layout string and build a corresponding layout cell structure.


# Functions

---
### layout_find_bottomright<!-- {{#callable:layout_find_bottomright}} -->
The function [`layout_find_bottomright`](#layout_find_bottomright) recursively finds and returns the bottom-right cell in a layout tree.
- **Inputs**:
    - `lc`: A pointer to a `layout_cell` structure, representing the starting cell from which to find the bottom-right cell.
- **Control Flow**:
    - Check if the current cell `lc` is of type `LAYOUT_WINDOWPANE`; if so, return it as it is the bottom-right cell.
    - If not, retrieve the last cell in the `lc->cells` list using `TAILQ_LAST`.
    - Recursively call [`layout_find_bottomright`](#layout_find_bottomright) with the last cell obtained in the previous step.
- **Output**:
    - Returns a pointer to the `layout_cell` structure that is the bottom-right cell in the layout tree.
- **Functions called**:
    - [`layout_find_bottomright`](#layout_find_bottomright)


---
### layout_checksum<!-- {{#callable:layout_checksum}} -->
The `layout_checksum` function calculates a checksum for a given layout string by iterating over each character and updating a checksum value using bitwise operations.
- **Inputs**:
    - `layout`: A constant character pointer representing the layout string for which the checksum is to be calculated.
- **Control Flow**:
    - Initialize a variable `csum` to 0 to store the checksum.
    - Iterate over each character in the `layout` string until the null terminator is reached.
    - For each character, update `csum` by performing a right bitwise shift by 1, adding the leftmost bit shifted to the 15th position, and then adding the ASCII value of the current character.
    - Return the final checksum value stored in `csum`.
- **Output**:
    - The function returns an unsigned short integer representing the calculated checksum of the input layout string.


---
### layout_dump<!-- {{#callable:layout_dump}} -->
The `layout_dump` function generates a string representation of a layout tree, including a checksum, from a given root layout cell.
- **Inputs**:
    - `root`: A pointer to the root `layout_cell` structure representing the layout tree to be dumped.
- **Control Flow**:
    - Initialize an empty string `layout` with a fixed size of 8192 bytes.
    - Call [`layout_append`](#layout_append) to append the layout information from the `root` cell into the `layout` string; if it fails, return `NULL`.
    - Calculate a checksum of the `layout` string using [`layout_checksum`](#layout_checksum).
    - Format the checksum and the `layout` string into a new string `out` using `xasprintf`.
    - Return the formatted string `out`.
- **Output**:
    - A dynamically allocated string containing the checksum and the layout information, or `NULL` if an error occurs during layout appending.
- **Functions called**:
    - [`layout_append`](#layout_append)
    - [`layout_checksum`](#layout_checksum)


---
### layout_append<!-- {{#callable:layout_append}} -->
The [`layout_append`](#layout_append) function appends a string representation of a layout cell and its children to a buffer, handling different layout types and ensuring the buffer does not overflow.
- **Inputs**:
    - `lc`: A pointer to a `layout_cell` structure representing the current layout cell to be processed.
    - `buf`: A character buffer where the layout string representation will be appended.
    - `len`: The size of the buffer `buf`, indicating the maximum number of characters it can hold.
- **Control Flow**:
    - Check if the buffer length `len` is zero and return -1 if true, indicating an error.
    - Format the layout cell's dimensions and offsets into a temporary string `tmp`, including the window pane ID if present.
    - Check if the formatted string `tmp` exceeds its buffer size and return -1 if true.
    - Append the formatted string `tmp` to the buffer `buf` and return -1 if it exceeds the buffer size `len`.
    - Determine the type of layout cell (`LAYOUT_LEFTRIGHT`, `LAYOUT_TOPBOTTOM`, or `LAYOUT_WINDOWPANE`) and set the appropriate bracket characters.
    - For `LAYOUT_LEFTRIGHT` and `LAYOUT_TOPBOTTOM`, append the closing bracket to the buffer and iterate over child cells, recursively calling [`layout_append`](#layout_append) for each child.
    - Append a comma after each child cell's string representation, then replace the last comma with the opening bracket character.
    - Return 0 to indicate successful completion.
- **Output**:
    - Returns 0 on success, or -1 if an error occurs (such as buffer overflow or invalid input).
- **Functions called**:
    - [`layout_append`](#layout_append)


---
### layout_check<!-- {{#callable:layout_check}} -->
The [`layout_check`](#layout_check) function verifies that a given layout cell and its children have consistent dimensions according to their layout type.
- **Inputs**:
    - `lc`: A pointer to a `layout_cell` structure representing the root of the layout tree to be checked.
- **Control Flow**:
    - Initialize a counter `n` to zero.
    - Switch on the type of the layout cell `lc`.
    - If the type is `LAYOUT_WINDOWPANE`, do nothing and break out of the switch.
    - If the type is `LAYOUT_LEFTRIGHT`, iterate over each child cell `lcchild` in `lc->cells`.
    - For each child, check if its height `sy` matches the parent `lc->sy`; if not, return 0.
    - Recursively call [`layout_check`](#layout_check) on each child; if any call returns 0, return 0.
    - Accumulate the widths `sx` of the children plus one into `n`.
    - After the loop, check if `n - 1` equals `lc->sx`; if not, return 0.
    - If the type is `LAYOUT_TOPBOTTOM`, iterate over each child cell `lcchild` in `lc->cells`.
    - For each child, check if its width `sx` matches the parent `lc->sx`; if not, return 0.
    - Recursively call [`layout_check`](#layout_check) on each child; if any call returns 0, return 0.
    - Accumulate the heights `sy` of the children plus one into `n`.
    - After the loop, check if `n - 1` equals `lc->sy`; if not, return 0.
    - Return 1 if all checks pass.
- **Output**:
    - Returns 1 if the layout is consistent, otherwise returns 0 if any inconsistency is found.
- **Functions called**:
    - [`layout_check`](#layout_check)


---
### layout_parse<!-- {{#callable:layout_parse}} -->
The `layout_parse` function parses a layout string to arrange a window's panes according to the specified layout, ensuring the layout is valid and fits the window's current configuration.
- **Inputs**:
    - `w`: A pointer to the `struct window` that represents the window to be arranged according to the layout.
    - `layout`: A constant character pointer to the layout string that describes the desired arrangement of panes within the window.
    - `cause`: A pointer to a character pointer where an error message will be stored if the function fails.
- **Control Flow**:
    - The function begins by validating the layout string's checksum using `sscanf` and [`layout_checksum`](#layout_checksum) to ensure it matches the expected format.
    - If the checksum is invalid, the function sets an error message in `cause` and returns -1.
    - The function constructs the layout tree using [`layout_construct`](#layout_construct), and if construction fails, it sets an error message and returns -1.
    - The function checks if the number of panes in the window matches the number of cells in the layout; if there are more panes than cells, it sets an error message and returns -1.
    - If there are fewer panes than cells, it removes the bottom-right cell until the numbers match.
    - The function adjusts the layout's top cell size if it is incorrect, based on the sizes of its child cells.
    - The function verifies the layout's validity with [`layout_check`](#layout_check); if invalid, it sets an error message and returns -1.
    - The window is resized to match the layout's size using `window_resize`.
    - The old layout is destroyed, and the new layout is assigned to the window's root layout cell.
    - Panes are assigned to the layout cells using [`layout_assign`](#layout_assign).
    - Pane offsets and sizes are updated with `layout_fix_offsets` and `layout_fix_panes`, and sizes are recalculated.
    - The function logs the layout and notifies that the window layout has changed.
    - If any step fails, the function cleans up by freeing the layout cell and returns -1.
- **Output**:
    - The function returns 0 on success, indicating the layout was successfully parsed and applied, or -1 on failure, with an error message set in `cause`.
- **Functions called**:
    - [`layout_checksum`](#layout_checksum)
    - [`layout_construct`](#layout_construct)
    - [`layout_find_bottomright`](#layout_find_bottomright)
    - [`layout_check`](#layout_check)
    - [`layout_assign`](#layout_assign)


---
### layout_assign<!-- {{#callable:layout_assign}} -->
The [`layout_assign`](#layout_assign) function assigns window panes to layout cells based on the cell type, recursively handling nested cells.
- **Inputs**:
    - `wp`: A pointer to a pointer to a `window_pane` structure, representing the current window pane to be assigned.
    - `lc`: A pointer to a `layout_cell` structure, representing the current layout cell to which a window pane may be assigned.
- **Control Flow**:
    - The function checks the type of the layout cell `lc`.
    - If `lc` is of type `LAYOUT_WINDOWPANE`, it calls `layout_make_leaf` to assign the current window pane `*wp` to the cell `lc`, then updates `*wp` to the next window pane in the queue.
    - If `lc` is of type `LAYOUT_LEFTRIGHT` or `LAYOUT_TOPBOTTOM`, it iterates over each child cell of `lc` and recursively calls [`layout_assign`](#layout_assign) for each child cell.
- **Output**:
    - The function does not return a value; it modifies the layout by assigning window panes to layout cells.
- **Functions called**:
    - [`layout_assign`](#layout_assign)


---
### layout_construct<!-- {{#callable:layout_construct}} -->
The [`layout_construct`](#layout_construct) function parses a layout string to construct a hierarchical layout cell structure, representing a window layout.
- **Inputs**:
    - `lcparent`: A pointer to the parent layout cell, which will contain the newly constructed cell.
    - `layout`: A pointer to a string representing the layout, which is parsed to create the layout cell structure.
- **Control Flow**:
    - Check if the layout string starts with a digit; if not, return NULL.
    - Parse the layout string for dimensions and offsets using `sscanf`; if parsing fails, return NULL.
    - Advance the layout pointer past the parsed numbers and check for expected delimiters ('x', ','); if any are missing, return NULL.
    - Create a new layout cell using `layout_create_cell` and set its dimensions and offsets.
    - Determine the type of layout (LAYOUT_LEFTRIGHT or LAYOUT_TOPBOTTOM) based on the next character in the layout string.
    - Recursively construct child layout cells if the layout type is LAYOUT_LEFTRIGHT or LAYOUT_TOPBOTTOM, inserting them into the parent cell's list of children.
    - Check for the correct closing character ('}' for LAYOUT_LEFTRIGHT, ']' for LAYOUT_TOPBOTTOM); if incorrect, go to fail.
    - Return the constructed layout cell if successful, or free the cell and return NULL on failure.
- **Output**:
    - A pointer to the newly constructed layout cell, or NULL if the layout string is invalid or an error occurs during construction.
- **Functions called**:
    - [`layout_construct`](#layout_construct)


# Function Declarations (Public API)

---
### layout_find_bottomright<!-- {{#callable_declaration:layout_find_bottomright}} -->
Finds the bottom-right cell in a layout.
- **Description**: This function is used to locate the bottom-right cell within a given layout structure. It is typically called when there is a need to identify the last cell in a layout, which may be necessary for operations such as resizing or rearranging layout components. The function assumes that the input layout cell is part of a valid layout structure and will recursively traverse the layout to find the bottom-right cell. It is important to ensure that the input cell is not null and is part of a properly initialized layout.
- **Inputs**:
    - `lc`: A pointer to a layout_cell structure representing the starting point for the search. Must not be null and should be part of a valid layout structure. The function will handle the traversal internally.
- **Output**: Returns a pointer to the bottom-right layout_cell in the layout structure.
- **See also**: [`layout_find_bottomright`](#layout_find_bottomright)  (Implementation)


---
### layout_checksum<!-- {{#callable_declaration:layout_checksum}} -->
Calculates a checksum for a given layout string.
- **Description**: Use this function to compute a checksum for a layout string, which can be used to verify the integrity or consistency of the layout data. This function processes each character in the input string to generate a checksum value. It is important to ensure that the input string is null-terminated and valid, as the function iterates over the string until it reaches the null character. The function does not handle null pointers, so the input must not be null.
- **Inputs**:
    - `layout`: A pointer to a null-terminated string representing the layout. The string must be valid and not null, as the function does not perform null checks.
- **Output**: Returns a 16-bit unsigned short representing the checksum of the input layout string.
- **See also**: [`layout_checksum`](#layout_checksum)  (Implementation)


---
### layout_append<!-- {{#callable_declaration:layout_append}} -->
Appends layout cell information to a buffer.
- **Description**: This function is used to append the string representation of a layout cell to a provided buffer, which is typically used to serialize the layout structure. It should be called with a valid layout cell and a buffer with sufficient space to hold the resulting string. The function handles different types of layout cells, including window panes and composite layouts, and appends the appropriate string representation. It returns an error if the buffer is too small or if any other issue occurs during the string construction.
- **Inputs**:
    - `lc`: A pointer to a layout_cell structure representing the layout cell to be serialized. Must not be null.
    - `buf`: A character buffer where the layout string will be appended. Must not be null and should have enough space to accommodate the appended data.
    - `len`: The total size of the buffer in bytes. Must be greater than zero.
- **Output**: Returns 0 on success, or -1 if an error occurs, such as if the buffer is too small to hold the appended data.
- **See also**: [`layout_append`](#layout_append)  (Implementation)


---
### layout_construct<!-- {{#callable_declaration:layout_construct}} -->
Constructs a layout cell from a layout string.
- **Description**: This function constructs a layout cell tree from a given layout string, which describes the dimensions and offsets of the layout. It is used to parse and build a hierarchical structure of layout cells, which can represent complex window arrangements. The function expects a valid layout string format and a parent layout cell, if applicable. It returns a pointer to the root of the constructed layout cell tree or NULL if the layout string is invalid or parsing fails.
- **Inputs**:
    - `lcparent`: A pointer to the parent layout cell, or NULL if this is the root cell. The caller retains ownership and must ensure it is valid.
    - `layout`: A pointer to a string representing the layout. The string must be in a specific format, containing dimensions and offsets, and must not be NULL. The function will parse and modify this string as it processes it.
- **Output**: Returns a pointer to the constructed layout cell if successful, or NULL if the layout string is invalid or an error occurs during construction.
- **See also**: [`layout_construct`](#layout_construct)  (Implementation)


---
### layout_assign<!-- {{#callable_declaration:layout_assign}} -->
Assigns window panes to layout cells.
- **Description**: This function is used to recursively assign window panes to the appropriate layout cells within a layout tree. It should be called when you need to map panes to a predefined layout structure. The function expects a valid layout cell structure and a pointer to a window pane. It traverses the layout tree, assigning panes to leaf cells and updating the pointer to the next pane. This function must be used in contexts where the layout and panes are already initialized and valid.
- **Inputs**:
    - `wp`: A pointer to a pointer to a window pane. This must not be null and should point to the first pane to be assigned. The pointer is updated to point to the next pane after assignment.
    - `lc`: A pointer to a layout cell structure representing the root of the layout tree. This must not be null and should be a valid layout cell with a defined type.
- **Output**: None
- **See also**: [`layout_assign`](#layout_assign)  (Implementation)


