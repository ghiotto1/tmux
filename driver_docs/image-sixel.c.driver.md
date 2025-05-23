# Purpose
This C source code file is part of a software system that processes and manipulates Sixel images, which are a format for encoding bitmap graphics. The file provides a collection of functions to parse, manipulate, and render Sixel images. It includes structures such as `sixel_image`, `sixel_line`, and `sixel_chunk` to represent the image data, lines, and chunks of Sixel data, respectively. The code handles tasks like expanding image lines, setting and getting pixel values, parsing Sixel attributes and colors, and compressing and printing Sixel data. It also includes functions to scale images, convert them to a screen representation, and free allocated resources.

The file is not an executable but rather a library intended to be used by other parts of the software, likely as part of a terminal emulator or a similar application that needs to handle Sixel graphics. It defines several static functions for internal use and public functions like [`sixel_parse`](#sixel_parse), [`sixel_free`](#sixel_free), [`sixel_log`](#sixel_log), [`sixel_scale`](#sixel_scale), [`sixel_print`](#sixel_print), and [`sixel_to_screen`](#sixel_to_screen), which provide an API for working with Sixel images. The code ensures that images do not exceed predefined width and height limits and includes logging for debugging purposes. The presence of functions for scaling and converting images to a screen format suggests that this library is designed to integrate Sixel graphics into a terminal or graphical user interface.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Data Structures

---
### sixel_line
- **Type**: `struct`
- **Members**:
    - `x`: An unsigned integer representing the width of the line.
    - `data`: A pointer to an array of uint16_t values representing pixel data for the line.
- **Description**: The `sixel_line` structure is used to represent a single line of pixel data in a sixel image, where `x` denotes the width of the line and `data` is a dynamically allocated array that holds the pixel values for that line. This structure is part of a larger system for handling sixel images, which are a format for encoding bitmap graphics.


---
### sixel_image
- **Type**: `struct`
- **Members**:
    - `x`: The width of the image in pixels.
    - `y`: The height of the image in pixels.
    - `xpixel`: The width of each pixel in the image.
    - `ypixel`: The height of each pixel in the image.
    - `set_ra`: Flag indicating if the aspect ratio is set.
    - `ra_x`: The aspect ratio width.
    - `ra_y`: The aspect ratio height.
    - `colours`: Pointer to an array of colors used in the image.
    - `ncolours`: The number of colors in the image.
    - `p2`: A parameter related to the image's color depth or format.
    - `dx`: The current x position for drawing.
    - `dy`: The current y position for drawing.
    - `dc`: The current color index for drawing.
    - `lines`: Pointer to an array of sixel_line structures representing the image lines.
- **Description**: The `sixel_image` structure is used to represent an image in the sixel graphics format, which is a bitmap graphics format used in some terminal emulators. It contains information about the image's dimensions, pixel size, color palette, and the actual pixel data organized in lines. The structure supports operations such as setting and getting pixel values, expanding the image dimensions, and handling color attributes, making it suitable for rendering and manipulating sixel images.


---
### sixel_chunk
- **Type**: `struct`
- **Members**:
    - `next_x`: Stores the next x-coordinate for the chunk.
    - `next_y`: Stores the next y-coordinate for the chunk.
    - `count`: Holds the count of repeated patterns.
    - `pattern`: Represents the current pattern character.
    - `next_pattern`: Represents the next pattern character to be used.
    - `len`: Indicates the total length of the data buffer.
    - `used`: Tracks the amount of the data buffer currently used.
    - `data`: Pointer to a character array holding the chunk's data.
- **Description**: The `sixel_chunk` structure is used to manage a segment of a sixel image, which is a format for encoding bitmap graphics. It keeps track of the current and next positions in the image grid, the pattern being used, and the buffer that stores the image data. This structure is essential for handling the compression and decompression of sixel data, allowing for efficient storage and manipulation of image patterns.


# Functions

---
### sixel_parse_expand_lines<!-- {{#callable:sixel_parse_expand_lines}} -->
The `sixel_parse_expand_lines` function expands the number of lines in a `sixel_image` structure to accommodate a specified height, ensuring it does not exceed a predefined limit.
- **Inputs**:
    - `si`: A pointer to a `sixel_image` structure, which contains information about the image including its current dimensions and lines.
    - `y`: An unsigned integer representing the new desired height for the image in terms of lines.
- **Control Flow**:
    - Check if the desired height `y` is less than or equal to the current height `si->y`; if so, return 0 indicating no expansion is needed.
    - Check if the desired height `y` exceeds the `SIXEL_HEIGHT_LIMIT`; if so, return 1 indicating an error due to exceeding the limit.
    - Use [`xrecallocarray`](xmalloc.c.driver.md#xrecallocarray) to reallocate memory for `si->lines` to accommodate the new height `y`, expanding from the current height `si->y`.
    - Update `si->y` to the new height `y`.
    - Return 0 to indicate successful expansion.
- **Output**:
    - Returns 0 on successful expansion or if no expansion is needed, and 1 if the desired height exceeds the limit.
- **Functions called**:
    - [`xrecallocarray`](xmalloc.c.driver.md#xrecallocarray)


---
### sixel_parse_expand_line<!-- {{#callable:sixel_parse_expand_line}} -->
The `sixel_parse_expand_line` function expands a `sixel_line` to a specified width, reallocating memory if necessary, while ensuring the width does not exceed a predefined limit.
- **Inputs**:
    - `si`: A pointer to a `sixel_image` structure, which represents the image being processed.
    - `sl`: A pointer to a `sixel_line` structure, which represents the line within the image that needs to be expanded.
    - `x`: An unsigned integer representing the new width to which the line should be expanded.
- **Control Flow**:
    - Check if the current width of the line (`sl->x`) is greater than or equal to `x`; if so, return 0 as no expansion is needed.
    - Check if `x` exceeds the `SIXEL_WIDTH_LIMIT`; if so, return 1 to indicate an error due to exceeding the width limit.
    - If `x` is greater than the current width of the image (`si->x`), update `si->x` to `x`.
    - Reallocate memory for `sl->data` to accommodate the new width using [`xrecallocarray`](xmalloc.c.driver.md#xrecallocarray), expanding from `sl->x` to `si->x`.
    - Update `sl->x` to reflect the new width `si->x`.
    - Return 0 to indicate successful expansion.
- **Output**:
    - Returns 0 on successful expansion or 1 if the width exceeds the `SIXEL_WIDTH_LIMIT`.
- **Functions called**:
    - [`xrecallocarray`](xmalloc.c.driver.md#xrecallocarray)


---
### sixel_get_pixel<!-- {{#callable:sixel_get_pixel}} -->
The `sixel_get_pixel` function retrieves the pixel value at a specified (x, y) coordinate from a sixel image structure.
- **Inputs**:
    - `si`: A pointer to a `sixel_image` structure, which contains the image data and dimensions.
    - `x`: An unsigned integer representing the x-coordinate of the pixel to retrieve.
    - `y`: An unsigned integer representing the y-coordinate of the pixel to retrieve.
- **Control Flow**:
    - Check if the y-coordinate is greater than or equal to the height of the image (`si->y`); if so, return 0.
    - Retrieve the `sixel_line` structure corresponding to the y-coordinate from the `lines` array in the `sixel_image`.
    - Check if the x-coordinate is greater than or equal to the width of the line (`sl->x`); if so, return 0.
    - Return the pixel value from the `data` array at the specified x-coordinate.
- **Output**:
    - The function returns the pixel value at the specified (x, y) coordinate, or 0 if the coordinates are out of bounds.


---
### sixel_set_pixel<!-- {{#callable:sixel_set_pixel}} -->
The `sixel_set_pixel` function sets a pixel at a specified position in a sixel image to a given color value, expanding the image's dimensions if necessary.
- **Inputs**:
    - `si`: A pointer to a `sixel_image` structure representing the image where the pixel is to be set.
    - `x`: An unsigned integer representing the x-coordinate of the pixel to be set.
    - `y`: An unsigned integer representing the y-coordinate of the pixel to be set.
    - `c`: An unsigned integer representing the color value to set the pixel to.
- **Control Flow**:
    - The function first checks if the image needs to be expanded vertically to accommodate the y-coordinate by calling [`sixel_parse_expand_lines`](#sixel_parse_expand_lines) with `y + 1` as the argument.
    - If the vertical expansion fails (returns non-zero), the function returns 1 indicating an error.
    - If successful, it retrieves the line at the y-coordinate from the image's lines array.
    - The function then checks if the line needs to be expanded horizontally to accommodate the x-coordinate by calling [`sixel_parse_expand_line`](#sixel_parse_expand_line) with `x + 1` as the argument.
    - If the horizontal expansion fails (returns non-zero), the function returns 1 indicating an error.
    - If successful, it sets the pixel at the x-coordinate of the line to the color value `c`.
    - Finally, the function returns 0 indicating success.
- **Output**:
    - The function returns 0 on success, or 1 if it fails to expand the image to accommodate the specified pixel position.
- **Functions called**:
    - [`sixel_parse_expand_lines`](#sixel_parse_expand_lines)
    - [`sixel_parse_expand_line`](#sixel_parse_expand_line)


---
### sixel_parse_write<!-- {{#callable:sixel_parse_write}} -->
The `sixel_parse_write` function writes a sixel character to a sixel image by expanding lines and setting pixel data based on the character's bit pattern.
- **Inputs**:
    - `si`: A pointer to a `sixel_image` structure representing the image being modified.
    - `ch`: An unsigned integer representing the sixel character to be written, where each bit corresponds to a pixel in a vertical column.
- **Control Flow**:
    - The function first attempts to expand the lines of the sixel image to accommodate six more lines by calling [`sixel_parse_expand_lines`](#sixel_parse_expand_lines) with `si->dy + 6` as the argument.
    - If the line expansion fails, the function returns 1, indicating an error.
    - A pointer `sl` is set to the current line in the image at `si->dy`.
    - A loop iterates over six bits (0 to 5) of the input character `ch`.
    - For each bit, the function attempts to expand the current line to accommodate one more pixel by calling [`sixel_parse_expand_line`](#sixel_parse_expand_line) with `si->dx + 1` as the argument.
    - If the line expansion fails, the function returns 1, indicating an error.
    - If the current bit in `ch` is set, the function sets the pixel at the current position `si->dx` in the line `sl` to the current drawing color `si->dc`.
    - The line pointer `sl` is incremented to move to the next line.
    - After processing all six bits, the function returns 0, indicating success.
- **Output**:
    - The function returns 0 on success or 1 on failure, indicating whether the sixel character was successfully written to the image.
- **Functions called**:
    - [`sixel_parse_expand_lines`](#sixel_parse_expand_lines)
    - [`sixel_parse_expand_line`](#sixel_parse_expand_line)


---
### sixel_parse_attributes<!-- {{#callable:sixel_parse_attributes}} -->
The `sixel_parse_attributes` function parses and validates the attributes of a sixel image from a given string, updating the image's dimensions and attributes if valid.
- **Inputs**:
    - `si`: A pointer to a `sixel_image` structure that will be updated with parsed attributes.
    - `cp`: A pointer to the start of the string containing the sixel attributes to be parsed.
    - `end`: A pointer to the end of the string, marking the limit for parsing.
- **Control Flow**:
    - Initialize `last` to `cp` and iterate through the string until a non-numeric or non-semicolon character is found.
    - Use `strtoul` to parse the first number from `cp` to `endptr`, checking if parsing was successful and if the next character is a semicolon.
    - Parse the second number from `endptr + 1` to `endptr`, ensuring it is followed by a semicolon.
    - Parse the third number (x) from `endptr + 1` to `endptr`, checking for a semicolon and ensuring `x` does not exceed `SIXEL_WIDTH_LIMIT`.
    - Parse the fourth number (y) from `endptr + 1` to `endptr`, ensuring no extra characters remain and `y` does not exceed `SIXEL_HEIGHT_LIMIT`.
    - Update the `sixel_image` structure with the parsed `x` and `y` values, expand lines if necessary, and set the `set_ra`, `ra_x`, and `ra_y` attributes.
    - Return `last` if parsing is successful, or `NULL` if any validation fails.
- **Output**:
    - Returns a pointer to the last successfully parsed character if parsing is successful, or `NULL` if any validation fails.
- **Functions called**:
    - [`sixel_parse_expand_lines`](#sixel_parse_expand_lines)


---
### sixel_parse_colour<!-- {{#callable:sixel_parse_colour}} -->
The `sixel_parse_colour` function parses a color definition from a string and updates the `sixel_image` structure with the parsed color information.
- **Inputs**:
    - `si`: A pointer to a `sixel_image` structure where the parsed color information will be stored.
    - `cp`: A pointer to the start of the color definition string to be parsed.
    - `end`: A pointer to the end of the string, marking the limit for parsing.
- **Control Flow**:
    - Initialize `last` to `cp` and iterate through the string until a non-numeric or non-semicolon character is found.
    - Parse the color index `c` using `strtoul` and check if it exceeds `SIXEL_COLOUR_REGISTERS`; if so, log an error and return `NULL`.
    - Set `si->dc` to `c + 1` and check if the parsing reached the end or if the next character is not a semicolon; if so, return `last`.
    - Parse the color type `type` and check for parsing errors or missing semicolons, logging errors and returning `NULL` if any issues are found.
    - Parse the red, green, and blue components (`r`, `g`, `b`) in sequence, checking for parsing errors or missing semicolons after each, logging errors and returning `NULL` if any issues are found.
    - Validate the `type` to ensure it is either 1 or 2, logging an error and returning `NULL` if invalid.
    - If the color index `c + 1` exceeds the current number of colors in `si`, reallocate the `colours` array to accommodate the new color index.
    - Store the parsed color in the `colours` array at index `c` using a bitwise operation to combine `type`, `r`, `g`, and `b`.
    - Return `last`, the position in the string after the parsed color definition.
- **Output**:
    - Returns a pointer to the position in the string after the parsed color definition, or `NULL` if an error occurs during parsing.
- **Functions called**:
    - [`xrecallocarray`](xmalloc.c.driver.md#xrecallocarray)


---
### sixel_parse_repeat<!-- {{#callable:sixel_parse_repeat}} -->
The `sixel_parse_repeat` function parses a repeat command in a SIXEL image data stream, writing a specified character multiple times to the image structure.
- **Inputs**:
    - `si`: A pointer to a `sixel_image` structure where the parsed data will be stored.
    - `cp`: A pointer to the current position in the input character stream, marking the start of the repeat command.
    - `end`: A pointer to the end of the input character stream, marking the limit for parsing.
- **Control Flow**:
    - Initialize `last` to `cp` and `n` to 0, and declare a temporary buffer `tmp` to store numeric characters.
    - Iterate over the input stream starting from `last` to extract numeric characters, storing them in `tmp` until a non-numeric character is encountered or the buffer is full.
    - If no numeric characters are found or the end of the stream is reached, log an error and return `NULL`.
    - Convert the numeric string in `tmp` to an integer `n` using [`strtonum`](compat/strtonum.c.driver.md#strtonum), ensuring it is within the valid range; log an error and return `NULL` if conversion fails.
    - Extract the character to be repeated from the stream, adjusting its value by subtracting 0x3f.
    - Loop `n` times, writing the character to the `sixel_image` using [`sixel_parse_write`](#sixel_parse_write); increment `si->dx` after each write.
    - If writing fails due to width limits, log an error and return `NULL`.
    - Return the updated position `last` in the input stream.
- **Output**:
    - Returns a pointer to the position in the input stream immediately after the parsed repeat command, or `NULL` if an error occurs.
- **Functions called**:
    - [`strtonum`](compat/strtonum.c.driver.md#strtonum)
    - [`sixel_parse_write`](#sixel_parse_write)


---
### sixel_parse<!-- {{#callable:sixel_parse}} -->
The `sixel_parse` function parses a buffer containing a sixel image and constructs a `sixel_image` structure representing the image.
- **Inputs**:
    - `buf`: A pointer to a character buffer containing the sixel image data to be parsed.
    - `len`: The length of the buffer `buf`.
    - `p2`: An unsigned integer representing a parameter for the sixel image.
    - `xpixel`: An unsigned integer representing the horizontal pixel density of the image.
    - `ypixel`: An unsigned integer representing the vertical pixel density of the image.
- **Control Flow**:
    - Check if the buffer length is 0 or 1 or if the first character is not 'q'; if so, log a debug message and return NULL.
    - Allocate memory for a `sixel_image` structure and initialize its fields with the provided parameters.
    - Iterate over the buffer, processing each character according to its type: attributes ('"'), color ('#'), repeat ('!'), line end ('-'), or line break ('$').
    - For each character type, call the corresponding parsing function ([`sixel_parse_attributes`](#sixel_parse_attributes), [`sixel_parse_colour`](#sixel_parse_colour), [`sixel_parse_repeat`](#sixel_parse_repeat)) and handle errors by freeing the allocated memory and returning NULL.
    - For default characters, check if they are within the valid range and write them to the image using [`sixel_parse_write`](#sixel_parse_write); increment the x-coordinate after writing.
    - After processing the buffer, check if the image dimensions are valid; if not, free the memory and return NULL.
    - Return the constructed `sixel_image` structure if parsing is successful.
- **Output**:
    - Returns a pointer to a `sixel_image` structure representing the parsed image, or NULL if parsing fails.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`sixel_parse_attributes`](#sixel_parse_attributes)
    - [`sixel_parse_colour`](#sixel_parse_colour)
    - [`sixel_parse_repeat`](#sixel_parse_repeat)
    - [`sixel_parse_write`](#sixel_parse_write)


---
### sixel_free<!-- {{#callable:sixel_free}} -->
The `sixel_free` function deallocates memory associated with a `sixel_image` structure, including its lines, colours, and the structure itself.
- **Inputs**:
    - `si`: A pointer to a `sixel_image` structure that needs to be freed.
- **Control Flow**:
    - Iterates over each line in the `sixel_image` structure, freeing the `data` associated with each line.
    - Frees the `lines` array of the `sixel_image` structure.
    - Frees the `colours` array of the `sixel_image` structure.
    - Frees the `sixel_image` structure itself.
- **Output**:
    - This function does not return any value; it performs memory deallocation.


---
### sixel_log<!-- {{#callable:sixel_log}} -->
The `sixel_log` function logs detailed information about a sixel image, including its dimensions, colors, and pixel data for each line.
- **Inputs**:
    - `si`: A pointer to a `sixel_image` structure, which contains the image data to be logged.
- **Control Flow**:
    - Call [`sixel_size_in_cells`](#sixel_size_in_cells) to calculate the image size in terminal cells and store the results in `cx` and `cy`.
    - Log the image dimensions and cell size using `log_debug`.
    - Iterate over each color in the image and log its index and value.
    - For each line in the image, iterate over each pixel position.
    - For each pixel, determine if it is out of bounds, has a non-zero value, or is zero, and set the corresponding character in the string `s`.
    - Terminate the string `s` with a null character and log the line number and pixel data using `log_debug`.
- **Output**:
    - The function does not return any value; it outputs log messages to the debug log.
- **Functions called**:
    - [`sixel_size_in_cells`](#sixel_size_in_cells)


---
### sixel_size_in_cells<!-- {{#callable:sixel_size_in_cells}} -->
The `sixel_size_in_cells` function calculates the number of terminal cells required to display a sixel image based on its pixel dimensions and the size of each cell in pixels.
- **Inputs**:
    - `si`: A pointer to a `sixel_image` structure that contains the dimensions of the sixel image and the size of each pixel.
    - `x`: A pointer to an unsigned integer where the function will store the calculated number of cells required in the horizontal direction.
    - `y`: A pointer to an unsigned integer where the function will store the calculated number of cells required in the vertical direction.
- **Control Flow**:
    - Check if the width of the image (`si->x`) is evenly divisible by the width of a cell in pixels (`si->xpixel`).
    - If divisible, set `*x` to the quotient of the division; otherwise, set `*x` to the quotient plus one.
    - Check if the height of the image (`si->y`) is evenly divisible by the height of a cell in pixels (`si->ypixel`).
    - If divisible, set `*y` to the quotient of the division; otherwise, set `*y` to the quotient plus one.
- **Output**:
    - The function outputs the number of terminal cells required to display the sixel image in both horizontal and vertical directions, stored in the variables pointed to by `x` and `y`.


---
### sixel_scale<!-- {{#callable:sixel_scale}} -->
The `sixel_scale` function scales a section of a sixel image to a specified size and returns a new sixel image structure.
- **Inputs**:
    - `si`: A pointer to the original sixel image structure to be scaled.
    - `xpixel`: The desired width in pixels for the new image; if zero, the original image's width is used.
    - `ypixel`: The desired height in pixels for the new image; if zero, the original image's height is used.
    - `ox`: The x-coordinate of the origin point in the original image from where scaling should start.
    - `oy`: The y-coordinate of the origin point in the original image from where scaling should start.
    - `sx`: The width in cells of the section to be scaled from the original image.
    - `sy`: The height in cells of the section to be scaled from the original image.
    - `colours`: A flag indicating whether to copy the color information from the original image to the new image.
- **Control Flow**:
    - Calculate the size of the original image in cells using [`sixel_size_in_cells`](#sixel_size_in_cells) and store in `cx` and `cy`.
    - Check if the origin point `ox, oy` is within the bounds of the original image; return NULL if out of bounds.
    - Adjust `sx` and `sy` to ensure the section to be scaled does not exceed the original image's dimensions.
    - Set `xpixel` and `ypixel` to the original image's dimensions if they are zero.
    - Calculate pixel-based origin and size (`pox`, `poy`, `psx`, `psy`) for the section to be scaled.
    - Calculate the target size in pixels (`tsx`, `tsy`) for the new image, ensuring `tsy` is a multiple of 6.
    - Allocate memory for a new `sixel_image` structure and initialize its properties.
    - Adjust the region of interest (`ra_x`, `ra_y`) in the new image based on the original image's region and the scaling factors.
    - Iterate over each pixel in the target size, map it to the corresponding pixel in the original image, and set the pixel in the new image using [`sixel_set_pixel`](#sixel_set_pixel).
    - If `colours` is true, allocate memory for the color array in the new image and copy the colors from the original image.
    - Return the newly created and scaled sixel image.
- **Output**:
    - A pointer to the newly created and scaled sixel image structure, or NULL if the scaling parameters are invalid.
- **Functions called**:
    - [`sixel_size_in_cells`](#sixel_size_in_cells)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`sixel_set_pixel`](#sixel_set_pixel)
    - [`sixel_get_pixel`](#sixel_get_pixel)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)


---
### sixel_print_add<!-- {{#callable:sixel_print_add}} -->
The `sixel_print_add` function appends a string to a dynamically allocated buffer, expanding the buffer size if necessary.
- **Inputs**:
    - `buf`: A pointer to the buffer where the string will be appended.
    - `len`: A pointer to the size of the buffer, which may be increased if the buffer is expanded.
    - `used`: A pointer to the current number of bytes used in the buffer.
    - `s`: The string to be appended to the buffer.
    - `slen`: The length of the string `s` to be appended.
- **Control Flow**:
    - Check if the current used size plus the length of the string to be added exceeds the current buffer size.
    - If the buffer is too small, double the buffer size and reallocate memory for the buffer using [`xrealloc`](xmalloc.c.driver.md#xrealloc).
    - Copy the string `s` into the buffer at the position indicated by `used`.
    - Update the `used` size to reflect the addition of the new string.
- **Output**:
    - The function does not return a value; it modifies the buffer and its size in place.
- **Functions called**:
    - [`xrealloc`](xmalloc.c.driver.md#xrealloc)


---
### sixel_print_repeat<!-- {{#callable:sixel_print_repeat}} -->
The `sixel_print_repeat` function appends a character to a buffer a specified number of times, using a special format for counts greater than three.
- **Inputs**:
    - `buf`: A pointer to a character buffer where the repeated character will be added.
    - `len`: A pointer to a size_t variable representing the current allocated length of the buffer.
    - `used`: A pointer to a size_t variable representing the current used length of the buffer.
    - `count`: An unsigned integer specifying how many times the character should be repeated.
    - `ch`: The character to be repeated and added to the buffer.
- **Control Flow**:
    - If `count` is 1, the character `ch` is added once to the buffer using [`sixel_print_add`](#sixel_print_add).
    - If `count` is 2, the character `ch` is added twice to the buffer using [`sixel_print_add`](#sixel_print_add).
    - If `count` is 3, the character `ch` is added three times to the buffer using [`sixel_print_add`](#sixel_print_add).
    - If `count` is greater than 3 and not zero, a formatted string `!<count><ch>` is created and added to the buffer using [`sixel_print_add`](#sixel_print_add).
- **Output**:
    - The function does not return a value; it modifies the buffer pointed to by `buf` to include the repeated character.
- **Functions called**:
    - [`sixel_print_add`](#sixel_print_add)
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)


---
### sixel_print_compress_colors<!-- {{#callable:sixel_print_compress_colors}} -->
The `sixel_print_compress_colors` function processes a sixel image to compress color data into chunks for efficient printing.
- **Inputs**:
    - `si`: A pointer to a `sixel_image` structure representing the image to be processed.
    - `chunks`: An array of `sixel_chunk` structures where compressed color data will be stored.
    - `y`: The starting y-coordinate in the image from which to begin processing.
    - `active`: An array to store indices of active color chunks that have been modified.
    - `nactive`: A pointer to an unsigned integer that tracks the number of active color chunks.
- **Control Flow**:
    - Initialize a loop over the x-coordinate of the image from 0 to `si->x`.
    - For each x-coordinate, initialize a loop over six lines starting from the given y-coordinate.
    - For each line, check if the current pixel is non-zero and within bounds, then update the `colors` array and the `next_pattern` of the corresponding chunk.
    - After processing the six lines, iterate over the `colors` array to update the `chunks` array for non-zero colors.
    - For each non-zero color, check if the chunk's `next_x` is not equal to the current x-coordinate plus one, and update `next_y` and `active` array if necessary.
    - Calculate the difference `dx` between the current x-coordinate and the chunk's `next_x`.
    - If the chunk's current pattern differs from `next_pattern` or `dx` is non-zero, call [`sixel_print_repeat`](#sixel_print_repeat) to update the chunk's data and reset the pattern and count.
    - Increment the chunk's count and update `next_x` to the current x-coordinate plus one.
- **Output**:
    - The function does not return a value but modifies the `chunks` array to store compressed color data and updates the `active` array with indices of modified chunks.
- **Functions called**:
    - [`sixel_print_repeat`](#sixel_print_repeat)


---
### sixel_print<!-- {{#callable:sixel_print}} -->
The `sixel_print` function generates a SIXEL encoded string from a given sixel image structure, optionally using a color map, and returns the encoded string along with its size.
- **Inputs**:
    - `si`: A pointer to a `sixel_image` structure representing the image to be encoded.
    - `map`: A pointer to a `sixel_image` structure representing a color map to be used for encoding, or NULL if the image's own colors should be used.
    - `size`: A pointer to a `size_t` variable where the size of the generated SIXEL string will be stored, or NULL if the size is not needed.
- **Control Flow**:
    - Initialize variables and determine the color palette to use based on the presence of a color map.
    - Allocate memory for the output buffer and initialize it with the SIXEL header sequence.
    - If the image has a defined aspect ratio, append the aspect ratio settings to the buffer.
    - Allocate memory for chunks and active color arrays based on the number of colors.
    - Iterate over the image in vertical slices of 6 pixels, compressing and encoding each slice's colors into the buffer.
    - For each active color in the slice, append the color index and encoded data to the buffer, handling repeated patterns.
    - Ensure the buffer does not end with a redundant character and append the SIXEL termination sequence.
    - Store the final size of the buffer if the size pointer is provided.
    - Free allocated memory for chunks and active color arrays.
    - Return the generated SIXEL encoded string.
- **Output**:
    - Returns a pointer to a dynamically allocated string containing the SIXEL encoded image, or NULL if there are no colors to encode.
- **Functions called**:
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)
    - [`sixel_print_add`](#sixel_print_add)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`sixel_print_compress_colors`](#sixel_print_compress_colors)
    - [`sixel_print_repeat`](#sixel_print_repeat)


---
### sixel_to_screen<!-- {{#callable:sixel_to_screen}} -->
The `sixel_to_screen` function converts a sixel image into a screen representation by initializing a screen with the appropriate dimensions and filling it with grid cells based on the image size.
- **Inputs**:
    - `si`: A pointer to a `sixel_image` structure, which contains the sixel image data to be converted into a screen.
- **Control Flow**:
    - Call [`sixel_size_in_cells`](#sixel_size_in_cells) to determine the screen dimensions (`sx` and `sy`) based on the sixel image size.
    - Allocate memory for a new `screen` structure and initialize it with the calculated dimensions.
    - Copy the default grid cell attributes and modify them to include charset and dim attributes, setting the character data to '~'.
    - Start a screen write context for the new screen.
    - If the screen dimensions are 1x1, fill the entire grid with the modified grid cell.
    - Otherwise, draw a box around the screen and fill the inner area with the modified grid cell.
    - Stop the screen write context.
    - Return the initialized and filled screen.
- **Output**:
    - Returns a pointer to a `screen` structure that represents the sixel image as a grid of cells.
- **Functions called**:
    - [`sixel_size_in_cells`](#sixel_size_in_cells)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`screen_init`](screen.c.driver.md#screen_init)
    - [`utf8_set`](utf8.c.driver.md#utf8_set)
    - [`screen_write_start`](screen-write.c.driver.md#screen_write_start)
    - [`grid_view_set_cell`](grid-view.c.driver.md#grid_view_set_cell)
    - [`screen_write_box`](screen-write.c.driver.md#screen_write_box)
    - [`screen_write_stop`](screen-write.c.driver.md#screen_write_stop)


