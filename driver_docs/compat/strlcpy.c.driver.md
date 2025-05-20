# Purpose
The provided C source code file implements the [`strlcpy`](#strlcpy) function, which is a safer alternative to the standard `strcpy` function. This function is designed to copy a string from a source (`src`) to a destination (`dst`) buffer, ensuring that the destination buffer is not overflowed. It copies at most `siz-1` characters to the destination buffer and always null-terminates the result, unless the size (`siz`) is zero. The function returns the total length of the string it tried to create, which is the length of `src`. If the return value is greater than or equal to `siz`, it indicates that truncation occurred. This behavior allows the caller to detect truncation by comparing the return value to the size of the destination buffer.

The code is a standalone implementation of the [`strlcpy`](#strlcpy) function, which is often used in environments where buffer overflow vulnerabilities need to be mitigated. It is not part of the standard C library, but it is widely adopted in various systems, particularly in OpenBSD, from which this implementation originates. The file includes necessary headers and a compatibility header (`compat.h`), suggesting it might be part of a larger codebase that ensures compatibility across different systems. The function does not define a public API or external interface beyond the [`strlcpy`](#strlcpy) function itself, focusing solely on providing this specific functionality.
# Imports and Dependencies

---
- `sys/types.h`
- `string.h`
- `compat.h`


# Functions

---
### strlcpy<!-- {{#callable:strlcpy}} -->
The `strlcpy` function copies a string from `src` to `dst` ensuring it is NUL-terminated and returns the length of `src`.
- **Inputs**:
    - `dst`: A pointer to the destination buffer where the string will be copied.
    - `src`: A pointer to the source string to be copied.
    - `siz`: The size of the destination buffer, which determines how many characters can be copied.
- **Control Flow**:
    - Initialize pointers `d` and `s` to `dst` and `src` respectively, and `n` to `siz`.
    - Check if `n` is not zero and decrement `n`. If `n` is still not zero, enter a loop to copy characters from `src` to `dst` until a NUL character is encountered or `n` becomes zero.
    - If `n` becomes zero, check if `siz` is not zero and NUL-terminate `dst`.
    - Continue traversing the rest of `src` to calculate its length.
    - Return the length of `src` excluding the NUL terminator.
- **Output**:
    - The function returns the length of the source string `src`, excluding the NUL terminator.


