# Purpose
The provided C source code file implements a set of functions for visually encoding characters, primarily for the purpose of making non-printable or special characters visible in a human-readable format. This functionality is encapsulated in functions such as [`vis`](#vis), [`strvis`](#strvis), [`strnvis`](#strnvis), [`stravis`](#stravis), and [`strvisx`](#strvisx). The [`vis`](#vis) function is the core utility that encodes a single character based on specified flags, which determine how characters like spaces, tabs, and control characters are represented. The other functions, such as [`strvis`](#strvis) and [`strnvis`](#strnvis), extend this functionality to strings, allowing for the encoding of entire strings while managing buffer sizes and ensuring null-termination.

This code is part of a library intended to be used by other programs that require character encoding capabilities. It provides a public API for encoding strings, which can be useful in scenarios where data needs to be safely displayed or logged without misinterpretation of special characters. The code includes careful handling of buffer sizes and memory allocation, ensuring that the encoded strings do not exceed the allocated space. The use of flags allows for flexible encoding options, such as whether to include escape sequences for certain characters or to encode using octal representations. Overall, this file provides a focused and robust solution for character visibility encoding in C programs.
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
The `vis` function encodes a character into a visually safe representation based on specified flags.
- **Inputs**:
    - `dst`: A pointer to the destination buffer where the encoded character will be stored.
    - `c`: The character to be encoded.
    - `flag`: An integer representing various encoding options and flags.
    - `nextc`: The next character in the sequence, used for certain encoding decisions.
- **Control Flow**:
    - Check if the character `c` is visible using `isvisible(c, flag)`; if so, handle special cases for double quotes and backslashes, then copy `c` to `dst` and return.
    - If the `VIS_CSTYLE` flag is set, encode certain control characters (e.g., newline, carriage return) using C-style escape sequences and return.
    - If the character should be encoded in octal (based on flags or character type), encode it as an octal sequence and return.
    - If the `VIS_NOSLASH` flag is not set, prepend a backslash to the encoding.
    - If the character has the 8th bit set, encode it as a 'M-' sequence and clear the 8th bit.
    - If the character is a control character, encode it using caret notation (e.g., `^A` for control-A); otherwise, encode it as a `-` followed by the character.
    - Terminate the string in `dst` with a null character and return the updated `dst` pointer.
- **Output**:
    - The function returns a pointer to the end of the encoded string in the destination buffer `dst`.


---
### strvis<!-- {{#callable:strvis}} -->
The `strvis` function encodes characters from a source string into a destination string using a visual encoding scheme, based on specified flags.
- **Inputs**:
    - `dst`: A pointer to the destination buffer where the encoded string will be stored.
    - `src`: A pointer to the source string that needs to be encoded.
    - `flag`: An integer representing the encoding flags that dictate how characters should be encoded.
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
The `strnvis` function encodes a source string into a destination buffer with visual representation, ensuring the output does not exceed a specified size and is null-terminated.
- **Inputs**:
    - `dst`: A pointer to the destination buffer where the encoded string will be stored.
    - `src`: A pointer to the source string that needs to be encoded.
    - `siz`: The maximum size of the destination buffer, including space for the null terminator.
    - `flag`: An integer flag that specifies encoding options, such as whether to escape certain characters.
- **Control Flow**:
    - Initialize pointers `start` and `end` to mark the beginning and end of the destination buffer, and a temporary buffer `tbuf` for character encoding.
    - Iterate over each character `c` in the source string `src` until the end of the string or the destination buffer is reached.
    - Check if the character `c` is visible using the `isvisible` function and the provided `flag`.
    - If `c` is visible and requires escaping (e.g., double quotes or backslashes), check if there is enough space in the destination buffer to add an escape character and the character itself; if not, break the loop.
    - If `c` is visible and does not require escaping, copy it directly to the destination buffer and increment the source pointer.
    - If `c` is not visible, use the [`vis`](#vis) function to encode it into `tbuf`, then copy the encoded result to the destination buffer if there is enough space; otherwise, break the loop.
    - Ensure the destination buffer is null-terminated if its size is greater than zero.
    - If the destination buffer was truncated, continue encoding the remaining source characters to calculate the total length needed for full encoding.
    - Return the number of characters written to the destination buffer, excluding the null terminator.
- **Output**:
    - The function returns the number of characters written to the destination buffer, excluding the null terminator, and accounts for any truncation that may have occurred.
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
    - Attempt to reallocate `buf` to fit the exact size of the encoded string plus one for the null terminator, and assign the result to `*outp`.
    - If reallocation fails, restore `errno` to `serrno` and assign `buf` to `*outp` to ensure the caller still receives a valid pointer.
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
    - Enter a loop that continues while `len` is greater than 1, decrementing `len` with each iteration.
    - In each iteration, assign the current character from `src` to `c` and call the [`vis`](#vis) function to encode `c` into `dst`, using the next character in `src` as context for encoding.
    - After the loop, if `len` is still greater than 0, encode the last character from `src` into `dst` using [`vis`](#vis), with a null character as context.
    - Terminate the encoded string in `dst` with a null character.
    - Return the number of bytes written to `dst`, calculated as the difference between the current position of `dst` and the initial position `start`.
- **Output**:
    - The function returns the length of the encoded string in the destination buffer, excluding the null terminator.
- **Functions called**:
    - [`vis`](#vis)


