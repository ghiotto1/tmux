# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it focuses on managing and manipulating styles within the application. The code provides functionality to parse, apply, and convert styles, which are configurations that define the appearance of text and interface elements in tmux. The styles are specified in a string format, such as "fg=colour,bg=colour,bright", and the code includes functions to parse these strings into structured data, apply them to grid cells, and convert them back into strings for display or storage.

The file defines several key functions, such as [`style_parse`](#style_parse), which interprets style strings and updates style structures accordingly, and [`style_tostring`](#style_tostring), which converts a style structure back into a string representation. It also includes functions to apply styles to grid cells, either by adding them on top of existing styles or by setting them as defaults. The code is structured to handle various style attributes, including foreground and background colors, alignment, and range types. This file is a crucial component of tmux's styling system, allowing for flexible and dynamic customization of the user interface.
# Imports and Dependencies

---
- `sys/types.h`
- `ctype.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### style_default
- **Type**: `struct style`
- **Description**: The `style_default` variable is a static instance of the `struct style` data structure, which defines default styling attributes for a user interface component. It includes settings for alignment, list behavior, range type, width, padding, and default base style, among others.
- **Use**: This variable is used as a baseline style configuration that can be copied or modified to apply consistent styling across different components in the application.


# Functions

---
### style_set_range_string<!-- {{#callable:style_set_range_string}} -->
The `style_set_range_string` function copies a given string into the `range_string` field of a `style` structure.
- **Inputs**:
    - `sy`: A pointer to a `struct style` where the `range_string` will be set.
    - `s`: A constant character pointer to the string that will be copied into the `range_string` field of the `style` structure.
- **Control Flow**:
    - The function uses `strlcpy` to copy the string `s` into the `range_string` field of the `style` structure pointed to by `sy`.
    - The size of the destination buffer is specified as `sizeof sy->range_string` to ensure no buffer overflow occurs.
- **Output**:
    - The function does not return any value; it modifies the `range_string` field of the `style` structure in place.


---
### style_parse<!-- {{#callable:style_parse}} -->
The `style_parse` function parses a style string and applies the specified style attributes to a given style structure.
- **Inputs**:
    - `sy`: A pointer to a `struct style` that will be modified based on the parsed input string.
    - `base`: A pointer to a `const struct grid_cell` that provides default style attributes for certain cases.
    - `in`: A `const char*` representing the input style string to be parsed.
- **Control Flow**:
    - Check if the input string is empty and return 0 if true.
    - Save the current state of the style structure in case of an error during parsing.
    - Iterate over the input string, skipping delimiters and processing each style attribute.
    - For each attribute, check its type and update the style structure accordingly, handling special cases like 'default', 'ignore', 'list', 'range', 'align', 'fill', 'fg', 'bg', 'us', 'none', and 'width'.
    - If an error occurs during parsing, restore the original style structure and return -1.
    - Continue parsing until the end of the input string is reached.
    - Return 0 if parsing completes successfully without errors.
- **Output**:
    - Returns 0 on successful parsing and application of the style, or -1 if an error occurs during parsing.
- **Functions called**:
    - [`style_copy`](#style_copy)
    - [`style_set_range_string`](#style_set_range_string)


---
### style_tostring<!-- {{#callable:style_tostring}} -->
The `style_tostring` function converts a `style` structure into a human-readable string representation, detailing various style attributes.
- **Inputs**:
    - `sy`: A pointer to a `struct style` which contains style attributes to be converted into a string.
- **Control Flow**:
    - Initialize a static character array `s` to store the resulting string and set the initial offset `off` to 0.
    - Check if `sy->list` is not `STYLE_LIST_OFF` and append the corresponding list type to `s`.
    - Check if `sy->range_type` is not `STYLE_RANGE_NONE` and append the corresponding range type and argument to `s`.
    - Check if `sy->align` is not `STYLE_ALIGN_DEFAULT` and append the corresponding alignment type to `s`.
    - Check if `sy->default_type` is not `STYLE_DEFAULT_BASE` and append the corresponding default type to `s`.
    - Check if `sy->fill` is not 8 and append the fill color to `s`.
    - Check if `gc->fg`, `gc->bg`, and `gc->us` are not 8 and append the foreground, background, and underscore colors to `s` respectively.
    - Check if `gc->attr` is not 0 and append the attributes to `s`.
    - Check if `sy->width` and `sy->pad` are non-negative and append them to `s`.
    - If `s` is still empty, return "default"; otherwise, return the constructed string `s`.
- **Output**:
    - A string representing the style attributes of the given `style` structure, or "default" if no attributes are set.


---
### style_add<!-- {{#callable:style_add}} -->
The `style_add` function applies a style to a given grid cell by modifying its attributes based on a style retrieved from options.
- **Inputs**:
    - `gc`: A pointer to a `grid_cell` structure that will be modified to include the new style attributes.
    - `oo`: A pointer to an `options` structure from which the style is retrieved.
    - `name`: A string representing the name of the style to be applied.
    - `ft`: A pointer to a `format_tree` structure used for formatting, which can be NULL.
- **Control Flow**:
    - Check if the `ft` (format tree) is NULL; if so, create a new format tree `ft0` with `format_create` and assign it to `ft`.
    - Retrieve the style `sy` from the options `oo` using the style name `name` and the format tree `ft`.
    - If the retrieved style `sy` is NULL, use the default style `style_default`.
    - If the foreground color `fg` of `sy` is not 8, set the foreground color of `gc` to `sy->gc.fg`.
    - If the background color `bg` of `sy` is not 8, set the background color of `gc` to `sy->gc.bg`.
    - If the underline style `us` of `sy` is not 8, set the underline style of `gc` to `sy->gc.us`.
    - Apply the attributes from `sy->gc.attr` to `gc->attr` using a bitwise OR operation.
    - If a new format tree `ft0` was created, free it using `format_free`.
- **Output**:
    - The function modifies the `grid_cell` structure pointed to by `gc` to include the style attributes from the specified style.


---
### style_apply<!-- {{#callable:style_apply}} -->
The `style_apply` function initializes a grid cell with default style settings and then applies additional style modifications based on specified options.
- **Inputs**:
    - `gc`: A pointer to a `struct grid_cell` that will be initialized and modified with style settings.
    - `oo`: A pointer to a `struct options` that contains style options to be applied.
    - `name`: A string representing the name of the style to be applied.
    - `ft`: A pointer to a `struct format_tree` used for formatting, which can be NULL.
- **Control Flow**:
    - Copy the default grid cell settings into the provided `gc` structure using `memcpy`.
    - Call [`style_add`](#style_add) to apply additional style settings from the options `oo` and name `name` onto the `gc` structure.
- **Output**:
    - The function modifies the `gc` structure in place, applying the default and specified styles.
- **Functions called**:
    - [`style_add`](#style_add)


---
### style_set<!-- {{#callable:style_set}} -->
The `style_set` function initializes a `style` structure with default values and then updates its `grid_cell` component with the values from a provided `grid_cell` structure.
- **Inputs**:
    - `sy`: A pointer to a `style` structure that will be initialized and updated.
    - `gc`: A pointer to a `grid_cell` structure whose values will be used to update the `grid_cell` component of the `style` structure.
- **Control Flow**:
    - Copy the default style values from `style_default` into the `style` structure pointed to by `sy`.
    - Copy the values from the `grid_cell` structure pointed to by `gc` into the `gc` component of the `style` structure pointed to by `sy`.
- **Output**:
    - The function does not return a value; it modifies the `style` structure pointed to by `sy` in place.


---
### style_copy<!-- {{#callable:style_copy}} -->
The `style_copy` function copies the contents of one `struct style` to another using `memcpy`.
- **Inputs**:
    - `dst`: A pointer to the destination `struct style` where the data will be copied to.
    - `src`: A pointer to the source `struct style` from which the data will be copied.
- **Control Flow**:
    - The function uses `memcpy` to copy the contents from the source `struct style` (`src`) to the destination `struct style` (`dst`).
    - The size of the data copied is determined by the size of the destination `struct style` (`sizeof *dst`).
- **Output**:
    - The function does not return any value; it performs an in-place copy of the style data from `src` to `dst`.


---
### style_set_scrollbar_style_from_option<!-- {{#callable:style_set_scrollbar_style_from_option}} -->
The function `style_set_scrollbar_style_from_option` sets the scrollbar style based on the options provided, defaulting to predefined values if no specific style is found.
- **Inputs**:
    - `sb_style`: A pointer to a `struct style` where the scrollbar style will be set.
    - `oo`: A pointer to a `struct options` from which the scrollbar style option will be retrieved.
- **Control Flow**:
    - Retrieve the scrollbar style from the options using `options_string_to_style` with the key 'pane-scrollbars-style'.
    - If the retrieved style is NULL, set `sb_style` to the default grid cell style, and assign default width, padding, and character values.
    - If a style is retrieved, copy it to `sb_style`.
    - Check if the width of `sb_style` is less than 1, and if so, set it to the default width.
    - Check if the padding of `sb_style` is less than 0, and if so, set it to the default padding.
    - Set the character data of `sb_style` to the default scrollbar character.
- **Output**:
    - The function does not return a value; it modifies the `sb_style` structure in place.
- **Functions called**:
    - [`style_set`](#style_set)
    - [`style_copy`](#style_copy)


