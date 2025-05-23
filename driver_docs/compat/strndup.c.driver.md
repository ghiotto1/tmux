# Purpose
This C source code file provides an implementation of the [`strndup`](#strndup) function, which is a utility function used to duplicate a string up to a specified maximum length. The function takes a pointer to a string `str` and a `size_t` value `maxlen`, determining the maximum number of characters to duplicate. It uses `strnlen` to calculate the actual length to copy, allocates memory for the new string using `malloc`, and then copies the content using `memcpy`, ensuring the new string is null-terminated. This implementation is useful in environments where [`strndup`](#strndup) is not available in the standard library, providing a safe way to duplicate strings with a length constraint. The file includes necessary headers for memory management and string operations, and it is distributed under a permissive license, allowing for broad use and modification.
# Imports and Dependencies

---
- `sys/types.h`
- `stddef.h`
- `stdlib.h`
- `string.h`
- `compat.h`


# Functions

---
### strndup<!-- {{#callable:strndup}} -->
The `strndup` function duplicates a string up to a specified maximum length, allocating memory for the new string.
- **Inputs**:
    - `str`: A pointer to the null-terminated string to be duplicated.
    - `maxlen`: The maximum number of characters to duplicate from the input string.
- **Control Flow**:
    - Calculate the length of the string to be duplicated using `strnlen`, which is the lesser of the string's length or `maxlen`.
    - Allocate memory for the new string, including space for the null terminator, using `malloc`.
    - If memory allocation is successful, copy the string up to the calculated length using `memcpy` and append a null terminator.
    - Return the pointer to the newly allocated and duplicated string.
- **Output**:
    - A pointer to the newly allocated string that is a duplicate of the input string up to `maxlen` characters, or NULL if memory allocation fails.


