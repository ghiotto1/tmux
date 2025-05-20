# Purpose
This C source code file provides a specific utility function, [`getline`](#getline), which is used to read a line of text from a file stream. The file is part of the NetBSD project and includes a copyright notice from The NetBSD Foundation, indicating its open-source nature and the conditions under which it can be redistributed. The primary functionality of this file is encapsulated in the [`getdelim`](#getdelim) function, which is a static helper function that reads characters from a file stream until a specified delimiter is encountered, dynamically resizing the buffer as needed to accommodate the input. The [`getline`](#getline) function is a public interface that utilizes [`getdelim`](#getdelim) with the newline character as the delimiter, making it specifically tailored for reading lines of text.

The code is designed to be robust and efficient, handling dynamic memory allocation and resizing to ensure that lines of any length can be read without causing buffer overflows. It checks for end-of-file and error conditions, returning appropriate values to signal success or failure. This file is likely intended to be part of a larger library or application, such as the `tmux` terminal multiplexer, as indicated by the inclusion of "tmux.h". The [`getline`](#getline) function is a common utility in C programming, often used for reading input from files or standard input, and this implementation provides a reliable and flexible version of that functionality.
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
    - If the buffer is about to overflow, double its size using realloc, update pointers accordingly, and continue reading.
- **Output**:
    - The function returns the number of characters read, including the delimiter, or -1 on error or if no characters are read before EOF.


---
### getline<!-- {{#callable:getline}} -->
The `getline` function reads an entire line from a file stream, storing it in a buffer, and returns the number of characters read.
- **Inputs**:
    - `buf`: A pointer to a character pointer where the address of the buffer will be stored; the buffer will contain the line read from the file.
    - `bufsiz`: A pointer to a size_t variable that holds the size of the buffer; it may be updated if the buffer is reallocated.
    - `fp`: A file pointer to the input stream from which the line is to be read.
- **Control Flow**:
    - The function calls [`getdelim`](#getdelim) with the newline character '\n' as the delimiter to read a line from the file stream `fp`.
    - [`getdelim`](#getdelim) checks if the buffer is NULL or of size zero, and if so, allocates an initial buffer of size `BUFSIZ`.
    - It enters a loop to read characters from the file stream one by one using `fgetc`.
    - If `fgetc` returns -1, it checks for end-of-file and returns the number of characters read if any, or -1 on error.
    - Each character read is stored in the buffer, and if the delimiter '\n' is encountered, the function null-terminates the string and returns the number of characters read.
    - If the buffer is about to overflow, it reallocates the buffer to double its current size and continues reading.
- **Output**:
    - The function returns the number of characters read, including the delimiter, or -1 on error or end-of-file without reading any characters.
- **Functions called**:
    - [`getdelim`](#getdelim)


