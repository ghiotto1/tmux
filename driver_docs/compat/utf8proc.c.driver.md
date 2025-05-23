# Purpose
This C source code file provides a set of functions for handling UTF-8 encoded characters, specifically focusing on converting between wide characters (`wchar_t`) and multibyte UTF-8 sequences. The file includes three primary functions: [`utf8proc_wcwidth`](#utf8proc_wcwidth), [`utf8proc_mbtowc`](#utf8proc_mbtowc), and [`utf8proc_wctomb`](#utf8proc_wctomb). These functions utilize the `utf8proc` library to perform their operations. The [`utf8proc_wcwidth`](#utf8proc_wcwidth) function determines the display width of a wide character, accounting for special cases like private use codepoints, which are assigned a width of 1. The [`utf8proc_mbtowc`](#utf8proc_mbtowc) function converts a multibyte UTF-8 sequence to a wide character, returning the number of bytes processed or an error if the sequence is invalid. Conversely, [`utf8proc_wctomb`](#utf8proc_wctomb) converts a wide character to a multibyte UTF-8 sequence, ensuring the validity of the codepoint before encoding.

The file is designed to be part of a larger system, likely serving as a utility for text processing applications that require precise handling of UTF-8 encoded data. It does not define a main function, indicating that it is not an executable but rather a library component intended to be integrated into other software. The inclusion of the `utf8proc` library suggests that the code leverages existing functionality for UTF-8 processing, enhancing it with specific handling for character width and conversion. The file's purpose is to provide reliable and efficient character encoding and decoding capabilities, which are essential for applications dealing with internationalization and multilingual text.
# Imports and Dependencies

---
- `sys/types.h`
- `utf8proc.h`
- `compat.h`


# Functions

---
### utf8proc_wcwidth<!-- {{#callable:utf8proc_wcwidth}} -->
The `utf8proc_wcwidth` function determines the display width of a wide character, particularly handling private use characters with a fixed width of 1.
- **Inputs**:
    - `wc`: A wide character (wchar_t) whose display width is to be determined.
- **Control Flow**:
    - Call `utf8proc_category` with `wc` to determine its category and store the result in `cat`.
    - Check if `cat` is equal to `UTF8PROC_CATEGORY_CO`, which represents private use characters.
    - If `cat` is `UTF8PROC_CATEGORY_CO`, return a width of 1, as these characters have an ambiguous width.
    - If `cat` is not `UTF8PROC_CATEGORY_CO`, call `utf8proc_charwidth` with `wc` and return its result as the character width.
- **Output**:
    - The function returns an integer representing the display width of the given wide character.


---
### utf8proc_mbtowc<!-- {{#callable:utf8proc_mbtowc}} -->
The function `utf8proc_mbtowc` converts a multibyte UTF-8 sequence to a wide character and returns the number of bytes processed or an error indicator.
- **Inputs**:
    - `pwc`: A pointer to a `wchar_t` where the resulting wide character will be stored.
    - `s`: A pointer to the input string containing the UTF-8 encoded multibyte sequence.
    - `n`: The maximum number of bytes to process from the input string `s`.
- **Control Flow**:
    - Check if the input string `s` is NULL; if so, return 0 indicating no conversion is performed.
    - Call `utf8proc_iterate` to convert the UTF-8 sequence in `s` to a wide character, storing the result in `pwc` and capturing the number of bytes processed in `slen`.
    - Check if the conversion resulted in an invalid codepoint (`*pwc == -1`) or an error (`slen < 0`); if so, return -1 indicating an error.
    - Return `slen`, the number of bytes processed, if the conversion is successful.
- **Output**:
    - The function returns the number of bytes processed from the input string `s` if successful, or -1 if an error occurs during conversion.


---
### utf8proc_wctomb<!-- {{#callable:utf8proc_wctomb}} -->
The function `utf8proc_wctomb` converts a wide character to a multibyte UTF-8 encoded string.
- **Inputs**:
    - `s`: A pointer to a character array where the UTF-8 encoded result will be stored.
    - `wc`: The wide character (wchar_t) to be converted to UTF-8.
- **Control Flow**:
    - Check if the pointer `s` is NULL; if so, return 0 indicating no conversion is performed.
    - Validate the wide character `wc` using `utf8proc_codepoint_valid`; if invalid, return -1 indicating an error.
    - Call `utf8proc_encode_char` to encode the wide character `wc` into the UTF-8 format and store it in `s`, returning the result of this function.
- **Output**:
    - The function returns 0 if `s` is NULL, -1 if `wc` is invalid, or the result of `utf8proc_encode_char` if successful.


