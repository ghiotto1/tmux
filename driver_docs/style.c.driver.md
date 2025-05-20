# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it focuses on managing and manipulating styles within the application. The code provides functionality to parse, apply, and convert styles, which are configurations that define the appearance of text elements, such as foreground and background colors, attributes, and alignment. The file includes functions to parse style strings, apply styles to grid cells, and convert styles to string representations. It also defines a default style and provides mechanisms to copy and set styles based on existing grid cell configurations.

The code is structured around a central theme of style management, with key functions like [`style_parse`](#style_parse), [`style_tostring`](#style_tostring), [`style_add`](#style_add), and [`style_apply`](#style_apply) that handle different aspects of style processing. The [`style_parse`](#style_parse) function is particularly important as it interprets style strings and updates style structures accordingly. The file does not define a public API or external interfaces directly but is likely intended to be used internally within the tmux codebase to handle style-related operations. The inclusion of headers like `tmux.h` suggests that this file is part of a larger system and relies on shared data structures and functions defined elsewhere in the project.
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
- **Description**: The `style_default` variable is a static instance of the `struct style` data structure, which is used to define default styling attributes for a user interface component. It includes fields for alignment, list style, range type, width, padding, and other graphical attributes.
- **Use**: This variable is used as a baseline style configuration that can be copied or modified to apply consistent styling across different components in the application.


# Functions

---
### style_set_range_string<!-- {{#callable:style_set_range_string}} -->
The function `style_set_range_string` copies a given string into the `range_string` field of a `style` structure.
- **Inputs**:
    - `sy`: A pointer to a `struct style` where the `range_string` field will be updated.
    - `s`: A constant character pointer to the string that will be copied into the `range_string` field of the `style` structure.
- **Control Flow**:
    - The function uses [`strlcpy`](compat/strlcpy.c.driver.md#strlcpy) to copy the string `s` into the `range_string` field of the `style` structure pointed to by `sy`.
    - The size of the destination buffer is specified as the size of `sy->range_string`.
- **Output**:
    - The function does not return any value; it modifies the `range_string` field of the `style` structure in place.
- **Functions called**:
    - [`strlcpy`](compat/strlcpy.c.driver.md#strlcpy)


---
### style_parse<!-- {{#callable:style_parse}} -->
The `style_parse` function parses a style string and applies the specified style attributes to a given style structure.
- **Inputs**:
    - `sy`: A pointer to a `struct style` that will be modified based on the parsed input string.
    - `base`: A pointer to a `struct grid_cell` that provides default style values for certain attributes.
    - `in`: A string containing style attributes to be parsed and applied to the `sy` structure.
- **Control Flow**:
    - Check if the input string is empty and return 0 if true.
    - Save the current state of the style structure `sy` to `saved`.
    - Iterate over the input string, skipping delimiters and processing each style attribute.
    - For each attribute, copy it into a temporary buffer `tmp` and determine its type by comparing it against known style keywords.
    - Apply the corresponding style changes to the `sy` structure based on the attribute type.
    - If an error occurs during parsing, restore the original style from `saved` and return -1.
    - Continue parsing until the end of the input string is reached.
    - Return 0 if parsing completes successfully without errors.
- **Output**:
    - Returns 0 on successful parsing and application of styles, or -1 if an error occurs during parsing.
- **Functions called**:
    - [`style_copy`](#style_copy)
    - [`strlcpy`](compat/strlcpy.c.driver.md#strlcpy)
    - [`style_set_range_string`](#style_set_range_string)
    - [`strtonum`](compat/strtonum.c.driver.md#strtonum)
    - [`colour_fromstring`](colour.c.driver.md#colour_fromstring)
    - [`attributes_fromstring`](attributes.c.driver.md#attributes_fromstring)


---
### style_tostring<!-- {{#callable:style_tostring}} -->
The `style_tostring` function converts a `style` structure into a human-readable string representation, detailing various style attributes.
- **Inputs**:
    - `sy`: A pointer to a `struct style` which contains style attributes to be converted into a string.
- **Control Flow**:
    - Initialize a static character array `s` to store the resulting string and set the initial offset `off` to 0.
    - Check if the `list` attribute of the style is not `STYLE_LIST_OFF` and append the corresponding string representation to `s`.
    - Check if the `range_type` attribute is not `STYLE_RANGE_NONE` and append the corresponding string representation to `s`, using `range_argument` or `range_string` if necessary.
    - Check if the `align` attribute is not `STYLE_ALIGN_DEFAULT` and append the corresponding string representation to `s`.
    - Check if the `default_type` attribute is not `STYLE_DEFAULT_BASE` and append the corresponding string representation to `s`.
    - Check if the `fill` attribute is not 8 and append the corresponding color string to `s`.
    - Check if the foreground (`fg`), background (`bg`), and underline (`us`) colors of the grid cell are not 8 and append their corresponding color strings to `s`.
    - Check if the `attr` attribute of the grid cell is not 0 and append the corresponding attribute string to `s`.
    - Check if the `width` and `pad` attributes are non-negative and append their values to `s`.
    - If the resulting string `s` is empty, return "default"; otherwise, return `s`.
- **Output**:
    - A string representation of the style attributes, or "default" if no attributes are set.
- **Functions called**:
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)
    - [`colour_tostring`](colour.c.driver.md#colour_tostring)
    - [`attributes_tostring`](attributes.c.driver.md#attributes_tostring)


---
### style_add<!-- {{#callable:style_add}} -->
The `style_add` function applies a style to a given grid cell based on options and a format tree.
- **Inputs**:
    - `gc`: A pointer to a `struct grid_cell` where the style will be applied.
    - `oo`: A pointer to a `struct options` which contains style options.
    - `name`: A string representing the name of the style to be applied.
    - `ft`: A pointer to a `struct format_tree` used for formatting, which can be NULL.
- **Control Flow**:
    - Check if the `ft` (format tree) is NULL; if so, create a new format tree `ft0` with [`format_create`](format.c.driver.md#format_create) and assign it to `ft`.
    - Convert the style name to a `struct style` using [`options_string_to_style`](options.c.driver.md#options_string_to_style); if it returns NULL, use the default style `style_default`.
    - If the style's foreground (`fg`), background (`bg`), or underscore (`us`) colors are not set to 8, update the corresponding fields in the `gc` (grid cell).
    - Apply the style's attributes to the grid cell by OR-ing the `attr` field.
    - If a new format tree `ft0` was created, free it using [`format_free`](format.c.driver.md#format_free).
- **Output**:
    - The function does not return a value; it modifies the `gc` (grid cell) in place.
- **Functions called**:
    - [`format_create`](format.c.driver.md#format_create)
    - [`options_string_to_style`](options.c.driver.md#options_string_to_style)
    - [`format_free`](format.c.driver.md#format_free)


---
### style_apply<!-- {{#callable:style_apply}} -->
The `style_apply` function initializes a grid cell with default values and then applies additional style settings from options.
- **Inputs**:
    - `gc`: A pointer to a `struct grid_cell` that will be initialized and styled.
    - `oo`: A pointer to a `struct options` containing style options to be applied.
    - `name`: A string representing the name of the style to be applied.
    - `ft`: A pointer to a `struct format_tree` used for formatting, can be NULL.
- **Control Flow**:
    - Copy the default grid cell values into the provided `gc` structure using `memcpy`.
    - Call [`style_add`](#style_add) to apply additional style settings from the options `oo` using the style name `name` and format tree `ft`.
- **Output**:
    - The function does not return a value; it modifies the `gc` structure in place to apply the default and additional styles.
- **Functions called**:
    - [`style_add`](#style_add)


---
### style_set<!-- {{#callable:style_set}} -->
The `style_set` function initializes a `style` structure with default values and then updates its `grid_cell` component with the values from a provided `grid_cell` structure.
- **Inputs**:
    - `sy`: A pointer to a `style` structure that will be initialized and updated.
    - `gc`: A pointer to a `grid_cell` structure whose values will be used to update the `grid_cell` component of the `style` structure.
- **Control Flow**:
    - Copy the default style values into the `style` structure pointed to by `sy` using `memcpy`.
    - Copy the values from the `grid_cell` structure pointed to by `gc` into the `gc` component of the `style` structure pointed to by `sy` using `memcpy`.
- **Output**:
    - The function does not return a value; it modifies the `style` structure pointed to by `sy` in place.


---
### style_copy<!-- {{#callable:style_copy}} -->
The `style_copy` function copies the contents of one `style` structure to another using `memcpy`.
- **Inputs**:
    - `dst`: A pointer to the destination `style` structure where the data will be copied to.
    - `src`: A pointer to the source `style` structure from which the data will be copied.
- **Control Flow**:
    - The function uses `memcpy` to copy the entire contents of the `src` structure to the `dst` structure.
    - The size of the data copied is determined by the size of the `dst` structure.
- **Output**:
    - The function does not return any value; it performs the copy operation directly on the provided `style` structures.


---
### style_set_scrollbar_style_from_option<!-- {{#callable:style_set_scrollbar_style_from_option}} -->
The function `style_set_scrollbar_style_from_option` sets the scrollbar style for a pane based on the options provided, defaulting to predefined values if no specific style is found.
- **Inputs**:
    - `sb_style`: A pointer to a `struct style` where the scrollbar style will be set.
    - `oo`: A pointer to a `struct options` from which the scrollbar style option will be retrieved.
- **Control Flow**:
    - Retrieve the scrollbar style from the options using [`options_string_to_style`](options.c.driver.md#options_string_to_style) with the key 'pane-scrollbars-style'.
    - If the retrieved style (`sy`) is NULL, set `sb_style` to the default grid cell style, and assign default width, padding, and character values for the scrollbar.
    - If a style is retrieved, copy it to `sb_style`.
    - Check if the width of `sb_style` is less than 1, and if so, set it to the default width.
    - Check if the padding of `sb_style` is less than 0, and if so, set it to the default padding.
    - Set the character data of `sb_style` to the default scrollbar character.
- **Output**:
    - The function does not return a value; it modifies the `sb_style` structure in place.
- **Functions called**:
    - [`options_string_to_style`](options.c.driver.md#options_string_to_style)
    - [`style_set`](#style_set)
    - [`utf8_set`](utf8.c.driver.md#utf8_set)
    - [`style_copy`](#style_copy)


