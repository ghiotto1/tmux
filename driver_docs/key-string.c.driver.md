# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it provides functionality for handling and interpreting key codes. The file defines a mapping between string representations of keys and their corresponding key codes, which are used within tmux to identify and process keyboard and mouse input. The code includes a comprehensive table of key strings, covering function keys, control characters, arrow keys, numeric keypad keys, and mouse events, each associated with a specific key code. This mapping allows tmux to convert user input from string format into a format that can be processed programmatically.

The file contains several static functions that facilitate the conversion between key strings and key codes. The [`key_string_search_table`](#key_string_search_table) function searches for a key string in the predefined table and returns the corresponding key code. The [`key_string_get_modifiers`](#key_string_get_modifiers) function extracts modifier keys (such as Control, Meta, and Shift) from a string. The [`key_string_lookup_string`](#key_string_lookup_string) function is a public API that converts a string representation of a key into a key code, considering modifiers and special cases like hexadecimal values and UTF-8 characters. Conversely, the [`key_string_lookup_key`](#key_string_lookup_key) function converts a key code back into its string representation, optionally including flags that describe the key's attributes. This file is crucial for enabling tmux to interpret and respond to user input accurately, making it a fundamental component of the software's input handling system.
# Imports and Dependencies

---
- `sys/types.h`
- `ctype.h`
- `stdlib.h`
- `string.h`
- `wchar.h`
- `tmux.h`


# Global Variables

---
### key_string_table
- **Type**: `array of struct`
- **Description**: The `key_string_table` is a static constant array of structures, where each structure contains a string representation of a key and its corresponding key code. This table maps various key names, including function keys, control characters, arrow keys, numeric keypad keys, and mouse keys, to their respective key codes, often with additional modifiers like `KEYC_IMPLIED_META`. The table is used to facilitate the conversion between string representations of keys and their internal key code equivalents.
- **Use**: This variable is used to look up key codes based on string representations of keys, enabling the conversion of key names to their corresponding key codes in the program.


# Functions

---
### key_string_search_table<!-- {{#callable:key_string_search_table}} -->
The `key_string_search_table` function searches for a given string in a predefined key string table and returns the corresponding key code, or a user-defined key code, or an unknown key code if not found.
- **Inputs**:
    - `string`: A constant character pointer representing the string to be searched in the key string table.
- **Control Flow**:
    - Initialize variables `i` and `user` for iteration and user key code storage, respectively.
    - Iterate over the `key_string_table` array using a for loop.
    - For each entry in the table, compare the input `string` with the `string` field of the current table entry using `strcasecmp` for case-insensitive comparison.
    - If a match is found, return the corresponding `key` from the table entry.
    - If no match is found in the table, use `sscanf` to check if the `string` matches the format 'User%u' and if the extracted user number is less than `KEYC_NUSER`.
    - If the user format is valid, return the user key code by adding the user number to `KEYC_USER`.
    - If neither a table match nor a valid user format is found, return `KEYC_UNKNOWN`.
- **Output**:
    - The function returns a `key_code` which is either a key code from the table, a user-defined key code, or `KEYC_UNKNOWN` if no match is found.


---
### key_string_get_modifiers<!-- {{#callable:key_string_get_modifiers}} -->
The `key_string_get_modifiers` function parses a string to identify and return key modifier flags such as Control, Meta, and Shift.
- **Inputs**:
    - `string`: A pointer to a string that represents key modifiers, which is modified to point past the parsed modifiers.
- **Control Flow**:
    - Initialize `modifiers` to 0.
    - Enter a loop that continues as long as the current character is not null and the next character is a hyphen ('-').
    - Check the current character to determine if it is 'C', 'M', or 'S' (case insensitive) and set the corresponding modifier flag using bitwise OR.
    - If the character is not recognized, set `*string` to NULL and return 0.
    - Advance the string pointer by two characters to skip the current modifier and the hyphen.
    - Return the accumulated `modifiers`.
- **Output**:
    - The function returns a `key_code` value representing the combined modifier flags found in the input string.


---
### key_string_lookup_string<!-- {{#callable:key_string_lookup_string}} -->
The `key_string_lookup_string` function converts a string representation of a key into its corresponding key code, handling various formats and modifiers.
- **Inputs**:
    - `string`: A constant character pointer representing the string to be converted into a key code.
- **Control Flow**:
    - Check if the input string is 'None' or 'Any' and return the corresponding key code.
    - If the string starts with '0x', attempt to parse it as a hexadecimal value and convert it to a key code, handling UTF-8 conversion if necessary.
    - Check for a short Ctrl key notation (e.g., '^A') and apply the Ctrl modifier if present.
    - Extract any modifiers (Ctrl, Meta, Shift) from the string using [`key_string_get_modifiers`](#key_string_get_modifiers).
    - If the string is a single ASCII character, return its key code unless it is a control character.
    - Attempt to interpret the string as a UTF-8 character and return its key code if successful.
    - If none of the above, search for the string in a predefined key table and return the corresponding key code.
    - Combine the determined key code with any extracted modifiers and return the result.
- **Output**:
    - The function returns a `key_code` which is the numeric representation of the key described by the input string, or `KEYC_UNKNOWN` if the string cannot be converted.
- **Functions called**:
    - [`utf8_fromcstr`](utf8.c.driver.md#utf8_fromcstr)
    - [`utf8_from_data`](utf8.c.driver.md#utf8_from_data)
    - [`key_string_get_modifiers`](#key_string_get_modifiers)
    - [`utf8_open`](utf8.c.driver.md#utf8_open)
    - [`utf8_append`](utf8.c.driver.md#utf8_append)
    - [`key_string_search_table`](#key_string_search_table)


---
### key_string_lookup_key<!-- {{#callable:key_string_lookup_key}} -->
The `key_string_lookup_key` function converts a key code into a human-readable string representation, optionally including flag information.
- **Inputs**:
    - `key`: A `key_code` value representing the key to be converted into a string.
    - `with_flags`: An integer indicating whether to include flag information in the output string.
- **Control Flow**:
    - Initialize an empty output string `out` and save the original key code.
    - Check if the key is a literal key and convert it directly to a character if so.
    - Append modifier prefixes (C-, M-, S-) to `out` if the key has control, meta, or shift modifiers.
    - Mask the key to remove modifiers and check for special key values like `KEYC_NONE`, `KEYC_UNKNOWN`, etc., appending their string representations to `out`.
    - If the key is a user-defined key, append 'User' followed by the user key number.
    - Search the key in the `key_string_table` and append the corresponding string if found.
    - If the key is a Unicode key, convert it to UTF-8 and append to `out`.
    - Handle invalid keys by appending an error string.
    - For printable ASCII keys, append the character or its representation to `out`.
    - If `with_flags` is true, append flag information enclosed in brackets to `out`.
- **Output**:
    - A pointer to a static character array containing the string representation of the key code, including any specified flags.
- **Functions called**:
    - [`strlcat`](compat/strlcat.c.driver.md#strlcat)
    - [`utf8_to_data`](utf8.c.driver.md#utf8_to_data)
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)


