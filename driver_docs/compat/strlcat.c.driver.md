# Purpose
The provided C source code file implements the [`strlcat`](#strlcat) function, which is a safer alternative to the standard `strncat` function for string concatenation. This function is designed to append the source string (`src`) to the destination string (`dst`) while ensuring that the total length of the destination string does not exceed the specified buffer size (`siz`). Unlike `strncat`, which requires the space left in the buffer, [`strlcat`](#strlcat) takes the full size of the destination buffer, making it easier to use and less error-prone. The function guarantees that the destination string is always null-terminated unless the buffer size is zero or less than the length of the initial destination string. It returns the total length of the string it tried to create, which is the sum of the initial length of `dst` and the length of `src`. If the return value is greater than or equal to `siz`, it indicates that truncation occurred.

This code is part of a library intended to be used in other C programs, providing a more robust and secure way to handle string concatenation. It is not an executable on its own but rather a utility function that can be included in other projects to enhance string handling capabilities. The function is implemented with careful attention to buffer boundaries, which helps prevent common security vulnerabilities such as buffer overflows. The inclusion of a permissive license allows for wide usage and modification, making it a valuable addition to any C programmer's toolkit.
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
    - Iterate through `dst` to find its end, decrementing `n` until `n` is zero or the end of `dst` is reached.
    - Calculate the length of `dst` (`dlen`) and update `n` to reflect the remaining space in the buffer.
    - If no space is left (`n == 0`), return the sum of `dlen` and the length of `src`.
    - Iterate through `src`, appending characters to `dst` while there is space (`n != 1`), decrementing `n` with each character appended.
    - Null-terminate the resulting string in `dst`.
    - Return the total length of the string that would have been created if there was unlimited space, which is `dlen` plus the length of `src`.
- **Output**:
    - The function returns the total length of the string it tried to create, which is the initial length of `dst` plus the length of `src`; if the return value is greater than or equal to `siz`, it indicates that truncation occurred.


