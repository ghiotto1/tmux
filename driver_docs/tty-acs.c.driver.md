# Purpose
This C source code file is part of the tmux terminal multiplexer project and is responsible for handling the translation between ASCII Control Sequence (ACS) characters and their UTF-8 equivalents. The file defines several static tables that map ACS characters to UTF-8 strings and vice versa, allowing tmux to render line-drawing characters and other special symbols correctly in terminals that support UTF-8. The code includes functions to retrieve the appropriate character representation based on the terminal's capabilities, such as [`tty_acs_get`](#tty_acs_get) for converting ACS to UTF-8 and [`tty_acs_reverse_get`](#tty_acs_reverse_get) for the reverse operation. Additionally, it provides functions to determine if a terminal should use ACS instead of UTF-8 for line drawing, based on terminal flags and capabilities.

The file is not an executable but rather a component of the tmux library, intended to be used internally by other parts of the tmux codebase. It does not define public APIs or external interfaces but instead focuses on the internal logic required to support character rendering in different terminal environments. The code is structured around the concept of character translation, with a clear emphasis on ensuring compatibility across various terminal types by dynamically choosing the appropriate character set based on the terminal's features.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### tty_acs_table
- **Type**: ``static const struct tty_acs_entry[]``
- **Description**: The `tty_acs_table` is a static constant array of `tty_acs_entry` structures that maps ASCII characters to their corresponding UTF-8 encoded strings. Each entry in the table consists of a key, which is an ASCII character, and a string, which is the UTF-8 representation of a graphical character or symbol.
- **Use**: This variable is used to translate ASCII characters into UTF-8 encoded strings for terminal display purposes.


---
### tty_acs_reverse2
- **Type**: ``struct tty_acs_reverse_entry[]``
- **Description**: The `tty_acs_reverse2` is a static constant array of `struct tty_acs_reverse_entry` that maps a specific UTF-8 character sequence to an ACS (Alternate Character Set) key. In this case, it contains a single entry that maps the UTF-8 sequence for a bullet character ("\302\267") to the ACS key '~'. This mapping is used to reverse translate UTF-8 characters back to their corresponding ACS keys.
- **Use**: This variable is used to reverse map a specific UTF-8 character sequence to an ACS key for terminals that require ACS instead of UTF-8.


---
### tty_acs_reverse3
- **Type**: ``struct tty_acs_reverse_entry[]``
- **Description**: The `tty_acs_reverse3` is a static constant array of `struct tty_acs_reverse_entry` that maps UTF-8 encoded strings to their corresponding ACS (Alternate Character Set) keys. Each entry in the array consists of a UTF-8 string and a single character key, facilitating the reverse lookup from UTF-8 to ACS characters.
- **Use**: This variable is used to reverse map UTF-8 strings to ACS keys for terminals that require ACS instead of UTF-8 for line drawing.


---
### tty_acs_double_borders_list
- **Type**: ``struct utf8_data[]``
- **Description**: The `tty_acs_double_borders_list` is a static constant array of `struct utf8_data` that contains UTF-8 encoded characters representing double border line drawing characters. Each element in the array corresponds to a specific line drawing character, such as vertical or horizontal lines, corners, and intersections, encoded in UTF-8 format.
- **Use**: This array is used to map cell types to their corresponding UTF-8 double border characters for rendering in terminal applications.


---
### tty_acs_heavy_borders_list
- **Type**: ``struct utf8_data[]``
- **Description**: The `tty_acs_heavy_borders_list` is a static constant array of `struct utf8_data` that defines a set of UTF-8 encoded characters representing heavy border line drawing characters. Each element in the array contains a UTF-8 string and associated metadata, such as the length of the string and other attributes. These characters are used to render heavy borders in terminal applications that support UTF-8.
- **Use**: This variable is used to provide UTF-8 encoded heavy border characters for rendering in terminal applications.


---
### tty_acs_rounded_borders_list
- **Type**: `struct utf8_data[]`
- **Description**: The `tty_acs_rounded_borders_list` is a static constant array of `struct utf8_data` that defines a set of UTF-8 encoded characters used for rendering rounded border styles in a terminal. Each element in the array represents a specific border character, with its UTF-8 byte sequence and associated metadata. This array is part of a larger system for handling different border styles in terminal applications.
- **Use**: This variable is used to provide UTF-8 character data for rendering rounded borders in terminal applications.


# Data Structures

---
### tty_acs_entry
- **Type**: `struct`
- **Members**:
    - `key`: A single character (u_char) that represents an ACS entry.
    - `string`: A constant character pointer to a UTF-8 string that corresponds to the ACS entry.
- **Description**: The `tty_acs_entry` structure is used to map ASCII Control Sequence (ACS) entries to their corresponding UTF-8 representations. It consists of a `key`, which is a single character representing the ACS entry, and a `string`, which is a pointer to the UTF-8 string that visually represents the ACS character. This structure is part of a table that facilitates the conversion of ACS characters to UTF-8, allowing for proper display of line-drawing characters in terminal applications that support UTF-8.


---
### tty_acs_reverse_entry
- **Type**: `struct`
- **Members**:
    - `string`: A pointer to a constant character string representing a UTF-8 sequence.
    - `key`: An unsigned character representing the ACS key corresponding to the UTF-8 sequence.
- **Description**: The `tty_acs_reverse_entry` structure is used to map UTF-8 sequences back to their corresponding ACS (Alternate Character Set) keys. This is part of a mechanism to support terminals that may not fully support UTF-8 line drawing characters, allowing them to fall back to using ACS characters. The structure contains a UTF-8 string and its associated ACS key, facilitating reverse lookup operations.


# Functions

---
### tty_acs_double_borders<!-- {{#callable:tty_acs_double_borders}} -->
The function `tty_acs_double_borders` retrieves a UTF-8 character representation for a double border style based on a given cell type index.
- **Inputs**:
    - `cell_type`: An integer representing the index of the desired double border character in the `tty_acs_double_borders_list` array.
- **Control Flow**:
    - The function takes an integer `cell_type` as input.
    - It accesses the `tty_acs_double_borders_list` array using the `cell_type` index.
    - It returns a pointer to the `utf8_data` structure at the specified index in the array.
- **Output**:
    - A pointer to a `utf8_data` structure representing the UTF-8 character for the specified double border style.


---
### tty_acs_heavy_borders<!-- {{#callable:tty_acs_heavy_borders}} -->
The function `tty_acs_heavy_borders` returns a pointer to a UTF-8 data structure representing a heavy border character based on the given cell type index.
- **Inputs**:
    - `cell_type`: An integer representing the index of the desired heavy border character in the `tty_acs_heavy_borders_list` array.
- **Control Flow**:
    - The function takes an integer `cell_type` as input.
    - It accesses the `tty_acs_heavy_borders_list` array using the `cell_type` as an index.
    - It returns a pointer to the `utf8_data` structure at the specified index in the array.
- **Output**:
    - A pointer to a `utf8_data` structure representing a heavy border character.


---
### tty_acs_rounded_borders<!-- {{#callable:tty_acs_rounded_borders}} -->
The function `tty_acs_rounded_borders` retrieves a UTF-8 character representation for a rounded border style based on a given cell type index.
- **Inputs**:
    - `cell_type`: An integer representing the index of the desired rounded border character in the `tty_acs_rounded_borders_list` array.
- **Control Flow**:
    - The function accesses the `tty_acs_rounded_borders_list` array using the provided `cell_type` index.
    - It returns a pointer to the `utf8_data` structure at the specified index in the array.
- **Output**:
    - A pointer to a `utf8_data` structure representing the UTF-8 character for the specified rounded border style.


---
### tty_acs_cmp<!-- {{#callable:tty_acs_cmp}} -->
The function `tty_acs_cmp` compares a given key with the key of a `tty_acs_entry` structure to facilitate searching or sorting operations.
- **Inputs**:
    - `key`: A pointer to a `u_char` representing the key to be compared.
    - `value`: A pointer to a `tty_acs_entry` structure containing a key and a string.
- **Control Flow**:
    - The function casts the `value` pointer to a `tty_acs_entry` structure pointer.
    - It retrieves the `key` from the `tty_acs_entry` structure.
    - It casts the `key` pointer to a `u_char` and dereferences it to get the integer value `test`.
    - The function returns the result of subtracting the `entry->key` from `test`.
- **Output**:
    - The function returns an integer that is the difference between the `test` value and the `entry->key`, which indicates the relative order of the two keys.


---
### tty_acs_reverse_cmp<!-- {{#callable:tty_acs_reverse_cmp}} -->
The function `tty_acs_reverse_cmp` compares a given UTF-8 string with the string in a `tty_acs_reverse_entry` structure.
- **Inputs**:
    - `key`: A pointer to a UTF-8 string that is being searched for.
    - `value`: A pointer to a `tty_acs_reverse_entry` structure containing a string and a corresponding ACS key.
- **Control Flow**:
    - The function casts the `value` pointer to a `tty_acs_reverse_entry` structure pointer.
    - It casts the `key` pointer to a `char` pointer representing the UTF-8 string to be compared.
    - The function uses `strcmp` to compare the UTF-8 string pointed to by `key` with the `string` field of the `tty_acs_reverse_entry` structure pointed to by `value`.
    - The result of `strcmp` is returned, indicating the lexicographical order of the two strings.
- **Output**:
    - The function returns an integer that indicates the result of the string comparison: zero if the strings are equal, a negative value if the `key` string is less than the `entry->string`, or a positive value if the `key` string is greater.


---
### tty_acs_needed<!-- {{#callable:tty_acs_needed}} -->
The `tty_acs_needed` function determines whether a terminal should use Alternate Character Set (ACS) instead of UTF-8 for line drawing.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal for which the decision is being made.
- **Control Flow**:
    - Check if the `tty` pointer is NULL; if so, return 0 indicating ACS is not needed.
    - Check if the terminal has the U8 capability and its value is 0; if true, return 1 to force ACS usage.
    - Check if the client associated with the terminal has the CLIENT_UTF8 flag set; if true, return 0 to use UTF-8 line drawing.
    - If none of the above conditions are met, return 1 to indicate ACS should be used.
- **Output**:
    - Returns an integer: 1 if ACS is needed, or 0 if UTF-8 line drawing should be used.
- **Functions called**:
    - [`tty_term_has`](tty-term.c.driver.md#tty_term_has)
    - [`tty_term_number`](tty-term.c.driver.md#tty_term_number)


---
### tty_acs_get<!-- {{#callable:tty_acs_get}} -->
The `tty_acs_get` function retrieves the appropriate character representation for a given character, either from the ACS set or as a UTF-8 string, based on terminal capabilities.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` which represents the terminal state and capabilities.
    - `ch`: An unsigned character (`u_char`) representing the character for which the ACS or UTF-8 representation is sought.
- **Control Flow**:
    - Check if the terminal needs to use the ACS set instead of UTF-8 by calling `tty_acs_needed(tty)`.
    - If ACS is needed and the ACS entry for the character `ch` is not empty, return the ACS character from `tty->term->acs[ch]`.
    - If ACS is not needed, perform a binary search (`bsearch`) on `tty_acs_table` to find the UTF-8 translation for the character `ch`.
    - If a matching entry is found in the table, return the UTF-8 string from the entry.
    - If no matching entry is found, return `NULL`.
- **Output**:
    - Returns a pointer to a character string representing the ACS or UTF-8 equivalent of the input character `ch`, or `NULL` if no representation is found.
- **Functions called**:
    - [`tty_acs_needed`](#tty_acs_needed)


---
### tty_acs_reverse_get<!-- {{#callable:tty_acs_reverse_get}} -->
The function `tty_acs_reverse_get` retrieves the ACS (Alternate Character Set) key corresponding to a given UTF-8 string of length 2 or 3.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure, which is unused in this function.
    - `s`: A pointer to a UTF-8 string for which the corresponding ACS key is to be found.
    - `slen`: The length of the UTF-8 string `s`, expected to be either 2 or 3.
- **Control Flow**:
    - Check if `slen` is 2; if so, set `table` to `tty_acs_reverse2` and `items` to the number of elements in `tty_acs_reverse2`.
    - If `slen` is 3, set `table` to `tty_acs_reverse3` and `items` to the number of elements in `tty_acs_reverse3`.
    - If `slen` is neither 2 nor 3, return -1 indicating an error.
    - Perform a binary search on `table` using `s` to find a matching entry using `tty_acs_reverse_cmp`.
    - If no matching entry is found, return -1.
    - If a matching entry is found, return the `key` from the entry.
- **Output**:
    - Returns the ACS key corresponding to the UTF-8 string `s` if found, otherwise returns -1.


