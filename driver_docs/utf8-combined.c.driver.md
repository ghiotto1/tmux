# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it provides specialized functionality for handling UTF-8 encoded data. The file defines several functions that analyze UTF-8 data to determine specific characteristics, such as whether a given sequence contains a zero-width joiner (ZWJ), is a ZWJ, is a variation selector, or is part of a modifier table. These functions are crucial for correctly interpreting and rendering complex text sequences, particularly those involving emoji or other characters that require special handling in terminal environments.

The code is focused on narrow functionality, specifically the identification and classification of certain UTF-8 sequences. It does not define a public API or external interfaces but rather provides utility functions likely used internally within the tmux application. The functions rely on the `utf8_data` structure, which is presumably defined elsewhere in the tmux codebase, and they utilize standard C library functions such as `memcmp` for byte comparison. The inclusion of the `tmux.h` header suggests that these functions are part of a larger set of utilities for managing UTF-8 data within the tmux project.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `wchar.h`
- `tmux.h`


# Functions

---
### utf8_has_zwj<!-- {{#callable:utf8_has_zwj}} -->
The function `utf8_has_zwj` checks if a given UTF-8 encoded string ends with a zero width joiner (ZWJ) character.
- **Inputs**:
    - `ud`: A pointer to a `utf8_data` structure, which contains the UTF-8 encoded string data and its size.
- **Control Flow**:
    - Check if the size of the UTF-8 data is less than 3; if so, return 0 indicating no ZWJ is present.
    - Use `memcmp` to compare the last three bytes of the UTF-8 data with the byte sequence representing the ZWJ character (`\342\200\215`).
    - Return 1 if the comparison is equal, indicating the presence of a ZWJ at the end, otherwise return 0.
- **Output**:
    - The function returns an integer: 1 if the UTF-8 data ends with a zero width joiner, otherwise 0.


---
### utf8_is_zwj<!-- {{#callable:utf8_is_zwj}} -->
The function `utf8_is_zwj` checks if a given UTF-8 encoded data represents a zero width joiner (ZWJ) character.
- **Inputs**:
    - `ud`: A pointer to a `utf8_data` structure containing the UTF-8 encoded data to be checked.
- **Control Flow**:
    - Check if the size of the UTF-8 data is exactly 3 bytes; if not, return 0 (false).
    - Compare the 3-byte data with the UTF-8 encoding of the zero width joiner character (`\342\200\215`); if they match, return 1 (true), otherwise return 0 (false).
- **Output**:
    - Returns 1 if the UTF-8 data represents a zero width joiner character, otherwise returns 0.


---
### utf8_is_vs<!-- {{#callable:utf8_is_vs}} -->
The function `utf8_is_vs` checks if a given UTF-8 encoded data represents a variation selector character.
- **Inputs**:
    - `ud`: A pointer to a `struct utf8_data` which contains the UTF-8 encoded data to be checked.
- **Control Flow**:
    - Check if the size of the UTF-8 data is exactly 3 bytes; if not, return 0 (false).
    - Compare the 3-byte data with the specific byte sequence for a variation selector ("\357\270\217").
    - Return 1 (true) if the data matches the variation selector sequence, otherwise return 0 (false).
- **Output**:
    - The function returns an integer: 1 if the data is a variation selector, and 0 otherwise.


---
### utf8_is_modifier<!-- {{#callable:utf8_is_modifier}} -->
The function `utf8_is_modifier` checks if a given UTF-8 encoded character is a modifier by comparing it against a predefined set of Unicode code points.
- **Inputs**:
    - `ud`: A pointer to a `utf8_data` structure representing the UTF-8 encoded character to be checked.
- **Control Flow**:
    - Convert the UTF-8 encoded character in `ud` to a wide character `wc` using `utf8_towc` function.
    - If the conversion is not successful (i.e., `utf8_towc` does not return `UTF8_DONE`), return 0 indicating the character is not a modifier.
    - Use a switch statement to check if `wc` matches any of the predefined Unicode code points that represent modifier characters.
    - If `wc` matches one of the predefined code points, return 1 indicating the character is a modifier.
    - If `wc` does not match any of the predefined code points, return 0 indicating the character is not a modifier.
- **Output**:
    - The function returns an integer: 1 if the character is a modifier, and 0 otherwise.


