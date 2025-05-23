# Purpose
The provided C source code file implements a function named [`memmem`](#memmem), which is designed to locate the first occurrence of a byte string `s` within another byte string `l`. This function is a utility that extends the standard C library's string manipulation capabilities by providing a way to search for a substring within a larger string, similar to the `strstr` function but for arbitrary byte sequences. The function takes four parameters: a pointer to the larger byte string `l`, its length `l_len`, a pointer to the substring `s`, and its length `s_len`. It returns a pointer to the first occurrence of `s` within `l`, or `NULL` if `s` is not found.

The code is structured to handle several edge cases efficiently. It immediately returns the haystack if the needle's length is zero, and it returns `NULL` if the needle is longer than the haystack. For single-byte needles, it leverages the `memchr` function for optimized searching. The function iterates through the possible starting positions in `l` where `s` could fit, using `memcmp` to check for a match. This implementation is a standalone utility function, likely intended to be part of a larger library or application, providing a specific and narrow functionality focused on byte string searching. The inclusion of a copyright notice and redistribution conditions suggests that this code is intended for use in open-source projects, adhering to the specified licensing terms.
# Imports and Dependencies

---
- `string.h`
- `compat.h`


# Functions

---
### memmem<!-- {{#callable:memmem}} -->
The `memmem` function searches for the first occurrence of a byte string `s` within another byte string `l`.
- **Inputs**:
    - `l`: A pointer to the byte string (haystack) in which to search.
    - `l_len`: The length of the byte string `l`.
    - `s`: A pointer to the byte string (needle) to search for within `l`.
    - `s_len`: The length of the byte string `s`.
- **Control Flow**:
    - Check if `s_len` is zero; if so, return the start of `l` as the needle is considered found at the beginning.
    - Check if `l_len` is less than `s_len`; if so, return NULL as the needle cannot fit in the haystack.
    - If `s_len` is 1, use `memchr` to find the single character in `l`.
    - Calculate the last possible starting position in `l` where `s` can fit.
    - Iterate over `l` from the start to the last possible position, checking if the current position matches the start of `s` and if the subsequent bytes match using `memcmp`.
    - Return the current position if a match is found, otherwise return NULL.
- **Output**:
    - A pointer to the first occurrence of `s` in `l`, or NULL if `s` is not found.


