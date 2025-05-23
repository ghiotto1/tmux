# Purpose
This C source code file is part of the tmux terminal multiplexer project, specifically handling the management and manipulation of screen objects within tmux. The code provides a comprehensive set of functions to initialize, reinitialize, resize, and manage the state of a screen, which is a fundamental component in tmux for displaying terminal content. The file defines several key structures, such as `screen_sel` for managing selected areas and `screen_title_entry` for handling a stack of screen titles. It includes functions for setting and clearing selections, managing screen titles, handling cursor styles and colors, and managing alternate screen modes, which are essential for terminal emulation.

The code is designed to be integrated into the larger tmux application, providing specific functionality related to screen management. It does not define a public API or external interfaces directly but rather serves as an internal component of tmux. The functions are primarily focused on manipulating the screen's grid, handling selections, and managing screen states like alternate modes and cursor properties. The file includes static functions for internal operations, such as resizing and reflowing the screen, and public functions for initializing and freeing screen resources. This modular approach allows tmux to efficiently manage multiple terminal sessions, providing users with a robust and flexible terminal multiplexing experience.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `unistd.h`
- `tmux.h`


# Data Structures

---
### screen_sel
- **Type**: `struct`
- **Members**:
    - `hidden`: Indicates whether the selection is hidden.
    - `rectangle`: Specifies if the selection is rectangular.
    - `modekeys`: Defines the mode keys used for selection.
    - `sx`: The starting x-coordinate of the selection.
    - `sy`: The starting y-coordinate of the selection.
    - `ex`: The ending x-coordinate of the selection.
    - `ey`: The ending y-coordinate of the selection.
    - `cell`: Represents the grid cell associated with the selection.
- **Description**: The `screen_sel` structure is used to define a selected area within a screen, typically in a terminal or text-based interface. It includes information about whether the selection is hidden, if it is rectangular, and the mode keys used. The structure also stores the starting and ending coordinates of the selection, as well as a `grid_cell` structure that represents the visual attributes of the selected area.


---
### screen_title_entry
- **Type**: `struct`
- **Members**:
    - `text`: A pointer to a character array (string) representing the text of the title entry.
    - `entry`: A TAILQ_ENTRY macro that links this entry into a tail queue of screen_title_entry structures.
- **Description**: The `screen_title_entry` structure is used to represent an entry in a stack of screen titles. It contains a pointer to a string (`text`) that holds the title text and a `TAILQ_ENTRY` (`entry`) which allows it to be part of a doubly-linked tail queue. This structure is part of a larger system for managing a stack of screen titles, where each entry can be pushed or popped from the stack, allowing for dynamic title management in a terminal-like environment.


# Functions

---
### screen_free_titles<!-- {{#callable:screen_free_titles}} -->
The `screen_free_titles` function deallocates memory associated with the title stack of a screen structure.
- **Inputs**:
    - `s`: A pointer to a `struct screen`, which contains a linked list of title entries to be freed.
- **Control Flow**:
    - Check if the `titles` field of the screen structure is NULL; if so, return immediately as there is nothing to free.
    - Iterate over the linked list of title entries using `TAILQ_FIRST` to get the first entry and `TAILQ_REMOVE` to remove it from the list.
    - For each title entry, free the memory allocated for the `text` field and then free the title entry itself.
    - After all entries are removed and freed, free the `titles` list itself and set the `titles` pointer to NULL.
- **Output**:
    - The function does not return any value; it performs memory deallocation to prevent memory leaks.


---
### screen_init<!-- {{#callable:screen_init}} -->
The `screen_init` function initializes a `screen` structure with specified dimensions and default settings.
- **Inputs**:
    - `s`: A pointer to a `screen` structure that will be initialized.
    - `sx`: An unsigned integer representing the width of the screen grid.
    - `sy`: An unsigned integer representing the height of the screen grid.
    - `hlimit`: An unsigned integer representing the history limit for the screen grid.
- **Control Flow**:
    - Create a grid for the screen using `grid_create` with the specified width, height, and history limit.
    - Set the `saved_grid` pointer to NULL, indicating no saved grid initially.
    - Initialize the screen's title to an empty string and set the `titles` and `path` pointers to NULL.
    - Set the cursor style and mode to default values, and initialize color settings to default values.
    - Initialize the `tabs` and `sel` pointers to NULL, indicating no tab stops or selection initially.
    - If the `ENABLE_SIXEL` macro is defined, initialize the `images` queue.
    - Set the `write_list` and `hyperlinks` pointers to NULL, indicating no write operations or hyperlinks initially.
    - Call [`screen_reinit`](#screen_reinit) to further initialize or reset the screen state.
- **Output**:
    - The function does not return a value; it initializes the provided `screen` structure in place.
- **Functions called**:
    - [`screen_reinit`](#screen_reinit)


---
### screen_reinit<!-- {{#callable:screen_reinit}} -->
The `screen_reinit` function resets a screen structure to its initial state, clearing various settings and data.
- **Inputs**:
    - `s`: A pointer to a `struct screen` which represents the screen to be reinitialized.
- **Control Flow**:
    - Set the cursor position `cx` and `cy` to 0.
    - Set the scroll region `rupper` to 0 and `rlower` to the height of the screen minus 1.
    - Set the screen mode to include `MODE_CURSOR` and `MODE_WRAP`, and retain `MODE_CRLF` if it was previously set.
    - Check if the 'extended-keys' option is set to 2, and if so, adjust the mode to include `MODE_KEYS_EXTENDED` while removing `EXTENDED_KEY_MODES`.
    - If the screen is in alternate mode, call [`screen_alternate_off`](#screen_alternate_off) to exit it.
    - Set `saved_cx` and `saved_cy` to `UINT_MAX` to indicate no saved cursor position.
    - Call [`screen_reset_tabs`](#screen_reset_tabs) to reset tab stops to default positions.
    - Clear the grid lines from the screen using `grid_clear_lines`.
    - Clear any current selection on the screen with [`screen_clear_selection`](#screen_clear_selection).
    - Free any titles associated with the screen using [`screen_free_titles`](#screen_free_titles).
    - If SIXEL support is enabled, free all images associated with the screen using `image_free_all`.
    - Reset hyperlinks associated with the screen using [`screen_reset_hyperlinks`](#screen_reset_hyperlinks).
- **Output**:
    - The function does not return a value; it modifies the screen structure in place.
- **Functions called**:
    - [`screen_alternate_off`](#screen_alternate_off)
    - [`screen_reset_tabs`](#screen_reset_tabs)
    - [`screen_clear_selection`](#screen_clear_selection)
    - [`screen_free_titles`](#screen_free_titles)
    - [`screen_reset_hyperlinks`](#screen_reset_hyperlinks)


---
### screen_reset_hyperlinks<!-- {{#callable:screen_reset_hyperlinks}} -->
The `screen_reset_hyperlinks` function initializes or resets the hyperlinks associated with a given screen structure.
- **Inputs**:
    - `s`: A pointer to a `struct screen` which represents the screen whose hyperlinks are to be reset or initialized.
- **Control Flow**:
    - Check if the `hyperlinks` member of the screen structure `s` is `NULL`.
    - If `hyperlinks` is `NULL`, initialize it using `hyperlinks_init()`.
    - If `hyperlinks` is not `NULL`, reset it using `hyperlinks_reset()`.
- **Output**:
    - The function does not return a value; it modifies the `hyperlinks` member of the `struct screen` in place.


---
### screen_free<!-- {{#callable:screen_free}} -->
The `screen_free` function deallocates memory and resources associated with a `screen` structure.
- **Inputs**:
    - `s`: A pointer to a `screen` structure that holds various attributes and resources related to a screen, such as selection, tabs, path, title, write list, grid, saved grid, hyperlinks, and possibly images.
- **Control Flow**:
    - Free the memory allocated for the screen's selection (`s->sel`).
    - Free the memory allocated for the screen's tabs (`s->tabs`).
    - Free the memory allocated for the screen's path (`s->path`).
    - Free the memory allocated for the screen's title (`s->title`).
    - If the screen's write list (`s->write_list`) is not NULL, call `screen_write_free_list` to free it.
    - If the screen is in alternate mode (`SCREEN_IS_ALTERNATE(s)`), destroy the saved grid (`s->saved_grid`).
    - Destroy the main grid (`s->grid`).
    - If the screen's hyperlinks (`s->hyperlinks`) are not NULL, free them using `hyperlinks_free`.
    - Free the titles stack using [`screen_free_titles`](#screen_free_titles).
    - If the `ENABLE_SIXEL` preprocessor directive is defined, free all images associated with the screen using `image_free_all`.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the `screen` structure to prevent memory leaks.
- **Functions called**:
    - [`screen_free_titles`](#screen_free_titles)


---
### screen_reset_tabs<!-- {{#callable:screen_reset_tabs}} -->
The `screen_reset_tabs` function resets the tab stops of a screen to default positions, spaced eight columns apart.
- **Inputs**:
    - `s`: A pointer to a `screen` structure, representing the screen whose tab stops are to be reset.
- **Control Flow**:
    - Free the existing tab stops array in the screen structure `s`.
    - Allocate a new bit array for tab stops with a size equal to the width of the screen (`screen_size_x(s)`).
    - If the allocation fails, terminate the program with a fatal error message.
    - Iterate over the screen width starting from column 8, setting a tab stop every 8 columns using the `bit_set` function.
- **Output**:
    - The function does not return a value; it modifies the `tabs` field of the `screen` structure in place.


---
### screen_set_default_cursor<!-- {{#callable:screen_set_default_cursor}} -->
The function `screen_set_default_cursor` sets the default cursor color and style for a given screen based on options provided.
- **Inputs**:
    - `s`: A pointer to a `struct screen` which represents the screen whose default cursor settings are to be configured.
    - `oo`: A pointer to a `struct options` which contains the configuration options for cursor color and style.
- **Control Flow**:
    - Retrieve the cursor color from the options using `options_get_number` and assign it to `s->default_ccolour`.
    - Retrieve the cursor style from the options using `options_get_number`.
    - Initialize `s->default_mode` to 0.
    - Call [`screen_set_cursor_style`](#screen_set_cursor_style) with the retrieved cursor style, a pointer to `s->default_cstyle`, and a pointer to `s->default_mode` to set the cursor style and mode.
- **Output**:
    - The function does not return a value; it modifies the `screen` structure pointed to by `s` to set its default cursor color and style.
- **Functions called**:
    - [`screen_set_cursor_style`](#screen_set_cursor_style)


---
### screen_set_cursor_style<!-- {{#callable:screen_set_cursor_style}} -->
The `screen_set_cursor_style` function sets the cursor style and mode based on the provided style code.
- **Inputs**:
    - `style`: An unsigned integer representing the desired cursor style, where each value corresponds to a specific cursor appearance and behavior.
    - `cstyle`: A pointer to an enum `screen_cursor_style` where the function will store the resulting cursor style.
    - `mode`: A pointer to an integer representing the current mode, which will be modified to include or exclude the blinking attribute based on the style.
- **Control Flow**:
    - The function uses a switch statement to determine the action based on the `style` value.
    - For each case from 0 to 6, it sets the `cstyle` to a specific cursor style (default, block, underline, or bar).
    - For styles 1, 3, and 5, it sets the `MODE_CURSOR_BLINKING` bit in `mode` using the bitwise OR operator.
    - For styles 2, 4, and 6, it clears the `MODE_CURSOR_BLINKING` bit in `mode` using the bitwise AND NOT operator.
- **Output**:
    - The function modifies the values pointed to by `cstyle` and `mode` to reflect the selected cursor style and blinking mode.


---
### screen_set_cursor_colour<!-- {{#callable:screen_set_cursor_colour}} -->
The function `screen_set_cursor_colour` sets the cursor color of a given screen to a specified color value.
- **Inputs**:
    - `s`: A pointer to a `struct screen` which represents the screen whose cursor color is to be set.
    - `colour`: An integer representing the color value to set for the cursor.
- **Control Flow**:
    - The function assigns the provided color value to the `ccolour` field of the `screen` structure pointed to by `s`.
- **Output**:
    - The function does not return any value; it modifies the `ccolour` field of the `screen` structure in place.


---
### screen_set_title<!-- {{#callable:screen_set_title}} -->
The `screen_set_title` function sets the title of a screen object to a new UTF-8 validated string.
- **Inputs**:
    - `s`: A pointer to a `struct screen` object whose title is to be set.
    - `title`: A constant character pointer to the new title string to be set for the screen.
- **Control Flow**:
    - Check if the provided title string is a valid UTF-8 string using `utf8_isvalid`; if not, return 0.
    - Free the current title of the screen object to prevent memory leaks.
    - Duplicate the new title string using `xstrdup` and assign it to the screen's title.
    - Return 1 to indicate successful title setting.
- **Output**:
    - Returns an integer: 1 if the title was successfully set, or 0 if the title was not a valid UTF-8 string.


---
### screen_set_path<!-- {{#callable:screen_set_path}} -->
The `screen_set_path` function sets the path for a screen object by freeing the existing path and converting a new path string to a specific format.
- **Inputs**:
    - `s`: A pointer to a `screen` structure whose path is to be set.
    - `path`: A constant character pointer representing the new path to be set for the screen.
- **Control Flow**:
    - Free the memory allocated for the current path in the screen structure `s`.
    - Convert the input `path` string to a format suitable for the screen's path using `utf8_stravis`, with specific visibility flags, and assign it to `s->path`.
- **Output**:
    - The function does not return a value; it modifies the `path` member of the `screen` structure in place.


---
### screen_push_title<!-- {{#callable:screen_push_title}} -->
The `screen_push_title` function pushes the current screen title onto a stack of titles for a given screen.
- **Inputs**:
    - `s`: A pointer to a `struct screen` which represents the screen whose title is to be pushed onto the stack.
- **Control Flow**:
    - Check if the `titles` stack in the screen structure is `NULL`; if so, allocate memory for it and initialize it as a tail queue.
    - Allocate memory for a new `screen_title_entry` structure to hold the current title.
    - Duplicate the current screen title string and assign it to the `text` field of the new `screen_title_entry`.
    - Insert the new `screen_title_entry` at the head of the `titles` tail queue.
- **Output**:
    - The function does not return any value; it modifies the `titles` stack within the provided `screen` structure.


---
### screen_pop_title<!-- {{#callable:screen_pop_title}} -->
The `screen_pop_title` function removes the top title from a screen's title stack and sets it as the current screen title.
- **Inputs**:
    - `s`: A pointer to a `struct screen` which contains the title stack and other screen-related data.
- **Control Flow**:
    - Check if the `titles` stack in the screen `s` is NULL; if so, return immediately.
    - Retrieve the first title entry from the `titles` stack using `TAILQ_FIRST`.
    - If a title entry is found, set the screen's title to the text of the title entry using [`screen_set_title`](#screen_set_title).
    - Remove the title entry from the `titles` stack using `TAILQ_REMOVE`.
    - Free the memory allocated for the title entry's text and the title entry itself.
- **Output**:
    - The function does not return a value; it modifies the screen's title and the title stack in place.
- **Functions called**:
    - [`screen_set_title`](#screen_set_title)


---
### screen_resize_cursor<!-- {{#callable:screen_resize_cursor}} -->
The `screen_resize_cursor` function resizes a screen's dimensions and adjusts the cursor position accordingly, with optional reflow and empty line handling.
- **Inputs**:
    - `s`: A pointer to a `screen` structure representing the screen to be resized.
    - `sx`: An unsigned integer representing the new width of the screen.
    - `sy`: An unsigned integer representing the new height of the screen.
    - `reflow`: An integer flag indicating whether to reflow the screen content when resizing.
    - `eat_empty`: An integer flag indicating whether to remove empty lines from the bottom when resizing.
    - `cursor`: An integer flag indicating whether to adjust the cursor position during reflow.
- **Control Flow**:
    - Initialize local variables `cx` and `cy` to the current cursor position and the sum of the grid's history size and current y position, respectively.
    - If the screen's write list is not NULL, free it using `screen_write_free_list`.
    - Log the current and new screen sizes and cursor positions for debugging purposes.
    - Ensure the new screen dimensions `sx` and `sy` are at least 1.
    - If the new width `sx` differs from the current width, update the grid's width, reset tabs, and set `reflow` to 0 if the width hasn't changed.
    - If the new height `sy` differs from the current height, call [`screen_resize_y`](#screen_resize_y) to adjust the height and update `cy`.
    - If the `ENABLE_SIXEL` flag is defined, free all images associated with the screen.
    - If `reflow` is true, call [`screen_reflow`](#screen_reflow) to adjust the screen content and cursor position based on the new width.
    - Adjust the cursor position based on the updated `cy` value, setting it to the new position relative to the grid's history size.
    - Log the final cursor position for debugging purposes.
    - If the screen's write list is not NULL, recreate it using `screen_write_make_list`.
- **Output**:
    - The function does not return a value; it modifies the screen structure in place, adjusting its size and cursor position.
- **Functions called**:
    - [`screen_reset_tabs`](#screen_reset_tabs)
    - [`screen_resize_y`](#screen_resize_y)
    - [`screen_reflow`](#screen_reflow)


---
### screen_resize<!-- {{#callable:screen_resize}} -->
The `screen_resize` function resizes a screen to specified dimensions and optionally reflows its content.
- **Inputs**:
    - `s`: A pointer to a `screen` structure representing the screen to be resized.
    - `sx`: An unsigned integer representing the new width of the screen.
    - `sy`: An unsigned integer representing the new height of the screen.
    - `reflow`: An integer flag indicating whether the screen content should be reflowed to fit the new dimensions.
- **Control Flow**:
    - The function calls [`screen_resize_cursor`](#screen_resize_cursor) with the provided screen, width, height, reflow flag, and additional parameters `1` and `1` for `eat_empty` and `cursor`, respectively.
    - The [`screen_resize_cursor`](#screen_resize_cursor) function handles the actual resizing and reflowing of the screen content.
- **Output**:
    - The function does not return a value; it modifies the screen structure in place.
- **Functions called**:
    - [`screen_resize_cursor`](#screen_resize_cursor)


---
### screen_resize_y<!-- {{#callable:screen_resize_y}} -->
The `screen_resize_y` function adjusts the vertical size of a screen by either deleting lines or adding blank lines, while managing the screen's history and cursor position.
- **Inputs**:
    - `s`: A pointer to a `screen` structure representing the screen to be resized.
    - `sy`: An unsigned integer representing the new vertical size of the screen.
    - `eat_empty`: An integer flag indicating whether to delete empty lines from the bottom when reducing size.
    - `cy`: A pointer to an unsigned integer representing the current cursor y-position, which may be adjusted during resizing.
- **Control Flow**:
    - Check if the new size `sy` is zero and terminate with an error if true.
    - Store the current vertical size of the screen in `oldy`.
    - If the new size `sy` is less than `oldy`, calculate the number of lines to delete (`needed`) and attempt to delete lines from the bottom if `eat_empty` is true.
    - If there are still lines to delete after removing from the bottom, increase the history size if history is enabled, or delete lines from the top if not.
    - Adjust the grid's line array to accommodate the new size.
    - If the new size `sy` is greater than `oldy`, calculate the number of lines to add (`needed`) and attempt to pull lines from the scrolled history if enabled.
    - Fill any remaining needed lines with blanks at the bottom of the screen.
    - Set the new vertical size of the grid and reset the scroll region.
- **Output**:
    - The function does not return a value but modifies the screen's grid and cursor position to reflect the new vertical size.


---
### screen_set_selection<!-- {{#callable:screen_set_selection}} -->
The `screen_set_selection` function initializes or updates the selection area on a screen with specified coordinates, rectangle mode, mode keys, and grid cell attributes.
- **Inputs**:
    - `s`: A pointer to a `struct screen` which represents the screen where the selection is to be set.
    - `sx`: An unsigned integer representing the starting x-coordinate of the selection.
    - `sy`: An unsigned integer representing the starting y-coordinate of the selection.
    - `ex`: An unsigned integer representing the ending x-coordinate of the selection.
    - `ey`: An unsigned integer representing the ending y-coordinate of the selection.
    - `rectangle`: An unsigned integer indicating whether the selection is rectangular (non-zero) or not (zero).
    - `modekeys`: An integer representing the mode keys used for the selection, which can affect how the selection behaves.
    - `gc`: A pointer to a `struct grid_cell` that contains the attributes of the grid cell to be used for the selection.
- **Control Flow**:
    - Check if the selection (`s->sel`) is NULL; if so, allocate memory for it using `xcalloc`.
    - Copy the grid cell attributes from `gc` to `s->sel->cell` using `memcpy`.
    - Set the `hidden` attribute of the selection to 0, making it visible.
    - Set the `rectangle` attribute of the selection to the provided `rectangle` value.
    - Set the `modekeys` attribute of the selection to the provided `modekeys` value.
    - Assign the starting and ending coordinates (`sx`, `sy`, `ex`, `ey`) to the corresponding attributes of the selection.
- **Output**:
    - The function does not return a value; it modifies the `screen` structure's selection attributes in place.


---
### screen_clear_selection<!-- {{#callable:screen_clear_selection}} -->
The `screen_clear_selection` function deallocates memory for the selection in a screen and sets the selection pointer to NULL.
- **Inputs**:
    - `s`: A pointer to a `struct screen` which represents the screen whose selection is to be cleared.
- **Control Flow**:
    - The function calls `free` on the `sel` member of the `struct screen` pointed to by `s`, which deallocates the memory used for the selection.
    - It then sets the `sel` member of the `struct screen` to `NULL`, indicating that there is no current selection.
- **Output**:
    - The function does not return any value; it modifies the `struct screen` in place.


---
### screen_hide_selection<!-- {{#callable:screen_hide_selection}} -->
The `screen_hide_selection` function hides the current selection on a screen by setting its `hidden` attribute to 1.
- **Inputs**:
    - `s`: A pointer to a `struct screen`, which represents the screen whose selection is to be hidden.
- **Control Flow**:
    - Check if the `sel` attribute of the screen `s` is not NULL.
    - If `sel` is not NULL, set its `hidden` attribute to 1, effectively hiding the selection.
- **Output**:
    - The function does not return any value; it modifies the `hidden` attribute of the `sel` structure within the `screen`.


---
### screen_check_selection<!-- {{#callable:screen_check_selection}} -->
The `screen_check_selection` function determines if a given cell (px, py) is within the current selection on a screen.
- **Inputs**:
    - `s`: A pointer to a `screen` structure, which contains the selection information.
    - `px`: The x-coordinate of the cell to check.
    - `py`: The y-coordinate of the cell to check.
- **Control Flow**:
    - Check if the selection (`sel`) is NULL or hidden; if so, return 0 (not selected).
    - If the selection is a rectangle, determine if the y-coordinate (py) is within the selection's vertical bounds, considering the direction of selection (upward, downward, or single line).
    - For rectangle selection, check if the x-coordinate (px) is within the horizontal bounds of the selection, adjusting for the direction of selection (left or right).
    - If the selection is not a rectangle, handle it like Emacs, keeping the top-left-most character and dropping the bottom-right-most, adjusting for the mode keys (Emacs or not).
    - For non-rectangle selection, check if the y-coordinate (py) is within the selection's vertical bounds, and adjust the x-coordinate (px) checks based on the mode keys and selection direction.
    - Return 1 if the cell is within the selection, otherwise return 0.
- **Output**:
    - Returns 1 if the cell (px, py) is within the selection, otherwise returns 0.


---
### screen_select_cell<!-- {{#callable:screen_select_cell}} -->
The `screen_select_cell` function copies a selected grid cell from a screen's selection to a destination cell, applying default colors and attributes from a source cell if necessary.
- **Inputs**:
    - `s`: A pointer to a `struct screen` which contains the screen's state and selection information.
    - `dst`: A pointer to a `struct grid_cell` where the selected cell's data will be copied to.
    - `src`: A pointer to a `const struct grid_cell` which provides default color and attribute values if the selected cell has default colors.
- **Control Flow**:
    - Check if the screen's selection (`s->sel`) is NULL or hidden; if so, return immediately without doing anything.
    - Copy the selected cell (`s->sel->cell`) into the destination cell (`dst`).
    - If the foreground color of the destination cell is default, set it to the foreground color of the source cell.
    - If the background color of the destination cell is default, set it to the background color of the source cell.
    - Copy the UTF-8 data from the source cell to the destination cell.
    - Clear the charset attribute from the destination cell's attributes and set it to the charset attribute from the source cell.
    - Copy the flags from the source cell to the destination cell.
- **Output**:
    - The function does not return a value; it modifies the `dst` grid cell in place.


---
### screen_reflow<!-- {{#callable:screen_reflow}} -->
The `screen_reflow` function adjusts the layout of a screen's grid to accommodate a new width, optionally repositioning the cursor.
- **Inputs**:
    - `s`: A pointer to a `screen` structure representing the screen to be reflowed.
    - `new_x`: An unsigned integer representing the new width of the screen grid.
    - `cx`: A pointer to an unsigned integer representing the current x-coordinate of the cursor.
    - `cy`: A pointer to an unsigned integer representing the current y-coordinate of the cursor.
    - `cursor`: An integer flag indicating whether the cursor position should be adjusted (non-zero) or not (zero).
- **Control Flow**:
    - If the `cursor` flag is set, the current cursor position (`cx`, `cy`) is wrapped to grid coordinates (`wx`, `wy`) using `grid_wrap_position`.
    - A debug log is generated to record the current cursor position.
    - The grid is reflowed to the new width `new_x` using `grid_reflow`.
    - If the `cursor` flag is set, the cursor position is unwrapped back to screen coordinates using `grid_unwrap_position`, and a debug log is generated for the new cursor position.
    - If the `cursor` flag is not set, the cursor is reset to the top-left corner of the screen (`cx` = 0, `cy` = `s->grid->hsize`).
- **Output**:
    - The function does not return a value; it modifies the screen's grid and potentially the cursor position in place.


---
### screen_alternate_on<!-- {{#callable:screen_alternate_on}} -->
The `screen_alternate_on` function enables the alternate screen mode by saving the current screen state and clearing the visible grid.
- **Inputs**:
    - `s`: A pointer to a `screen` structure representing the current screen state.
    - `gc`: A pointer to a `grid_cell` structure representing the current grid cell attributes to be saved.
    - `cursor`: An integer flag indicating whether to save the current cursor position.
- **Control Flow**:
    - Check if the screen is already in alternate mode using `SCREEN_IS_ALTERNATE(s)`; if true, return immediately.
    - Retrieve the current screen dimensions using `screen_size_x(s)` and `screen_size_y(s)`.
    - Create a new grid for saving the current screen state with `grid_create(sx, sy, 0)`.
    - Duplicate the current visible lines from the existing grid to the saved grid using `grid_duplicate_lines`.
    - If the `cursor` flag is set, save the current cursor position (`s->cx` and `s->cy`).
    - Copy the current grid cell attributes from `gc` to `s->saved_cell`.
    - Clear the visible grid using `grid_view_clear` to prepare for the alternate screen.
    - Save the current grid flags and disable the history flag by modifying `s->grid->flags`.
- **Output**:
    - The function does not return a value; it modifies the `screen` structure to enable alternate screen mode.


---
### screen_alternate_off<!-- {{#callable:screen_alternate_off}} -->
The `screen_alternate_off` function exits the alternate screen mode, restoring the saved grid and cursor position, and resizes the screen back to its original dimensions.
- **Inputs**:
    - `s`: A pointer to a `screen` structure representing the current screen state.
    - `gc`: A pointer to a `grid_cell` structure where the saved cell state will be restored, if not NULL.
    - `cursor`: An integer flag indicating whether to restore the cursor position and cell state.
- **Control Flow**:
    - Retrieve the current screen dimensions using `screen_size_x` and `screen_size_y`.
    - Check if the screen is in alternate mode using `SCREEN_IS_ALTERNATE`; if so, resize the screen to the saved grid's dimensions.
    - If the `cursor` flag is set and saved cursor positions are valid, restore the cursor position and optionally copy the saved cell state to `gc`.
    - If the screen is not in alternate mode, adjust the cursor position to ensure it is within the screen bounds and return.
    - Duplicate the saved grid lines back to the current grid using `grid_duplicate_lines`.
    - Restore the grid's history flag if it was previously set, and resize the screen back to its original dimensions.
    - Destroy the saved grid and set the `saved_grid` pointer to NULL.
    - Ensure the cursor position is within the current screen bounds.
- **Output**:
    - The function does not return a value; it modifies the screen state in place.
- **Functions called**:
    - [`screen_resize`](#screen_resize)


---
### screen_mode_to_string<!-- {{#callable:screen_mode_to_string}} -->
The function `screen_mode_to_string` converts a given screen mode integer into a human-readable string representation of the active modes.
- **Inputs**:
    - `mode`: An integer representing the screen mode, where each bit corresponds to a specific mode flag.
- **Control Flow**:
    - Initialize a static character array `tmp` to store the resulting string.
    - Check if the mode is 0, and if so, return "NONE".
    - Check if the mode is equal to `ALL_MODES`, and if so, return "ALL".
    - Initialize `tmp` to an empty string.
    - For each mode flag (e.g., `MODE_CURSOR`, `MODE_INSERT`, etc.), check if the corresponding bit is set in `mode`.
    - If a bit is set, append the corresponding mode name (e.g., "CURSOR,") to `tmp` using `strlcat`.
    - After processing all mode flags, replace the last comma in `tmp` with a null terminator to properly end the string.
    - Return the resulting string `tmp`.
- **Output**:
    - A string representing the active modes, separated by commas, or "NONE" or "ALL" if applicable.


# Function Declarations (Public API)

---
### screen_resize_y<!-- {{#callable_declaration:screen_resize_y}} -->
Adjusts the vertical size of a screen.
- **Description**: This function is used to change the vertical size of a screen, either increasing or decreasing its height. When the height is reduced, lines are removed from the bottom, and if necessary, lines are pushed into the history buffer. When the height is increased, lines are pulled from the history buffer if available, or filled with blank lines. This function should be called when the screen's vertical size needs to be adjusted, ensuring that the screen's content and history are managed appropriately. It must not be called with a zero size, as this will result in a fatal error.
- **Inputs**:
    - `s`: A pointer to a 'screen' structure representing the screen to be resized. Must not be null.
    - `sy`: An unsigned integer representing the new vertical size of the screen. Must be greater than zero.
    - `eat_empty`: An integer flag indicating whether to remove empty lines from the bottom of the screen when decreasing size. Non-zero values enable this behavior.
    - `cy`: A pointer to an unsigned integer representing the current cursor y-position. This value may be adjusted based on the resizing operation. Must not be null.
- **Output**: None
- **See also**: [`screen_resize_y`](#screen_resize_y)  (Implementation)


---
### screen_reflow<!-- {{#callable_declaration:screen_reflow}} -->
Reflows the screen to a new width, adjusting the cursor position if necessary.
- **Description**: This function adjusts the layout of a screen to accommodate a new width, specified by `new_x`. It should be used when the screen's width changes, such as during a window resize. If the `cursor` parameter is true, the function will also adjust the cursor position to maintain its logical position relative to the content. The function assumes that the screen's grid is already initialized and valid. It modifies the cursor position parameters `cx` and `cy` based on the new layout.
- **Inputs**:
    - `s`: A pointer to a `struct screen` representing the screen to be reflowed. Must not be null.
    - `new_x`: An unsigned integer specifying the new width of the screen. Must be greater than zero.
    - `cx`: A pointer to an unsigned integer representing the current x-coordinate of the cursor. Must not be null. The value may be modified to reflect the new cursor position.
    - `cy`: A pointer to an unsigned integer representing the current y-coordinate of the cursor. Must not be null. The value may be modified to reflect the new cursor position.
    - `cursor`: An integer flag indicating whether the cursor position should be adjusted. Non-zero values enable cursor adjustment.
- **Output**: None
- **See also**: [`screen_reflow`](#screen_reflow)  (Implementation)


