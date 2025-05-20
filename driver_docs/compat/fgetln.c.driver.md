# Purpose
This C source code file provides a portable implementation of the `fgetln()` function, which is not reentrant. The function reads a line from a given file stream (`FILE *fp`) and returns a pointer to the line, while also setting the length of the line in the provided `size_t *len` variable. The implementation uses a static buffer to store the line, dynamically resizing it as needed using `reallocarray()` to accommodate longer lines. This approach ensures that the function can handle lines of arbitrary length, limited only by available memory. The function is designed to be used in environments where `fgetln()` is not natively available, offering a compatible alternative.

The code is structured to handle errors gracefully, setting `errno` appropriately when invalid arguments are provided or when memory allocation fails. The use of static variables for the buffer and its size means that the function is not thread-safe, as concurrent calls could lead to data races. This implementation is intended to be included in larger projects where a portable line-reading function is required, particularly in systems that do not support `fgetln()` natively. The file does not define a public API or external interface beyond the `fgetln()` function itself, focusing on providing a specific utility function for file handling.
# Imports and Dependencies

---
- `stdio.h`
- `stdlib.h`
- `errno.h`
- `compat.h`


# Functions

---
### fgetln<!-- {{#callable:fgetln}} -->
The `fgetln` function reads a line from a file stream into a dynamically allocated buffer, returning the line and its length.
- **Inputs**:
    - `fp`: A pointer to a FILE object that specifies the input stream to read from.
    - `len`: A pointer to a size_t variable where the length of the read line will be stored.
- **Control Flow**:
    - Check if the file pointer `fp` or the length pointer `len` is NULL; if so, set `errno` to `EINVAL` and return NULL.
    - Initialize a static buffer `buf` and its size `bufsz` if they are not already initialized.
    - Read characters from the file stream one by one using `getc(fp)` until EOF or a newline character is encountered.
    - Store each character in the buffer `buf`, incrementing the index `r`.
    - If the buffer is full (`r == bufsz`), attempt to double its size using [`reallocarray`](reallocarray.c.driver.md#reallocarray); if reallocation fails, free the buffer, set `errno`, and return NULL.
    - Break the loop if a newline character is read.
    - Set the length of the read line to `r` and return the buffer if any characters were read, otherwise return NULL.
- **Output**:
    - Returns a pointer to the buffer containing the line read from the file, or NULL if an error occurs or no characters are read.
- **Functions called**:
    - [`reallocarray`](reallocarray.c.driver.md#reallocarray)


