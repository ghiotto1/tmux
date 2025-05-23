# Purpose
This C source code file provides a portable implementation of the `fgetln()` function, which is used to read a line from a file stream. The function is not reentrant, meaning it is not safe to use in a multi-threaded environment without additional synchronization. The `fgetln()` function reads characters from a given file pointer `fp` until it encounters a newline character or the end of the file. It dynamically allocates and resizes a buffer to store the line, ensuring that it can handle lines of arbitrary length. The function returns a pointer to the buffer containing the line and sets the length of the line in the provided `len` parameter. If an error occurs, such as an invalid file pointer or memory allocation failure, the function returns `NULL` and sets the `errno` to indicate the error.

The code includes necessary headers for standard input/output and memory management functions, and it references a "compat.h" header, suggesting that it may be part of a larger compatibility library. This implementation is designed to be portable across different systems, providing functionality that may not be available in all standard C libraries. The use of static variables for the buffer and its size indicates that the function maintains state between calls, which is why it is not reentrant. This file is likely intended to be part of a library that can be included in other projects to provide consistent line-reading functionality across various platforms.
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
    - If the buffer is full (`r == bufsz`), attempt to double its size using `reallocarray`; if reallocation fails, free the buffer, set `errno`, and return NULL.
    - Break the loop if a newline character is read.
    - Set the length of the read line in `*len` and return the buffer if any characters were read; otherwise, return NULL.
- **Output**:
    - Returns a pointer to the buffer containing the line read from the file, or NULL if an error occurs or EOF is reached without reading any characters.


