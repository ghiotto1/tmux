# Purpose
This C source code file is part of the tmux terminal multiplexer project, specifically handling the management and manipulation of screen objects within tmux. The code provides a comprehensive set of functions to initialize, reinitialize, resize, and free screen structures, which are essential for managing the terminal's display state. It includes functionality for handling screen titles, cursor styles, and selections, as well as managing alternate screen modes, which are crucial for applications that require a separate screen buffer, such as text editors or pagers.

The file defines several key structures, such as `screen_sel` for managing selected areas on the screen and `screen_title_entry` for handling a stack of screen titles. It also includes functions for setting and clearing selections, managing cursor styles and colors, and handling screen resizing with options for reflowing text. The code is designed to be integrated into the larger tmux application, providing essential screen management capabilities that allow tmux to efficiently handle multiple terminal sessions. The functions are primarily intended for internal use within the tmux codebase, as indicated by the static keyword for some functions, suggesting they are not part of a public API.
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
    - `cell`: A grid_cell structure representing the cell properties of the selection.
- **Description**: The `screen_sel` structure is used to represent a selected area on a screen, typically in a terminal or text-based interface. It includes information about whether the selection is hidden, if it is rectangular, and the mode keys used. The structure also stores the starting and ending coordinates of the selection, as well as a `grid_cell` structure that holds the properties of the selected cell, such as its appearance and attributes.


---
### screen_title_entry
- **Type**: `struct`
- **Members**:
    - `text`: A pointer to a character array representing the text of the title entry.
    - `entry`: A TAILQ_ENTRY macro that links this entry into a tail queue of screen_title_entry structures.
- **Description**: The `screen_title_entry` structure is used to represent an entry in a stack of screen titles, where each entry contains a text string representing the title and a link to the next entry in a tail queue. This structure is part of a larger system for managing screen titles in a terminal multiplexer, allowing for operations such as pushing and popping titles from a stack.


# Functions

---
### screen_free_titles<!-- {{#callable:screen_free_titles}} -->
The `screen_free_titles` function deallocates memory associated with the title stack of a screen structure.
- **Inputs**:
    - `s`: A pointer to a `struct screen` which contains a linked list of title entries to be freed.
- **Control Flow**:
    - Check if the `titles` field of the screen structure `s` is NULL; if so, return immediately as there is nothing to free.
    - Iterate over the linked list of title entries using `TAILQ_FIRST` to get the first entry and `TAILQ_REMOVE` to remove it from the list.
    - For each title entry, free the memory allocated for the `text` field and then free the title entry itself.
    - After all entries are freed, free the `titles` list itself and set the `titles` pointer in the screen structure to NULL.
- **Output**:
    - The function does not return any value; it performs memory deallocation for the title stack of the screen.


---
### screen_init<!-- {{#callable:screen_init}} -->
The `screen_init` function initializes a `screen` structure with specified dimensions and history limit, setting default values for various screen attributes.
- **Inputs**:
    - `s`: A pointer to a `screen` structure that will be initialized.
    - `sx`: An unsigned integer representing the width of the screen grid.
    - `sy`: An unsigned integer representing the height of the screen grid.
    - `hlimit`: An unsigned integer representing the history limit for the screen grid.
- **Control Flow**:
    - Create a grid for the screen using [`grid_create`](grid.c.driver.md#grid_create) with the specified width, height, and history limit.
    - Initialize the `saved_grid` pointer to `NULL`.
    - Set the screen's title to an empty string using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - Initialize the `titles` and `path` pointers to `NULL`.
    - Set the cursor style and default cursor style to `SCREEN_CURSOR_DEFAULT`.
    - Set the screen mode to `MODE_CURSOR` and default mode to `0`.
    - Initialize the cursor color and default cursor color to `-1`.
    - Set the `tabs` and `sel` pointers to `NULL`.
    - If `ENABLE_SIXEL` is defined, initialize the `images` queue.
    - Set the `write_list` and `hyperlinks` pointers to `NULL`.
    - Call [`screen_reinit`](#screen_reinit) to reinitialize the screen.
- **Output**:
    - The function does not return a value; it initializes the provided `screen` structure.
- **Functions called**:
    - [`grid_create`](grid.c.driver.md#grid_create)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`screen_reinit`](#screen_reinit)


---
### screen_reinit<!-- {{#callable:screen_reinit}} -->
The `screen_reinit` function resets a screen structure to its initial state, clearing various settings and data.
- **Inputs**:
    - `s`: A pointer to a `struct screen` which represents the screen to be reinitialized.
- **Control Flow**:
    - Set the cursor position to the top-left corner (0,0).
    - Set the scroll region to cover the entire screen height.
    - Initialize the screen mode with cursor and wrap modes, and conditionally set extended key modes based on global options.
    - If the screen is in alternate mode, switch it off.
    - Set saved cursor positions to maximum unsigned integer values, indicating they are unset.
    - Reset tab stops to default positions.
    - Clear all lines in the screen's grid from the history size to the screen size.
    - Clear any active selection on the screen.
    - Free any stored screen titles.
    - If SIXEL support is enabled, free all images associated with the screen.
    - Reset any hyperlinks associated with the screen.
- **Output**:
    - The function does not return a value; it modifies the screen structure in place.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`screen_alternate_off`](#screen_alternate_off)
    - [`screen_reset_tabs`](#screen_reset_tabs)
    - [`grid_clear_lines`](grid.c.driver.md#grid_clear_lines)
    - [`screen_clear_selection`](#screen_clear_selection)
    - [`screen_free_titles`](#screen_free_titles)
    - [`image_free_all`](image.c.driver.md#image_free_all)
    - [`screen_reset_hyperlinks`](#screen_reset_hyperlinks)


---
### screen_reset_hyperlinks<!-- {{#callable:screen_reset_hyperlinks}} -->
The `screen_reset_hyperlinks` function initializes or resets the hyperlinks associated with a given screen structure.
- **Inputs**:
    - `s`: A pointer to a `struct screen` which represents the screen whose hyperlinks are to be reset or initialized.
- **Control Flow**:
    - Check if the `hyperlinks` field of the screen structure `s` is `NULL`.
    - If `hyperlinks` is `NULL`, initialize it using `hyperlinks_init()`.
    - If `hyperlinks` is not `NULL`, reset it using `hyperlinks_reset()`.
- **Output**:
    - The function does not return a value; it modifies the `hyperlinks` field of the provided screen structure `s`.
- **Functions called**:
    - [`hyperlinks_init`](hyperlinks.c.driver.md#hyperlinks_init)
    - [`hyperlinks_reset`](hyperlinks.c.driver.md#hyperlinks_reset)


---
### screen_free<!-- {{#callable:screen_free}} -->
The `screen_free` function deallocates memory and resources associated with a `screen` structure.
- **Inputs**:
    - `s`: A pointer to a `screen` structure whose resources are to be freed.
- **Control Flow**:
    - Free the memory allocated for the selection (`sel`), tabs, path, and title of the screen.
    - If the `write_list` is not NULL, call [`screen_write_free_list`](screen-write.c.driver.md#screen_write_free_list) to free it.
    - If the screen is in alternate mode, destroy the saved grid using [`grid_destroy`](grid.c.driver.md#grid_destroy).
    - Destroy the main grid of the screen using [`grid_destroy`](grid.c.driver.md#grid_destroy).
    - If hyperlinks are present, free them using [`hyperlinks_free`](hyperlinks.c.driver.md#hyperlinks_free).
    - Call [`screen_free_titles`](#screen_free_titles) to free the titles stack.
    - If the `ENABLE_SIXEL` macro is defined, call [`image_free_all`](image.c.driver.md#image_free_all) to free all images associated with the screen.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the `screen` structure.
- **Functions called**:
    - [`screen_write_free_list`](screen-write.c.driver.md#screen_write_free_list)
    - [`grid_destroy`](grid.c.driver.md#grid_destroy)
    - [`hyperlinks_free`](hyperlinks.c.driver.md#hyperlinks_free)
    - [`screen_free_titles`](#screen_free_titles)
    - [`image_free_all`](image.c.driver.md#image_free_all)


---
### screen_reset_tabs<!-- {{#callable:screen_reset_tabs}} -->
The `screen_reset_tabs` function resets the tab stops of a screen to default positions, spaced eight columns apart.
- **Inputs**:
    - `s`: A pointer to a `screen` structure, representing the screen whose tab stops are to be reset.
- **Control Flow**:
    - Free the current tab stops array `s->tabs` to release any previously allocated memory.
    - Allocate a new bit array for tab stops using `bit_alloc` with the width of the screen obtained from `screen_size_x(s)`.
    - If the allocation fails, call `fatal` to handle the error.
    - Iterate over the screen width starting from column 8, setting a tab stop every 8 columns using `bit_set`.
- **Output**:
    - The function does not return a value; it modifies the `tabs` field of the `screen` structure in place.


---
### screen_set_default_cursor<!-- {{#callable:screen_set_default_cursor}} -->
The function `screen_set_default_cursor` sets the default cursor color and style for a given screen based on options provided.
- **Inputs**:
    - `s`: A pointer to a `struct screen` which represents the screen whose default cursor settings are to be configured.
    - `oo`: A pointer to a `struct options` which contains the configuration options for the cursor color and style.
- **Control Flow**:
    - Retrieve the cursor color from the options using [`options_get_number`](options.c.driver.md#options_get_number) and assign it to the screen's `default_ccolour` field.
    - Retrieve the cursor style from the options using [`options_get_number`](options.c.driver.md#options_get_number).
    - Initialize the screen's `default_mode` to 0.
    - Call [`screen_set_cursor_style`](#screen_set_cursor_style) with the retrieved cursor style, and pointers to the screen's `default_cstyle` and `default_mode` to set the cursor style and mode.
- **Output**:
    - The function does not return a value; it modifies the `struct screen` pointed to by `s` to set its default cursor color and style.
- **Functions called**:
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`screen_set_cursor_style`](#screen_set_cursor_style)


---
### screen_set_cursor_style<!-- {{#callable:screen_set_cursor_style}} -->
The `screen_set_cursor_style` function sets the cursor style and mode based on the provided style code.
- **Inputs**:
    - `style`: An unsigned integer representing the desired cursor style, where each value corresponds to a specific cursor appearance and behavior.
    - `cstyle`: A pointer to an enum variable where the function will store the resulting cursor style.
    - `mode`: A pointer to an integer representing the current mode, which will be modified to include or exclude the blinking attribute based on the style.
- **Control Flow**:
    - The function uses a switch statement to determine the action based on the `style` value.
    - For `style` 0, it sets `cstyle` to `SCREEN_CURSOR_DEFAULT`.
    - For `style` 1, it sets `cstyle` to `SCREEN_CURSOR_BLOCK` and enables blinking by setting the `MODE_CURSOR_BLINKING` bit in `mode`.
    - For `style` 2, it sets `cstyle` to `SCREEN_CURSOR_BLOCK` and disables blinking by clearing the `MODE_CURSOR_BLINKING` bit in `mode`.
    - For `style` 3, it sets `cstyle` to `SCREEN_CURSOR_UNDERLINE` and enables blinking.
    - For `style` 4, it sets `cstyle` to `SCREEN_CURSOR_UNDERLINE` and disables blinking.
    - For `style` 5, it sets `cstyle` to `SCREEN_CURSOR_BAR` and enables blinking.
    - For `style` 6, it sets `cstyle` to `SCREEN_CURSOR_BAR` and disables blinking.
- **Output**:
    - The function does not return a value; it modifies the values pointed to by `cstyle` and `mode` to reflect the selected cursor style and mode.


---
### screen_set_cursor_colour<!-- {{#callable:screen_set_cursor_colour}} -->
The function `screen_set_cursor_colour` sets the cursor color of a given screen to a specified color value.
- **Inputs**:
    - `s`: A pointer to a `struct screen` which represents the screen whose cursor color is to be set.
    - `colour`: An integer representing the color value to set for the cursor.
- **Control Flow**:
    - The function directly assigns the provided color value to the `ccolour` field of the `screen` structure pointed to by `s`.
- **Output**:
    - This function does not return any value; it modifies the `ccolour` field of the `screen` structure in place.


---
### screen_set_title<!-- {{#callable:screen_set_title}} -->
The `screen_set_title` function sets the title of a screen object to a new value if the title is valid UTF-8.
- **Inputs**:
    - `s`: A pointer to a `struct screen` object whose title is to be set.
    - `title`: A constant character pointer to the new title string to be set for the screen.
- **Control Flow**:
    - Check if the provided title is valid UTF-8 using [`utf8_isvalid`](utf8.c.driver.md#utf8_isvalid); if not, return 0.
    - Free the current title of the screen using `free`.
    - Duplicate the new title string using [`xstrdup`](xmalloc.c.driver.md#xstrdup) and assign it to the screen's title.
    - Return 1 to indicate success.
- **Output**:
    - Returns 1 if the title is successfully set, otherwise returns 0 if the title is not valid UTF-8.
- **Functions called**:
    - [`utf8_isvalid`](utf8.c.driver.md#utf8_isvalid)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### screen_set_path<!-- {{#callable:screen_set_path}} -->
The `screen_set_path` function sets the path for a given screen structure by freeing any existing path and converting the new path to a specific format using [`utf8_stravis`](utf8.c.driver.md#utf8_stravis).
- **Inputs**:
    - `s`: A pointer to a `screen` structure whose path is to be set.
    - `path`: A constant character pointer representing the new path to be set for the screen.
- **Control Flow**:
    - The function begins by freeing the existing path stored in the `screen` structure `s` using `free(s->path);`.
    - It then calls [`utf8_stravis`](utf8.c.driver.md#utf8_stravis) to convert the provided `path` into a format suitable for storage in `s->path`, using specific visibility flags: `VIS_OCTAL`, `VIS_CSTYLE`, `VIS_TAB`, and `VIS_NL`.
- **Output**:
    - The function does not return a value; it modifies the `path` member of the `screen` structure `s` in place.
- **Functions called**:
    - [`utf8_stravis`](utf8.c.driver.md#utf8_stravis)


---
### screen_push_title<!-- {{#callable:screen_push_title}} -->
The `screen_push_title` function pushes the current screen title onto a stack of titles for a given screen.
- **Inputs**:
    - `s`: A pointer to a `struct screen` which represents the screen whose title is to be pushed onto the stack.
- **Control Flow**:
    - Check if the `titles` stack in the screen `s` is initialized; if not, allocate memory for it and initialize it as an empty queue.
    - Allocate memory for a new `screen_title_entry` structure to hold the current title.
    - Duplicate the current title string from the screen `s` and assign it to the `text` field of the new `screen_title_entry`.
    - Insert the new `screen_title_entry` at the head of the `titles` queue in the screen `s`.
- **Output**:
    - The function does not return any value; it modifies the `titles` stack of the given screen `s` by adding the current title to it.
- **Functions called**:
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### screen_pop_title<!-- {{#callable:screen_pop_title}} -->
The `screen_pop_title` function removes the top title from a screen's title stack and sets it as the current screen title.
- **Inputs**:
    - `s`: A pointer to a `struct screen` which contains the title stack and other screen-related data.
- **Control Flow**:
    - Check if the screen's title stack (`s->titles`) is NULL; if so, return immediately as there are no titles to pop.
    - Retrieve the first title entry from the title stack using `TAILQ_FIRST`.
    - If a title entry is found, set the screen's title to the text of the title entry using [`screen_set_title`](#screen_set_title).
    - Remove the title entry from the title stack using `TAILQ_REMOVE`.
    - Free the memory allocated for the title entry's text and the title entry itself.
- **Output**:
    - The function does not return a value; it modifies the screen's title and the title stack in place.
- **Functions called**:
    - [`screen_set_title`](#screen_set_title)


---
### screen_resize_cursor<!-- {{#callable:screen_resize_cursor}} -->
The `screen_resize_cursor` function adjusts the size of a screen and repositions the cursor accordingly, with optional reflow and handling of empty lines.
- **Inputs**:
    - `s`: A pointer to a `screen` structure representing the screen to be resized.
    - `sx`: An unsigned integer representing the new width of the screen.
    - `sy`: An unsigned integer representing the new height of the screen.
    - `reflow`: An integer flag indicating whether to reflow the screen content when resizing.
    - `eat_empty`: An integer flag indicating whether to remove empty lines from the bottom when resizing.
    - `cursor`: An integer flag indicating whether to adjust the cursor position during reflow.
- **Control Flow**:
    - Initialize local variables `cx` and `cy` to the current cursor position and the sum of the grid's history size and current y position, respectively.
    - If the screen has a write list, free it using [`screen_write_free_list`](screen-write.c.driver.md#screen_write_free_list).
    - Log the current and new screen sizes and cursor positions for debugging purposes.
    - Ensure the new screen dimensions `sx` and `sy` are at least 1.
    - If the new width `sx` differs from the current width, update the grid's width, reset tabs, and set `reflow` to 0 if the width hasn't changed.
    - If the new height `sy` differs from the current height, call [`screen_resize_y`](#screen_resize_y) to adjust the height and update `cy`.
    - If `ENABLE_SIXEL` is defined, free all images associated with the screen.
    - If `reflow` is true, call [`screen_reflow`](#screen_reflow) to adjust the screen content and cursor position based on the new width.
    - Adjust the cursor position based on the updated `cy` value, setting it to the new `cx` and `cy` values if `cy` is within the grid's history size, or to (0,0) otherwise.
    - Log the final cursor position for debugging purposes.
    - If the screen has a write list, recreate it using [`screen_write_make_list`](screen-write.c.driver.md#screen_write_make_list).
- **Output**:
    - The function does not return a value; it modifies the screen structure in place.
- **Functions called**:
    - [`screen_write_free_list`](screen-write.c.driver.md#screen_write_free_list)
    - [`screen_reset_tabs`](#screen_reset_tabs)
    - [`screen_resize_y`](#screen_resize_y)
    - [`image_free_all`](image.c.driver.md#image_free_all)
    - [`screen_reflow`](#screen_reflow)
    - [`screen_write_make_list`](screen-write.c.driver.md#screen_write_make_list)


---
### screen_resize<!-- {{#callable:screen_resize}} -->
The `screen_resize` function resizes a screen to specified dimensions and optionally reflows its content.
- **Inputs**:
    - `s`: A pointer to a `screen` structure representing the screen to be resized.
    - `sx`: An unsigned integer representing the new width of the screen.
    - `sy`: An unsigned integer representing the new height of the screen.
    - `reflow`: An integer flag indicating whether the screen content should be reflowed to fit the new dimensions.
- **Control Flow**:
    - The function calls [`screen_resize_cursor`](#screen_resize_cursor) with the provided screen, new dimensions, reflow flag, and additional parameters `1` and `1` to handle the resizing and cursor adjustment.
- **Output**:
    - The function does not return a value; it modifies the screen structure in place.
- **Functions called**:
    - [`screen_resize_cursor`](#screen_resize_cursor)


---
### screen_resize_y<!-- {{#callable:screen_resize_y}} -->
The `screen_resize_y` function adjusts the vertical size of a screen's grid, handling both increases and decreases in size by managing grid lines and history.
- **Inputs**:
    - `s`: A pointer to a `screen` structure representing the screen to be resized.
    - `sy`: An unsigned integer representing the new vertical size (height) of the screen.
    - `eat_empty`: An integer flag indicating whether to delete empty lines from the bottom when reducing size.
    - `cy`: A pointer to an unsigned integer representing the current y-coordinate of the cursor, which may be adjusted during resizing.
- **Control Flow**:
    - Check if the new size `sy` is zero and terminate with an error if so.
    - Store the current screen height in `oldy`.
    - If the new size `sy` is less than `oldy`, calculate the number of lines to remove (`needed`) and attempt to delete lines from the bottom if `eat_empty` is true.
    - If lines remain to be removed, increase the history size if history is enabled, or delete lines from the top if not.
    - Adjust the grid's line array to accommodate the new size.
    - If the new size `sy` is greater than `oldy`, calculate the number of lines to add (`needed`) and attempt to pull lines from the scrolled history if enabled.
    - Fill any remaining needed lines with blanks at the bottom of the grid.
    - Set the grid's new vertical size and reset the screen's scroll region.
- **Output**:
    - The function modifies the screen's grid to reflect the new vertical size, adjusting the cursor position and history as necessary.
- **Functions called**:
    - [`grid_view_delete_lines`](grid-view.c.driver.md#grid_view_delete_lines)
    - [`grid_adjust_lines`](grid.c.driver.md#grid_adjust_lines)
    - [`grid_empty_line`](grid.c.driver.md#grid_empty_line)


---
### screen_set_selection<!-- {{#callable:screen_set_selection}} -->
The `screen_set_selection` function initializes and sets the selection area on a screen with specified coordinates, rectangle mode, mode keys, and grid cell attributes.
- **Inputs**:
    - `s`: A pointer to a `screen` structure where the selection will be set.
    - `sx`: The starting x-coordinate of the selection.
    - `sy`: The starting y-coordinate of the selection.
    - `ex`: The ending x-coordinate of the selection.
    - `ey`: The ending y-coordinate of the selection.
    - `rectangle`: An unsigned integer indicating whether the selection is rectangular.
    - `modekeys`: An integer representing the mode keys used for the selection.
    - `gc`: A pointer to a `grid_cell` structure containing the attributes to be applied to the selection.
- **Control Flow**:
    - Check if the `sel` field of the `screen` structure is `NULL`; if so, allocate memory for it using [`xcalloc`](xmalloc.c.driver.md#xcalloc).
    - Copy the contents of the `grid_cell` pointed to by `gc` into the `cell` field of the `sel` structure.
    - Set the `hidden` field of the `sel` structure to 0, indicating the selection is visible.
    - Set the `rectangle` field of the `sel` structure to the value of the `rectangle` parameter.
    - Set the `modekeys` field of the `sel` structure to the value of the `modekeys` parameter.
    - Assign the starting and ending coordinates (`sx`, `sy`, `ex`, `ey`) to the corresponding fields in the `sel` structure.
- **Output**:
    - The function does not return a value; it modifies the `screen` structure's selection fields in place.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)


---
### screen_clear_selection<!-- {{#callable:screen_clear_selection}} -->
The `screen_clear_selection` function deallocates memory for the selection structure in a screen and sets the selection pointer to NULL.
- **Inputs**:
    - `s`: A pointer to a `struct screen` which represents the screen whose selection is to be cleared.
- **Control Flow**:
    - The function calls `free` on the `sel` member of the `struct screen` pointed to by `s`, which deallocates the memory previously allocated for the selection.
    - The function then sets the `sel` member of the `struct screen` to `NULL`, indicating that there is no current selection.
- **Output**:
    - The function does not return any value; it modifies the `struct screen` in place.


---
### screen_hide_selection<!-- {{#callable:screen_hide_selection}} -->
The `screen_hide_selection` function sets the `hidden` attribute of the selection in a screen to 1, effectively hiding the selection if it exists.
- **Inputs**:
    - `s`: A pointer to a `struct screen`, which represents the screen object containing the selection to be hidden.
- **Control Flow**:
    - Check if the `sel` attribute of the screen `s` is not NULL, indicating that a selection exists.
    - If a selection exists, set its `hidden` attribute to 1, marking it as hidden.
- **Output**:
    - The function does not return any value; it modifies the `hidden` attribute of the selection within the provided screen structure.


---
### screen_check_selection<!-- {{#callable:screen_check_selection}} -->
The function `screen_check_selection` determines if a given cell (px, py) is within the current selection on a screen.
- **Inputs**:
    - `s`: A pointer to a `struct screen` which contains the selection information.
    - `px`: The x-coordinate of the cell to check.
    - `py`: The y-coordinate of the cell to check.
- **Control Flow**:
    - Check if the selection is NULL or hidden; if so, return 0 (not selected).
    - If the selection is a rectangle, determine if the y-coordinate (py) is within the selection's vertical bounds, considering the direction of selection (upward, downward, or single line).
    - For rectangle selection, check if the x-coordinate (px) is within the horizontal bounds of the selection, considering which side (start or end) is on the left.
    - If the selection is not a rectangle, handle it like Emacs, keeping the top-left-most character and dropping the bottom-right-most, adjusting for the modekeys setting.
    - For non-rectangle selection, check if the cell is within the selection bounds, considering the direction of selection and modekeys setting.
    - Return 1 if the cell is within the selection, otherwise return 0.
- **Output**:
    - Returns 1 if the cell (px, py) is within the selection, otherwise returns 0.


---
### screen_select_cell<!-- {{#callable:screen_select_cell}} -->
The `screen_select_cell` function copies a selected grid cell from a screen's selection to a destination cell, applying default colors and attributes from a source cell if necessary.
- **Inputs**:
    - `s`: A pointer to a `struct screen` which contains the screen data and selection information.
    - `dst`: A pointer to a `struct grid_cell` where the selected cell data will be copied to.
    - `src`: A pointer to a `const struct grid_cell` which provides default color and attribute values if the selected cell has default colors.
- **Control Flow**:
    - Check if the screen's selection (`s->sel`) is NULL or hidden; if so, return immediately without doing anything.
    - Copy the selected cell (`s->sel->cell`) into the destination cell (`dst`).
    - If the foreground color of the destination cell is default, set it to the foreground color of the source cell.
    - If the background color of the destination cell is default, set it to the background color of the source cell.
    - Copy the UTF-8 data from the source cell to the destination cell.
    - Clear the charset attribute from the destination cell and set it to the charset attribute of the source cell.
    - Copy the flags from the source cell to the destination cell.
- **Output**:
    - The function does not return a value; it modifies the `dst` grid cell in place.
- **Functions called**:
    - [`utf8_copy`](utf8.c.driver.md#utf8_copy)


---
### screen_reflow<!-- {{#callable:screen_reflow}} -->
The `screen_reflow` function adjusts the layout of a screen's grid to accommodate a new width, optionally repositioning the cursor.
- **Inputs**:
    - `s`: A pointer to a `screen` structure representing the screen to be reflowed.
    - `new_x`: An unsigned integer representing the new width of the screen grid.
    - `cx`: A pointer to an unsigned integer representing the current x-coordinate of the cursor, which may be updated.
    - `cy`: A pointer to an unsigned integer representing the current y-coordinate of the cursor, which may be updated.
    - `cursor`: An integer flag indicating whether the cursor position should be adjusted (non-zero) or not (zero).
- **Control Flow**:
    - If the `cursor` flag is set, the function calls [`grid_wrap_position`](grid.c.driver.md#grid_wrap_position) to convert the current cursor position (`cx`, `cy`) into wrapped coordinates (`wx`, `wy`).
    - A debug log is generated to record the original cursor position and its wrapped coordinates.
    - The function calls [`grid_reflow`](grid.c.driver.md#grid_reflow) to adjust the grid layout to the new width `new_x`.
    - If the `cursor` flag is set, the function calls [`grid_unwrap_position`](grid.c.driver.md#grid_unwrap_position) to convert the wrapped coordinates back to the new cursor position (`cx`, `cy`) after reflowing.
    - Another debug log is generated to record the new cursor position.
    - If the `cursor` flag is not set, the cursor is reset to the top-left corner of the screen (`cx` = 0) and the bottom of the grid's history (`cy` = `s->grid->hsize`).
- **Output**:
    - The function does not return a value, but it modifies the screen's grid layout and potentially updates the cursor position through the pointers `cx` and `cy`.
- **Functions called**:
    - [`grid_wrap_position`](grid.c.driver.md#grid_wrap_position)
    - [`grid_reflow`](grid.c.driver.md#grid_reflow)
    - [`grid_unwrap_position`](grid.c.driver.md#grid_unwrap_position)


---
### screen_alternate_on<!-- {{#callable:screen_alternate_on}} -->
The `screen_alternate_on` function enables the alternate screen mode by saving the current screen state and clearing the visible grid.
- **Inputs**:
    - `s`: A pointer to a `screen` structure representing the current screen state.
    - `gc`: A pointer to a `grid_cell` structure representing the current grid cell attributes to be saved.
    - `cursor`: An integer flag indicating whether to save the current cursor position (non-zero value) or not (zero value).
- **Control Flow**:
    - Check if the screen is already in alternate mode using `SCREEN_IS_ALTERNATE(s)`; if true, return immediately.
    - Retrieve the current screen dimensions using `screen_size_x(s)` and `screen_size_y(s)`.
    - Create a new grid for saving the current screen state using [`grid_create`](grid.c.driver.md#grid_create) with the retrieved dimensions.
    - Duplicate the current screen lines into the saved grid using [`grid_duplicate_lines`](grid.c.driver.md#grid_duplicate_lines).
    - If the `cursor` flag is set, save the current cursor position (`s->cx` and `s->cy`) into `s->saved_cx` and `s->saved_cy`.
    - Copy the current grid cell attributes from `gc` to `s->saved_cell` using `memcpy`.
    - Clear the current grid view using [`grid_view_clear`](grid-view.c.driver.md#grid_view_clear) to prepare for the alternate screen.
    - Save the current grid flags into `s->saved_flags` and disable the `GRID_HISTORY` flag in the current grid.
- **Output**:
    - The function does not return a value; it modifies the `screen` structure to enable alternate screen mode.
- **Functions called**:
    - [`grid_create`](grid.c.driver.md#grid_create)
    - [`grid_duplicate_lines`](grid.c.driver.md#grid_duplicate_lines)
    - [`grid_view_clear`](grid-view.c.driver.md#grid_view_clear)


---
### screen_alternate_off<!-- {{#callable:screen_alternate_off}} -->
The `screen_alternate_off` function exits the alternate screen mode by restoring the saved grid and cursor position, and resizing the screen back to its original dimensions.
- **Inputs**:
    - `s`: A pointer to a `screen` structure representing the current screen state.
    - `gc`: A pointer to a `grid_cell` structure where the saved cell state will be restored, if not NULL.
    - `cursor`: An integer flag indicating whether to restore the cursor position and cell state.
- **Control Flow**:
    - Check if the screen is in alternate mode; if so, resize to the saved grid's dimensions.
    - If the cursor flag is set and saved cursor positions are valid, restore the cursor position and cell state.
    - If not in alternate mode, adjust the cursor position to fit within the current screen size and return.
    - Duplicate lines from the saved grid back to the current grid to restore the screen content.
    - Restore the history flag if it was previously set, and resize the screen back to its original dimensions.
    - Destroy the saved grid and set the saved grid pointer to NULL.
    - Adjust the cursor position to ensure it is within the bounds of the current screen size.
- **Output**:
    - The function does not return a value; it modifies the screen state by restoring the previous screen content and cursor position.
- **Functions called**:
    - [`screen_resize`](#screen_resize)
    - [`grid_duplicate_lines`](grid.c.driver.md#grid_duplicate_lines)
    - [`grid_destroy`](grid.c.driver.md#grid_destroy)


---
### screen_mode_to_string<!-- {{#callable:screen_mode_to_string}} -->
The function `screen_mode_to_string` converts a given screen mode integer into a human-readable string representation of the active modes.
- **Inputs**:
    - `mode`: An integer representing the screen mode, where each bit corresponds to a specific mode flag.
- **Control Flow**:
    - Initialize a static character array `tmp` to store the resulting string.
    - Check if the mode is 0, and if so, return the string "NONE".
    - Check if the mode is equal to `ALL_MODES`, and if so, return the string "ALL".
    - Initialize the `tmp` string to be empty.
    - For each mode flag (e.g., `MODE_CURSOR`, `MODE_INSERT`, etc.), check if the corresponding bit is set in the `mode` integer.
    - If a mode flag is set, append the corresponding string (e.g., "CURSOR,", "INSERT,") to `tmp` using [`strlcat`](compat/strlcat.c.driver.md#strlcat).
    - After processing all mode flags, remove the trailing comma from `tmp`.
    - Return the `tmp` string containing the concatenated mode names.
- **Output**:
    - A string representing the active modes, concatenated and separated by commas, or "NONE" or "ALL" if applicable.
- **Functions called**:
    - [`strlcat`](compat/strlcat.c.driver.md#strlcat)


