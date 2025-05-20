# Purpose
This C source code file is part of a larger software system, likely related to terminal emulation or text processing, given its focus on handling UTF-8 encoded characters. The file provides a comprehensive set of functions for managing UTF-8 data, including encoding and decoding UTF-8 sequences, calculating the display width of Unicode characters, and maintaining a cache of character widths. The code utilizes red-black trees to efficiently store and retrieve UTF-8 data and character width information, which suggests that performance and quick access to this data are important considerations.

The file defines several static functions and data structures, indicating that it is intended to be used internally within a larger application rather than as a standalone library or public API. The presence of functions for converting between UTF-8 and wide characters, as well as functions for sanitizing and validating UTF-8 strings, highlights its role in ensuring that text data is correctly processed and displayed. The code also includes mechanisms for handling special cases, such as non-printable characters, and provides utilities for padding and formatting UTF-8 strings. Overall, this file is a specialized component focused on robust UTF-8 character handling, likely contributing to the text rendering or manipulation capabilities of the software it is part of.
# Imports and Dependencies

---
- `sys/types.h`
- `ctype.h`
- `errno.h`
- `stdlib.h`
- `string.h`
- `wchar.h`
- `compat.h`
- `tmux.h`


# Global Variables

---
### utf8_width_cache
- **Type**: `struct utf8_width_cache`
- **Description**: The `utf8_width_cache` is a global variable of type `struct utf8_width_cache`, which is a red-black tree data structure used to store and manage the width of UTF-8 characters. It is initialized using the `RB_INITIALIZER` macro, which sets up the tree for use.
- **Use**: This variable is used to cache the width of UTF-8 characters for efficient retrieval and management of character widths in the application.


---
### utf8_default_width_cache
- **Type**: ``struct utf8_width_item[]``
- **Description**: The `utf8_default_width_cache` is a static array of `struct utf8_width_item` that holds predefined Unicode code points and their corresponding display widths. Each entry in the array consists of a wide character (`wc`) and its width, which is set to 2 for all entries in this cache. This array is used to provide default width values for specific Unicode characters.
- **Use**: This variable is used to initialize and populate the UTF-8 width cache with default width values for certain Unicode characters.


---
### utf8_data_tree
- **Type**: `struct utf8_data_tree`
- **Description**: The `utf8_data_tree` is a static global variable of type `struct utf8_data_tree`, which is a red-black tree data structure used to store and manage UTF-8 items. Each item in the tree is represented by a `struct utf8_item`, which contains UTF-8 data and its size.
- **Use**: This variable is used to efficiently store and retrieve UTF-8 items based on their data, allowing for quick access and manipulation of UTF-8 encoded characters.


---
### utf8_index_tree
- **Type**: `struct utf8_index_tree`
- **Description**: The `utf8_index_tree` is a static global variable of type `struct utf8_index_tree`, which is initialized using the `RB_INITIALIZER` macro. This structure is part of a red-black tree implementation used to manage UTF-8 items by their index.
- **Use**: This variable is used to store and manage UTF-8 items in a red-black tree, allowing efficient retrieval and manipulation of UTF-8 data by index.


---
### utf8_no_width
- **Type**: `int`
- **Description**: The `utf8_no_width` variable is a static integer used within the file to indicate whether the width of a UTF-8 character should be considered or not. It is set to 1 when the width should be ignored, and 0 otherwise.
- **Use**: This variable is used to control whether the width of UTF-8 characters is calculated during certain operations.


---
### utf8_next_index
- **Type**: `u_int`
- **Description**: The `utf8_next_index` is a static unsigned integer variable used to keep track of the next available index for UTF-8 items in the data structure. It is initialized to zero and is incremented each time a new UTF-8 item is added to the data structure.
- **Use**: This variable is used to assign unique indices to new UTF-8 items as they are added to the data structure.


# Data Structures

---
### utf8_width_item
- **Type**: `struct`
- **Members**:
    - `wc`: A wide character represented by the `wchar_t` type.
    - `width`: An unsigned integer representing the width of the character.
    - `allocated`: An integer indicating if the item is dynamically allocated.
    - `entry`: A Red-Black tree entry for managing the structure in a tree.
- **Description**: The `utf8_width_item` structure is designed to store information about a UTF-8 character's width. It includes a wide character (`wc`), its display width (`width`), and a flag (`allocated`) to indicate if the structure was dynamically allocated. The `entry` member is used to integrate the structure into a Red-Black tree, allowing efficient storage and retrieval of character width information. This structure is part of a caching mechanism to optimize the handling of UTF-8 character widths in applications.


---
### utf8_item
- **Type**: `struct`
- **Members**:
    - `index_entry`: A Red-Black tree entry for indexing the utf8_item.
    - `index`: An unsigned integer representing the index of the utf8_item.
    - `data_entry`: A Red-Black tree entry for storing the utf8_item data.
    - `data`: A character array to store UTF-8 data, with a size defined by UTF8_SIZE.
    - `size`: An unsigned character representing the size of the UTF-8 data stored.
- **Description**: The `utf8_item` structure is designed to manage UTF-8 data within a Red-Black tree structure, allowing for efficient indexing and data storage. It contains two Red-Black tree entries, `index_entry` and `data_entry`, which facilitate the organization of items by index and data, respectively. The `index` field is used to uniquely identify each item, while the `data` array holds the actual UTF-8 encoded data, with its size specified by the `size` field. This structure is integral to managing UTF-8 data efficiently in applications that require fast lookup and storage capabilities.


# Functions

---
### utf8_width_cache_cmp<!-- {{#callable:utf8_width_cache_cmp}} -->
The `utf8_width_cache_cmp` function compares two `utf8_width_item` structures based on their `wc` (wide character) values.
- **Inputs**:
    - `uw1`: A pointer to the first `utf8_width_item` structure to be compared.
    - `uw2`: A pointer to the second `utf8_width_item` structure to be compared.
- **Control Flow**:
    - Check if the `wc` value of `uw1` is less than that of `uw2`; if true, return -1.
    - Check if the `wc` value of `uw1` is greater than that of `uw2`; if true, return 1.
    - If neither condition is met, return 0, indicating the `wc` values are equal.
- **Output**:
    - An integer indicating the result of the comparison: -1 if `uw1` is less than `uw2`, 1 if `uw1` is greater than `uw2`, and 0 if they are equal.


---
### utf8_data_cmp<!-- {{#callable:utf8_data_cmp}} -->
The `utf8_data_cmp` function compares two UTF-8 items based on their size and data content.
- **Inputs**:
    - `ui1`: A pointer to the first `utf8_item` structure to be compared.
    - `ui2`: A pointer to the second `utf8_item` structure to be compared.
- **Control Flow**:
    - Check if the size of `ui1` is less than the size of `ui2`; if true, return -1.
    - Check if the size of `ui1` is greater than the size of `ui2`; if true, return 1.
    - If sizes are equal, use `memcmp` to compare the data of `ui1` and `ui2` and return the result.
- **Output**:
    - An integer indicating the result of the comparison: -1 if `ui1` is smaller, 1 if `ui1` is larger, or the result of `memcmp` if they are equal in size.


---
### utf8_index_cmp<!-- {{#callable:utf8_index_cmp}} -->
The `utf8_index_cmp` function compares the indices of two `utf8_item` structures and returns an integer indicating their relative order.
- **Inputs**:
    - `ui1`: A pointer to the first `utf8_item` structure to be compared.
    - `ui2`: A pointer to the second `utf8_item` structure to be compared.
- **Control Flow**:
    - Check if the index of `ui1` is less than the index of `ui2`; if true, return -1.
    - Check if the index of `ui1` is greater than the index of `ui2`; if true, return 1.
    - If neither condition is met, return 0, indicating the indices are equal.
- **Output**:
    - An integer: -1 if `ui1`'s index is less than `ui2`'s, 1 if greater, and 0 if they are equal.


---
### utf8_item_by_data<!-- {{#callable:utf8_item_by_data}} -->
The function `utf8_item_by_data` retrieves a UTF-8 item from a red-black tree based on the provided data and size.
- **Inputs**:
    - `data`: A pointer to the UTF-8 data to be searched for in the tree.
    - `size`: The size of the data in bytes.
- **Control Flow**:
    - A `utf8_item` structure `ui` is declared and initialized.
    - The `data` is copied into `ui.data` using `memcpy`, and `ui.size` is set to `size`.
    - The function returns the result of `RB_FIND`, which searches the `utf8_data_tree` for an item matching `ui`.
- **Output**:
    - A pointer to the `utf8_item` found in the `utf8_data_tree` that matches the given data and size, or `NULL` if no match is found.


---
### utf8_item_by_index<!-- {{#callable:utf8_item_by_index}} -->
The function `utf8_item_by_index` retrieves a UTF-8 item from a red-black tree based on a given index.
- **Inputs**:
    - `index`: An unsigned integer representing the index of the UTF-8 item to be retrieved.
- **Control Flow**:
    - A `utf8_item` structure `ui` is initialized.
    - The `index` field of `ui` is set to the input `index`.
    - The function `RB_FIND` is called with the `utf8_index_tree` and `ui` to search for the item with the specified index in the red-black tree.
    - The result of `RB_FIND`, which is a pointer to the found `utf8_item` or NULL if not found, is returned.
- **Output**:
    - A pointer to the `utf8_item` structure with the specified index, or NULL if no such item exists in the tree.


---
### utf8_find_in_width_cache<!-- {{#callable:utf8_find_in_width_cache}} -->
The function `utf8_find_in_width_cache` searches for a `wchar_t` codepoint in a red-black tree cache of UTF-8 width items and returns the corresponding item if found.
- **Inputs**:
    - `wc`: A `wchar_t` representing the Unicode codepoint to search for in the width cache.
- **Control Flow**:
    - A `utf8_width_item` structure `uw` is initialized with the input `wchar_t` `wc`.
    - The function `RB_FIND` is called with the `utf8_width_cache` tree and the `uw` item to search for a matching item in the cache.
    - The result of `RB_FIND`, which is either a pointer to the found `utf8_width_item` or `NULL` if not found, is returned.
- **Output**:
    - A pointer to a `utf8_width_item` structure if the codepoint is found in the cache, or `NULL` if it is not found.


---
### utf8_update_width_cache<!-- {{#callable:utf8_update_width_cache}} -->
The `utf8_update_width_cache` function refreshes the UTF-8 width cache by clearing existing entries, inserting default widths, and adding custom widths from global options.
- **Inputs**:
    - None
- **Control Flow**:
    - Iterate over each item in the `utf8_width_cache` and safely remove it, freeing memory if the item was allocated.
    - Insert each item from `utf8_default_width_cache` into the `utf8_width_cache`.
    - Retrieve the 'codepoint-widths' option from `global_options` and iterate over its array items.
    - For each array item, call `utf8_add_to_width_cache` to add the custom width to the cache.
- **Output**:
    - The function does not return a value; it updates the global `utf8_width_cache` with new width data.
- **Functions called**:
    - [`options_get`](options.c.driver.md#options_get)
    - [`options_array_first`](options.c.driver.md#options_array_first)
    - [`options_array_item_value`](options.c.driver.md#options_array_item_value)
    - [`options_array_next`](options.c.driver.md#options_array_next)


---
### utf8_put_item<!-- {{#callable:utf8_put_item}} -->
The `utf8_put_item` function adds a UTF-8 item to a data structure, assigning it a unique index if it doesn't already exist.
- **Inputs**:
    - `data`: A pointer to the UTF-8 data to be added.
    - `size`: The size of the UTF-8 data.
    - `index`: A pointer to an unsigned integer where the index of the UTF-8 item will be stored.
- **Control Flow**:
    - The function first attempts to find an existing UTF-8 item in the data tree using [`utf8_item_by_data`](#utf8_item_by_data) with the provided data and size.
    - If the item is found, the function sets the `index` to the found item's index and logs a debug message, then returns 0.
    - If the item is not found, the function checks if the `utf8_next_index` has reached its maximum value (0xffffff + 1); if so, it returns -1 indicating an error.
    - If the index is still available, the function allocates memory for a new `utf8_item`, assigns it the next available index, and inserts it into both the index and data trees.
    - The function copies the provided data into the new item's data field, sets its size, and logs a debug message indicating the addition of the new item.
    - Finally, the function sets the `index` to the new item's index and returns 0.
- **Output**:
    - The function returns 0 on success, indicating the item was added or found, and -1 if the index limit is reached.
- **Functions called**:
    - [`utf8_item_by_data`](#utf8_item_by_data)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)


---
### utf8_from_data<!-- {{#callable:utf8_from_data}} -->
The `utf8_from_data` function converts UTF-8 data into a UTF-8 character representation, handling both valid and invalid data cases.
- **Inputs**:
    - `ud`: A pointer to a `utf8_data` structure containing the UTF-8 data to be converted.
    - `uc`: A pointer to a `utf8_char` where the resulting UTF-8 character will be stored.
- **Control Flow**:
    - Check if the width of the UTF-8 data is greater than 2, and if so, terminate the program with an error message.
    - If the size of the UTF-8 data exceeds the maximum allowed size (`UTF8_SIZE`), jump to the fail label.
    - If the size of the UTF-8 data is 3 or less, calculate the index by combining the data bytes into a single integer.
    - If the size is greater than 3, attempt to store the data in the UTF-8 item tree and retrieve an index; if this fails, jump to the fail label.
    - Set the UTF-8 character `uc` by combining the size, width, and index into a single value.
    - Log the conversion process and return `UTF8_DONE` to indicate success.
    - In the fail case, set `uc` to a default value based on the width and return `UTF8_ERROR` to indicate failure.
- **Output**:
    - Returns an `enum utf8_state` indicating the success (`UTF8_DONE`) or failure (`UTF8_ERROR`) of the conversion process.
- **Functions called**:
    - [`utf8_put_item`](#utf8_put_item)


---
### utf8_to_data<!-- {{#callable:utf8_to_data}} -->
The `utf8_to_data` function converts a UTF-8 character into a structured data format, populating a `utf8_data` structure with the character's size, width, and data bytes.
- **Inputs**:
    - `uc`: A UTF-8 character represented as an integer (`utf8_char`).
    - `ud`: A pointer to a `utf8_data` structure where the converted data will be stored.
- **Control Flow**:
    - Initialize the `utf8_data` structure `ud` to zero using `memset`.
    - Set the `size` and `have` fields of `ud` using the macro `UTF8_GET_SIZE` applied to `uc`.
    - Set the `width` field of `ud` using the macro `UTF8_GET_WIDTH` applied to `uc`.
    - If the size of the UTF-8 character is 3 bytes or less, extract each byte from `uc` and store them in `ud->data`.
    - If the size is greater than 3 bytes, retrieve the index from `uc` and attempt to find a corresponding `utf8_item` using [`utf8_item_by_index`](#utf8_item_by_index).
    - If a matching `utf8_item` is found, copy its data into `ud->data`; otherwise, fill `ud->data` with spaces.
    - Log the conversion process using `log_debug`.
- **Output**:
    - The function does not return a value; it modifies the `utf8_data` structure pointed to by `ud` to contain the converted UTF-8 character data.
- **Functions called**:
    - [`utf8_item_by_index`](#utf8_item_by_index)


---
### utf8_build_one<!-- {{#callable:utf8_build_one}} -->
The `utf8_build_one` function constructs a UTF-8 character representation from a single ASCII character by setting its size and width to 1.
- **Inputs**:
    - `ch`: A single unsigned character (u_char) representing an ASCII character to be converted into a UTF-8 character.
- **Control Flow**:
    - The function takes a single ASCII character as input.
    - It uses the macros `UTF8_SET_SIZE(1)` and `UTF8_SET_WIDTH(1)` to set the size and width of the UTF-8 character to 1.
    - The input character `ch` is combined with these settings using bitwise OR operations to form the final UTF-8 character representation.
- **Output**:
    - The function returns an unsigned integer (u_int) representing the UTF-8 character with size and width set to 1, combined with the input character.


---
### utf8_set<!-- {{#callable:utf8_set}} -->
The `utf8_set` function initializes a `utf8_data` structure with a single ASCII character.
- **Inputs**:
    - `ud`: A pointer to a `utf8_data` structure that will be initialized.
    - `ch`: A single ASCII character (`u_char`) to be set in the `utf8_data` structure.
- **Control Flow**:
    - A static constant `utf8_data` structure named `empty` is defined with default values.
    - The `memcpy` function is used to copy the contents of `empty` into the `utf8_data` structure pointed to by `ud`.
    - The first byte of the `data` array in the `utf8_data` structure is set to the character `ch`.
- **Output**:
    - The function does not return a value; it modifies the `utf8_data` structure pointed to by `ud`.


---
### utf8_copy<!-- {{#callable:utf8_copy}} -->
The `utf8_copy` function copies UTF-8 data from one `utf8_data` structure to another, ensuring that the destination's data array is null-terminated beyond the copied size.
- **Inputs**:
    - `to`: A pointer to the destination `utf8_data` structure where the data will be copied.
    - `from`: A pointer to the source `utf8_data` structure from which the data will be copied.
- **Control Flow**:
    - The function begins by copying the entire `utf8_data` structure from the source (`from`) to the destination (`to`) using `memcpy`.
    - It then iterates over the `data` array of the destination structure starting from the index equal to the `size` of the copied data.
    - For each index from `size` to the end of the `data` array, it sets the value to the null character (`'\0'`).
- **Output**:
    - The function does not return a value; it modifies the `to` structure in place.


---
### utf8_width<!-- {{#callable:utf8_width}} -->
The `utf8_width` function calculates the display width of a UTF-8 character represented by `utf8_data` and returns the result through a pointer.
- **Inputs**:
    - `ud`: A pointer to a `struct utf8_data` which contains the UTF-8 character data to be evaluated.
    - `width`: A pointer to an integer where the calculated width of the UTF-8 character will be stored.
- **Control Flow**:
    - Convert the UTF-8 data in `ud` to a wide character `wc` using `utf8_towc`; if conversion fails, return `UTF8_ERROR`.
    - Check if the wide character `wc` is in the width cache using [`utf8_find_in_width_cache`](#utf8_find_in_width_cache); if found, set `*width` to the cached width, log the result, and return `UTF8_DONE`.
    - If the character is not cached, calculate its width using [`utf8proc_wcwidth`](compat/utf8proc.c.driver.md#utf8proc_wcwidth) or `wcwidth` depending on the compilation flags; log the result.
    - If the width is negative and the character is a C1 control character, set the width to 0; otherwise, set it to 1.
    - If the calculated width is valid (between 0 and 255), return `UTF8_DONE`; otherwise, return `UTF8_ERROR`.
- **Output**:
    - The function returns an `enum utf8_state` indicating success (`UTF8_DONE`) or failure (`UTF8_ERROR`) in determining the width.
- **Functions called**:
    - [`utf8_find_in_width_cache`](#utf8_find_in_width_cache)
    - [`utf8proc_wcwidth`](compat/utf8proc.c.driver.md#utf8proc_wcwidth)


---
### utf8_fromwc<!-- {{#callable:utf8_fromwc}} -->
The `utf8_fromwc` function converts a wide character to a UTF-8 encoded character and determines its display width.
- **Inputs**:
    - `wc`: A wide character (wchar_t) to be converted to UTF-8.
    - `ud`: A pointer to a struct utf8_data where the UTF-8 encoded character and its properties will be stored.
- **Control Flow**:
    - The function first attempts to convert the wide character `wc` to a UTF-8 encoded character using either [`utf8proc_wctomb`](compat/utf8proc.c.driver.md#utf8proc_wctomb) or `wctomb`, depending on the availability of UTF8PROC.
    - If the conversion fails (size < 0), it logs the error, resets the conversion state, and returns `UTF8_ERROR`.
    - If the conversion size is zero, it returns `UTF8_ERROR`.
    - It sets the `size` and `have` fields of `ud` to the conversion size.
    - It calls [`utf8_width`](#utf8_width) to determine the display width of the UTF-8 character.
    - If [`utf8_width`](#utf8_width) returns `UTF8_DONE`, it sets the `width` field of `ud` and returns `UTF8_DONE`.
    - If [`utf8_width`](#utf8_width) does not return `UTF8_DONE`, it returns `UTF8_ERROR`.
- **Output**:
    - The function returns an enum `utf8_state`, which can be `UTF8_DONE` if the conversion and width determination are successful, or `UTF8_ERROR` if any step fails.
- **Functions called**:
    - [`utf8proc_wctomb`](compat/utf8proc.c.driver.md#utf8proc_wctomb)
    - [`utf8_width`](#utf8_width)


---
### utf8_open<!-- {{#callable:utf8_open}} -->
The `utf8_open` function initializes a UTF-8 data structure for a new UTF-8 sequence based on the first byte of the sequence and determines the expected size of the sequence.
- **Inputs**:
    - `ud`: A pointer to a `struct utf8_data` which will be initialized to store the UTF-8 sequence data.
    - `ch`: An unsigned character representing the first byte of a UTF-8 sequence.
- **Control Flow**:
    - The function begins by zeroing out the memory of the `utf8_data` structure pointed to by `ud`.
    - It checks the value of `ch` to determine the size of the UTF-8 sequence: 2 bytes if `ch` is between 0xC2 and 0xDF, 3 bytes if between 0xE0 and 0xEF, and 4 bytes if between 0xF0 and 0xF4.
    - If `ch` does not fall within these ranges, the function returns `UTF8_ERROR`.
    - If a valid size is determined, the function calls [`utf8_append`](#utf8_append) to add `ch` to the `utf8_data` structure.
    - Finally, the function returns `UTF8_MORE` to indicate that more bytes are expected to complete the sequence.
- **Output**:
    - The function returns an `enum utf8_state`, which is `UTF8_MORE` if the sequence is valid and more bytes are expected, or `UTF8_ERROR` if the first byte is invalid for starting a UTF-8 sequence.
- **Functions called**:
    - [`utf8_append`](#utf8_append)


---
### utf8_append<!-- {{#callable:utf8_append}} -->
The `utf8_append` function appends a UTF-8 character to a `utf8_data` structure and determines if the character sequence is complete or if an error has occurred.
- **Inputs**:
    - `ud`: A pointer to a `utf8_data` structure that holds the UTF-8 character data being constructed.
    - `ch`: A `u_char` representing the next byte of the UTF-8 character to append.
- **Control Flow**:
    - Check if the current number of bytes (`have`) in `ud` exceeds or equals the expected size (`size`); if so, trigger a fatal error for overflow.
    - Check if the expected size (`size`) of the UTF-8 character is larger than the buffer size; if so, trigger a fatal error for size too large.
    - If `have` is not zero and the byte `ch` does not match the continuation byte pattern, set `width` to 0xff to indicate an error.
    - Append the byte `ch` to the `data` array in `ud` and increment the `have` counter.
    - If `have` does not equal `size`, return `UTF8_MORE` indicating more bytes are needed.
    - If `utf8_no_width` is not set, check if `width` is 0xff and return `UTF8_ERROR` if true.
    - Call [`utf8_width`](#utf8_width) to calculate the width of the character; if it returns anything other than `UTF8_DONE`, return `UTF8_ERROR`.
    - Set the `width` in `ud` to the calculated width.
    - Return `UTF8_DONE` indicating the UTF-8 character is complete.
- **Output**:
    - The function returns an `enum utf8_state` which can be `UTF8_MORE` if more bytes are needed, `UTF8_DONE` if the character is complete, or `UTF8_ERROR` if an error occurred.
- **Functions called**:
    - [`utf8_width`](#utf8_width)


---
### utf8_strvis<!-- {{#callable:utf8_strvis}} -->
The `utf8_strvis` function encodes a given UTF-8 string into a visually safe format, handling special characters and incomplete UTF-8 sequences, and returns the length of the encoded string.
- **Inputs**:
    - `dst`: A pointer to the destination buffer where the encoded string will be stored.
    - `src`: A pointer to the source UTF-8 string that needs to be encoded.
    - `len`: The length of the source string to be processed.
    - `flag`: An integer flag that modifies the encoding behavior, such as handling special characters like '$'.
- **Control Flow**:
    - Initialize variables for UTF-8 data handling and set pointers for the start and end of the source string.
    - Enter a loop to process each character in the source string until the end is reached.
    - Attempt to open a UTF-8 sequence with [`utf8_open`](#utf8_open); if more bytes are needed, continue appending with [`utf8_append`](#utf8_append) until the sequence is complete or invalid.
    - If a complete UTF-8 character is formed, copy it to the destination buffer and continue to the next character.
    - If the character is not a valid UTF-8 sequence, adjust the source pointer to retry the sequence.
    - Check for special handling of the '$' character when the VIS_DQ flag is set, potentially escaping it with a backslash.
    - Use the [`vis`](compat/vis.c.driver.md#vis) function to encode non-UTF-8 characters or incomplete sequences, considering the next character for context.
    - Terminate the destination string with a null character and return the length of the encoded string.
- **Output**:
    - The function returns the length of the encoded string stored in the destination buffer.
- **Functions called**:
    - [`utf8_open`](#utf8_open)
    - [`utf8_append`](#utf8_append)
    - [`vis`](compat/vis.c.driver.md#vis)


---
### utf8_stravis<!-- {{#callable:utf8_stravis}} -->
The `utf8_stravis` function encodes a UTF-8 string into a visually safe format and allocates memory for the resulting string.
- **Inputs**:
    - `dst`: A pointer to a character pointer where the function will store the address of the newly allocated and encoded string.
    - `src`: The source string to be encoded, which is a null-terminated UTF-8 string.
    - `flag`: An integer flag that influences the encoding behavior, such as how certain characters are escaped.
- **Control Flow**:
    - Allocate memory for a buffer that can hold up to four times the length of the source string plus one for the null terminator.
    - Call [`utf8_strvis`](#utf8_strvis) to encode the source string into the buffer, using the specified flag for encoding options.
    - Reallocate the buffer to fit the exact length of the encoded string plus one for the null terminator.
    - Assign the reallocated buffer to the destination pointer `dst`.
    - Return the length of the encoded string.
- **Output**:
    - The function returns the length of the encoded string.
- **Functions called**:
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`utf8_strvis`](#utf8_strvis)
    - [`xrealloc`](xmalloc.c.driver.md#xrealloc)


---
### utf8_stravisx<!-- {{#callable:utf8_stravisx}} -->
The `utf8_stravisx` function encodes a given UTF-8 string into a buffer with a specified length and flag, allocating memory for the buffer dynamically.
- **Inputs**:
    - `dst`: A pointer to a character pointer where the address of the allocated buffer will be stored.
    - `src`: The source UTF-8 string to be encoded.
    - `srclen`: The length of the source string to be processed.
    - `flag`: An integer flag that influences the encoding behavior.
- **Control Flow**:
    - Allocate memory for a buffer using [`xreallocarray`](xmalloc.c.driver.md#xreallocarray) with a size of four times the source length plus one.
    - Call [`utf8_strvis`](#utf8_strvis) to encode the source string into the buffer, passing the buffer, source string, source length, and flag.
    - Reallocate the buffer to fit the exact length of the encoded string plus one for the null terminator using [`xrealloc`](xmalloc.c.driver.md#xrealloc).
    - Store the address of the reallocated buffer in the `dst` pointer.
    - Return the length of the encoded string.
- **Output**:
    - The function returns the length of the encoded string.
- **Functions called**:
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`utf8_strvis`](#utf8_strvis)
    - [`xrealloc`](xmalloc.c.driver.md#xrealloc)


---
### utf8_isvalid<!-- {{#callable:utf8_isvalid}} -->
The `utf8_isvalid` function checks if a given string is composed entirely of valid UTF-8 characters.
- **Inputs**:
    - `s`: A pointer to a null-terminated string that is to be checked for valid UTF-8 encoding.
- **Control Flow**:
    - Initialize a `utf8_data` structure `ud` and a pointer `end` to the end of the string `s`.
    - Iterate over each character in the string `s` until reaching the end.
    - For each character, attempt to open a UTF-8 sequence using [`utf8_open`](#utf8_open).
    - If the sequence requires more bytes (`UTF8_MORE`), continue appending bytes using [`utf8_append`](#utf8_append) until the sequence is complete (`UTF8_DONE`) or invalid.
    - If a sequence is invalid or a character is outside the printable ASCII range (0x20 to 0x7e), return 0 indicating invalid UTF-8.
    - If all characters are valid, return 1 indicating the string is valid UTF-8.
- **Output**:
    - Returns 1 if the string is valid UTF-8, otherwise returns 0.
- **Functions called**:
    - [`utf8_open`](#utf8_open)
    - [`utf8_append`](#utf8_append)


---
### utf8_sanitize<!-- {{#callable:utf8_sanitize}} -->
The `utf8_sanitize` function processes a UTF-8 encoded string, replacing invalid or non-printable characters with underscores ('_').
- **Inputs**:
    - `src`: A pointer to a null-terminated string containing UTF-8 encoded characters to be sanitized.
- **Control Flow**:
    - Initialize a destination string pointer `dst` to NULL and a size counter `n` to 0.
    - Iterate over each character in the source string `src` until the null terminator is reached.
    - For each character, attempt to open a UTF-8 sequence using [`utf8_open`](#utf8_open).
    - If the character starts a multi-byte UTF-8 sequence (`UTF8_MORE`), continue appending subsequent bytes using [`utf8_append`](#utf8_append) until the sequence is complete or invalid.
    - If a valid UTF-8 sequence is completed (`UTF8_DONE`), allocate space in `dst` for the sequence's width and fill it with underscores ('_').
    - If the character is a valid ASCII printable character (between 0x20 and 0x7E), copy it to `dst`.
    - If the character is not valid or printable, append an underscore ('_') to `dst`.
    - Reallocate `dst` to accommodate the new character or underscore, and increment the `src` pointer.
    - After processing all characters, append a null terminator to `dst` and return it.
- **Output**:
    - A newly allocated, null-terminated string with invalid or non-printable characters replaced by underscores, which the caller is responsible for freeing.
- **Functions called**:
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`utf8_open`](#utf8_open)
    - [`utf8_append`](#utf8_append)


---
### utf8_strlen<!-- {{#callable:utf8_strlen}} -->
The `utf8_strlen` function calculates the number of UTF-8 characters in a given UTF-8 data array.
- **Inputs**:
    - `s`: A pointer to an array of `utf8_data` structures, each representing a UTF-8 character, terminated by an element with a size of 0.
- **Control Flow**:
    - Initialize a size_t variable `i` to 0.
    - Iterate over the array `s` using `i` as the index, checking if `s[i].size` is not equal to 0.
    - Increment `i` for each iteration until a `utf8_data` element with size 0 is encountered.
- **Output**:
    - The function returns the number of UTF-8 characters in the array `s`, which is the value of `i` when the loop terminates.


---
### utf8_strwidth<!-- {{#callable:utf8_strwidth}} -->
The `utf8_strwidth` function calculates the total display width of a UTF-8 encoded string, up to a specified number of characters.
- **Inputs**:
    - `s`: A pointer to an array of `utf8_data` structures, each representing a UTF-8 character with its size and width.
    - `n`: A `ssize_t` value indicating the maximum number of characters to consider from the array; if `-1`, the entire array is considered.
- **Control Flow**:
    - Initialize a variable `width` to 0 to accumulate the total width.
    - Iterate over the `utf8_data` array using an index `i`, stopping when a character with size 0 is encountered, indicating the end of the string.
    - For each character, check if `n` is not `-1` and if `i` equals `n`; if so, break the loop to limit the number of characters considered.
    - Add the width of the current character to the `width` accumulator.
    - Return the accumulated `width` as the result.
- **Output**:
    - The function returns an unsigned integer representing the total width of the UTF-8 string, considering up to `n` characters or the entire string if `n` is `-1`.


---
### utf8_fromcstr<!-- {{#callable:utf8_fromcstr}} -->
The `utf8_fromcstr` function converts a null-terminated C string into a dynamically allocated array of UTF-8 data structures, each representing a UTF-8 character, and terminates the array with a structure of size zero.
- **Inputs**:
    - `src`: A pointer to a null-terminated C string that is to be converted into UTF-8 data structures.
- **Control Flow**:
    - Initialize a pointer `dst` to NULL and a size counter `n` to 0.
    - Enter a loop that continues until the end of the source string `src` is reached (i.e., until `*src` is '\0').
    - Reallocate memory for `dst` to accommodate one more `utf8_data` structure.
    - Attempt to open a UTF-8 sequence with the current character from `src` using [`utf8_open`](#utf8_open).
    - If the character is the start of a multi-byte UTF-8 sequence (`UTF8_MORE`), continue reading and appending subsequent bytes using [`utf8_append`](#utf8_append) until the sequence is complete (`UTF8_DONE`) or invalid.
    - If a complete UTF-8 sequence is formed, increment the counter `n` and continue to the next character.
    - If the sequence is invalid, backtrack `src` by the number of bytes read so far in the current `utf8_data` structure.
    - If the character is a single-byte UTF-8 character, set it directly in the `utf8_data` structure using [`utf8_set`](#utf8_set).
    - Increment the counter `n` and move to the next character in `src`.
    - After the loop, reallocate `dst` to add a final `utf8_data` structure with size zero to mark the end of the array.
    - Return the pointer `dst` to the caller.
- **Output**:
    - A pointer to a dynamically allocated array of `utf8_data` structures, each representing a UTF-8 character from the input string, terminated by a structure with size zero.
- **Functions called**:
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)
    - [`utf8_open`](#utf8_open)
    - [`utf8_append`](#utf8_append)
    - [`utf8_set`](#utf8_set)


---
### utf8_tocstr<!-- {{#callable:utf8_tocstr}} -->
The `utf8_tocstr` function converts a buffer of UTF-8 characters into a null-terminated C string.
- **Inputs**:
    - `src`: A pointer to a buffer of `struct utf8_data` elements, each representing a UTF-8 character, terminated by an element with size 0.
- **Control Flow**:
    - Initialize a null pointer `dst` for the destination string and a size_t variable `n` to 0 for tracking the length of the string.
    - Iterate over the `src` buffer until a `struct utf8_data` element with size 0 is encountered.
    - For each element in `src`, reallocate `dst` to accommodate the new data by increasing its size by the size of the current `utf8_data` element.
    - Copy the data from the current `utf8_data` element into the `dst` buffer at the current position `n`.
    - Increment `n` by the size of the current `utf8_data` element to update the position for the next data copy.
    - After the loop, reallocate `dst` to add space for the null terminator and set the last character of `dst` to '\0'.
- **Output**:
    - A pointer to the newly allocated C string containing the UTF-8 data from `src`, null-terminated.
- **Functions called**:
    - [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)


---
### utf8_cstrwidth<!-- {{#callable:utf8_cstrwidth}} -->
The `utf8_cstrwidth` function calculates the display width of a UTF-8 encoded C string.
- **Inputs**:
    - `s`: A pointer to a null-terminated UTF-8 encoded C string whose display width is to be calculated.
- **Control Flow**:
    - Initialize a temporary `utf8_data` structure and a `width` variable to zero.
    - Iterate over each character in the string `s` until the null terminator is reached.
    - For each character, attempt to open a UTF-8 sequence using [`utf8_open`](#utf8_open).
    - If the character is the start of a multi-byte UTF-8 sequence (`UTF8_MORE`), continue appending bytes using [`utf8_append`](#utf8_append) until the sequence is complete (`UTF8_DONE`) or invalid.
    - If a valid UTF-8 sequence is completed, add its width to the total `width` and continue to the next character.
    - If the character is a single-byte character and is printable (not a control character), increment the `width`.
    - Continue to the next character in the string.
    - Return the total calculated `width` of the string.
- **Output**:
    - The function returns an unsigned integer representing the total display width of the UTF-8 string.
- **Functions called**:
    - [`utf8_open`](#utf8_open)
    - [`utf8_append`](#utf8_append)


---
### utf8_padcstr<!-- {{#callable:utf8_padcstr}} -->
The `utf8_padcstr` function pads a UTF-8 string with spaces on the right to ensure it reaches a specified width.
- **Inputs**:
    - `s`: A pointer to the input UTF-8 string that needs to be padded.
    - `width`: An unsigned integer specifying the desired width of the output string after padding.
- **Control Flow**:
    - Calculate the current display width of the input string `s` using [`utf8_cstrwidth`](#utf8_cstrwidth).
    - If the current width `n` is greater than or equal to the specified `width`, return a duplicate of the input string using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - Determine the length of the input string `s` using `strlen`.
    - Allocate memory for the output string `out` to accommodate the original string and the additional padding spaces.
    - Copy the original string `s` into the newly allocated memory `out`.
    - Iterate from the current width `n` to the specified `width`, appending spaces to `out` to achieve the desired width.
    - Terminate the padded string with a null character.
    - Return the padded string `out`.
- **Output**:
    - A newly allocated string that is a padded version of the input string `s`, with spaces added to the right to reach the specified width.
- **Functions called**:
    - [`utf8_cstrwidth`](#utf8_cstrwidth)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)


---
### utf8_rpadcstr<!-- {{#callable:utf8_rpadcstr}} -->
The `utf8_rpadcstr` function right-pads a UTF-8 string with spaces to ensure it reaches a specified width.
- **Inputs**:
    - `s`: A pointer to the input UTF-8 string that needs to be right-padded.
    - `width`: An unsigned integer specifying the desired width of the output string in terms of character display width.
- **Control Flow**:
    - Calculate the display width of the input string `s` using [`utf8_cstrwidth`](#utf8_cstrwidth) and store it in `n`.
    - If `n` is greater than or equal to `width`, duplicate the input string `s` using [`xstrdup`](xmalloc.c.driver.md#xstrdup) and return it.
    - Calculate the length of the input string `s` using `strlen` and store it in `slen`.
    - Allocate memory for the output string `out` with a size sufficient to hold the original string plus the necessary padding spaces and a null terminator.
    - Fill the first `width - n` characters of `out` with spaces to achieve the desired padding.
    - Copy the original string `s` into `out` starting after the padding spaces.
    - Terminate the output string `out` with a null character and return it.
- **Output**:
    - A newly allocated string that is the right-padded version of the input string `s`, with a total display width of `width` characters.
- **Functions called**:
    - [`utf8_cstrwidth`](#utf8_cstrwidth)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)


---
### utf8_cstrhas<!-- {{#callable:utf8_cstrhas}} -->
The `utf8_cstrhas` function checks if a UTF-8 encoded string contains a specific UTF-8 character sequence.
- **Inputs**:
    - `s`: A C-style string (null-terminated) that is to be searched for the presence of a specific UTF-8 character sequence.
    - `ud`: A pointer to a `utf8_data` structure representing the UTF-8 character sequence to search for within the string `s`.
- **Control Flow**:
    - Convert the input C-style string `s` into a buffer of UTF-8 data structures using [`utf8_fromcstr`](#utf8_fromcstr).
    - Iterate over each UTF-8 data structure in the converted buffer until a structure with size 0 is encountered, indicating the end of the buffer.
    - For each UTF-8 data structure in the buffer, compare its size with the size of the input `utf8_data` structure `ud`.
    - If the sizes match, compare the data of the current UTF-8 data structure with the data in `ud` using `memcmp`.
    - If a match is found, set the `found` flag to 1 and break out of the loop.
    - Free the allocated memory for the UTF-8 data buffer.
    - Return the value of the `found` flag, which indicates whether the sequence was found.
- **Output**:
    - Returns an integer value: 1 if the UTF-8 character sequence represented by `ud` is found in the string `s`, otherwise 0.
- **Functions called**:
    - [`utf8_fromcstr`](#utf8_fromcstr)


