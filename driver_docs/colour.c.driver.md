# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it provides functionality for handling and converting colors within the application. The code is focused on converting RGB color values to the xterm 256-color palette, managing color palettes, and converting between different color representations, such as RGB, 256-color, and 16-color formats. It includes functions to find the closest xterm color for a given RGB triplet, join and split RGB values, convert colors to strings, and parse colors from strings, including X11 color names. The file also defines functions to initialize, clear, and manage color palettes, which are used to customize the appearance of tmux panes.

The code is structured around several key functions that perform specific tasks related to color management. For example, [`colour_find_rgb`](#colour_find_rgb) maps an RGB triplet to the nearest xterm color, while [`colour_fromstring`](#colour_fromstring) parses a color from a string representation. The file also includes a large static table for converting 256-color values to RGB, as well as a comprehensive list of X11 color names and their corresponding RGB values. This file is not an executable but rather a library intended to be used within the tmux application, providing essential color handling capabilities that enhance the visual customization options available to users.
# Imports and Dependencies

---
- `sys/types.h`
- `ctype.h`
- `stdlib.h`
- `string.h`
- `math.h`
- `tmux.h`


# Functions

---
### colour_dist_sq<!-- {{#callable:colour_dist_sq}} -->
The `colour_dist_sq` function calculates the squared Euclidean distance between two RGB color values.
- **Inputs**:
    - `R`: The red component of the first color.
    - `G`: The green component of the first color.
    - `B`: The blue component of the first color.
    - `r`: The red component of the second color.
    - `g`: The green component of the second color.
    - `b`: The blue component of the second color.
- **Control Flow**:
    - Calculate the difference between the red components of the two colors and square it.
    - Calculate the difference between the green components of the two colors and square it.
    - Calculate the difference between the blue components of the two colors and square it.
    - Sum the squared differences of the red, green, and blue components.
- **Output**:
    - The function returns an integer representing the squared Euclidean distance between the two RGB colors.


---
### colour_to_6cube<!-- {{#callable:colour_to_6cube}} -->
The `colour_to_6cube` function maps an integer value to a corresponding index in a 6x6x6 color cube used in xterm's 256 color palette.
- **Inputs**:
    - `v`: An integer representing a color value to be mapped to the 6x6x6 color cube.
- **Control Flow**:
    - Check if the input value `v` is less than 48; if true, return 0.
    - Check if the input value `v` is less than 114; if true, return 1.
    - For values of `v` 114 or greater, calculate and return the result of `(v - 35) / 40`.
- **Output**:
    - An integer representing the index in the 6x6x6 color cube.


---
### colour_find_rgb<!-- {{#callable:colour_find_rgb}} -->
The `colour_find_rgb` function converts an RGB color to the closest xterm 256-color palette index, considering both a 6x6x6 color cube and a set of 24 greyscales.
- **Inputs**:
    - `r`: The red component of the RGB color, represented as an unsigned char.
    - `g`: The green component of the RGB color, represented as an unsigned char.
    - `b`: The blue component of the RGB color, represented as an unsigned char.
- **Control Flow**:
    - Convert each RGB component to a 6-level cube index using [`colour_to_6cube`](#colour_to_6cube) and map it to the nearest color in the 6x6x6 cube using `q2c` array.
    - Check if the mapped cube color matches the input RGB color exactly; if so, return the corresponding 256-color index immediately.
    - Calculate the average of the RGB components to find the closest grey index and its corresponding grey value.
    - Compute the squared distance between the input RGB and the mapped cube color using [`colour_dist_sq`](#colour_dist_sq).
    - Compare the distance of the input RGB to the closest grey and the mapped cube color, and choose the one with the smaller distance.
    - Return the 256-color index of the closest color, either from the grey scale or the 6x6x6 cube, with the `COLOUR_FLAG_256` flag.
- **Output**:
    - The function returns an integer representing the closest xterm 256-color palette index to the input RGB color, with the `COLOUR_FLAG_256` flag set.
- **Functions called**:
    - [`colour_to_6cube`](#colour_to_6cube)
    - [`colour_dist_sq`](#colour_dist_sq)


---
### colour_join_rgb<!-- {{#callable:colour_join_rgb}} -->
The `colour_join_rgb` function combines three 8-bit RGB color components into a single integer with a specific flag.
- **Inputs**:
    - `r`: The red component of the RGB color, represented as an unsigned char.
    - `g`: The green component of the RGB color, represented as an unsigned char.
    - `b`: The blue component of the RGB color, represented as an unsigned char.
- **Control Flow**:
    - The function takes three unsigned char inputs representing the red, green, and blue components of a color.
    - Each component is masked with 0xff to ensure it is within the 8-bit range.
    - The red component is shifted left by 16 bits, the green component by 8 bits, and the blue component remains unshifted.
    - The shifted values are combined using bitwise OR operations.
    - A predefined flag, COLOUR_FLAG_RGB, is also combined using a bitwise OR operation to indicate that the result is an RGB color.
- **Output**:
    - The function returns an integer representing the combined RGB color with a specific flag indicating it is an RGB color.


---
### colour_split_rgb<!-- {{#callable:colour_split_rgb}} -->
The `colour_split_rgb` function extracts the red, green, and blue components from a 24-bit integer color value.
- **Inputs**:
    - `c`: An integer representing a 24-bit RGB color value, where the red component is in the highest 8 bits, the green component is in the middle 8 bits, and the blue component is in the lowest 8 bits.
    - `r`: A pointer to an unsigned char where the red component of the color will be stored.
    - `g`: A pointer to an unsigned char where the green component of the color will be stored.
    - `b`: A pointer to an unsigned char where the blue component of the color will be stored.
- **Control Flow**:
    - The function shifts the integer `c` 16 bits to the right to isolate the red component, then masks it with `0xff` to store the result in the location pointed to by `r`.
    - The function shifts the integer `c` 8 bits to the right to isolate the green component, then masks it with `0xff` to store the result in the location pointed to by `g`.
    - The function masks the integer `c` with `0xff` to isolate the blue component and stores the result in the location pointed to by `b`.
- **Output**:
    - The function does not return a value; it outputs the red, green, and blue components through the pointers `r`, `g`, and `b`.


---
### colour_force_rgb<!-- {{#callable:colour_force_rgb}} -->
The `colour_force_rgb` function converts a given colour code to its RGB equivalent if it is not already in RGB format.
- **Inputs**:
    - `c`: An integer representing a colour code, which may be in RGB, 256-colour, or basic colour format.
- **Control Flow**:
    - Check if the colour code `c` has the RGB flag set; if so, return `c` as it is already in RGB format.
    - If the colour code `c` has the 256-colour flag set, convert it to RGB using `colour_256toRGB(c)` and return the result.
    - If the colour code `c` is a basic colour (0 to 7), convert it to RGB using `colour_256toRGB(c)` and return the result.
    - If the colour code `c` is a bright colour (90 to 97), convert it to RGB using `colour_256toRGB(8 + c - 90)` and return the result.
    - If none of the above conditions are met, return -1 indicating an invalid or unconvertible colour code.
- **Output**:
    - An integer representing the RGB equivalent of the input colour code, or -1 if the conversion is not possible.
- **Functions called**:
    - [`colour_256toRGB`](#colour_256toRGB)


---
### colour_tostring<!-- {{#callable:colour_tostring}} -->
The `colour_tostring` function converts an integer color code into a human-readable string representation.
- **Inputs**:
    - `c`: An integer representing a color code, which can be a special value, an RGB value, a 256-color value, or a basic color index.
- **Control Flow**:
    - Check if the input color code `c` is -1, and return "none" if true.
    - Check if the color code `c` has the `COLOUR_FLAG_RGB` flag set, indicating an RGB color, and convert it to a string in the format `#RRGGBB`.
    - Check if the color code `c` has the `COLOUR_FLAG_256` flag set, indicating a 256-color palette, and convert it to a string in the format `colourN`.
    - Use a switch statement to match the color code `c` to predefined basic color names (e.g., black, red, green, etc.) and return the corresponding string.
    - If none of the above conditions are met, return "invalid".
- **Output**:
    - A constant character pointer to a string representing the color name or code.
- **Functions called**:
    - [`colour_split_rgb`](#colour_split_rgb)
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)


---
### colour_totheme<!-- {{#callable:colour_totheme}} -->
The [`colour_totheme`](#colour_totheme) function determines the theme (light or dark) of a given color based on its RGB or 256-color representation.
- **Inputs**:
    - `c`: An integer representing a color, which can be in RGB format, 256-color format, or a basic color code.
- **Control Flow**:
    - Check if the color `c` is -1, returning `THEME_UNKNOWN` if true.
    - If `c` is in RGB format, extract the red, green, and blue components, calculate their sum (brightness), and return `THEME_LIGHT` if the brightness is greater than 382, otherwise return `THEME_DARK`.
    - If `c` is in 256-color format, convert it to RGB using [`colour_256toRGB`](#colour_256toRGB) and recursively call [`colour_totheme`](#colour_totheme).
    - Use a switch statement to handle specific color codes: return `THEME_DARK` for codes 0 and 90, and `THEME_LIGHT` for codes 7 and 97.
    - For other color codes, convert them to RGB using [`colour_256toRGB`](#colour_256toRGB) and recursively call [`colour_totheme`](#colour_totheme).
    - Return `THEME_UNKNOWN` if none of the conditions are met.
- **Output**:
    - The function returns an `enum client_theme` value, which can be `THEME_LIGHT`, `THEME_DARK`, or `THEME_UNKNOWN`.
- **Functions called**:
    - [`colour_totheme`](#colour_totheme)
    - [`colour_256toRGB`](#colour_256toRGB)


---
### colour_fromstring<!-- {{#callable:colour_fromstring}} -->
The `colour_fromstring` function converts a string representation of a color into an integer value representing that color in a specific format.
- **Inputs**:
    - `s`: A constant character pointer representing the string input that specifies the color to be converted.
- **Control Flow**:
    - Check if the string starts with '#' and is 7 characters long, indicating an RGB hex color code.
    - If the string is a valid hex color, parse the RGB values and return the combined RGB integer using [`colour_join_rgb`](#colour_join_rgb).
    - Check if the string starts with 'colour' or 'color', parse the number following it, and return it with the `COLOUR_FLAG_256` flag if valid.
    - Check for specific color names like 'default', 'terminal', and basic colors (e.g., 'black', 'red') and return their corresponding integer values.
    - If none of the above conditions are met, attempt to find the color by name using [`colour_byname`](#colour_byname).
- **Output**:
    - An integer representing the color, or -1 if the input string is invalid or unrecognized.
- **Functions called**:
    - [`colour_join_rgb`](#colour_join_rgb)
    - [`strtonum`](compat/strtonum.c.driver.md#strtonum)
    - [`colour_byname`](#colour_byname)


---
### colour_256toRGB<!-- {{#callable:colour_256toRGB}} -->
The function `colour_256toRGB` converts a 256-color code to its corresponding RGB color value and applies an RGB flag.
- **Inputs**:
    - `c`: An integer representing a 256-color code.
- **Control Flow**:
    - The function uses a static table of 256 predefined RGB values corresponding to 256-color codes.
    - The input color code `c` is masked with `0xff` to ensure it is within the 0-255 range.
    - The function retrieves the RGB value from the table using the masked color code as an index.
    - The retrieved RGB value is combined with the `COLOUR_FLAG_RGB` flag using a bitwise OR operation.
    - The function returns the combined RGB value with the flag.
- **Output**:
    - The function returns an integer representing the RGB color value with an RGB flag.


---
### colour_256to16<!-- {{#callable:colour_256to16}} -->
The function `colour_256to16` converts a 256-color code to a 16-color code using a predefined mapping table.
- **Inputs**:
    - `c`: An integer representing a 256-color code.
- **Control Flow**:
    - The function uses a static table of 256 entries, where each entry maps a 256-color code to a corresponding 16-color code.
    - The input color code `c` is masked with `0xff` to ensure it is within the range of 0 to 255.
    - The function returns the value from the table corresponding to the masked color code.
- **Output**:
    - An integer representing the corresponding 16-color code.


---
### colour_byname<!-- {{#callable:colour_byname}} -->
The `colour_byname` function retrieves the RGB color value associated with a given color name, supporting both exact matches and specific gray shades.
- **Inputs**:
    - `name`: A string representing the name of the color to be looked up.
- **Control Flow**:
    - The function first checks if the input name starts with 'grey' or 'gray'.
    - If the name is exactly 'grey' or 'gray', it returns the RGB value for a standard gray color.
    - If the name is followed by a number, it attempts to convert the number to a percentage of gray and returns the corresponding RGB value.
    - If the name does not start with 'grey' or 'gray', it iterates through a predefined list of color names and their RGB values.
    - If a match is found in the list, it returns the corresponding RGB value.
    - If no match is found, it returns -1 indicating an error or unknown color.
- **Output**:
    - The function returns an integer representing the RGB color value with a flag indicating it's an RGB color, or -1 if the color name is not found.
- **Functions called**:
    - [`strtonum`](compat/strtonum.c.driver.md#strtonum)
    - [`colour_join_rgb`](#colour_join_rgb)


---
### colour_parseX11<!-- {{#callable:colour_parseX11}} -->
The `colour_parseX11` function parses a string representing a color in various formats and returns the corresponding RGB color value.
- **Inputs**:
    - `p`: A constant character pointer representing the color string to be parsed.
- **Control Flow**:
    - Initialize variables for color components and set default color to -1.
    - Determine the length of the input string `p`.
    - Check if the string matches various RGB formats (e.g., 'rgb:', '#', or comma-separated) and parse the RGB values if matched.
    - If the string matches a 16-bit RGB format, shift the values to 8-bit and parse them.
    - Check if the string matches CMYK or CMY formats, validate the range, and convert to RGB if valid.
    - Trim leading and trailing spaces from the string if no format matched.
    - Duplicate the trimmed string and attempt to find a color by name using [`colour_byname`](#colour_byname).
    - Log the parsed color and return the color value.
- **Output**:
    - Returns an integer representing the RGB color value, or -1 if the color could not be parsed.
- **Functions called**:
    - [`colour_join_rgb`](#colour_join_rgb)
    - [`xstrndup`](xmalloc.c.driver.md#xstrndup)
    - [`colour_byname`](#colour_byname)
    - [`colour_tostring`](#colour_tostring)


---
### colour_palette_init<!-- {{#callable:colour_palette_init}} -->
The `colour_palette_init` function initializes a `colour_palette` structure by setting its foreground and background colors to default values and its palette pointers to NULL.
- **Inputs**:
    - `p`: A pointer to a `colour_palette` structure that will be initialized.
- **Control Flow**:
    - Set the foreground color (`fg`) of the palette to 8, which typically represents the default color.
    - Set the background color (`bg`) of the palette to 8, which typically represents the default color.
    - Set the `palette` pointer to NULL, indicating no custom palette is currently set.
    - Set the `default_palette` pointer to NULL, indicating no default palette is currently set.
- **Output**:
    - The function does not return a value; it modifies the `colour_palette` structure pointed to by `p`.


---
### colour_palette_clear<!-- {{#callable:colour_palette_clear}} -->
The `colour_palette_clear` function resets the foreground and background colors of a `colour_palette` structure to default values and frees the memory allocated for its palette.
- **Inputs**:
    - `p`: A pointer to a `colour_palette` structure that is to be cleared.
- **Control Flow**:
    - Check if the input pointer `p` is not NULL.
    - Set the foreground (`fg`) and background (`bg`) color indices of the palette to 8, which typically represents the default color.
    - Free the memory allocated for the `palette` array within the `colour_palette` structure.
    - Set the `palette` pointer to NULL to indicate that it no longer points to a valid memory location.
- **Output**:
    - The function does not return any value; it modifies the `colour_palette` structure pointed to by `p`.


---
### colour_palette_free<!-- {{#callable:colour_palette_free}} -->
The `colour_palette_free` function deallocates memory associated with a colour palette and its default palette within a `colour_palette` structure.
- **Inputs**:
    - `p`: A pointer to a `colour_palette` structure that contains the palette and default_palette to be freed.
- **Control Flow**:
    - Check if the input pointer `p` is not NULL.
    - If `p` is not NULL, free the memory allocated for `p->palette` and set it to NULL.
    - Free the memory allocated for `p->default_palette` and set it to NULL.
- **Output**:
    - The function does not return any value; it performs memory deallocation for the specified `colour_palette` structure.


---
### colour_palette_get<!-- {{#callable:colour_palette_get}} -->
The `colour_palette_get` function retrieves a color value from a specified color palette based on a given color index, with adjustments for special color flags and ranges.
- **Inputs**:
    - `p`: A pointer to a `struct colour_palette` which contains the color palette and default palette arrays.
    - `c`: An integer representing the color index to retrieve from the palette.
- **Control Flow**:
    - Check if the palette pointer `p` is NULL, return -1 if true.
    - Adjust the color index `c` if it is in the range 90 to 97 by mapping it to the range 8 to 15.
    - Remove the COLOUR_FLAG_256 flag from `c` if it is set.
    - Return -1 if `c` is greater than or equal to 8 after adjustments.
    - Check if the palette array in `p` is not NULL and contains a valid color at index `c`, return that color if true.
    - Check if the default palette array in `p` is not NULL and contains a valid color at index `c`, return that color if true.
    - Return -1 if no valid color is found in either palette.
- **Output**:
    - Returns the color value from the palette or default palette at the specified index `c`, or -1 if the index is invalid or no color is found.


---
### colour_palette_set<!-- {{#callable:colour_palette_set}} -->
The `colour_palette_set` function sets a specific color in a color palette at a given index, initializing the palette if necessary.
- **Inputs**:
    - `p`: A pointer to a `colour_palette` structure where the color will be set.
    - `n`: An integer representing the index in the palette where the color should be set, must be between 0 and 255.
    - `c`: An integer representing the color to set at the specified index, or -1 to indicate no color.
- **Control Flow**:
    - Check if the palette pointer `p` is NULL or if the index `n` is greater than 255; if so, return 0 indicating failure.
    - Check if the color `c` is -1 and the palette is NULL; if so, return 0 indicating failure.
    - If the color `c` is not -1 and the palette is NULL, allocate memory for the palette and initialize all entries to -1.
    - Set the color `c` at the index `n` in the palette.
    - Return 1 indicating success.
- **Output**:
    - Returns 1 if the color is successfully set in the palette, or 0 if the operation fails due to invalid input or conditions.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)


---
### colour_palette_from_option<!-- {{#callable:colour_palette_from_option}} -->
The function `colour_palette_from_option` initializes and populates a colour palette from a given options structure.
- **Inputs**:
    - `p`: A pointer to a `struct colour_palette` which will be initialized and populated with colour values.
    - `oo`: A pointer to a `struct options` from which the colour values are retrieved.
- **Control Flow**:
    - Check if the palette pointer `p` is NULL and return immediately if it is.
    - Retrieve the options entry for 'pane-colours' from the options structure `oo`.
    - If the options array is empty, free the existing default palette in `p` if it exists, and return.
    - If the default palette in `p` is NULL, allocate memory for it to hold 256 integers and initialize all values to -1.
    - Iterate over each item in the options array, retrieve its index and value, and if the index is less than 256, set the corresponding index in the default palette to the retrieved value.
    - Move to the next item in the options array and repeat until all items are processed.
- **Output**:
    - The function does not return a value; it modifies the `default_palette` field of the `struct colour_palette` pointed to by `p`.
- **Functions called**:
    - [`options_get`](options.c.driver.md#options_get)
    - [`options_array_first`](options.c.driver.md#options_array_first)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`options_array_item_index`](options.c.driver.md#options_array_item_index)
    - [`options_array_item_value`](options.c.driver.md#options_array_item_value)
    - [`options_array_next`](options.c.driver.md#options_array_next)


