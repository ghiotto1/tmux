# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it provides functionality for converting between attribute representations and their string equivalents. The file defines two primary functions: [`attributes_tostring`](#attributes_tostring) and [`attributes_fromstring`](#attributes_fromstring). The [`attributes_tostring`](#attributes_tostring) function takes an integer representing a set of attributes and returns a string that describes these attributes in a human-readable format. Conversely, the [`attributes_fromstring`](#attributes_fromstring) function takes a string of attribute names and converts it back into the corresponding integer representation. This conversion is essential for managing and interpreting text attributes within the tmux environment, such as text styling options like bold, underscore, and italics.

The code is focused on handling text attributes, which are represented as bitwise flags. It includes a static buffer to store the string representation of attributes and uses a lookup table to map attribute names to their corresponding bitwise values. The file is not a standalone executable but rather a component of the larger tmux codebase, likely intended to be included and used by other parts of the tmux system. It does not define public APIs or external interfaces but provides internal utility functions to facilitate attribute management within tmux.
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
    - Check if the input attribute integer `attr` is zero; if so, return the string "none".
    - Use `xsnprintf` to format a string into `buf` by checking each attribute flag in `attr` and appending the corresponding attribute name followed by a comma if the flag is set.
    - If the formatted string length `len` is greater than zero, replace the last comma with a null terminator to properly end the string.
    - Return the formatted string `buf`.
- **Output**:
    - A static character buffer containing a comma-separated list of attribute names corresponding to the set bits in the input integer, or "none" if no attributes are set.


---
### attributes_fromstring<!-- {{#callable:attributes_fromstring}} -->
The `attributes_fromstring` function parses a string of attribute names separated by delimiters and returns a bitmask representing the corresponding attributes.
- **Inputs**:
    - `str`: A constant character pointer to a string containing attribute names separated by delimiters (spaces, commas, or pipes).
- **Control Flow**:
    - Check if the input string is empty or starts/ends with a delimiter, returning -1 if true.
    - Check if the string is 'default' or 'none', returning 0 if true.
    - Initialize an attribute bitmask to 0.
    - Iterate over the string, splitting it by delimiters and matching each part against a predefined table of attribute names.
    - If a match is found, update the bitmask with the corresponding attribute value.
    - If no match is found for a part, return -1.
    - Continue parsing until the end of the string is reached.
    - Return the final attribute bitmask.
- **Output**:
    - The function returns an integer bitmask representing the parsed attributes, or -1 if the input string is invalid or contains unrecognized attributes.


