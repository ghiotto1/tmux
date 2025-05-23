# Purpose
This C source code file implements a custom version of the [`strnlen`](#strnlen) function, which is used to determine the length of a string up to a specified maximum number of characters. The function iterates through the string, counting characters until it either reaches the null terminator or the specified maximum length, ensuring that it does not read beyond the intended limit. This implementation is useful in environments where [`strnlen`](#strnlen) might not be available as a standard library function, providing compatibility across different systems. The file includes necessary headers and a compatibility header, suggesting it is part of a larger codebase that aims to maintain cross-platform functionality. The code is distributed under a permissive license, allowing for broad use and modification.
# Imports and Dependencies

---
- `sys/types.h`
- `string.h`
- `compat.h`


# Functions

---
### strnlen<!-- {{#callable:strnlen}} -->
The `strnlen` function calculates the length of a string up to a maximum specified length.
- **Inputs**:
    - `str`: A pointer to the null-terminated string whose length is to be measured.
    - `maxlen`: The maximum number of characters to examine in the string.
- **Control Flow**:
    - Initialize a pointer `cp` to the start of the string `str`.
    - Enter a loop that continues as long as `maxlen` is not zero and the current character pointed to by `cp` is not the null terminator '\0'.
    - In each iteration of the loop, increment the pointer `cp` and decrement `maxlen`.
    - Exit the loop when either `maxlen` reaches zero or a null terminator is encountered in the string.
- **Output**:
    - The function returns the number of characters in the string up to the first null terminator or `maxlen`, whichever comes first, as a `size_t` type.


