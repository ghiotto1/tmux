# Purpose
This C source code file provides a specific functionality for reading lines from a file stream, implementing a custom version of the [`getline`](#getline) function. The file includes the implementation of two functions: [`getdelim`](#getdelim) and [`getline`](#getline). The [`getdelim`](#getdelim) function is a static helper function that reads characters from a given file stream (`FILE *fp`) until a specified delimiter is encountered, dynamically resizing the buffer as needed to accommodate the input. The [`getline`](#getline) function is a public interface that utilizes [`getdelim`](#getdelim) to read a line from the file stream, using the newline character (`'\n'`) as the delimiter. This implementation is particularly useful in environments where the standard [`getline`](#getline) function may not be available, providing similar functionality by reading input line-by-line and managing memory allocation for the buffer.

The file is part of the NetBSD project, as indicated by the comments and versioning information at the top, and it is intended to be used as part of a larger software system, potentially the `tmux` terminal multiplexer, as suggested by the inclusion of "tmux.h". The code is designed to be portable and robust, handling memory allocation and reallocation to ensure that lines of any length can be read from a file. The presence of copyright and licensing information indicates that the code is open-source, with permissions for redistribution and modification under certain conditions. This file is a utility component that can be integrated into other C programs to provide enhanced file reading capabilities.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `stdio.h`
- `unistd.h`
- `errno.h`
- `string.h`
- `tmux.h`


# Functions

---
### getdelim<!-- {{#callable:getdelim}} -->
The `getdelim` function reads characters from a file stream into a buffer until a specified delimiter is encountered, dynamically resizing the buffer as needed.
- **Inputs**:
    - `buf`: A pointer to a character pointer where the function will store the address of the buffer containing the read line.
    - `bufsiz`: A pointer to a size_t variable that holds the current size of the buffer and will be updated if the buffer is resized.
    - `delimiter`: An integer representing the character delimiter that determines the end of the line to be read.
    - `fp`: A pointer to a FILE object that specifies the input stream from which characters are read.
- **Control Flow**:
    - Check if the buffer is NULL or its size is zero; if so, allocate an initial buffer of size BUFSIZ.
    - Enter an infinite loop to read characters from the file stream one by one using fgetc.
    - If fgetc returns -1, check if the end of the file is reached; if so, return the number of characters read if any, otherwise return -1.
    - Store each read character into the buffer and check if it matches the delimiter; if it does, null-terminate the buffer and return the number of characters read.
    - If the buffer is close to being full, double its size using realloc and adjust pointers accordingly.
    - Continue reading characters until the delimiter is found or an error occurs.
- **Output**:
    - The function returns the number of characters read, including the delimiter, or -1 if an error occurs or if the end of the file is reached without reading any characters.


---
### getline<!-- {{#callable:getline}} -->
The `getline` function reads an entire line from a file stream, storing it in a buffer, and returns the number of characters read.
- **Inputs**:
    - `buf`: A pointer to a character pointer where the address of the buffer will be stored.
    - `bufsiz`: A pointer to a size_t variable that holds the size of the buffer.
    - `fp`: A file pointer to the input stream from which the line is to be read.
- **Control Flow**:
    - The function calls [`getdelim`](#getdelim) with the newline character '\n' as the delimiter.
    - [`getdelim`](#getdelim) checks if the buffer is NULL or of size zero, and allocates memory if necessary.
    - It reads characters from the file stream one by one using `fgetc`.
    - If the end-of-file is reached and some characters have been read, it null-terminates the string and returns the number of characters read.
    - If a newline character is encountered, it null-terminates the string and returns the number of characters read.
    - If the buffer is about to overflow, it reallocates the buffer to double its current size.
- **Output**:
    - The function returns the number of characters read, including the delimiter, or -1 on error or end-of-file without reading any characters.
- **Functions called**:
    - [`getdelim`](#getdelim)


