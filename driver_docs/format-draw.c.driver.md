# Purpose
This C source code file is part of the tmux project, a terminal multiplexer that allows users to switch between several programs in one terminal. The file is responsible for handling the formatting and drawing of text on the screen, particularly focusing on how different sections of text are aligned and styled. It defines a structure `format_range` to manage different formatting ranges and uses various functions to draw text with specific styles and alignments, such as left, center, right, and absolute center. The code also handles special cases like lists and markers within the text, ensuring that these elements are displayed correctly according to the specified styles.

The file includes several static functions that manage the creation, updating, and drawing of these formatted text ranges. It also provides utility functions to calculate the width of formatted text and to trim text from the left or right while considering special formatting characters. The code is designed to be integrated into the larger tmux application, where it can be used to render formatted text in a terminal session. The functions are not intended to be public APIs but rather internal utilities to support the text rendering capabilities of tmux.
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
    - `start`: An unsigned integer indicating the starting position of the format range.
    - `end`: An unsigned integer indicating the ending position of the format range.
    - `type`: An enumeration value representing the type of style range.
    - `argument`: An unsigned integer used as an argument for certain style range types.
    - `string`: A character array of size 16 used to store a string associated with the format range.
    - `entry`: A TAILQ_ENTRY structure for linking format_range elements in a tail queue.
- **Description**: The `format_range` structure is used to define a range within a screen format, including its start and end positions, associated screen, and style type. It includes an index for identification, a type to specify the style range, and an argument or string for additional range-specific data. The structure is designed to be part of a linked list using the TAILQ_ENTRY macro, allowing multiple format ranges to be managed in a queue.


# Functions

---
### format_is_type<!-- {{#callable:format_is_type}} -->
The `format_is_type` function checks if a given format range matches a specified style based on their type and additional attributes.
- **Inputs**:
    - `fr`: A pointer to a `struct format_range` which contains information about the format range to be checked.
    - `sy`: A pointer to a `struct style` which contains information about the style to be compared against the format range.
- **Control Flow**:
    - Check if the type of the format range (`fr->type`) is not equal to the style's range type (`sy->range_type`); if so, return 0 indicating no match.
    - Use a switch statement to handle different types of format ranges.
    - For `STYLE_RANGE_NONE`, `STYLE_RANGE_LEFT`, and `STYLE_RANGE_RIGHT`, return 1 indicating a match since these types do not require further checks.
    - For `STYLE_RANGE_PANE`, `STYLE_RANGE_WINDOW`, and `STYLE_RANGE_SESSION`, compare the `argument` fields of the format range and style; return 1 if they match, otherwise return 0.
    - For `STYLE_RANGE_USER`, compare the `string` fields of the format range and style using `strcmp`; return 1 if they are equal, otherwise return 0.
    - If none of the cases match, return 1 by default.
- **Output**:
    - Returns an integer: 1 if the format range matches the style, 0 otherwise.


---
### format_free_range<!-- {{#callable:format_free_range}} -->
The `format_free_range` function removes a `format_range` entry from a `format_ranges` list and deallocates its memory.
- **Inputs**:
    - `frs`: A pointer to a `format_ranges` structure, which is a list of `format_range` entries.
    - `fr`: A pointer to a `format_range` structure that is to be removed from the list and freed.
- **Control Flow**:
    - The function calls `TAILQ_REMOVE` to remove the `format_range` entry `fr` from the `format_ranges` list `frs` using the `entry` field as the list link.
    - The function then calls `free` to deallocate the memory occupied by the `format_range` entry `fr`.
- **Output**:
    - The function does not return any value; it performs operations on the provided data structures and frees memory.


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
    - Check if the `frs` pointer is NULL; if so, return immediately as there is nothing to update.
    - Iterate over each format range `fr` in the `frs` list using `TAILQ_FOREACH_SAFE`.
    - For each range, check if its associated screen `fr->s` matches the provided screen `s`; if not, continue to the next range.
    - Check if the range is completely outside the specified start and width bounds; if so, free the range and continue to the next one.
    - Adjust the start and end of the range to fit within the specified start and width bounds.
    - If the adjusted start and end are equal, free the range as it is now empty.
    - Apply the offset to the start and end of the range to finalize the update.
- **Output**:
    - The function does not return a value; it modifies the `format_ranges` list in place, updating or removing ranges as necessary.
- **Functions called**:
    - [`format_free_range`](#format_free_range)


---
### format_draw_put<!-- {{#callable:format_draw_put}} -->
The `format_draw_put` function moves the cursor on a target screen and copies a specified portion of a source screen to it, updating format ranges accordingly.
- **Inputs**:
    - `octx`: A pointer to a `screen_write_ctx` structure representing the context of the screen where drawing operations are performed.
    - `ocx`: An unsigned integer representing the x-coordinate of the cursor on the target screen.
    - `ocy`: An unsigned integer representing the y-coordinate of the cursor on the target screen.
    - `s`: A pointer to a `screen` structure representing the source screen from which content is copied.
    - `frs`: A pointer to a `format_ranges` structure that holds the format ranges to be updated.
    - `offset`: An unsigned integer representing the offset from the cursor on the target screen where the content will be placed.
    - `start`: An unsigned integer representing the starting position on the source screen from which content will be copied.
    - `width`: An unsigned integer representing the width of the content to be copied from the source screen.
- **Control Flow**:
    - The function first moves the cursor on the target screen to the position specified by `ocx + offset` and `ocy` using `screen_write_cursormove`.
    - It then copies a portion of the source screen `s` starting at `start` with a width of `width` to the target screen using `screen_write_fast_copy`.
    - Finally, it updates the format ranges `frs` to reflect the changes made by the copy operation using [`format_update_ranges`](#format_update_ranges).
- **Output**:
    - The function does not return a value; it performs operations on the provided screen context and updates the format ranges in place.
- **Functions called**:
    - [`format_update_ranges`](#format_update_ranges)


---
### format_draw_put_list<!-- {{#callable:format_draw_put_list}} -->
The `format_draw_put_list` function draws a portion of a list on a screen, ensuring that the focused section remains visible and adding markers if the list is trimmed.
- **Inputs**:
    - `octx`: A pointer to the screen write context where the list will be drawn.
    - `ocx`: The x-coordinate on the screen where drawing starts.
    - `ocy`: The y-coordinate on the screen where drawing starts.
    - `offset`: The offset from the starting x-coordinate for drawing.
    - `width`: The width available for drawing the list.
    - `list`: A pointer to the screen structure representing the list to be drawn.
    - `list_left`: A pointer to the screen structure representing the left marker to indicate more content to the left.
    - `list_right`: A pointer to the screen structure representing the right marker to indicate more content to the right.
    - `focus_start`: The starting index of the focus area within the list.
    - `focus_end`: The ending index of the focus area within the list.
    - `frs`: A pointer to the format ranges structure used for managing style ranges.
- **Control Flow**:
    - Check if the available width is sufficient to draw the entire list; if so, draw the list and return.
    - Calculate the center of the focus area and determine the starting point for drawing the list to keep the focus visible.
    - Adjust the starting point if it exceeds the list's width, ensuring it fits within the available width.
    - If the list is trimmed on the left, draw the left marker and adjust the offset, start, and width accordingly.
    - If the list is trimmed on the right, draw the right marker and adjust the width accordingly.
    - Draw the visible portion of the list using the calculated start and width.
- **Output**:
    - The function does not return a value; it modifies the screen context to display the list with appropriate markers and focus.
- **Functions called**:
    - [`format_draw_put`](#format_draw_put)


---
### format_draw_none<!-- {{#callable:format_draw_none}} -->
The `format_draw_none` function arranges and draws sections of a screen (left, centre, right, and absolute centre) within a given available width, prioritizing the left and right sections over the centre.
- **Inputs**:
    - `octx`: A pointer to a `screen_write_ctx` structure, which represents the context for writing to a screen.
    - `available`: An unsigned integer representing the total available width for drawing.
    - `ocx`: An unsigned integer representing the x-coordinate offset on the screen where drawing begins.
    - `ocy`: An unsigned integer representing the y-coordinate offset on the screen where drawing begins.
    - `left`: A pointer to a `screen` structure representing the left section of the screen to be drawn.
    - `centre`: A pointer to a `screen` structure representing the centre section of the screen to be drawn.
    - `right`: A pointer to a `screen` structure representing the right section of the screen to be drawn.
    - `abs_centre`: A pointer to a `screen` structure representing the absolute centre section of the screen to be drawn.
    - `frs`: A pointer to a `format_ranges` structure, which holds format range information for the drawing operation.
- **Control Flow**:
    - Initialize the widths of the left, centre, right, and absolute centre sections from their respective screen structures.
    - Enter a loop to reduce the widths of the sections if their combined width exceeds the available width, prioritizing reducing the centre, then the right, and finally the left section.
    - Call [`format_draw_put`](#format_draw_put) to draw the left section starting at the beginning of the available space.
    - Call [`format_draw_put`](#format_draw_put) to draw the right section starting at the end of the available space minus the width of the right section.
    - Calculate the position for the centre section to be drawn halfway between the left and right sections, and call [`format_draw_put`](#format_draw_put) to draw it.
    - Adjust the width of the absolute centre section if it exceeds the available width, then call [`format_draw_put`](#format_draw_put) to draw it in the perfect centre of the available space.
- **Output**:
    - The function does not return a value; it performs drawing operations on the screen context provided.
- **Functions called**:
    - [`format_draw_put`](#format_draw_put)


---
### format_draw_left<!-- {{#callable:format_draw_left}} -->
The `format_draw_left` function arranges and draws multiple screen sections within a given width, prioritizing the left section and handling a list section with focus management.
- **Inputs**:
    - `octx`: A pointer to the screen write context where the drawing operations will be applied.
    - `available`: The total available width for drawing the sections.
    - `ocx`: The x-coordinate offset in the output context where drawing begins.
    - `ocy`: The y-coordinate offset in the output context where drawing begins.
    - `left`: A pointer to the screen structure representing the left section to be drawn.
    - `centre`: A pointer to the screen structure representing the centre section to be drawn.
    - `right`: A pointer to the screen structure representing the right section to be drawn.
    - `abs_centre`: A pointer to the screen structure representing the absolute centre section to be drawn.
    - `list`: A pointer to the screen structure representing the list section to be drawn.
    - `list_left`: A pointer to the screen structure representing the left marker of the list.
    - `list_right`: A pointer to the screen structure representing the right marker of the list.
    - `after`: A pointer to the screen structure representing the section to be drawn after the list.
    - `focus_start`: The starting index of the focus within the list section.
    - `focus_end`: The ending index of the focus within the list section.
    - `frs`: A pointer to a structure holding format ranges for style management.
- **Control Flow**:
    - Initialize the widths of all sections from their respective screen structures.
    - Iteratively reduce the widths of sections (centre, list, right, after, left) until their total fits within the available width.
    - If the list section is completely trimmed, call [`format_draw_none`](#format_draw_none) to handle drawing without a list.
    - Draw the left section starting at position 0.
    - Draw the right section aligned to the right edge of the available space.
    - Draw the 'after' section immediately following the list section.
    - Calculate the position for the centre section and draw it centered between the list and right sections.
    - If no focus is specified, default the focus to the start of the list.
    - Draw the list section with focus management using [`format_draw_put_list`](#format_draw_put_list).
    - Draw the absolute centre section perfectly centered in the available space.
- **Output**:
    - The function does not return a value; it performs drawing operations on the provided screen write context.
- **Functions called**:
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
    - `focus_start`: The starting index of the focus range within the list.
    - `focus_end`: The ending index of the focus range within the list.
    - `frs`: A pointer to the format ranges structure used to manage style ranges.
- **Control Flow**:
    - Initialize the widths of all sections (left, centre, right, abs_centre, list, after) from their respective screen structures.
    - Enter a loop to trim the widths of sections in the order: list, after, centre, right, left, until their total width fits within the available width.
    - If the list width is reduced to zero, copy the 'after' section to the centre and call [`format_draw_none`](#format_draw_none) to handle drawing without a list.
    - Draw the left section starting at position 0 using [`format_draw_put`](#format_draw_put).
    - Draw the right section starting at position `available - width_right` using [`format_draw_put`](#format_draw_put).
    - Calculate the middle position of the available space and use it to position the centre, after, and list sections around the middle.
    - Draw the centre section offset from the middle by `- width_list / 2 - width_centre` using [`format_draw_put`](#format_draw_put).
    - Draw the after section offset from the middle by `- width_list / 2 + width_list` using [`format_draw_put`](#format_draw_put).
    - Determine the focus range for the list, defaulting to the centre if not specified, and draw the list using [`format_draw_put_list`](#format_draw_put_list).
    - Draw the abs_centre section in the perfect centre of the available space using [`format_draw_put`](#format_draw_put).
- **Output**:
    - The function does not return a value; it performs drawing operations on the provided screen write context.
- **Functions called**:
    - [`format_draw_none`](#format_draw_none)
    - [`format_draw_put`](#format_draw_put)
    - [`format_draw_put_list`](#format_draw_put_list)


---
### format_draw_right<!-- {{#callable:format_draw_right}} -->
The `format_draw_right` function arranges and draws multiple screen sections within a given available width, focusing on placing a list on the right side of the layout.
- **Inputs**:
    - `octx`: A pointer to the screen write context where the drawing operations will be applied.
    - `available`: The total available width for drawing the sections.
    - `ocx`: The starting x-coordinate on the output screen.
    - `ocy`: The starting y-coordinate on the output screen.
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
    - Initialize the widths of each section (left, centre, right, abs_centre, list, after) based on their respective screen's current x-coordinate (cx).
    - Iteratively reduce the widths of sections (starting with centre, then list, right, after, and left) until their combined width fits within the available width.
    - If the list section's width is reduced to zero, copy the 'after' section to the 'right' section and call [`format_draw_none`](#format_draw_none) to handle drawing without a list.
    - Draw the left section starting at position 0.
    - Draw the 'after' section at the position calculated as available - width_after.
    - Draw the right section at the position calculated as available - width_right - width_list - width_after.
    - Draw the centre section at a position calculated to be halfway between the left section and the right section's starting position, adjusted for the centre's width.
    - If no focus is specified, set the focus to the start of the list; otherwise, draw the list section using [`format_draw_put_list`](#format_draw_put_list) with the specified focus range.
    - Draw the abs_centre section in the perfect centre of the available horizontal space.
- **Output**:
    - The function does not return a value; it performs drawing operations directly on the provided screen write context.
- **Functions called**:
    - [`format_draw_none`](#format_draw_none)
    - [`format_draw_put`](#format_draw_put)
    - [`format_draw_put_list`](#format_draw_put_list)


---
### format_draw_absolute_centre<!-- {{#callable:format_draw_absolute_centre}} -->
The `format_draw_absolute_centre` function arranges and draws multiple screen sections within a given available width, ensuring that a list and an absolute center section are perfectly centered horizontally.
- **Inputs**:
    - `octx`: A pointer to a `screen_write_ctx` structure, which represents the context for writing to a screen.
    - `available`: An unsigned integer representing the total available width for drawing.
    - `ocx`: An unsigned integer representing the starting x-coordinate on the screen.
    - `ocy`: An unsigned integer representing the starting y-coordinate on the screen.
    - `left`: A pointer to a `screen` structure representing the left section to be drawn.
    - `centre`: A pointer to a `screen` structure representing the center section to be drawn.
    - `right`: A pointer to a `screen` structure representing the right section to be drawn.
    - `abs_centre`: A pointer to a `screen` structure representing the absolute center section to be drawn.
    - `list`: A pointer to a `screen` structure representing the list section to be drawn.
    - `list_left`: A pointer to a `screen` structure representing the left marker of the list.
    - `list_right`: A pointer to a `screen` structure representing the right marker of the list.
    - `after`: A pointer to a `screen` structure representing the section to be drawn after the list.
    - `focus_start`: An integer indicating the start position of the focus within the list, or -1 if no focus is specified.
    - `focus_end`: An integer indicating the end position of the focus within the list, or -1 if no focus is specified.
    - `frs`: A pointer to a `format_ranges` structure, which holds information about format ranges for styling.
- **Control Flow**:
    - Initialize width variables for each section based on their respective screen's current width (`cx`).
    - Trim the widths of the left, center, and right sections to fit within the available width, prioritizing trimming the center first, then the right, and finally the left.
    - Independently trim the widths of the list, after, and absolute center sections to fit within the available width, prioritizing trimming the list first, then after, and finally the absolute center.
    - Draw the left section starting at position 0 using [`format_draw_put`](#format_draw_put).
    - Draw the right section starting at `available - width_right` using [`format_draw_put`](#format_draw_put).
    - Calculate the middle position for the center section and draw it at `middle - width_centre` using [`format_draw_put`](#format_draw_put).
    - If no focus is specified, set the focus to the center of the list.
    - Calculate the offset for the absolute center and list to be perfectly centered horizontally.
    - Draw the absolute center section before the list using [`format_draw_put`](#format_draw_put).
    - Draw the list section in the absolute center using [`format_draw_put_list`](#format_draw_put_list).
    - Draw the after section at the end of the center using [`format_draw_put`](#format_draw_put).
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
    - Initialize the count of leading '#' characters to zero and iterate through the string, incrementing the count for each '#' found.
    - If no '#' characters are found, set the width to zero and return the original string pointer.
    - If the character following the '#' sequence is not '[', calculate the width as half the count of '#' characters, rounding up if the count is odd, and return a pointer to the position after the '#' characters.
    - If the character following the '#' sequence is '[', calculate the width as half the count of '#' characters.
    - If the count of '#' characters is even, return a pointer to the position after the '#' characters, indicating that all '#' characters are escaped.
    - If the count of '#' characters is odd, return a pointer to the last '#' character, indicating a style.
- **Output**:
    - A pointer to a position in the string after processing the leading '#' characters, which may point to the original string, the position after the '#' characters, or the last '#' character, depending on the conditions met.


---
### format_draw_many<!-- {{#callable:format_draw_many}} -->
The `format_draw_many` function writes a specified character multiple times to a screen context using a given style.
- **Inputs**:
    - `ctx`: A pointer to a `screen_write_ctx` structure representing the screen context where the characters will be drawn.
    - `sy`: A pointer to a `style` structure that defines the style attributes to be applied to the characters being drawn.
    - `ch`: A character that will be drawn repeatedly on the screen.
    - `n`: An unsigned integer specifying the number of times the character `ch` should be drawn.
- **Control Flow**:
    - The function begins by setting the UTF-8 data of the style's grid cell to the character `ch` using `utf8_set`.
    - A loop iterates `n` times, where `n` is the number of times the character should be drawn.
    - In each iteration, the function calls `screen_write_cell` to draw the character on the screen using the style's grid cell.
- **Output**:
    - The function does not return a value; it performs its operation by modifying the screen context directly.


---
### format_draw<!-- {{#callable:format_draw}} -->
The `format_draw` function processes a formatted string with style specifications and draws it onto a screen context, handling alignment and style ranges.
- **Inputs**:
    - `octx`: A pointer to a `screen_write_ctx` structure representing the screen context where the formatted output will be drawn.
    - `base`: A pointer to a `grid_cell` structure representing the base style for drawing.
    - `available`: An unsigned integer representing the available width for drawing on the screen.
    - `expanded`: A constant character pointer to the formatted string that needs to be drawn.
    - `srs`: A pointer to a `style_ranges` structure where style ranges will be stored.
    - `default_colours`: An integer flag indicating whether to use default colors for the drawing.
- **Control Flow**:
    - Initialize variables and structures for screen contexts and style handling.
    - Iterate over the `expanded` string, parsing and applying styles, and drawing characters to the appropriate screen context based on alignment and style.
    - Handle special sequences of '#' characters and UTF-8 characters, updating the current screen context accordingly.
    - Manage style changes, including pushing and popping default styles, and handle list states for different alignments.
    - After processing the string, clear the available area if a fill color is specified.
    - Draw the prepared screens onto the output context `octx` based on the determined list alignment.
    - Create and log style ranges for the drawn content, storing them in the provided `style_ranges` structure.
    - Free allocated resources and restore the original cursor position on the screen context.
- **Output**:
    - The function does not return a value but modifies the screen context `octx` to display the formatted and styled output, and updates the `style_ranges` structure with the style ranges used.
- **Functions called**:
    - [`format_draw_many`](#format_draw_many)
    - [`format_free_range`](#format_free_range)
    - [`format_is_type`](#format_is_type)
    - [`format_draw_none`](#format_draw_none)
    - [`format_draw_left`](#format_draw_left)
    - [`format_draw_centre`](#format_draw_centre)
    - [`format_draw_right`](#format_draw_right)
    - [`format_draw_absolute_centre`](#format_draw_absolute_centre)


---
### format_width<!-- {{#callable:format_width}} -->
The `format_width` function calculates the display width of a string, taking into account special formatting sequences and UTF-8 characters.
- **Inputs**:
    - `expanded`: A constant character pointer to the string whose width is to be calculated.
- **Control Flow**:
    - Initialize pointers and variables for tracking position and width.
    - Iterate over each character in the input string `expanded`.
    - If a '#' character is encountered, call [`format_leading_hashes`](#format_leading_hashes) to determine the number of leading '#' characters and their contribution to the width.
    - If the '#' sequence is followed by another '#', skip the style sequence enclosed in brackets using `format_skip`.
    - If a UTF-8 character is detected, use `utf8_open` and `utf8_append` to process the character and add its width to the total if valid.
    - For ASCII characters between 0x20 and 0x7E, increment the width by one.
    - Continue iterating until the end of the string is reached.
    - Return the total calculated width.
- **Output**:
    - The function returns an unsigned integer representing the calculated width of the input string, considering special formatting and UTF-8 characters.
- **Functions called**:
    - [`format_leading_hashes`](#format_leading_hashes)


---
### format_trim_left<!-- {{#callable:format_trim_left}} -->
The `format_trim_left` function trims a string from the left, taking into account special formatting characters and a specified width limit.
- **Inputs**:
    - `expanded`: A constant character pointer to the input string that may contain special formatting characters.
    - `limit`: An unsigned integer specifying the maximum width of the output string after trimming.
- **Control Flow**:
    - Allocate memory for the output string, which is twice the length of the input string plus one for the null terminator.
    - Initialize pointers for traversing the input string and writing to the output string.
    - Iterate over the input string until the end is reached or the accumulated width exceeds the specified limit.
    - If a '#' character is encountered, determine the number of leading '#' characters and their width using [`format_leading_hashes`](#format_leading_hashes).
    - Adjust the leading width if it exceeds the remaining width limit and copy the appropriate number of '#' characters to the output string.
    - If a UTF-8 character is detected, append it to the output string if it fits within the width limit.
    - For ASCII characters between 0x20 and 0x7E, copy them to the output string if they fit within the width limit.
    - Terminate the output string with a null character and return it.
- **Output**:
    - A dynamically allocated string that is a trimmed version of the input, respecting the width limit and special formatting.
- **Functions called**:
    - [`format_leading_hashes`](#format_leading_hashes)


---
### format_trim_right<!-- {{#callable:format_trim_right}} -->
The `format_trim_right` function trims a given string from the right to fit within a specified width limit, taking into account special formatting characters and UTF-8 encoding.
- **Inputs**:
    - `expanded`: A constant character pointer to the input string that needs to be trimmed.
    - `limit`: An unsigned integer specifying the maximum width the output string should have.
- **Control Flow**:
    - Calculate the total width of the input string using [`format_width`](#format_width) function.
    - If the total width is less than or equal to the limit, return a duplicate of the input string.
    - Calculate the number of characters to skip by subtracting the limit from the total width.
    - Allocate memory for the output string, which is twice the length of the input string plus one for the null terminator.
    - Iterate over each character in the input string, handling special cases for '#' characters and UTF-8 sequences.
    - For '#' characters, determine the number of leading hashes and adjust the copy width based on the skip value.
    - For UTF-8 sequences, append the sequence to the output if the current width is greater than or equal to the skip value.
    - For ASCII characters, append them to the output if the current width is greater than or equal to the skip value.
    - Terminate the output string with a null character and return it.
- **Output**:
    - A dynamically allocated string that is a trimmed version of the input, fitting within the specified width limit.
- **Functions called**:
    - [`format_width`](#format_width)
    - [`format_leading_hashes`](#format_leading_hashes)


