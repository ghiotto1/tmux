# Purpose
This C source code file is part of a software system that processes and manipulates Sixel images, a format used for encoding bitmap graphics. The file provides a collection of functions to parse, manipulate, and render Sixel images, which are typically used in terminal emulators to display graphics. The code defines several structures, such as `sixel_image`, `sixel_line`, and `sixel_chunk`, which are used to represent the image data, lines within the image, and chunks of image data, respectively. The primary functionality includes parsing Sixel data from a buffer, expanding image lines and pixels, setting and getting pixel values, handling color attributes, and compressing and printing the image data in Sixel format.

The file is not an executable but rather a library intended to be used by other parts of the software, likely within a terminal emulator or a similar application that requires Sixel image support. It does not define public APIs or external interfaces directly but provides internal functions that can be utilized by other components of the system. The code includes functions for scaling images, converting Sixel images to a screen representation, and freeing allocated resources, indicating its role in managing the lifecycle of Sixel images within the application. The presence of logging and error handling suggests that the code is designed to be robust and informative during execution, aiding in debugging and development.
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
    - `p2`: A parameter related to the image, possibly a mode or setting.
    - `dx`: The current x position for drawing.
    - `dy`: The current y position for drawing.
    - `dc`: The current color index for drawing.
    - `lines`: Pointer to an array of sixel_line structures representing the image lines.
- **Description**: The `sixel_image` structure is used to represent an image in the sixel graphics format, which is a bitmap graphics format used in some terminal emulators. It contains information about the image dimensions, pixel size, color palette, and the actual image data stored in lines. The structure supports operations such as setting and getting pixel values, expanding the image dimensions, and managing color information. It is designed to handle the constraints of terminal-based graphics, such as limited color registers and pixel dimensions.


---
### sixel_chunk
- **Type**: `struct`
- **Members**:
    - `next_x`: Stores the next x-coordinate for the chunk.
    - `next_y`: Stores the next y-coordinate for the chunk.
    - `count`: Tracks the number of times a pattern is repeated.
    - `pattern`: Holds the current pattern character for the chunk.
    - `next_pattern`: Holds the next pattern character for the chunk.
    - `len`: Indicates the allocated length of the data buffer.
    - `used`: Indicates the used length of the data buffer.
    - `data`: Pointer to a character array storing the chunk's data.
- **Description**: The `sixel_chunk` structure is used to represent a segment of a sixel image, which is a format for encoding bitmap graphics. It contains information about the position and pattern of the chunk, as well as a buffer to store the chunk's data. This structure is essential for managing the encoding and manipulation of sixel image data, allowing for efficient handling of image segments during processing.


# Functions

---
### sixel_parse_expand_lines<!-- {{#callable:sixel_parse_expand_lines}} -->
The `sixel_parse_expand_lines` function expands the number of lines in a `sixel_image` structure to accommodate a specified height, reallocating memory as necessary.
- **Inputs**:
    - `si`: A pointer to a `sixel_image` structure, which contains information about the image including its current dimensions and line data.
    - `y`: An unsigned integer representing the new height to which the image's lines should be expanded.
- **Control Flow**:
    - Check if the current height `y` is less than or equal to the existing height in `si->y`; if so, return 0 as no expansion is needed.
    - Check if `y` exceeds the defined `SIXEL_HEIGHT_LIMIT`; if so, return 1 to indicate an error due to exceeding the limit.
    - Reallocate the `lines` array in the `sixel_image` structure to accommodate the new height `y` using `xrecallocarray`, which adjusts the size of the memory block.
    - Update the `y` field in the `sixel_image` structure to reflect the new height.
    - Return 0 to indicate successful expansion.
- **Output**:
    - Returns 0 on successful expansion or if no expansion is needed, and 1 if the requested height exceeds the limit.


---
### sixel_parse_expand_line<!-- {{#callable:sixel_parse_expand_line}} -->
The `sixel_parse_expand_line` function expands a `sixel_line` to a specified width, reallocating memory as necessary, while ensuring the width does not exceed a predefined limit.
- **Inputs**:
    - `si`: A pointer to a `sixel_image` structure, which represents the image being processed.
    - `sl`: A pointer to a `sixel_line` structure, which represents the line within the image that needs to be expanded.
    - `x`: An unsigned integer representing the new width to which the line should be expanded.
- **Control Flow**:
    - Check if the current width of the line (`sl->x`) is greater than or equal to `x`; if so, return 0 as no expansion is needed.
    - Check if `x` exceeds the `SIXEL_WIDTH_LIMIT`; if so, return 1 to indicate an error due to exceeding the width limit.
    - If `x` is greater than the current width of the image (`si->x`), update `si->x` to `x`.
    - Reallocate memory for `sl->data` to accommodate the new width using `xrecallocarray`, expanding from `sl->x` to `si->x`.
    - Update `sl->x` to reflect the new width `si->x`.
    - Return 0 to indicate successful expansion.
- **Output**:
    - Returns 0 on successful expansion or 1 if the width exceeds the limit.


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
    - Return the pixel value at the specified x-coordinate from the `data` array of the `sixel_line`.
- **Output**:
    - Returns the pixel value at the specified (x, y) coordinate, or 0 if the coordinates are out of bounds.


---
### sixel_set_pixel<!-- {{#callable:sixel_set_pixel}} -->
The `sixel_set_pixel` function sets a pixel at a specified position in a sixel image to a given color value, expanding the image's dimensions if necessary.
- **Inputs**:
    - `si`: A pointer to a `sixel_image` structure representing the image where the pixel is to be set.
    - `x`: The x-coordinate (horizontal position) of the pixel to be set.
    - `y`: The y-coordinate (vertical position) of the pixel to be set.
    - `c`: The color value to set the pixel to.
- **Control Flow**:
    - The function first checks if the image needs to be expanded vertically to accommodate the y-coordinate by calling [`sixel_parse_expand_lines`](#sixel_parse_expand_lines) with `y + 1` as the argument.
    - If the vertical expansion fails (returns non-zero), the function returns 1 indicating an error.
    - If successful, it retrieves the `sixel_line` structure for the specified y-coordinate.
    - The function then checks if the line needs to be expanded horizontally to accommodate the x-coordinate by calling [`sixel_parse_expand_line`](#sixel_parse_expand_line) with `x + 1` as the argument.
    - If the horizontal expansion fails (returns non-zero), the function returns 1 indicating an error.
    - If successful, it sets the pixel at the specified x-coordinate in the line's data array to the color value `c`.
    - Finally, the function returns 0 indicating success.
- **Output**:
    - The function returns 0 on success, or 1 if it fails to expand the image to accommodate the specified coordinates.
- **Functions called**:
    - [`sixel_parse_expand_lines`](#sixel_parse_expand_lines)
    - [`sixel_parse_expand_line`](#sixel_parse_expand_line)


---
### sixel_parse_write<!-- {{#callable:sixel_parse_write}} -->
The `sixel_parse_write` function writes a sixel character to a sixel image by expanding lines and setting pixel data based on the character's bit pattern.
- **Inputs**:
    - `si`: A pointer to a `sixel_image` structure representing the image to which the sixel character will be written.
    - `ch`: An unsigned integer representing the sixel character to be written, where each bit corresponds to a pixel in a vertical column of six pixels.
- **Control Flow**:
    - The function first attempts to expand the lines of the sixel image to accommodate six more lines by calling [`sixel_parse_expand_lines`](#sixel_parse_expand_lines) with `si->dy + 6` as the argument.
    - If the line expansion fails, the function returns 1, indicating an error.
    - A pointer `sl` is set to the current line in the sixel image at `si->dy`.
    - A loop iterates over six iterations (i = 0 to 5), corresponding to the six pixels in the vertical column represented by the sixel character.
    - Within the loop, the function attempts to expand the current line to accommodate one more pixel by calling [`sixel_parse_expand_line`](#sixel_parse_expand_line) with `si->dx + 1` as the argument.
    - If the line expansion fails, the function returns 1, indicating an error.
    - If the bit at position `i` in `ch` is set, the function sets the pixel at the current position in the line to `si->dc`, the current drawing color.
    - The line pointer `sl` is incremented to point to the next line.
- **Output**:
    - The function returns 0 on success, indicating that the sixel character was successfully written to the image, or 1 on failure, indicating an error in expanding lines or setting pixel data.
- **Functions called**:
    - [`sixel_parse_expand_lines`](#sixel_parse_expand_lines)
    - [`sixel_parse_expand_line`](#sixel_parse_expand_line)


---
### sixel_parse_attributes<!-- {{#callable:sixel_parse_attributes}} -->
The `sixel_parse_attributes` function parses and validates the attributes of a sixel image from a given string, updating the image's dimensions and attributes accordingly.
- **Inputs**:
    - `si`: A pointer to a `sixel_image` structure that will be updated with parsed attributes.
    - `cp`: A pointer to the start of the string containing the sixel attributes to be parsed.
    - `end`: A pointer to the end of the string, marking the limit for parsing.
- **Control Flow**:
    - Initialize `last` to `cp` and iterate through the string until a non-numeric or non-semicolon character is found.
    - Use `strtoul` to parse the first number from `cp` to `endptr`, checking if parsing was successful and if the next character is a semicolon.
    - Parse the second number from `endptr + 1` to `endptr`, ensuring it ends with a semicolon.
    - Parse the third number (x) from `endptr + 1` to `endptr`, checking for semicolon termination and ensuring it does not exceed `SIXEL_WIDTH_LIMIT`.
    - Parse the fourth number (y) from `endptr + 1` to `endptr`, ensuring no extra characters remain and it does not exceed `SIXEL_HEIGHT_LIMIT`.
    - Update the `sixel_image` structure with the parsed x and y values, expand lines if necessary, and set the `set_ra`, `ra_x`, and `ra_y` attributes.
    - Return `last` if parsing is successful, or `NULL` if any validation fails.
- **Output**:
    - Returns a pointer to the last successfully parsed character in the string, or `NULL` if parsing fails due to format errors or dimension limits.
- **Functions called**:
    - [`sixel_parse_expand_lines`](#sixel_parse_expand_lines)


---
### sixel_parse_colour<!-- {{#callable:sixel_parse_colour}} -->
The `sixel_parse_colour` function parses a color definition from a string and updates the color information in a `sixel_image` structure.
- **Inputs**:
    - `si`: A pointer to a `sixel_image` structure where the parsed color information will be stored.
    - `cp`: A pointer to the start of the color definition string to be parsed.
    - `end`: A pointer to the end of the string, marking the limit for parsing.
- **Control Flow**:
    - Initialize `last` to `cp` and iterate through the string until a non-numeric or non-semicolon character is found.
    - Parse the color index `c` using `strtoul` and check if it exceeds `SIXEL_COLOUR_REGISTERS`; if so, log an error and return `NULL`.
    - Set `si->dc` to `c + 1` and check if the next character is a semicolon; if not, return `last`.
    - Parse the color type `type` and ensure it is followed by a semicolon; if not, log an error and return `NULL`.
    - Parse the red, green, and blue components (`r`, `g`, `b`) in sequence, each followed by a semicolon; if any are missing, log an error and return `NULL`.
    - Check if `type` is valid (either 1 or 2); if not, log an error and return `NULL`.
    - If the color index `c + 1` exceeds the current number of colors in `si`, reallocate the colors array to accommodate the new index.
    - Store the parsed color in `si->colours[c]` using a bitwise operation to combine `type`, `r`, `g`, and `b`.
    - Return `last`, the position in the string after the parsed color definition.
- **Output**:
    - Returns a pointer to the position in the string after the parsed color definition, or `NULL` if an error occurs during parsing.


---
### sixel_parse_repeat<!-- {{#callable:sixel_parse_repeat}} -->
The `sixel_parse_repeat` function parses a repeat command in a SIXEL image data stream, writing a specified character multiple times to the image structure.
- **Inputs**:
    - `si`: A pointer to a `sixel_image` structure where the parsed data will be written.
    - `cp`: A pointer to the current position in the input character stream, marking the start of the repeat command.
    - `end`: A pointer to the end of the input character stream, used to determine the bounds of parsing.
- **Control Flow**:
    - Initialize `last` to `cp` and set up a temporary buffer `tmp` to store numeric characters.
    - Iterate over the input stream starting from `last`, collecting numeric characters into `tmp` until a non-numeric character is encountered or the buffer is full.
    - If no numeric characters are collected or the end of the stream is reached, log an error and return `NULL`.
    - Convert the collected numeric string in `tmp` to an integer `n` using `strtonum`, ensuring it is within the valid range (1 to `SIXEL_WIDTH_LIMIT`).
    - If the conversion fails or the number is out of range, log an error and return `NULL`.
    - Calculate the character to be repeated by subtracting 0x3f from the current character pointed by `last`.
    - For `n` times, call [`sixel_parse_write`](#sixel_parse_write) to write the character to the `sixel_image` structure, incrementing the `dx` position each time.
    - If [`sixel_parse_write`](#sixel_parse_write) fails, log an error and return `NULL`.
    - Return the updated position `last` in the input stream.
- **Output**:
    - Returns a pointer to the position in the input stream after the repeat command, or `NULL` if an error occurs during parsing.
- **Functions called**:
    - [`sixel_parse_write`](#sixel_parse_write)


---
### sixel_parse<!-- {{#callable:sixel_parse}} -->
The `sixel_parse` function parses a buffer containing a Sixel image format and constructs a `sixel_image` structure representing the image.
- **Inputs**:
    - `buf`: A pointer to a character buffer containing the Sixel image data to be parsed.
    - `len`: The length of the buffer `buf`.
    - `p2`: An unsigned integer representing a parameter used in the Sixel image processing.
    - `xpixel`: An unsigned integer representing the horizontal pixel density of the image.
    - `ypixel`: An unsigned integer representing the vertical pixel density of the image.
- **Control Flow**:
    - Check if the buffer length is 0 or 1 or if the first character is not 'q'; if so, log a debug message and return NULL.
    - Allocate memory for a `sixel_image` structure and initialize its fields with the provided parameters.
    - Iterate over the buffer, processing each character according to its type (e.g., attribute, color, repeat, line break, or pixel data).
    - For each character, call the appropriate parsing function (e.g., [`sixel_parse_attributes`](#sixel_parse_attributes), [`sixel_parse_colour`](#sixel_parse_colour), [`sixel_parse_repeat`](#sixel_parse_repeat)) and handle errors by freeing the allocated memory and returning NULL.
    - If a character represents a pixel, call [`sixel_parse_write`](#sixel_parse_write) to write the pixel data and increment the x-coordinate.
    - After processing the buffer, check if the image dimensions are valid; if not, free the memory and return NULL.
    - Return the constructed `sixel_image` structure if parsing is successful.
- **Output**:
    - Returns a pointer to a `sixel_image` structure representing the parsed image, or NULL if parsing fails.
- **Functions called**:
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
    - Iterate over each line in the `sixel_image` structure using a loop that runs from 0 to `si->y`.
    - For each line, free the memory allocated for `data` in `sixel_line`.
    - After freeing all line data, free the memory allocated for the `lines` array itself.
    - Free the memory allocated for the `colours` array.
    - Finally, free the memory allocated for the `sixel_image` structure itself.
- **Output**:
    - The function does not return any value; it performs memory deallocation for the given `sixel_image` structure.


---
### sixel_log<!-- {{#callable:sixel_log}} -->
The `sixel_log` function logs detailed information about a sixel image, including its dimensions, colors, and pixel data for each line.
- **Inputs**:
    - `si`: A pointer to a `sixel_image` structure, which contains information about the sixel image such as dimensions, colors, and pixel data.
- **Control Flow**:
    - Call [`sixel_size_in_cells`](#sixel_size_in_cells) to calculate the image size in terminal cells and store the results in `cx` and `cy`.
    - Log the image dimensions and calculated cell size using `log_debug`.
    - Iterate over each color in the image and log its index and value.
    - For each line in the image, iterate over each pixel position.
    - For each pixel, determine its character representation based on its value and store it in the string `s`.
    - Log the constructed string `s` for each line using `log_debug`.
- **Output**:
    - The function does not return any value; it outputs log messages detailing the image's properties and pixel data.
- **Functions called**:
    - [`sixel_size_in_cells`](#sixel_size_in_cells)


---
### sixel_size_in_cells<!-- {{#callable:sixel_size_in_cells}} -->
The `sixel_size_in_cells` function calculates the number of terminal cells required to display a sixel image based on its pixel dimensions and the size of each cell in pixels.
- **Inputs**:
    - `si`: A pointer to a `sixel_image` structure containing the image's dimensions and pixel size.
    - `x`: A pointer to an unsigned integer where the function will store the calculated number of cells in the x-direction.
    - `y`: A pointer to an unsigned integer where the function will store the calculated number of cells in the y-direction.
- **Control Flow**:
    - Check if the width of the image (`si->x`) is evenly divisible by the width of a cell (`si->xpixel`).
    - If divisible, set `*x` to the quotient of the division; otherwise, set `*x` to the quotient plus one.
    - Check if the height of the image (`si->y`) is evenly divisible by the height of a cell (`si->ypixel`).
    - If divisible, set `*y` to the quotient of the division; otherwise, set `*y` to the quotient plus one.
- **Output**:
    - The function outputs the number of terminal cells required in the x and y directions to display the sixel image, stored in the variables pointed to by `x` and `y`.


---
### sixel_scale<!-- {{#callable:sixel_scale}} -->
The `sixel_scale` function scales a section of a sixel image to a specified size and returns a new sixel image with the scaled content.
- **Inputs**:
    - `si`: A pointer to the original sixel image structure to be scaled.
    - `xpixel`: The desired width in pixels for the new image; if zero, the original image's width is used.
    - `ypixel`: The desired height in pixels for the new image; if zero, the original image's height is used.
    - `ox`: The x-coordinate of the top-left corner of the section to be scaled, in image cells.
    - `oy`: The y-coordinate of the top-left corner of the section to be scaled, in image cells.
    - `sx`: The width of the section to be scaled, in image cells.
    - `sy`: The height of the section to be scaled, in image cells.
    - `colours`: A flag indicating whether to copy the color palette from the original image to the new image.
- **Control Flow**:
    - Calculate the size of the original image in cells using [`sixel_size_in_cells`](#sixel_size_in_cells) and store in `cx` and `cy`.
    - Check if the specified section (ox, oy, sx, sy) is within the bounds of the original image; adjust `sx` and `sy` if necessary.
    - Set `xpixel` and `ypixel` to the original image's pixel dimensions if they are zero.
    - Calculate the pixel coordinates of the section to be scaled (`pox`, `poy`, `psx`, `psy`).
    - Calculate the target size in pixels (`tsx`, `tsy`) for the new image.
    - Allocate memory for a new `sixel_image` structure and initialize its pixel dimensions and other properties.
    - Adjust the region attributes (`ra_x`, `ra_y`) of the new image based on the original image's attributes and the section's origin and size.
    - Iterate over each pixel in the target size (`tsx`, `tsy`), map it to the corresponding pixel in the original section, and set the pixel in the new image using [`sixel_set_pixel`](#sixel_set_pixel).
    - If the `colours` flag is set, copy the color palette from the original image to the new image.
    - Return the newly created and scaled sixel image.
- **Output**:
    - A pointer to the newly created and scaled sixel image structure, or NULL if the specified section is out of bounds.
- **Functions called**:
    - [`sixel_size_in_cells`](#sixel_size_in_cells)
    - [`sixel_set_pixel`](#sixel_set_pixel)
    - [`sixel_get_pixel`](#sixel_get_pixel)


---
### sixel_print_add<!-- {{#callable:sixel_print_add}} -->
The `sixel_print_add` function appends a string to a dynamically allocated buffer, expanding the buffer size if necessary.
- **Inputs**:
    - `buf`: A pointer to the buffer where the string will be appended.
    - `len`: A pointer to the current length of the buffer, which may be updated if the buffer is expanded.
    - `used`: A pointer to the current number of bytes used in the buffer, which will be updated after appending the string.
    - `s`: The string to be appended to the buffer.
    - `slen`: The length of the string `s` to be appended.
- **Control Flow**:
    - Check if the current used size plus the length of the string to be added exceeds the current buffer length.
    - If the buffer is too small, double the buffer size using `xrealloc` to allocate more memory.
    - Copy the string `s` of length `slen` into the buffer starting at the current used position.
    - Update the used size by adding `slen` to it.
- **Output**:
    - The function does not return a value, but it modifies the buffer and its associated metadata (length and used size) in place.


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
    - If `count` is 1, the character `ch` is added to the buffer once using [`sixel_print_add`](#sixel_print_add).
    - If `count` is 2, the character `ch` is added to the buffer twice using [`sixel_print_add`](#sixel_print_add).
    - If `count` is 3, the character `ch` is added to the buffer three times using [`sixel_print_add`](#sixel_print_add).
    - If `count` is greater than 3 and not zero, a formatted string `!<count><ch>` is created and added to the buffer using [`sixel_print_add`](#sixel_print_add).
- **Output**:
    - The function does not return a value; it modifies the buffer in place to include the repeated character.
- **Functions called**:
    - [`sixel_print_add`](#sixel_print_add)


---
### sixel_print_compress_colors<!-- {{#callable:sixel_print_compress_colors}} -->
The `sixel_print_compress_colors` function processes a sixel image to compress color data into chunks for efficient printing.
- **Inputs**:
    - `si`: A pointer to a `sixel_image` structure representing the image to be processed.
    - `chunks`: An array of `sixel_chunk` structures used to store compressed color data.
    - `y`: The starting y-coordinate in the image from which to begin processing.
    - `active`: An array to store indices of active color chunks.
    - `nactive`: A pointer to an unsigned integer that tracks the number of active color chunks.
- **Control Flow**:
    - Initialize an array `colors` to store color data for six lines starting from y-coordinate `y`.
    - Iterate over each x-coordinate in the image width `si->x`.
    - For each x-coordinate, iterate over six lines starting from `y` to populate `colors` with non-zero color data and update `next_pattern` in the corresponding chunk.
    - For each non-zero color in `colors`, update the corresponding chunk's `next_y` and `active` array if necessary.
    - Calculate the difference `dx` between the current x-coordinate and the chunk's `next_x`.
    - If the chunk's current pattern differs from `next_pattern` or `dx` is non-zero, call [`sixel_print_repeat`](#sixel_print_repeat) to print the current pattern and update the chunk's data.
    - Increment the chunk's `count` and reset `next_pattern` and `next_x` for the next iteration.
- **Output**:
    - The function does not return a value but modifies the `chunks`, `active`, and `nactive` arrays to reflect compressed color data for the specified y-coordinate range.
- **Functions called**:
    - [`sixel_print_repeat`](#sixel_print_repeat)


---
### sixel_print<!-- {{#callable:sixel_print}} -->
The `sixel_print` function generates a SIXEL encoded string from a given sixel image, optionally using a color map, and returns the encoded string along with its size.
- **Inputs**:
    - `si`: A pointer to a `sixel_image` structure representing the image to be encoded.
    - `map`: A pointer to a `sixel_image` structure representing a color map to be used for encoding, or NULL if no map is used.
    - `size`: A pointer to a `size_t` variable where the size of the generated SIXEL string will be stored, or NULL if the size is not needed.
- **Control Flow**:
    - Initialize color information from the map if provided, otherwise from the image itself.
    - Check if there are any colors to process; if not, return NULL.
    - Allocate an initial buffer for the SIXEL string and prepare the SIXEL header with the image's P2 parameter.
    - If the image has a set RA (aspect ratio), append the RA information to the buffer.
    - Allocate memory for chunks and active color arrays based on the number of colors.
    - Iterate over each color to prepare color definitions and initialize chunk data.
    - For each row of the image, compress the colors and update the active colors list.
    - For each active color, append the color index and its compressed data to the buffer, handling pattern repetition.
    - Ensure the buffer does not end with a '$' or '-' character, adjusting as necessary.
    - Append the SIXEL termination sequence to the buffer.
    - Store the final size of the buffer if the size pointer is provided.
    - Free allocated memory for chunks and active color arrays.
    - Return the generated SIXEL string.
- **Output**:
    - A dynamically allocated string containing the SIXEL encoded image, or NULL if no colors are present.
- **Functions called**:
    - [`sixel_print_add`](#sixel_print_add)
    - [`sixel_print_compress_colors`](#sixel_print_compress_colors)
    - [`sixel_print_repeat`](#sixel_print_repeat)


---
### sixel_to_screen<!-- {{#callable:sixel_to_screen}} -->
The `sixel_to_screen` function converts a sixel image into a screen representation by initializing a screen with the appropriate dimensions and filling it with grid cells based on the image size.
- **Inputs**:
    - `si`: A pointer to a `sixel_image` structure representing the sixel image to be converted into a screen.
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
    - A pointer to a `screen` structure that represents the sixel image as a screen with grid cells.
- **Functions called**:
    - [`sixel_size_in_cells`](#sixel_size_in_cells)


