# Purpose
The provided C source code file implements a set of functions for visually encoding characters, primarily for the purpose of making non-printable or special characters visible in a human-readable format. This functionality is encapsulated in functions such as [`vis`](#vis), [`strvis`](#strvis), [`strnvis`](#strnvis), [`stravis`](#stravis), and [`strvisx`](#strvisx). The core function, [`vis`](#vis), encodes a single character based on specified flags, which determine how characters like spaces, tabs, newlines, and control characters are represented. The encoding can include C-style escape sequences, octal representations, or other visual markers to ensure that the output is both safe and readable.

The file is part of a library intended to be used by other programs that require character encoding capabilities. It does not define a standalone executable but rather provides a set of utilities that can be integrated into larger applications. The functions handle various encoding scenarios, such as encoding entire strings ([`strvis`](#strvis), [`strnvis`](#strnvis), [`stravis`](#stravis)) or specific lengths of data ([`strvisx`](#strvisx)). The code is designed to be robust, with considerations for buffer sizes and memory allocation, ensuring that the encoded output is correctly terminated and that potential buffer overflows are avoided. This library is particularly useful in environments where data needs to be safely displayed or logged, preserving the integrity of special characters.
# Imports and Dependencies

---
- `sys/types.h`
- `errno.h`
- `ctype.h`
- `limits.h`
- `string.h`
- `stdlib.h`
- `compat.h`


# Functions

---
### vis<!-- {{#callable:vis}} -->
The `vis` function encodes a character into a visually safe representation based on specified flags and stores it in the destination buffer.
- **Inputs**:
    - `dst`: A pointer to the destination buffer where the encoded character will be stored.
    - `c`: The character to be encoded.
    - `flag`: An integer representing various encoding options and flags.
    - `nextc`: The next character in the sequence, used for certain encoding decisions.
- **Control Flow**:
    - Check if the character `c` is visible using `isvisible(c, flag)`; if true, handle special cases for double quotes and backslashes, then store the character in `dst` and return.
    - If the `VIS_CSTYLE` flag is set, encode certain control characters (e.g., newline, carriage return) using C-style escape sequences and store them in `dst`.
    - If the character `c` is a space, or if `VIS_OCTAL` or `VIS_GLOB` flags are set, encode `c` using octal representation and store it in `dst`.
    - If `VIS_NOSLASH` is not set, prepend a backslash to the encoded character.
    - If `c` has the 8th bit set, encode it as a Meta character by stripping the 8th bit and prepending 'M'.
    - If `c` is a control character, encode it using caret notation (e.g., `^A` for control-A).
    - If none of the above conditions apply, encode `c` as a normal character prefixed by a dash.
    - Terminate the string in `dst` with a null character and return the updated `dst` pointer.
- **Output**:
    - A pointer to the end of the encoded string in the destination buffer `dst`.


---
### strvis<!-- {{#callable:strvis}} -->
The `strvis` function encodes characters from a source string into a destination string using a visual encoding scheme, based on specified flags.
- **Inputs**:
    - `dst`: A pointer to the destination buffer where the encoded string will be stored.
    - `src`: A pointer to the source string that will be encoded.
    - `flag`: An integer representing flags that modify the encoding behavior.
- **Control Flow**:
    - Initialize a pointer `start` to the beginning of the destination buffer `dst`.
    - Iterate over each character `c` in the source string `src` until the null terminator is reached.
    - For each character `c`, call the [`vis`](#vis) function to encode it according to the specified `flag` and store the result in `dst`.
    - Advance the `src` pointer to the next character after each call to [`vis`](#vis).
    - After the loop, append a null terminator to the end of the encoded string in `dst`.
    - Return the length of the encoded string, which is the difference between the current position of `dst` and the initial position `start`.
- **Output**:
    - The function returns the length of the encoded string, excluding the null terminator.
- **Functions called**:
    - [`vis`](#vis)


---
### strnvis<!-- {{#callable:strnvis}} -->
The `strnvis` function encodes a string from `src` into `dst` with visual representation, ensuring the output does not exceed `siz-1` bytes, and returns the number of bytes needed to fully encode the string.
- **Inputs**:
    - `dst`: A pointer to the destination buffer where the encoded string will be stored.
    - `src`: A pointer to the source string that needs to be encoded.
    - `siz`: The size of the destination buffer, which limits the number of bytes that can be written to `dst`.
    - `flag`: An integer flag that modifies the encoding behavior, such as handling of special characters.
- **Control Flow**:
    - Initialize pointers `start` and `end` to mark the beginning and end of the destination buffer, and a temporary buffer `tbuf` for encoding.
    - Iterate over each character `c` in `src` while `dst` is within bounds of `end`.
    - Check if the character `c` is visible using `isvisible(c, flag)`.
    - If `c` is visible and requires special handling (e.g., `"` or `\`), ensure there is space for an extra `\` and append it to `dst`.
    - If `c` is visible, append it directly to `dst` and increment `src`.
    - If `c` is not visible, use [`vis`](#vis) to encode it into `tbuf`, then copy `tbuf` to `dst` if space allows, otherwise break the loop.
    - Null-terminate `dst` if `siz` is greater than 0.
    - If the encoding was truncated, continue encoding the remaining characters to adjust the return value for truncation.
- **Output**:
    - The function returns the number of bytes that would have been written to `dst` if there was enough space, not including the null terminator.
- **Functions called**:
    - [`vis`](#vis)


---
### stravis<!-- {{#callable:stravis}} -->
The `stravis` function encodes a source string into a visually encoded format and dynamically allocates memory for the output string.
- **Inputs**:
    - `outp`: A pointer to a char pointer where the address of the newly allocated and encoded string will be stored.
    - `src`: The source string to be visually encoded.
    - `flag`: An integer flag that specifies encoding options for the [`strvis`](#strvis) function.
- **Control Flow**:
    - Allocate memory for `buf` to hold the encoded string, with a size four times the length of `src` plus one for the null terminator.
    - Check if memory allocation for `buf` fails, and return -1 if it does.
    - Call [`strvis`](#strvis) to encode `src` into `buf` using the specified `flag`, and store the length of the encoded string in `len`.
    - Save the current value of `errno` in `serrno` to preserve it across memory operations.
    - Attempt to reallocate `buf` to the exact size needed for the encoded string plus one for the null terminator, and store the result in `*outp`.
    - If reallocation fails, restore `errno` to `serrno` and assign `buf` to `*outp` to ensure the caller still has access to the encoded string.
    - Return the length of the encoded string.
- **Output**:
    - The function returns the length of the visually encoded string, or -1 if memory allocation fails.
- **Functions called**:
    - [`strvis`](#strvis)


---
### strvisx<!-- {{#callable:strvisx}} -->
The `strvisx` function encodes a specified number of bytes from a source string into a destination string using visual encoding rules.
- **Inputs**:
    - `dst`: A pointer to the destination buffer where the encoded string will be stored.
    - `src`: A pointer to the source string that will be encoded.
    - `len`: The number of bytes from the source string to encode.
    - `flag`: An integer flag that specifies encoding options.
- **Control Flow**:
    - Initialize a character pointer `start` to the beginning of the destination buffer `dst`.
    - Iterate over the source string `src` for `len - 1` bytes, encoding each character using the [`vis`](#vis) function and updating the `dst` pointer.
    - If there is one byte left to process (`len` is not zero), encode the last character with a null character as the next character using [`vis`](#vis).
    - Terminate the destination string with a null character.
    - Return the number of bytes written to the destination buffer, excluding the null terminator, by calculating the difference between the current `dst` pointer and the `start` pointer.
- **Output**:
    - The function returns the length of the encoded string in the destination buffer, excluding the null terminator.
- **Functions called**:
    - [`vis`](#vis)


