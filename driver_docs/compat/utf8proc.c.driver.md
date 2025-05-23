# Purpose
This C source code file provides functionality for handling UTF-8 encoded characters, specifically focusing on character width determination and conversion between multibyte and wide character representations. The file includes functions that interface with the `utf8proc` library, which is a library for processing UTF-8 encoded Unicode strings. The primary functions defined in this file are [`utf8proc_wcwidth`](#utf8proc_wcwidth), [`utf8proc_mbtowc`](#utf8proc_mbtowc), and [`utf8proc_wctomb`](#utf8proc_wctomb). These functions are designed to determine the display width of a wide character, convert a multibyte string to a wide character, and convert a wide character to a multibyte string, respectively. The code handles special cases such as private use characters, which are given a default width of 1, and checks for valid codepoints during conversions.

The file is intended to be part of a larger codebase, likely serving as a utility for applications that need to handle UTF-8 encoded text. It is not a standalone executable but rather a component that can be included in other projects to provide specific UTF-8 processing capabilities. The inclusion of the `compat.h` header suggests that this file may also address compatibility issues across different systems or environments. The functions defined here do not expose a public API directly but are likely intended to be used internally within a project or library that requires robust UTF-8 character handling.
# Imports and Dependencies

---
- `sys/types.h`
- `utf8proc.h`
- `compat.h`


# Functions

---
### utf8proc_wcwidth<!-- {{#callable:utf8proc_wcwidth}} -->
The function `utf8proc_wcwidth` determines the display width of a wide character, accounting for special cases like private use characters.
- **Inputs**:
    - `wc`: A wide character (wchar_t) whose display width is to be determined.
- **Control Flow**:
    - The function calls `utf8proc_category` to get the category of the wide character `wc`.
    - It checks if the category is `UTF8PROC_CATEGORY_CO`, which represents private use characters.
    - If the category is `UTF8PROC_CATEGORY_CO`, the function returns a width of 1, as these characters have an ambiguous width.
    - If the category is not `UTF8PROC_CATEGORY_CO`, the function calls `utf8proc_charwidth` to determine the width of the character and returns that value.
- **Output**:
    - The function returns an integer representing the display width of the wide character `wc`.


---
### utf8proc_mbtowc<!-- {{#callable:utf8proc_mbtowc}} -->
The `utf8proc_mbtowc` function converts a multibyte UTF-8 sequence to a wide character.
- **Inputs**:
    - `pwc`: A pointer to a `wchar_t` where the resulting wide character will be stored.
    - `s`: A pointer to the input string containing the UTF-8 encoded multibyte sequence.
    - `n`: The maximum number of bytes to read from the input string `s`.
- **Control Flow**:
    - Check if the input string `s` is NULL; if so, return 0 immediately.
    - Call `utf8proc_iterate` to convert the UTF-8 sequence in `s` to a wide character, storing the result in `pwc` and the length of the sequence in `slen`.
    - Check if the conversion resulted in an invalid codepoint (`*pwc == -1`) or an error (`slen < 0`); if so, return -1.
    - If the conversion is successful, return the length of the multibyte sequence (`slen`).
- **Output**:
    - The function returns the length of the multibyte sequence if successful, 0 if the input string is NULL, or -1 if there is an error or invalid codepoint.


---
### utf8proc_wctomb<!-- {{#callable:utf8proc_wctomb}} -->
The `utf8proc_wctomb` function converts a wide character to a multibyte UTF-8 sequence and stores it in a provided buffer.
- **Inputs**:
    - `s`: A pointer to a character buffer where the UTF-8 encoded result will be stored.
    - `wc`: The wide character (wchar_t) to be converted to a UTF-8 multibyte sequence.
- **Control Flow**:
    - Check if the buffer pointer `s` is NULL; if so, return 0 indicating no conversion is performed.
    - Verify if the wide character `wc` is a valid Unicode code point using `utf8proc_codepoint_valid`; if not, return -1 indicating an error.
    - If the character is valid, encode it into the buffer `s` using `utf8proc_encode_char` and return the result of this encoding function.
- **Output**:
    - The function returns 0 if the buffer is NULL, -1 if the wide character is invalid, or the number of bytes written to the buffer if successful.


