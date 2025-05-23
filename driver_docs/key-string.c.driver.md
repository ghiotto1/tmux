# Purpose
This C source code file is part of the tmux terminal multiplexer project and is responsible for handling key string conversions and mappings. It defines a set of functions and data structures that facilitate the conversion between string representations of keys and their corresponding key codes. The file includes a comprehensive table (`key_string_table`) that maps string representations of various keys, such as function keys, control characters, arrow keys, and mouse events, to their respective `key_code` values. This mapping is crucial for interpreting user input and translating it into actions within the tmux environment.

The file provides several key functions, including [`key_string_lookup_string`](#key_string_lookup_string), which converts a string representation of a key into a `key_code`, and [`key_string_lookup_key`](#key_string_lookup_key), which performs the reverse operation, converting a `key_code` back into a string. These functions handle various input formats, including ASCII, UTF-8, and hexadecimal representations, and support modifiers like Control, Meta, and Shift. The code is designed to be integrated into the broader tmux application, providing essential functionality for key event handling and user interaction. It does not define a public API but rather serves as an internal component of the tmux system, focusing on key input processing.
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
- **Description**: The `key_string_table` is a static constant array of structures, where each structure contains a string representation of a key and its corresponding key code. This table maps human-readable key names to their respective key codes, including function keys, control characters, arrow keys, numeric keypad keys, and mouse keys.
- **Use**: This variable is used to translate string representations of keys into their corresponding key codes for processing within the application.


# Functions

---
### key_string_search_table<!-- {{#callable:key_string_search_table}} -->
The `key_string_search_table` function searches for a given string in a predefined key string table and returns the corresponding key code.
- **Inputs**:
    - `string`: A constant character pointer representing the string to be searched in the key string table.
- **Control Flow**:
    - Iterates over the `key_string_table` array to compare each entry's string with the input `string` using a case-insensitive comparison (`strcasecmp`).
    - If a match is found, returns the corresponding key code from the table.
    - If no match is found, checks if the `string` matches the pattern 'User%u' and if the extracted user number is less than `KEYC_NUSER`, returns the key code `KEYC_USER` plus the user number.
    - If neither condition is met, returns `KEYC_UNKNOWN` indicating the string does not correspond to any known key.
- **Output**:
    - Returns a `key_code` which is the key code associated with the input string, or `KEYC_UNKNOWN` if the string does not match any known key.


---
### key_string_get_modifiers<!-- {{#callable:key_string_get_modifiers}} -->
The `key_string_get_modifiers` function parses a string to identify and return key modifier flags such as Control, Meta, and Shift.
- **Inputs**:
    - `string`: A pointer to a string that represents key modifiers, which is modified during parsing.
- **Control Flow**:
    - Initialize `modifiers` to 0.
    - Enter a loop that continues as long as the current character is not the null terminator and the next character is a hyphen ('-').
    - Check the current character to determine if it is 'C', 'M', or 'S' (case insensitive) and set the corresponding modifier flag using bitwise OR.
    - If the character is not recognized, set `*string` to NULL and return 0.
    - Advance the string pointer by two characters to skip the current modifier and the hyphen.
    - Return the accumulated `modifiers` value.
- **Output**:
    - The function returns a `key_code` value representing the combined modifier flags found in the input string.


---
### key_string_lookup_string<!-- {{#callable:key_string_lookup_string}} -->
The `key_string_lookup_string` function converts a string representation of a key into its corresponding key code, handling various formats and modifiers.
- **Inputs**:
    - `string`: A constant character pointer representing the string to be converted into a key code.
- **Control Flow**:
    - Check if the string is 'None' or 'Any' and return the corresponding key code.
    - If the string starts with '0x', attempt to parse it as a hexadecimal value and convert it to a key code.
    - Check for a short Ctrl key notation (e.g., '^A') and adjust the modifiers accordingly.
    - Extract any modifiers from the string using [`key_string_get_modifiers`](#key_string_get_modifiers).
    - If the string is a single ASCII character, return its key code if valid.
    - Attempt to interpret the string as a UTF-8 character and convert it to a key code if successful.
    - If none of the above, search for the string in a predefined key table and return the corresponding key code.
    - Combine the determined key code with any modifiers and return the result.
- **Output**:
    - The function returns a `key_code` which is the numeric representation of the key described by the input string, or `KEYC_UNKNOWN` if the string cannot be converted.
- **Functions called**:
    - [`key_string_get_modifiers`](#key_string_get_modifiers)
    - [`key_string_search_table`](#key_string_search_table)


---
### key_string_lookup_key<!-- {{#callable:key_string_lookup_key}} -->
The `key_string_lookup_key` function converts a key code into a human-readable string representation, optionally including modifier flags.
- **Inputs**:
    - `key`: A `key_code` value representing the key to be converted into a string.
    - `with_flags`: An integer flag indicating whether to include modifier flags in the output string.
- **Control Flow**:
    - Initialize an empty output string `out` and save the original key code.
    - Check if the key is a literal key and convert it directly to a character if so.
    - Append modifier prefixes (C-, M-, S-) to `out` if the key has control, meta, or shift modifiers, respectively.
    - Mask the key to remove modifiers and check for special key values like `KEYC_NONE`, `KEYC_UNKNOWN`, etc., appending their string representations to `out`.
    - If the key is a user-defined key, append 'User' followed by the user key number to `out`.
    - Search the key in the `key_string_table` and append the corresponding string if found.
    - If the key is a Unicode key, convert it to UTF-8 and append to `out`.
    - For invalid keys (greater than 255), append an error string to `out`.
    - For printable ASCII keys, append the character or its representation to `out`.
    - If `with_flags` is true and the original key had flags, append the flags in square brackets to `out`.
- **Output**:
    - A pointer to a static character array containing the string representation of the key code, including any specified modifiers and flags.


# Function Declarations (Public API)

---
### key_string_search_table<!-- {{#callable_declaration:key_string_search_table}} -->
Converts a string representation of a key to its corresponding key code.
- **Description**: This function is used to map a string that represents a key to its corresponding key code. It searches through a predefined table of key strings and returns the associated key code if a match is found. If the string matches the format 'UserX' where X is a number, it returns a user-defined key code if X is within a valid range. If no match is found, it returns a code indicating an unknown key. This function is useful for interpreting user input or configuration files where keys are specified as strings.
- **Inputs**:
    - `string`: A pointer to a null-terminated string representing the key. The string must not be null, and it should be a valid key representation. If the string is not found in the table or does not match the 'UserX' format with a valid X, the function will return a code for an unknown key.
- **Output**: Returns the key code corresponding to the input string, or a code indicating an unknown key if no match is found.
- **See also**: [`key_string_search_table`](#key_string_search_table)  (Implementation)


---
### key_string_get_modifiers<!-- {{#callable_declaration:key_string_get_modifiers}} -->
Extracts modifier keys from a string representation.
- **Description**: This function parses a string to identify and extract modifier keys such as Control, Meta, and Shift, represented by 'C-', 'M-', and 'S-' respectively. It is used when you need to interpret a string that specifies key combinations with modifiers. The function expects the string to be formatted with modifiers followed by a hyphen (e.g., 'C-x'). If an unrecognized modifier is encountered, the function sets the input string pointer to NULL and returns 0, indicating no valid modifiers were found. This function should be used when parsing key strings in environments where modifier keys are represented in this format.
- **Inputs**:
    - `string`: A pointer to a string pointer that contains the key string to be parsed. The string must not be null and should be formatted with modifiers followed by a hyphen. The function updates this pointer to point past the parsed modifiers. If an invalid modifier is encountered, the pointer is set to NULL.
- **Output**: Returns a key_code value representing the combined modifiers found in the string. If no valid modifiers are found, or an invalid modifier is encountered, it returns 0.
- **See also**: [`key_string_get_modifiers`](#key_string_get_modifiers)  (Implementation)


