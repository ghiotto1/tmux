# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it provides functionality for handling and converting colors within the application. The code includes functions to convert RGB values to the xterm 256-color palette, join and split RGB values, force colors to RGB, and convert colors to and from strings. It also includes functions to handle color palettes, allowing for the initialization, clearing, and setting of colors in a palette. The file defines several static and public functions, indicating that it is likely intended to be part of a larger library or module within tmux, rather than a standalone executable.

The code is focused on color manipulation, providing a range of utilities to convert between different color representations, such as RGB, xterm 256-color, and named colors from the X11 color names. It includes functions to determine the closest color in a palette, convert colors to themes (light or dark), and parse colors from strings, supporting various formats including hexadecimal and CMYK. The file also manages color palettes, allowing for the customization of colors used in tmux sessions. This functionality is crucial for tmux's ability to display colors accurately across different terminal environments, enhancing the user experience by allowing for customizable and consistent color schemes.
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
The `colour_find_rgb` function converts an RGB color triplet to the closest xterm 256-color palette index, considering both a 6x6x6 color cube and a set of 24 greyscales.
- **Inputs**:
    - `r`: The red component of the RGB color, represented as an unsigned char.
    - `g`: The green component of the RGB color, represented as an unsigned char.
    - `b`: The blue component of the RGB color, represented as an unsigned char.
- **Control Flow**:
    - Map each RGB component to a 6x6x6 color cube using the [`colour_to_6cube`](#colour_to_6cube) function and a predefined mapping array `q2c` to get the closest cube color values `cr`, `cg`, and `cb`.
    - Check if the mapped cube color matches the input RGB color exactly; if so, return the corresponding 256-color index immediately.
    - Calculate the average of the RGB components to determine the closest grey index and its corresponding grey value.
    - Compute the squared distance between the input RGB color and both the closest cube color and the closest grey color.
    - Compare these distances to decide whether the cube color or the grey color is closer, and set the index `idx` accordingly.
    - Return the calculated index combined with the `COLOUR_FLAG_256` flag.
- **Output**:
    - The function returns an integer representing the closest xterm 256-color palette index to the input RGB color, with the `COLOUR_FLAG_256` flag set.
- **Functions called**:
    - [`colour_to_6cube`](#colour_to_6cube)
    - [`colour_dist_sq`](#colour_dist_sq)


---
### colour_join_rgb<!-- {{#callable:colour_join_rgb}} -->
The `colour_join_rgb` function combines three 8-bit RGB color components into a single integer with a specific flag.
- **Inputs**:
    - `r`: The red component of the RGB color, represented as an unsigned 8-bit integer (u_char).
    - `g`: The green component of the RGB color, represented as an unsigned 8-bit integer (u_char).
    - `b`: The blue component of the RGB color, represented as an unsigned 8-bit integer (u_char).
- **Control Flow**:
    - The function takes three unsigned 8-bit integers representing the red, green, and blue components of a color.
    - Each component is masked with 0xff to ensure it is within the 8-bit range.
    - The red component is shifted left by 16 bits, the green component by 8 bits, and the blue component remains unshifted.
    - The shifted components are combined using bitwise OR operations to form a single integer.
    - A predefined flag, COLOUR_FLAG_RGB, is added to the result using a bitwise OR operation.
- **Output**:
    - The function returns an integer that represents the combined RGB color with an additional flag indicating it is an RGB color.


---
### colour_split_rgb<!-- {{#callable:colour_split_rgb}} -->
The `colour_split_rgb` function extracts the red, green, and blue components from a 24-bit integer color value.
- **Inputs**:
    - `c`: An integer representing a 24-bit RGB color value, where the red component is in the highest 8 bits, the green component is in the middle 8 bits, and the blue component is in the lowest 8 bits.
    - `r`: A pointer to an unsigned char where the red component of the color will be stored.
    - `g`: A pointer to an unsigned char where the green component of the color will be stored.
    - `b`: A pointer to an unsigned char where the blue component of the color will be stored.
- **Control Flow**:
    - The function shifts the integer `c` 16 bits to the right and masks it with `0xff` to extract the red component, storing it in the location pointed to by `r`.
    - The function shifts the integer `c` 8 bits to the right and masks it with `0xff` to extract the green component, storing it in the location pointed to by `g`.
    - The function masks the integer `c` with `0xff` to extract the blue component, storing it in the location pointed to by `b`.
- **Output**:
    - The function does not return a value; it outputs the red, green, and blue components through the pointers `r`, `g`, and `b`.


---
### colour_force_rgb<!-- {{#callable:colour_force_rgb}} -->
The `colour_force_rgb` function converts a given colour code to its RGB equivalent if it is not already in RGB format.
- **Inputs**:
    - `c`: An integer representing a colour code, which may be in RGB, 256-colour, or basic 8-colour format.
- **Control Flow**:
    - Check if the colour code `c` has the RGB flag set; if so, return `c` as it is already in RGB format.
    - Check if the colour code `c` has the 256-colour flag set; if so, convert it to RGB using `colour_256toRGB(c)` and return the result.
    - Check if the colour code `c` is within the range of basic 8 colours (0 to 7); if so, convert it to RGB using `colour_256toRGB(c)` and return the result.
    - Check if the colour code `c` is within the range of bright colours (90 to 97); if so, convert it to RGB using `colour_256toRGB(8 + c - 90)` and return the result.
    - If none of the above conditions are met, return -1 indicating an invalid or unconvertible colour code.
- **Output**:
    - Returns the RGB equivalent of the input colour code if conversion is possible, otherwise returns -1 for invalid or unconvertible codes.
- **Functions called**:
    - [`colour_256toRGB`](#colour_256toRGB)


---
### colour_tostring<!-- {{#callable:colour_tostring}} -->
The `colour_tostring` function converts an integer representing a color into a human-readable string format.
- **Inputs**:
    - `c`: An integer representing a color, which can be a special value (-1), an RGB color, a 256-color palette index, or a basic color index.
- **Control Flow**:
    - If the input `c` is -1, return the string "none".
    - If `c` has the `COLOUR_FLAG_RGB` flag, split it into RGB components, format it as a hexadecimal string, and return it.
    - If `c` has the `COLOUR_FLAG_256` flag, format it as "colour" followed by its index and return it.
    - Use a switch statement to match `c` against predefined color indices (0-9 and 90-97) and return the corresponding color name as a string.
    - If none of the above conditions are met, return "invalid".
- **Output**:
    - A string representing the color, such as a color name, a hexadecimal RGB value, or "none"/"invalid" for special cases.
- **Functions called**:
    - [`colour_split_rgb`](#colour_split_rgb)


---
### colour_totheme<!-- {{#callable:colour_totheme}} -->
The [`colour_totheme`](#colour_totheme) function determines the theme (light or dark) of a given color based on its RGB or 256-color representation.
- **Inputs**:
    - `c`: An integer representing a color, which can be in RGB format, 256-color format, or a basic color code.
- **Control Flow**:
    - Check if the color `c` is -1, returning `THEME_UNKNOWN` if true.
    - If `c` is in RGB format, extract the red, green, and blue components, calculate their combined brightness, and return `THEME_LIGHT` if the brightness is greater than 382, otherwise return `THEME_DARK`.
    - If `c` is in 256-color format, convert it to RGB using [`colour_256toRGB`](#colour_256toRGB) and recursively call [`colour_totheme`](#colour_totheme).
    - For basic color codes, return `THEME_DARK` for codes 0 and 90, and `THEME_LIGHT` for codes 7 and 97.
    - For other basic color codes, convert them to RGB using [`colour_256toRGB`](#colour_256toRGB) and recursively call [`colour_totheme`](#colour_totheme).
    - Return `THEME_UNKNOWN` if none of the conditions are met.
- **Output**:
    - The function returns an enum value of type `client_theme`, which can be `THEME_LIGHT`, `THEME_DARK`, or `THEME_UNKNOWN`.
- **Functions called**:
    - [`colour_totheme`](#colour_totheme)
    - [`colour_256toRGB`](#colour_256toRGB)


---
### colour_fromstring<!-- {{#callable:colour_fromstring}} -->
The `colour_fromstring` function converts a string representation of a color into an integer value representing that color in a specific format.
- **Inputs**:
    - `s`: A constant character pointer representing the string input that specifies the color.
- **Control Flow**:
    - Check if the string starts with '#' and has a length of 7, indicating an RGB hex color code.
    - If the string is a valid RGB hex code, parse the RGB values and return the combined RGB integer using [`colour_join_rgb`](#colour_join_rgb).
    - Check if the string starts with 'colour' or 'color', parse the number following it, and return it with the `COLOUR_FLAG_256` flag if valid.
    - Check for specific color names like 'default', 'terminal', and basic colors (e.g., 'black', 'red') and return their corresponding integer values.
    - If none of the above conditions are met, attempt to find the color by name using [`colour_byname`](#colour_byname).
- **Output**:
    - An integer representing the color, or -1 if the input string is invalid or unrecognized.
- **Functions called**:
    - [`colour_join_rgb`](#colour_join_rgb)
    - [`colour_byname`](#colour_byname)


---
### colour_256toRGB<!-- {{#callable:colour_256toRGB}} -->
The `colour_256toRGB` function converts a 256-color code to its corresponding RGB color value with an RGB flag.
- **Inputs**:
    - `c`: An integer representing a 256-color code.
- **Control Flow**:
    - The function uses a static table of 256 predefined RGB values corresponding to 256-color codes.
    - It retrieves the RGB value from the table using the input color code `c` masked with `0xff` to ensure it is within the 0-255 range.
    - The function returns the RGB value combined with the `COLOUR_FLAG_RGB` to indicate that the result is an RGB color.
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
    - The function returns the value from the table at the index corresponding to the masked color code.
- **Output**:
    - The function returns an integer representing the corresponding 16-color code.


---
### colour_byname<!-- {{#callable:colour_byname}} -->
The `colour_byname` function retrieves the RGB color value associated with a given X11 color name or a grey scale value if specified.
- **Inputs**:
    - `name`: A string representing the name of the color, which can be an X11 color name or a grey scale value prefixed with 'grey' or 'gray'.
- **Control Flow**:
    - Check if the input name starts with 'grey' or 'gray'.
    - If the name is exactly 'grey' or 'gray', return the RGB value for grey with a specific flag.
    - If the name is followed by a number, convert it to a grey scale value and return the corresponding RGB value.
    - Iterate over a predefined list of color names and their RGB values.
    - If a match is found for the input name, return the corresponding RGB value with a specific flag.
    - If no match is found, return -1 indicating an error or unknown color.
- **Output**:
    - The function returns an integer representing the RGB color value with a specific flag if the color name is found, or -1 if the name is not recognized.
- **Functions called**:
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
    - If none of the above formats match, trim leading and trailing spaces from the string, duplicate it, and attempt to find a color by name.
    - Log the parsed color and return the color value.
- **Output**:
    - Returns an integer representing the RGB color value, or -1 if the color could not be parsed.
- **Functions called**:
    - [`colour_join_rgb`](#colour_join_rgb)
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
    - The function does not return a value; it initializes the provided `colour_palette` structure in place.


---
### colour_palette_clear<!-- {{#callable:colour_palette_clear}} -->
The `colour_palette_clear` function resets a colour palette by setting its foreground and background to default values and freeing its palette memory.
- **Inputs**:
    - `p`: A pointer to a `struct colour_palette` which represents the colour palette to be cleared.
- **Control Flow**:
    - Check if the input pointer `p` is not NULL.
    - Set the foreground (`fg`) and background (`bg`) of the palette to 8, which typically represents a default or reset state.
    - Free the memory allocated for the `palette` array within the `colour_palette` structure.
    - Set the `palette` pointer to NULL to indicate that it no longer points to valid memory.
- **Output**:
    - The function does not return any value; it modifies the input `colour_palette` structure in place.


---
### colour_palette_free<!-- {{#callable:colour_palette_free}} -->
The `colour_palette_free` function deallocates memory associated with a `colour_palette` structure and sets its pointers to NULL.
- **Inputs**:
    - `p`: A pointer to a `colour_palette` structure that needs to be freed.
- **Control Flow**:
    - Check if the input pointer `p` is not NULL.
    - If `p` is not NULL, free the memory allocated for `p->palette` and set `p->palette` to NULL.
    - Free the memory allocated for `p->default_palette` and set `p->default_palette` to NULL.
- **Output**:
    - The function does not return any value; it performs memory deallocation and pointer nullification.


---
### colour_palette_get<!-- {{#callable:colour_palette_get}} -->
The `colour_palette_get` function retrieves a color value from a specified color palette based on a given color index, with adjustments for certain color flags and ranges.
- **Inputs**:
    - `p`: A pointer to a `struct colour_palette` which contains the color palette and default palette arrays.
    - `c`: An integer representing the color index to retrieve from the palette.
- **Control Flow**:
    - Check if the palette pointer `p` is NULL, return -1 if true.
    - Adjust the color index `c` if it is in the range 90 to 97 by mapping it to the range 8 to 15.
    - If the color index `c` has the `COLOUR_FLAG_256` flag, remove this flag from `c`.
    - Return -1 if the adjusted color index `c` is 8 or higher and not a 256-color flag.
    - Check if the `palette` array in `p` is not NULL and contains a valid color at index `c`, return that color if true.
    - Check if the `default_palette` array in `p` is not NULL and contains a valid color at index `c`, return that color if true.
    - Return -1 if no valid color is found in either palette.
- **Output**:
    - Returns the color value from the palette or default palette at the specified index `c`, or -1 if the color is not found or the input is invalid.


---
### colour_palette_set<!-- {{#callable:colour_palette_set}} -->
The `colour_palette_set` function sets a specific color in a color palette at a given index, initializing the palette if necessary.
- **Inputs**:
    - `p`: A pointer to a `struct colour_palette`, which represents the color palette to be modified.
    - `n`: An integer representing the index in the palette where the color should be set, must be between 0 and 255.
    - `c`: An integer representing the color to be set at the specified index, or -1 to indicate no color.
- **Control Flow**:
    - Check if the palette pointer `p` is NULL or if the index `n` is greater than 255; if so, return 0 indicating failure.
    - If the color `c` is -1 and the palette is NULL, return 0 indicating failure.
    - If the color `c` is not -1 and the palette is NULL, allocate memory for the palette and initialize all entries to -1.
    - Set the color at index `n` in the palette to `c`.
    - Return 1 indicating success.
- **Output**:
    - Returns 1 if the color is successfully set in the palette, or 0 if the operation fails due to invalid input or conditions.


---
### colour_palette_from_option<!-- {{#callable:colour_palette_from_option}} -->
The function `colour_palette_from_option` initializes and populates a colour palette from a specified options array.
- **Inputs**:
    - `p`: A pointer to a `struct colour_palette` which will be initialized and populated with colour values.
    - `oo`: A pointer to a `struct options` from which the colour values will be retrieved.
- **Control Flow**:
    - Check if the colour palette pointer `p` is NULL and return immediately if it is.
    - Retrieve the options entry for 'pane-colours' from the options `oo`.
    - If the options array is empty, free the existing default palette in `p` if it exists, and return.
    - Allocate memory for the default palette in `p` if it is not already allocated.
    - Initialize all entries in the default palette to -1.
    - Iterate over each item in the options array, retrieving the index and value for each item.
    - If the index is less than 256, set the corresponding entry in the default palette to the retrieved value.
    - Move to the next item in the options array and repeat until all items are processed.
- **Output**:
    - The function does not return a value; it modifies the `default_palette` field of the `struct colour_palette` pointed to by `p`.


