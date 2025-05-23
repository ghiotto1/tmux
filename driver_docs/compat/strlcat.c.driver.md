# Purpose
The provided C source code file implements the [`strlcat`](#strlcat) function, which is a safer alternative to the standard `strncat` function for string concatenation. This function is designed to append the source string (`src`) to the destination string (`dst`) while ensuring that the total length of the destination string does not exceed the specified buffer size (`siz`). Unlike `strncat`, which requires the size of the remaining space in the destination buffer, [`strlcat`](#strlcat) takes the full size of the destination buffer, making it easier to use correctly and reducing the risk of buffer overflows. The function guarantees that the destination string is always null-terminated unless the buffer size is zero or less than the initial length of the destination string. It returns the total length of the string it tried to create, which is the sum of the initial length of `dst` and the length of `src`. If the return value is greater than or equal to `siz`, it indicates that truncation occurred.

This code is part of a library intended to be used in other C programs, providing a public API for safer string manipulation. The function is implemented in a standalone manner, making it easy to integrate into various projects that require robust string handling. The inclusion of the `compat.h` header suggests that this file might be part of a larger compatibility layer, potentially providing similar safe string functions to enhance portability and security across different systems. The code is distributed under a permissive license, allowing for wide usage and modification, which is typical for utility functions that aim to improve standard library functionality.
# Imports and Dependencies

---
- `sys/types.h`
- `string.h`
- `compat.h`


# Functions

---
### strlcat<!-- {{#callable:strlcat}} -->
The `strlcat` function appends the source string to the destination string, ensuring the result is null-terminated and does not exceed the specified buffer size.
- **Inputs**:
    - `dst`: A pointer to the destination string buffer where the source string will be appended.
    - `src`: A pointer to the source string that will be appended to the destination string.
    - `siz`: The total size of the destination buffer, including space for the null terminator.
- **Control Flow**:
    - Initialize pointers `d` and `s` to `dst` and `src` respectively, and set `n` to `siz`.
    - Iterate through `dst` to find its end, decrementing `n` for each character until `n` is zero or the null terminator is found.
    - Calculate the length of `dst` (`dlen`) and update `n` to reflect the remaining space in the buffer after `dst`.
    - If no space is left (`n == 0`), return the sum of `dlen` and the length of `src`.
    - Iterate through `src`, appending each character to `dst` as long as there is space (`n != 1`), decrementing `n` for each character appended.
    - Terminate the resulting string in `dst` with a null character.
    - Return the total length of the string that would have been created if there was unlimited space, which is `dlen` plus the length of `src`.
- **Output**:
    - The function returns the total length of the string it tried to create, which is the initial length of `dst` plus the length of `src`.


