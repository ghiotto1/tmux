# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it provides functionality for formatting and drawing text with various styles and alignments on a screen. The code defines a set of functions that handle the rendering of text with specific formatting rules, such as alignment (left, center, right, absolute center), and the inclusion of lists with focus areas. The primary technical components include structures for managing format ranges and styles, functions for drawing formatted text on a screen, and utilities for handling UTF-8 characters and style parsing.

The file is not an executable but rather a library intended to be used within the tmux application. It does not define public APIs or external interfaces directly but provides internal functions that are likely used by other parts of the tmux codebase to render formatted text. The code handles complex formatting scenarios, including trimming text to fit within a specified width, managing style transitions, and ensuring that text is displayed correctly with respect to the specified styles and alignments. The use of structures like `format_range` and `format_ranges` indicates a focus on managing and applying styles over specific text ranges, which is crucial for the dynamic and flexible display capabilities of tmux.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Data Structures

---
### format_range
- **Type**: `struct`
- **Members**:
    - `index`: An unsigned integer representing the index of the format range.
    - `s`: A pointer to a screen structure associated with the format range.
    - `start`: An unsigned integer indicating the start position of the range.
    - `end`: An unsigned integer indicating the end position of the range.
    - `type`: An enumeration value representing the type of style range.
    - `argument`: An unsigned integer used as an argument for certain range types.
    - `string`: A character array of size 16 used to store a string associated with the range.
    - `entry`: A TAILQ_ENTRY structure for linking format_range elements in a tail queue.
- **Description**: The `format_range` structure is used to define a range within a screen, characterized by its start and end positions, and associated with a specific style type. It includes an index for identification, a pointer to the screen it belongs to, and additional fields for handling style arguments and strings. The structure is designed to be part of a linked list using the TAILQ_ENTRY macro, allowing multiple format ranges to be managed in a queue.


# Functions

---
### format_is_type<!-- {{#callable:format_is_type}} -->
The `format_is_type` function checks if a given format range matches a specified style based on type and additional criteria.
- **Inputs**:
    - `fr`: A pointer to a `format_range` structure representing the format range to be checked.
    - `sy`: A pointer to a `style` structure representing the style against which the format range is to be checked.
- **Control Flow**:
    - Check if the type of the format range (`fr->type`) is not equal to the style's range type (`sy->range_type`); if so, return 0 indicating no match.
    - Use a switch statement to handle different types of format ranges.
    - For `STYLE_RANGE_NONE`, `STYLE_RANGE_LEFT`, and `STYLE_RANGE_RIGHT`, return 1 indicating a match.
    - For `STYLE_RANGE_PANE`, `STYLE_RANGE_WINDOW`, and `STYLE_RANGE_SESSION`, compare the `argument` fields of the format range and style; return 1 if they match, otherwise return 0.
    - For `STYLE_RANGE_USER`, compare the `string` fields of the format range and style using `strcmp`; return 1 if they match, otherwise return 0.
    - If none of the cases match, return 1 by default.
- **Output**:
    - The function returns an integer: 1 if the format range matches the style, and 0 otherwise.


---
### format_free_range<!-- {{#callable:format_free_range}} -->
The `format_free_range` function removes a `format_range` from a `format_ranges` list and frees its memory.
- **Inputs**:
    - `frs`: A pointer to a `format_ranges` structure, which is a list of `format_range` elements.
    - `fr`: A pointer to a `format_range` structure that is to be removed from the list and freed.
- **Control Flow**:
    - The function calls `TAILQ_REMOVE` to remove the `format_range` element `fr` from the `format_ranges` list `frs` using the `entry` field as the list link.
    - The function then calls `free` to deallocate the memory occupied by `fr`.
- **Output**:
    - The function does not return any value; it performs an in-place modification of the list and frees memory.


---
### format_update_ranges<!-- {{#callable:format_update_ranges}} -->
The `format_update_ranges` function adjusts and updates the start and end positions of format ranges within a given screen, based on specified offset, start, and width parameters.
- **Inputs**:
    - `frs`: A pointer to a `format_ranges` structure, which is a list of format ranges to be updated.
    - `s`: A pointer to a `screen` structure, representing the screen associated with the format ranges.
    - `offset`: An unsigned integer representing the offset to be applied to the start and end positions of the format ranges.
    - `start`: An unsigned integer representing the starting position for the update operation.
    - `width`: An unsigned integer representing the width of the area to be updated.
- **Control Flow**:
    - Check if the `frs` (format ranges) is NULL, and return immediately if it is.
    - Iterate over each format range `fr` in the `frs` list using `TAILQ_FOREACH_SAFE`.
    - For each range, check if its associated screen `fr->s` matches the given screen `s`; if not, continue to the next range.
    - Check if the range is completely outside the specified start and width bounds; if so, free the range and continue to the next one.
    - Adjust the start and end of the range to fit within the specified start and width bounds.
    - If the adjusted start and end are equal, free the range and continue to the next one.
    - Subtract the start value from both the start and end of the range to normalize them to zero-based positions.
    - Add the offset to both the start and end of the range to apply the specified offset.
- **Output**:
    - The function does not return a value; it modifies the `format_ranges` list in place, updating or removing ranges as necessary.
- **Functions called**:
    - [`format_free_range`](#format_free_range)


---
### format_draw_put<!-- {{#callable:format_draw_put}} -->
The `format_draw_put` function moves the cursor on a target screen and copies a specified portion of a source screen to it, updating format ranges accordingly.
- **Inputs**:
    - `octx`: A pointer to a `screen_write_ctx` structure representing the context of the screen where the drawing will occur.
    - `ocx`: An unsigned integer representing the x-coordinate of the cursor on the target screen.
    - `ocy`: An unsigned integer representing the y-coordinate of the cursor on the target screen.
    - `s`: A pointer to a `screen` structure representing the source screen from which content will be copied.
    - `frs`: A pointer to a `format_ranges` structure that holds the format ranges to be updated.
    - `offset`: An unsigned integer representing the horizontal offset from the cursor position on the target screen.
    - `start`: An unsigned integer indicating the starting position on the source screen from which content will be copied.
    - `width`: An unsigned integer specifying the width of the content to be copied from the source screen.
- **Control Flow**:
    - The function begins by moving the cursor on the target screen to the position specified by `ocx + offset` and `ocy` using [`screen_write_cursormove`](screen-write.c.driver.md#screen_write_cursormove).
    - It then copies a portion of the source screen `s` starting at `start` with a width of `width` to the target screen using [`screen_write_fast_copy`](screen-write.c.driver.md#screen_write_fast_copy).
    - Finally, it updates the format ranges `frs` to reflect the changes made by the copy operation using [`format_update_ranges`](#format_update_ranges).
- **Output**:
    - The function does not return a value; it performs operations directly on the provided screen context and format ranges.
- **Functions called**:
    - [`screen_write_cursormove`](screen-write.c.driver.md#screen_write_cursormove)
    - [`screen_write_fast_copy`](screen-write.c.driver.md#screen_write_fast_copy)
    - [`format_update_ranges`](#format_update_ranges)


---
### format_draw_put_list<!-- {{#callable:format_draw_put_list}} -->
The `format_draw_put_list` function draws a list on a screen context, ensuring that the focus area is visible and adding markers if the list is trimmed.
- **Inputs**:
    - `octx`: A pointer to the screen write context where the list will be drawn.
    - `ocx`: The x-coordinate on the screen where drawing starts.
    - `ocy`: The y-coordinate on the screen where drawing starts.
    - `offset`: The offset from the starting x-coordinate for drawing.
    - `width`: The width available for drawing the list.
    - `list`: A pointer to the screen structure representing the list to be drawn.
    - `list_left`: A pointer to the screen structure representing the left marker to be drawn if the list is trimmed.
    - `list_right`: A pointer to the screen structure representing the right marker to be drawn if the list is trimmed.
    - `focus_start`: The starting index of the focus area within the list.
    - `focus_end`: The ending index of the focus area within the list.
    - `frs`: A pointer to the format ranges structure used for managing style ranges.
- **Control Flow**:
    - Check if the available width is sufficient to draw the entire list; if so, draw the list and return.
    - Calculate the center of the focus area and determine the starting point for drawing the list to keep the focus visible.
    - Adjust the starting point if it exceeds the list's width.
    - If the list is trimmed on the left, draw the left marker and adjust the offset, start, and width accordingly.
    - If the list is trimmed on the right, draw the right marker and adjust the width accordingly.
    - Draw the visible portion of the list using the calculated start and width.
- **Output**:
    - The function does not return a value; it modifies the screen context to display the list with appropriate markers and focus visibility.
- **Functions called**:
    - [`format_draw_put`](#format_draw_put)
    - [`screen_write_cursormove`](screen-write.c.driver.md#screen_write_cursormove)
    - [`screen_write_fast_copy`](screen-write.c.driver.md#screen_write_fast_copy)


---
### format_draw_none<!-- {{#callable:format_draw_none}} -->
The `format_draw_none` function arranges and draws sections of a screen (left, centre, right, and absolute centre) within a given available width, prioritizing the left and right sections over the centre.
- **Inputs**:
    - `octx`: A pointer to a `screen_write_ctx` structure, which represents the context for writing to a screen.
    - `available`: An unsigned integer representing the total available width for drawing.
    - `ocx`: An unsigned integer representing the x-coordinate offset on the target screen.
    - `ocy`: An unsigned integer representing the y-coordinate offset on the target screen.
    - `left`: A pointer to a `screen` structure representing the left section to be drawn.
    - `centre`: A pointer to a `screen` structure representing the centre section to be drawn.
    - `right`: A pointer to a `screen` structure representing the right section to be drawn.
    - `abs_centre`: A pointer to a `screen` structure representing the absolute centre section to be drawn.
    - `frs`: A pointer to a `format_ranges` structure, which holds format range information for styling.
- **Control Flow**:
    - Initialize the widths of the left, centre, right, and absolute centre sections from their respective screen structures.
    - Enter a loop to reduce the widths of the sections if their combined width exceeds the available width, prioritizing reducing the centre first, then the right, and finally the left.
    - Call [`format_draw_put`](#format_draw_put) to draw the left section starting at the beginning of the available space.
    - Call [`format_draw_put`](#format_draw_put) to draw the right section starting at the end of the available space minus the width of the right section.
    - Calculate the midpoint for the centre section and call [`format_draw_put`](#format_draw_put) to draw it centered between the left and right sections.
    - Adjust the width of the absolute centre section if it exceeds the available width, then call [`format_draw_put`](#format_draw_put) to draw it perfectly centered in the available space.
- **Output**:
    - The function does not return a value; it modifies the screen context `octx` to draw the specified sections within the available width.
- **Functions called**:
    - [`format_draw_put`](#format_draw_put)


---
### format_draw_left<!-- {{#callable:format_draw_left}} -->
The `format_draw_left` function arranges and draws multiple screen sections within a given available width, prioritizing the left section and handling a list section with focus management.
- **Inputs**:
    - `octx`: A pointer to a `screen_write_ctx` structure representing the output context where the drawing will occur.
    - `available`: An unsigned integer representing the total available width for drawing.
    - `ocx`: An unsigned integer representing the starting x-coordinate on the output context.
    - `ocy`: An unsigned integer representing the starting y-coordinate on the output context.
    - `left`: A pointer to a `screen` structure representing the left section to be drawn.
    - `centre`: A pointer to a `screen` structure representing the centre section to be drawn.
    - `right`: A pointer to a `screen` structure representing the right section to be drawn.
    - `abs_centre`: A pointer to a `screen` structure representing the absolute centre section to be drawn.
    - `list`: A pointer to a `screen` structure representing the list section to be drawn.
    - `list_left`: A pointer to a `screen` structure representing the left marker of the list section.
    - `list_right`: A pointer to a `screen` structure representing the right marker of the list section.
    - `after`: A pointer to a `screen` structure representing the section to be drawn after the list.
    - `focus_start`: An integer indicating the starting position of the focus within the list section.
    - `focus_end`: An integer indicating the ending position of the focus within the list section.
    - `frs`: A pointer to a `format_ranges` structure used to manage format ranges during drawing.
- **Control Flow**:
    - Initialize width variables for each section based on their respective screen's current x-coordinate (`cx`).
    - Enter a loop to trim the widths of sections in a specific order (centre, list, right, after, left) until their total width fits within the available width.
    - If the list section's width is reduced to zero, call [`format_draw_none`](#format_draw_none) to handle drawing without a list and return.
    - Draw the left section starting at position 0 using [`format_draw_put`](#format_draw_put).
    - Draw the right section at the position `available - width_right` using [`format_draw_put`](#format_draw_put).
    - Draw the after section at the position `width_left + width_list` using [`format_draw_put`](#format_draw_put).
    - Calculate the position for the centre section to be halfway between `width_left + width_list + width_after` and `available - width_right`, then draw it using [`format_draw_put`](#format_draw_put).
    - If no focus is specified, set the focus to the start of the list section.
    - Draw the list section using [`format_draw_put_list`](#format_draw_put_list), managing focus within the list.
    - Draw the abs_centre section in the perfect centre of the available horizontal space using [`format_draw_put`](#format_draw_put).
- **Output**:
    - The function does not return a value; it performs drawing operations on the provided `screen_write_ctx` output context.
- **Functions called**:
    - [`screen_write_start`](screen-write.c.driver.md#screen_write_start)
    - [`screen_write_fast_copy`](screen-write.c.driver.md#screen_write_fast_copy)
    - [`screen_write_stop`](screen-write.c.driver.md#screen_write_stop)
    - [`format_draw_none`](#format_draw_none)
    - [`format_draw_put`](#format_draw_put)
    - [`format_draw_put_list`](#format_draw_put_list)


---
### format_draw_centre<!-- {{#callable:format_draw_centre}} -->
The `format_draw_centre` function arranges and draws multiple screen sections within a given available width, centering a list section and adjusting other sections accordingly.
- **Inputs**:
    - `octx`: A pointer to the screen write context where the drawing operations will be applied.
    - `available`: The total available width for drawing the sections.
    - `ocx`: The x-coordinate offset in the output context where drawing begins.
    - `ocy`: The y-coordinate offset in the output context where drawing begins.
    - `left`: A pointer to the screen structure representing the left section.
    - `centre`: A pointer to the screen structure representing the centre section.
    - `right`: A pointer to the screen structure representing the right section.
    - `abs_centre`: A pointer to the screen structure representing the absolute centre section.
    - `list`: A pointer to the screen structure representing the list section to be centered.
    - `list_left`: A pointer to the screen structure representing the left marker of the list.
    - `list_right`: A pointer to the screen structure representing the right marker of the list.
    - `after`: A pointer to the screen structure representing the section after the list.
    - `focus_start`: The starting index of the focus within the list, or -1 if no focus is specified.
    - `focus_end`: The ending index of the focus within the list, or -1 if no focus is specified.
    - `frs`: A pointer to a structure holding format ranges for styling purposes.
- **Control Flow**:
    - Initialize width variables for each section based on their respective screen's current width (cx).
    - Enter a loop to trim sections in a specific order (list, after, centre, right, left) until their combined width fits within the available width.
    - If the list section is completely trimmed, call [`format_draw_none`](#format_draw_none) to handle drawing without a list and return.
    - Draw the left section starting at the beginning of the available space.
    - Draw the right section starting at the end of the available space minus its width.
    - Calculate the middle point of the available space to position the centre sections.
    - Draw the centre section offset from the middle point, adjusted by the list's width.
    - Draw the after section offset from the middle point, adjusted by the list's width.
    - Determine the focus range for the list if not specified, defaulting to the centre of the list.
    - Draw the list section centered around the middle point, with optional focus markers.
    - Draw the absolute centre section perfectly centered within the available space.
- **Output**:
    - The function does not return a value; it performs drawing operations on the provided screen write context.
- **Functions called**:
    - [`screen_write_start`](screen-write.c.driver.md#screen_write_start)
    - [`screen_write_fast_copy`](screen-write.c.driver.md#screen_write_fast_copy)
    - [`screen_write_stop`](screen-write.c.driver.md#screen_write_stop)
    - [`format_draw_none`](#format_draw_none)
    - [`format_draw_put`](#format_draw_put)
    - [`format_draw_put_list`](#format_draw_put_list)


---
### format_draw_right<!-- {{#callable:format_draw_right}} -->
The `format_draw_right` function arranges and draws multiple screen sections within a given available width, focusing on placing a list on the right side of the screen.
- **Inputs**:
    - `octx`: A pointer to the screen write context where the drawing operations will be applied.
    - `available`: The total available width for drawing the sections.
    - `ocx`: The x-coordinate offset on the target screen where drawing begins.
    - `ocy`: The y-coordinate offset on the target screen where drawing begins.
    - `left`: A pointer to the screen structure representing the left section.
    - `centre`: A pointer to the screen structure representing the centre section.
    - `right`: A pointer to the screen structure representing the right section.
    - `abs_centre`: A pointer to the screen structure representing the absolute centre section.
    - `list`: A pointer to the screen structure representing the list section.
    - `list_left`: A pointer to the screen structure representing the left marker of the list.
    - `list_right`: A pointer to the screen structure representing the right marker of the list.
    - `after`: A pointer to the screen structure representing the section after the list.
    - `focus_start`: The starting index of the focus within the list, or -1 if no focus is specified.
    - `focus_end`: The ending index of the focus within the list, or -1 if no focus is specified.
    - `frs`: A pointer to a structure holding format ranges for styling purposes.
- **Control Flow**:
    - Initialize the widths of each section (left, centre, right, abs_centre, list, after) based on their respective screen structures.
    - Iteratively reduce the widths of sections (centre, list, right, after, left) until their total width fits within the available width.
    - If the list width is zero, copy the 'after' section to the 'right' screen and call [`format_draw_none`](#format_draw_none) to handle drawing without a list.
    - Draw the left section starting at position 0 on the target screen.
    - Draw the 'after' section at the position calculated as `available - width_after`.
    - Draw the right section at the position calculated as `available - width_right - width_list - width_after`.
    - Draw the centre section at a position calculated to be halfway between the left section and the right section's starting position.
    - If no focus is specified, set the focus to the start of the list; otherwise, draw the list section with focus handling.
    - Draw the abs_centre section in the perfect centre of the available horizontal space.
- **Output**:
    - The function does not return a value; it performs drawing operations on the provided screen write context.
- **Functions called**:
    - [`screen_write_start`](screen-write.c.driver.md#screen_write_start)
    - [`screen_write_fast_copy`](screen-write.c.driver.md#screen_write_fast_copy)
    - [`screen_write_stop`](screen-write.c.driver.md#screen_write_stop)
    - [`format_draw_none`](#format_draw_none)
    - [`format_draw_put`](#format_draw_put)
    - [`format_draw_put_list`](#format_draw_put_list)


---
### format_draw_absolute_centre<!-- {{#callable:format_draw_absolute_centre}} -->
The `format_draw_absolute_centre` function arranges and draws multiple screen sections (left, centre, right, absolute centre, list, and after) within a given available width, ensuring that the list and absolute centre are perfectly centered horizontally.
- **Inputs**:
    - `octx`: A pointer to a `screen_write_ctx` structure, which represents the context for writing to a screen.
    - `available`: An unsigned integer representing the total available width for drawing.
    - `ocx`: An unsigned integer representing the starting x-coordinate on the output screen.
    - `ocy`: An unsigned integer representing the starting y-coordinate on the output screen.
    - `left`: A pointer to a `screen` structure representing the left section to be drawn.
    - `centre`: A pointer to a `screen` structure representing the centre section to be drawn.
    - `right`: A pointer to a `screen` structure representing the right section to be drawn.
    - `abs_centre`: A pointer to a `screen` structure representing the absolute centre section to be drawn.
    - `list`: A pointer to a `screen` structure representing the list section to be drawn.
    - `list_left`: A pointer to a `screen` structure representing the left marker of the list.
    - `list_right`: A pointer to a `screen` structure representing the right marker of the list.
    - `after`: A pointer to a `screen` structure representing the section to be drawn after the list.
    - `focus_start`: An integer indicating the start position of the focus within the list, or -1 if no focus is specified.
    - `focus_end`: An integer indicating the end position of the focus within the list, or -1 if no focus is specified.
    - `frs`: A pointer to a `format_ranges` structure, which holds information about format ranges for styling purposes.
- **Control Flow**:
    - Initialize width variables for each section (left, centre, right, abs_centre, list, after) using their respective `cx` values.
    - Trim the widths of the left, centre, and right sections to fit within the available width, prioritizing centre, then right, then left.
    - Trim the widths of the list, after, and abs_centre sections independently to fit within the available width, prioritizing list, then after, then abs_centre.
    - Draw the left section starting at position 0 using [`format_draw_put`](#format_draw_put).
    - Draw the right section starting at position `available - width_right` using [`format_draw_put`](#format_draw_put).
    - Calculate the middle position for the centre section and draw it at `middle - width_centre` using [`format_draw_put`](#format_draw_put).
    - If no focus is specified, set the focus to the middle of the list.
    - Calculate the offset for the abs_centre and list to be perfectly centered horizontally.
    - Draw the abs_centre section before the list using [`format_draw_put`](#format_draw_put).
    - Draw the list section in the absolute centre using [`format_draw_put_list`](#format_draw_put_list).
    - Draw the after section at the end of the centre using [`format_draw_put`](#format_draw_put).
- **Output**:
    - The function does not return a value; it performs drawing operations on the provided screen context (`octx`).
- **Functions called**:
    - [`format_draw_put`](#format_draw_put)
    - [`format_draw_put_list`](#format_draw_put_list)


---
### format_leading_hashes<!-- {{#callable:format_leading_hashes}} -->
The `format_leading_hashes` function calculates the number of leading '#' characters in a string and determines the width based on specific rules, returning a pointer to the position in the string after processing these characters.
- **Inputs**:
    - `cp`: A pointer to a constant character string where the function will count leading '#' characters.
    - `n`: A pointer to an unsigned integer where the function will store the count of leading '#' characters.
    - `width`: A pointer to an unsigned integer where the function will store the calculated width based on the number of leading '#' characters.
- **Control Flow**:
    - Initialize the count of leading '#' characters to zero and iterate through the string, incrementing the count for each '#' character encountered.
    - If no '#' characters are found, set the width to zero and return the original string pointer.
    - If the character following the '#' characters is not '[', calculate the width as half the count of '#' characters, rounding up if the count is odd, and return a pointer to the position after the '#' characters.
    - If the character following the '#' characters is '[', calculate the width as half the count of '#' characters.
    - If the count of '#' characters is even, return a pointer to the position after the '#' characters, indicating that all '#' characters are escaped.
    - If the count of '#' characters is odd, return a pointer to the last '#' character, indicating a style.
- **Output**:
    - Returns a pointer to a position in the string after processing the leading '#' characters, which may point to the character after the '#' characters or to a specific '#' character based on the rules applied.


---
### format_draw_many<!-- {{#callable:format_draw_many}} -->
The `format_draw_many` function draws a specified character multiple times on a screen context using a given style.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure representing the screen context where the characters will be drawn.
    - `sy`: A pointer to a `style` structure that defines the style attributes to be applied to the characters being drawn.
    - `ch`: A character to be drawn repeatedly on the screen.
    - `n`: An unsigned integer representing the number of times the character `ch` should be drawn.
- **Control Flow**:
    - The function begins by setting the UTF-8 data of the style's grid cell to the character `ch` using [`utf8_set`](utf8.c.driver.md#utf8_set).
    - A loop iterates `n` times, where `n` is the number of times the character should be drawn.
    - In each iteration, the function calls [`screen_write_cell`](screen-write.c.driver.md#screen_write_cell) to draw the character on the screen context `ctx` using the style `sy`.
- **Output**:
    - The function does not return a value; it performs its operation by modifying the screen context `ctx` directly.
- **Functions called**:
    - [`utf8_set`](utf8.c.driver.md#utf8_set)
    - [`screen_write_cell`](screen-write.c.driver.md#screen_write_cell)


---
### format_draw<!-- {{#callable:format_draw}} -->
The `format_draw` function processes a formatted string with style specifications and draws it onto a screen context, handling alignment and style ranges.
- **Inputs**:
    - `octx`: A pointer to a `screen_write_ctx` structure representing the screen context where the formatted string will be drawn.
    - `base`: A pointer to a `grid_cell` structure representing the base style for drawing.
    - `available`: An unsigned integer representing the available width for drawing on the screen.
    - `expanded`: A constant character pointer to the formatted string to be drawn.
    - `srs`: A pointer to a `style_ranges` structure to store style ranges if applicable.
    - `default_colours`: An integer flag indicating whether to use default colors for drawing.
- **Control Flow**:
    - Initialize variables and structures for screen contexts and style management.
    - Iterate over the `expanded` string, handling sequences of '#' and style specifications.
    - For each character or style, determine its type and draw it on the appropriate screen context based on alignment (left, center, right, etc.).
    - Manage style changes, including pushing and popping default styles, and handle list states for special formatting.
    - After processing the string, clear the available area if a fill color is specified.
    - Draw the formatted content on the screen context based on the determined alignment and list position.
    - Create and log style ranges if applicable, and free any allocated resources.
    - Restore the original cursor position on the screen context.
- **Output**:
    - The function does not return a value but modifies the screen context `octx` to display the formatted and styled string.
- **Functions called**:
    - [`style_set`](style.c.driver.md#style_set)
    - [`screen_init`](screen.c.driver.md#screen_init)
    - [`screen_write_start`](screen-write.c.driver.md#screen_write_start)
    - [`screen_write_clearendofline`](screen-write.c.driver.md#screen_write_clearendofline)
    - [`format_draw_many`](#format_draw_many)
    - [`utf8_set`](utf8.c.driver.md#utf8_set)
    - [`screen_write_cell`](screen-write.c.driver.md#screen_write_cell)
    - [`utf8_open`](utf8.c.driver.md#utf8_open)
    - [`utf8_append`](utf8.c.driver.md#utf8_append)
    - [`format_skip`](format.c.driver.md#format_skip)
    - [`format_free_range`](#format_free_range)
    - [`xstrndup`](xmalloc.c.driver.md#xstrndup)
    - [`style_copy`](style.c.driver.md#style_copy)
    - [`style_parse`](style.c.driver.md#style_parse)
    - [`style_tostring`](style.c.driver.md#style_tostring)
    - [`format_is_type`](#format_is_type)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`strlcpy`](compat/strlcpy.c.driver.md#strlcpy)
    - [`screen_write_stop`](screen-write.c.driver.md#screen_write_stop)
    - [`screen_write_putc`](screen-write.c.driver.md#screen_write_putc)
    - [`format_draw_none`](#format_draw_none)
    - [`format_draw_left`](#format_draw_left)
    - [`format_draw_centre`](#format_draw_centre)
    - [`format_draw_right`](#format_draw_right)
    - [`format_draw_absolute_centre`](#format_draw_absolute_centre)
    - [`screen_free`](screen.c.driver.md#screen_free)
    - [`screen_write_cursormove`](screen-write.c.driver.md#screen_write_cursormove)


---
### format_width<!-- {{#callable:format_width}} -->
The `format_width` function calculates the display width of a string, taking into account special formatting sequences and UTF-8 characters.
- **Inputs**:
    - `expanded`: A constant character pointer to the string whose width is to be calculated.
- **Control Flow**:
    - Initialize pointers and variables for tracking position and width.
    - Iterate over each character in the input string `expanded`.
    - If a '#' character is encountered, determine if it is part of a formatting sequence and adjust the width accordingly.
    - If a UTF-8 character is detected, process it to determine its width and add it to the total width.
    - For standard ASCII characters between 0x20 and 0x7E, increment the width by one.
    - Continue processing until the end of the string is reached.
- **Output**:
    - Returns an unsigned integer representing the calculated width of the input string, considering special formatting and UTF-8 characters.
- **Functions called**:
    - [`format_leading_hashes`](#format_leading_hashes)
    - [`format_skip`](format.c.driver.md#format_skip)
    - [`utf8_open`](utf8.c.driver.md#utf8_open)
    - [`utf8_append`](utf8.c.driver.md#utf8_append)


---
### format_trim_left<!-- {{#callable:format_trim_left}} -->
The `format_trim_left` function trims a string from the left, considering special formatting sequences, to fit within a specified width limit.
- **Inputs**:
    - `expanded`: A constant character pointer to the input string that may contain special formatting sequences.
    - `limit`: An unsigned integer specifying the maximum width the output string should occupy.
- **Control Flow**:
    - Allocate memory for the output string, which is twice the length of the input string plus one for the null terminator.
    - Initialize pointers for traversing the input string and writing to the output string.
    - Iterate over each character in the input string until the end of the string or the width limit is reached.
    - If a '#' character is encountered, determine the number of leading '#' characters and their width using [`format_leading_hashes`](#format_leading_hashes).
    - Adjust the width of leading '#' characters if they exceed the remaining width limit and copy them to the output string.
    - If a '#' is followed by a '[', find the matching ']' and copy the entire sequence to the output string.
    - For UTF-8 characters, determine their width and copy them to the output string if they fit within the remaining width limit.
    - For ASCII characters, copy them to the output string if they fit within the remaining width limit.
    - Terminate the output string with a null character.
- **Output**:
    - Returns a newly allocated string that is a trimmed version of the input, fitting within the specified width limit.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`format_leading_hashes`](#format_leading_hashes)
    - [`format_skip`](format.c.driver.md#format_skip)
    - [`utf8_open`](utf8.c.driver.md#utf8_open)
    - [`utf8_append`](utf8.c.driver.md#utf8_append)


---
### format_trim_right<!-- {{#callable:format_trim_right}} -->
The `format_trim_right` function trims a given string from the right to fit within a specified width limit, taking into account special formatting characters and UTF-8 encoding.
- **Inputs**:
    - `expanded`: A constant character pointer to the input string that needs to be trimmed.
    - `limit`: An unsigned integer specifying the maximum width the output string should have.
- **Control Flow**:
    - Calculate the total width of the input string using [`format_width`](#format_width) function.
    - If the total width is less than or equal to the limit, return a duplicate of the input string using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - Calculate the number of characters to skip by subtracting the limit from the total width.
    - Allocate memory for the output string using [`xcalloc`](xmalloc.c.driver.md#xcalloc).
    - Iterate over each character in the input string `expanded`.
    - If a '#' character is encountered, determine the number of leading '#' characters and their width using [`format_leading_hashes`](#format_leading_hashes).
    - Adjust the copy width based on the skip value and append the appropriate number of '#' characters to the output string.
    - If a UTF-8 character is encountered, process it using [`utf8_open`](utf8.c.driver.md#utf8_open) and [`utf8_append`](utf8.c.driver.md#utf8_append), and append it to the output string if it is beyond the skip width.
    - For ASCII characters, append them directly to the output string if they are beyond the skip width.
    - Continue processing until the end of the input string is reached.
    - Terminate the output string with a null character and return it.
- **Output**:
    - A dynamically allocated string that is a trimmed version of the input string, fitting within the specified width limit.
- **Functions called**:
    - [`format_width`](#format_width)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`format_leading_hashes`](#format_leading_hashes)
    - [`format_skip`](format.c.driver.md#format_skip)
    - [`utf8_open`](utf8.c.driver.md#utf8_open)
    - [`utf8_append`](utf8.c.driver.md#utf8_append)


