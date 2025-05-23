# Purpose
The provided C source code file implements the [`strlcpy`](#strlcpy) function, which is a safer alternative to the standard `strcpy` function. This function is designed to copy a string from a source (`src`) to a destination (`dst`) buffer, ensuring that the destination buffer is not overflowed. It copies up to `siz-1` characters from the source to the destination and always null-terminates the destination string, unless the size (`siz`) is zero. The function returns the total length of the source string, which allows the caller to detect if truncation occurred by comparing the return value to the size of the destination buffer.

The [`strlcpy`](#strlcpy) function is a part of the OpenBSD project and is often used in environments where buffer overflow vulnerabilities need to be mitigated. It is a single, focused utility function that provides a narrow but crucial functionality in string handling, particularly in C programming where manual memory management is required. The file includes necessary headers and a compatibility header, suggesting it might be part of a larger library or system where cross-platform compatibility is considered. The function does not define a public API or external interface by itself but is intended to be used as a safer string copying utility within larger applications or libraries.
# Imports and Dependencies

---
- `sys/types.h`
- `string.h`
- `compat.h`


# Functions

---
### strlcpy<!-- {{#callable:strlcpy}} -->
The `strlcpy` function copies a string from `src` to `dst` ensuring that the destination buffer is NUL-terminated and does not overflow, returning the length of the source string.
- **Inputs**:
    - `dst`: A pointer to the destination buffer where the string will be copied.
    - `src`: A pointer to the source string to be copied.
    - `siz`: The size of the destination buffer, including space for the NUL terminator.
- **Control Flow**:
    - Initialize pointers `d` and `s` to `dst` and `src` respectively, and `n` to `siz`.
    - Check if `n` is not zero and decrement `n` by one; if true, enter a loop to copy characters from `src` to `dst` until a NUL character is encountered or `n` becomes zero.
    - If `n` becomes zero, check if `siz` is not zero and NUL-terminate `dst`.
    - Continue traversing the rest of `src` to calculate its length.
    - Return the length of `src` minus one, which excludes the NUL terminator.
- **Output**:
    - The function returns the length of the source string `src`, not including the NUL terminator, which allows the caller to detect if truncation occurred by comparing the return value to `siz`.


