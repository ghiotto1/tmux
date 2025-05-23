# Purpose
This C source code file provides an implementation of the [`strnlen`](#strnlen) function, which is a utility function used to determine the length of a string up to a specified maximum number of characters. The function iterates through the string, counting characters until it reaches either the null terminator or the specified maximum length, ensuring that it does not read beyond the intended limit. This implementation is useful in environments where [`strnlen`](#strnlen) is not available as part of the standard library, providing compatibility across different systems. The file includes necessary headers and a custom "compat.h" header, suggesting it may be part of a larger compatibility layer for systems lacking certain standard functions. The code is distributed under a permissive license, allowing for broad use and modification.
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
    - Exit the loop when either `maxlen` reaches zero or a null terminator is encountered.
    - Calculate the length of the string by subtracting the original pointer `str` from the current pointer `cp`.
- **Output**:
    - The function returns the length of the string as a `size_t` value, which is the number of characters in the string up to the specified `maxlen` or the null terminator, whichever comes first.


