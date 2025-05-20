# Purpose
This C source code file is part of the tmux terminal multiplexer project and provides functionality for converting between integer attribute flags and their corresponding string representations. The file defines two primary functions: [`attributes_tostring`](#attributes_tostring) and [`attributes_fromstring`](#attributes_fromstring). The [`attributes_tostring`](#attributes_tostring) function takes an integer representing a combination of text attributes (such as bold, underscore, or italics) and returns a comma-separated string that describes these attributes. Conversely, the [`attributes_fromstring`](#attributes_fromstring) function takes a string of attribute names and converts it back into an integer flag that represents the combined attributes. This functionality is crucial for managing and interpreting text attributes within the tmux environment, allowing for flexible and human-readable configuration and manipulation of text display properties.

The code is structured to handle a variety of text attributes, including standard ones like bold and underscore, as well as more specific ones like double-underscore and overline. It uses a static buffer to construct the string representation and a lookup table to map string names to their corresponding attribute flags. The file does not define a public API or external interface but rather provides internal utility functions that are likely used by other parts of the tmux codebase to facilitate the handling of text attributes. The inclusion of headers like `<sys/types.h>` and `<string.h>`, along with the custom header `"tmux.h"`, indicates that this file is intended to be compiled as part of the larger tmux project rather than being a standalone library.
# Imports and Dependencies

---
- `sys/types.h`
- `string.h`
- `tmux.h`


# Functions

---
### attributes_tostring<!-- {{#callable:attributes_tostring}} -->
The `attributes_tostring` function converts an integer representing attribute flags into a comma-separated string of attribute names.
- **Inputs**:
    - `attr`: An integer representing a combination of attribute flags, where each bit corresponds to a specific attribute.
- **Control Flow**:
    - Declare a static buffer `buf` of size 512 to store the resulting string.
    - Check if `attr` is zero; if so, return the string "none".
    - Use [`xsnprintf`](xmalloc.c.driver.md#xsnprintf) to format the attribute names into `buf`, appending a comma after each name if the corresponding bit in `attr` is set.
    - Check if the length of the formatted string `len` is greater than zero; if so, replace the last comma with a null terminator to properly end the string.
    - Return the buffer `buf` containing the formatted attribute names.
- **Output**:
    - A pointer to a static character buffer containing a comma-separated string of attribute names corresponding to the set bits in `attr`, or "none" if `attr` is zero.
- **Functions called**:
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)


---
### attributes_fromstring<!-- {{#callable:attributes_fromstring}} -->
The function `attributes_fromstring` parses a string of attribute names separated by delimiters and returns a bitmask representing the corresponding attributes.
- **Inputs**:
    - `str`: A constant character pointer to a string containing attribute names separated by delimiters (spaces, commas, or pipes).
- **Control Flow**:
    - Check if the input string is empty or starts/ends with a delimiter; if so, return -1 indicating an error.
    - Check if the string is 'default' or 'none', and return 0 if true, indicating no attributes.
    - Initialize an attribute bitmask to 0.
    - Iterate over the string, splitting it by delimiters to extract each attribute name.
    - For each extracted name, compare it against a predefined table of attribute names and their corresponding bitmask values.
    - If a match is found, update the attribute bitmask using a bitwise OR operation with the matched attribute's value.
    - If no match is found for an attribute name, return -1 indicating an error.
    - Continue parsing until the end of the string is reached.
    - Return the final attribute bitmask.
- **Output**:
    - Returns an integer representing a bitmask of attributes parsed from the input string, or -1 if an error occurs.


