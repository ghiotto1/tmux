# Purpose
This C source code file is part of the tmux terminal multiplexer project and is responsible for handling the translation between ASCII Control Sequences (ACS) and UTF-8 characters for terminal line drawing. The file defines several static tables that map ACS characters to their corresponding UTF-8 representations and vice versa. These mappings are crucial for rendering line-drawing characters correctly in terminals that support UTF-8 encoding. The file includes functions to retrieve the appropriate UTF-8 character for a given ACS character and to determine if a terminal should use ACS instead of UTF-8 for line drawing, based on terminal capabilities and user preferences.

The code provides a narrow but essential functionality within the tmux project, focusing specifically on character encoding translation for terminal display. It includes functions to access different styles of border characters (double, heavy, and rounded) in UTF-8, which are used to draw borders in terminal windows. The file does not define a public API but rather serves as an internal component of tmux, facilitating the correct display of line-drawing characters across different terminal types. The use of bsearch for efficient lookup in the mapping tables highlights the importance of performance in this translation process, ensuring that tmux can render terminal interfaces quickly and accurately.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### tty_acs_table
- **Type**: `static const struct tty_acs_entry[]`
- **Description**: The `tty_acs_table` is a static constant array of `tty_acs_entry` structures that maps ASCII characters to their corresponding UTF-8 encoded strings. Each entry in the table consists of a key, which is an ASCII character, and a string, which is the UTF-8 representation of a graphical character or symbol.
- **Use**: This variable is used to translate ASCII characters into UTF-8 encoded strings for terminal display, particularly for drawing graphical elements like lines and corners.


---
### tty_acs_reverse2
- **Type**: ``struct tty_acs_reverse_entry[]``
- **Description**: The `tty_acs_reverse2` is a static constant array of `struct tty_acs_reverse_entry` that maps a specific UTF-8 string to an ACS (Alternate Character Set) character. In this case, it contains a single entry that maps the UTF-8 string "\302\267" to the ACS character '~'. This mapping is used to reverse translate UTF-8 characters back to their corresponding ACS characters.
- **Use**: This variable is used to reverse map a specific UTF-8 character to its corresponding ACS character in terminal applications.


---
### tty_acs_reverse3
- **Type**: `struct tty_acs_reverse_entry[]`
- **Description**: The `tty_acs_reverse3` is a static constant array of `tty_acs_reverse_entry` structures. Each entry in the array maps a UTF-8 string to a corresponding ACS (Alternate Character Set) character key. This mapping is used to reverse translate UTF-8 characters back to their ACS equivalents.
- **Use**: This variable is used to perform reverse lookups from UTF-8 strings to ACS characters, particularly for strings of length 3.


---
### tty_acs_double_borders_list
- **Type**: `struct utf8_data[]`
- **Description**: The `tty_acs_double_borders_list` is a static constant array of `utf8_data` structures that represents a set of UTF-8 encoded characters used for drawing double borders in a terminal interface. Each element in the array contains a UTF-8 string and associated metadata, such as the length of the string and other attributes. This array is used to map specific border drawing characters to their UTF-8 representations.
- **Use**: This variable is used to provide UTF-8 encoded characters for drawing double borders in terminal applications.


---
### tty_acs_heavy_borders_list
- **Type**: ``struct utf8_data[]``
- **Description**: The `tty_acs_heavy_borders_list` is a static constant array of `struct utf8_data` that defines a set of UTF-8 encoded characters representing heavy border line drawing characters. Each element in the array contains a UTF-8 string and associated metadata, such as the length of the string and other attributes. This array is used to map specific heavy border characters to their UTF-8 representations.
- **Use**: This variable is used to provide UTF-8 encoded heavy border characters for terminal display, particularly in applications that require line drawing.


---
### tty_acs_rounded_borders_list
- **Type**: `struct utf8_data[]`
- **Description**: The `tty_acs_rounded_borders_list` is a static constant array of `utf8_data` structures that defines a set of UTF-8 characters used for rendering rounded border styles in a terminal. Each entry in the array corresponds to a specific Unicode character, such as vertical and horizontal lines, corners, and intersections, which are used to create rounded borders in text-based user interfaces.
- **Use**: This variable is used to map specific ACS (Alternate Character Set) entries to their corresponding UTF-8 representations for rounded border styles.


# Data Structures

---
### tty_acs_entry
- **Type**: `struct`
- **Members**:
    - `key`: A single character (u_char) that represents an ACS (Alternate Character Set) entry.
    - `string`: A constant character pointer to a UTF-8 string that corresponds to the ACS entry.
- **Description**: The `tty_acs_entry` structure is used to map characters from the Alternate Character Set (ACS) to their corresponding UTF-8 representations. This is particularly useful in terminal applications where certain graphical characters need to be displayed using UTF-8 encoding. The structure contains a `key`, which is a character from the ACS, and a `string`, which is the UTF-8 encoded equivalent of that character. This mapping allows for the conversion of ACS characters to UTF-8, enabling better compatibility and display on modern terminals that support UTF-8.


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
    - The function accesses the `tty_acs_heavy_borders_list` array using the `cell_type` index.
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
The `tty_acs_cmp` function compares a given key with the key of a `tty_acs_entry` structure to facilitate searching or sorting operations.
- **Inputs**:
    - `key`: A pointer to a `u_char` representing the key to be compared.
    - `value`: A pointer to a `tty_acs_entry` structure containing the key to compare against.
- **Control Flow**:
    - The function casts the `value` pointer to a `tty_acs_entry` structure pointer.
    - It dereferences the `key` pointer to obtain the `u_char` value for comparison.
    - The function returns the difference between the `u_char` value from `key` and the `key` field of the `tty_acs_entry` structure.
- **Output**:
    - The function returns an integer representing the difference between the `key` and the `entry->key`, which is used to determine their relative order.


---
### tty_acs_reverse_cmp<!-- {{#callable:tty_acs_reverse_cmp}} -->
The function `tty_acs_reverse_cmp` compares a given string key with the string field of a `tty_acs_reverse_entry` structure.
- **Inputs**:
    - `key`: A pointer to a constant character string that serves as the key for comparison.
    - `value`: A pointer to a `tty_acs_reverse_entry` structure that contains a string to be compared against the key.
- **Control Flow**:
    - The function casts the `value` pointer to a `tty_acs_reverse_entry` structure pointer.
    - It retrieves the `string` field from the `tty_acs_reverse_entry` structure.
    - It compares the `key` string with the `string` field using the `strcmp` function.
    - The result of `strcmp` is returned, indicating the lexicographical order of the two strings.
- **Output**:
    - The function returns an integer that indicates the result of the string comparison: zero if the strings are equal, a negative value if the key is less than the entry's string, or a positive value if the key is greater.


---
### tty_acs_needed<!-- {{#callable:tty_acs_needed}} -->
The `tty_acs_needed` function determines whether a terminal should use Alternate Character Set (ACS) instead of UTF-8 for line drawing.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure representing the terminal for which the decision is being made.
- **Control Flow**:
    - Check if the `tty` pointer is NULL; if so, return 0 indicating ACS is not needed.
    - Check if the terminal has the U8 capability and its value is 0; if true, return 1 to force ACS usage.
    - Check if the client associated with the terminal has the UTF-8 flag set; if true, return 0 indicating ACS is not needed.
    - If none of the above conditions are met, return 1 to indicate ACS is needed.
- **Output**:
    - Returns an integer: 1 if ACS is needed, 0 if UTF-8 can be used for line drawing.


---
### tty_acs_get<!-- {{#callable:tty_acs_get}} -->
The `tty_acs_get` function retrieves the appropriate character representation for a given character, either from the Alternate Character Set (ACS) or as a UTF-8 string, based on the terminal's capabilities.
- **Inputs**:
    - `tty`: A pointer to a `struct tty` which represents the terminal for which the character representation is being retrieved.
    - `ch`: An `u_char` representing the character for which the ACS or UTF-8 representation is needed.
- **Control Flow**:
    - Check if the terminal needs to use the ACS set instead of UTF-8 by calling `tty_acs_needed(tty)`.
    - If ACS is needed and the ACS entry for the character `ch` is not empty, return the ACS character from `tty->term->acs[ch]`.
    - If ACS is not needed, perform a binary search on `tty_acs_table` to find the UTF-8 translation for the character `ch`.
    - If a matching entry is found in `tty_acs_table`, return the corresponding UTF-8 string; otherwise, return `NULL`.
- **Output**:
    - Returns a pointer to a string representing the character `ch` in either ACS or UTF-8 format, or `NULL` if no representation is found.
- **Functions called**:
    - [`tty_acs_needed`](#tty_acs_needed)


---
### tty_acs_reverse_get<!-- {{#callable:tty_acs_reverse_get}} -->
The function `tty_acs_reverse_get` converts a UTF-8 string to its corresponding ACS (Alternate Character Set) key if it exists.
- **Inputs**:
    - `tty`: A pointer to a `tty` structure, which is unused in this function.
    - `s`: A pointer to a UTF-8 string that needs to be converted to an ACS key.
    - `slen`: The length of the UTF-8 string `s`, which should be either 2 or 3.
- **Control Flow**:
    - Check if `slen` is 2 or 3; if not, return -1.
    - If `slen` is 2, set `table` to `tty_acs_reverse2` and `items` to the number of elements in `tty_acs_reverse2`.
    - If `slen` is 3, set `table` to `tty_acs_reverse3` and `items` to the number of elements in `tty_acs_reverse3`.
    - Perform a binary search on the `table` using the string `s` to find a matching entry.
    - If no matching entry is found, return -1.
    - If a matching entry is found, return the `key` from the entry.
- **Output**:
    - Returns the ACS key corresponding to the UTF-8 string if found, otherwise returns -1.


