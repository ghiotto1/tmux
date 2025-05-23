# Purpose
This C source code file provides a set of memory allocation and string manipulation functions that enhance the standard C library functions by incorporating error checking and handling. The functions defined in this file, such as [`xmalloc`](#xmalloc), [`xcalloc`](#xcalloc), [`xrealloc`](#xrealloc), [`xstrdup`](#xstrdup), and others, are designed to ensure that memory allocation operations do not fail silently. Instead of returning `NULL` on failure, these functions call a `fatalx` function, which is presumably defined elsewhere, to handle errors in a more controlled manner. This approach is particularly useful in applications where memory allocation failures need to be caught and handled immediately to prevent undefined behavior or crashes.

The file is part of a larger project, as indicated by the inclusion of "tmux.h", suggesting it is likely a component of the tmux terminal multiplexer. The functions provided are not intended to be a standalone library but rather to be integrated into the broader application, enhancing its robustness by ensuring that memory-related errors are managed consistently. The file does not define public APIs or external interfaces but instead offers internal utility functions that can be used throughout the application to replace standard memory and string functions with safer alternatives.
# Imports and Dependencies

---
- `errno.h`
- `limits.h`
- `stdint.h`
- `stdio.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Functions

---
### xmalloc<!-- {{#callable:xmalloc}} -->
The `xmalloc` function allocates memory of a specified size and terminates the program if the allocation fails.
- **Inputs**:
    - `size`: The size in bytes of the memory block to allocate.
- **Control Flow**:
    - Check if the requested size is zero; if so, call `fatalx` to terminate the program with an error message.
    - Attempt to allocate memory using `malloc` with the specified size.
    - If `malloc` returns `NULL`, indicating a failure to allocate memory, call `fatalx` with an error message including the size and error string.
    - Return the pointer to the allocated memory.
- **Output**:
    - A pointer to the allocated memory block, or the program is terminated if allocation fails.


---
### xcalloc<!-- {{#callable:xcalloc}} -->
The `xcalloc` function allocates memory for an array of elements, ensuring that the allocation is successful and aborting the program if it fails.
- **Inputs**:
    - `nmemb`: The number of elements to allocate memory for.
    - `size`: The size of each element to allocate.
- **Control Flow**:
    - Check if either `nmemb` or `size` is zero; if so, call `fatalx` to terminate the program with an error message indicating zero size allocation.
    - Call `calloc` to allocate memory for an array of `nmemb` elements, each of `size` bytes.
    - Check if the result of `calloc` is `NULL`, indicating a failure to allocate memory; if so, call `fatalx` to terminate the program with an error message including the number of bytes attempted to allocate and the error string from `strerror(errno)`.
    - Return the pointer to the allocated memory.
- **Output**:
    - A pointer to the allocated memory block, or the program is terminated if allocation fails.


---
### xrealloc<!-- {{#callable:xrealloc}} -->
The `xrealloc` function reallocates memory for a given pointer to a specified size, ensuring that the operation does not fail silently.
- **Inputs**:
    - `ptr`: A pointer to the memory block that needs to be reallocated.
    - `size`: The new size in bytes for the memory block.
- **Control Flow**:
    - The function calls [`xreallocarray`](#xreallocarray) with the provided pointer, a fixed number of elements (1), and the specified size.
    - The [`xreallocarray`](#xreallocarray) function handles the actual reallocation and error checking.
- **Output**:
    - A pointer to the newly allocated memory block, or the function will terminate the program if reallocation fails.
- **Functions called**:
    - [`xreallocarray`](#xreallocarray)


---
### xreallocarray<!-- {{#callable:xreallocarray}} -->
The `xreallocarray` function reallocates memory for an array, ensuring that the operation does not fail by terminating the program on error.
- **Inputs**:
    - `ptr`: A pointer to the previously allocated memory block that needs to be reallocated.
    - `nmemb`: The number of elements in the array to be allocated.
    - `size`: The size of each element in the array.
- **Control Flow**:
    - Check if either `nmemb` or `size` is zero; if so, call `fatalx` to terminate the program with an error message indicating zero size.
    - Call `reallocarray` to attempt to reallocate the memory block pointed to by `ptr` to accommodate `nmemb` elements each of `size` bytes.
    - Check if `reallocarray` returns `NULL`, indicating a failure to allocate memory; if so, call `fatalx` to terminate the program with an error message including the attempted allocation size and the error string from `strerror(errno)`.
    - Return the pointer to the newly allocated memory block.
- **Output**:
    - A pointer to the newly allocated memory block, or the program terminates if allocation fails.


---
### xrecallocarray<!-- {{#callable:xrecallocarray}} -->
The `xrecallocarray` function reallocates memory for an array, ensuring the new memory is zeroed and handling errors by terminating the program if allocation fails.
- **Inputs**:
    - `ptr`: A pointer to the existing memory block to be reallocated.
    - `oldnmemb`: The number of elements in the existing memory block.
    - `nmemb`: The number of elements in the new memory block.
    - `size`: The size of each element in the memory block.
- **Control Flow**:
    - Check if `nmemb` or `size` is zero, and if so, call `fatalx` to terminate the program with an error message.
    - Call `recallocarray` to attempt to reallocate the memory block with the specified number of elements and size, zeroing any new memory.
    - Check if `recallocarray` returns `NULL`, indicating a failure to allocate memory, and if so, call `fatalx` to terminate the program with an error message.
    - Return the pointer to the newly allocated memory block.
- **Output**:
    - A pointer to the newly allocated memory block, or the program terminates if allocation fails.


---
### xstrdup<!-- {{#callable:xstrdup}} -->
The `xstrdup` function duplicates a string and terminates the program if memory allocation fails.
- **Inputs**:
    - `str`: A pointer to the null-terminated string to be duplicated.
- **Control Flow**:
    - Attempt to duplicate the string `str` using `strdup` and assign the result to `cp`.
    - Check if `cp` is `NULL`, indicating a memory allocation failure.
    - If `cp` is `NULL`, call `fatalx` with an error message including the error string from `strerror(errno)`.
    - Return the duplicated string pointer `cp`.
- **Output**:
    - A pointer to the newly allocated duplicate of the input string `str`, or the program terminates if allocation fails.


---
### xstrndup<!-- {{#callable:xstrndup}} -->
The `xstrndup` function duplicates a string up to a specified maximum length and terminates the program if memory allocation fails.
- **Inputs**:
    - `str`: A pointer to the null-terminated string to be duplicated.
    - `maxlen`: The maximum number of characters to duplicate from the string, including the null terminator.
- **Control Flow**:
    - Call the `strndup` function to duplicate the string `str` up to `maxlen` characters.
    - Check if the result of `strndup` is `NULL`, indicating a memory allocation failure.
    - If `strndup` returns `NULL`, call `fatalx` to terminate the program with an error message.
    - Return the duplicated string pointer `cp`.
- **Output**:
    - A pointer to the newly allocated string that is a duplicate of the input string up to `maxlen` characters, or the program terminates if allocation fails.


---
### xasprintf<!-- {{#callable:xasprintf}} -->
The `xasprintf` function formats a string and allocates memory for it, ensuring that memory allocation errors are handled gracefully.
- **Inputs**:
    - `ret`: A pointer to a char pointer where the address of the allocated formatted string will be stored.
    - `fmt`: A format string that specifies how subsequent arguments are converted for output.
    - `...`: A variable number of arguments that are formatted according to the format string.
- **Control Flow**:
    - Initialize a variable argument list using `va_start` with the format string `fmt`.
    - Call [`xvasprintf`](#xvasprintf) with the provided arguments to perform the actual formatted string allocation and assignment.
    - End the variable argument list using `va_end`.
    - Return the result of [`xvasprintf`](#xvasprintf), which is the number of characters printed or a negative value if an error occurred.
- **Output**:
    - The function returns the number of characters printed (excluding the null byte) or a negative value if an error occurs.
- **Functions called**:
    - [`xvasprintf`](#xvasprintf)


---
### xvasprintf<!-- {{#callable:xvasprintf}} -->
The `xvasprintf` function formats a string and allocates memory for it, ensuring that any allocation failure results in program termination.
- **Inputs**:
    - `ret`: A pointer to a char pointer where the formatted string will be stored.
    - `fmt`: A format string that specifies how to format the output.
    - `ap`: A `va_list` object that contains the variable arguments to be formatted.
- **Control Flow**:
    - Call the standard library function `vasprintf` to format the string and allocate memory for it, storing the result in `ret` and returning the number of characters printed.
    - Check if `vasprintf` returned -1, indicating a failure to allocate memory.
    - If there is a failure, call `fatalx` with an error message, which will terminate the program.
    - Return the result of `vasprintf`, which is the number of characters printed (excluding the null byte).
- **Output**:
    - The function returns the number of characters printed (excluding the null byte) or terminates the program if memory allocation fails.


---
### xsnprintf<!-- {{#callable:xsnprintf}} -->
The `xsnprintf` function formats a string and stores it in a buffer, ensuring that the operation does not overflow the buffer size.
- **Inputs**:
    - `str`: A pointer to the buffer where the formatted string will be stored.
    - `len`: The maximum number of bytes to be written to the buffer, including the null terminator.
    - `fmt`: A format string that specifies how to format the data.
    - `...`: A variable number of arguments that are formatted according to the format string.
- **Control Flow**:
    - Initialize a variable argument list `ap` using `va_start`, with `fmt` as the last fixed argument.
    - Call [`xvsnprintf`](#xvsnprintf) with the provided buffer, length, format string, and the initialized variable argument list to perform the formatted output operation.
    - End the use of the variable argument list with `va_end`.
    - Return the result of the [`xvsnprintf`](#xvsnprintf) function call, which is the number of characters written, excluding the null terminator.
- **Output**:
    - The function returns an integer representing the number of characters written to the buffer, excluding the null terminator.
- **Functions called**:
    - [`xvsnprintf`](#xvsnprintf)


---
### xvsnprintf<!-- {{#callable:xvsnprintf}} -->
The `xvsnprintf` function safely formats a string into a buffer, ensuring the buffer size does not exceed `INT_MAX` and that no overflow occurs.
- **Inputs**:
    - `str`: A pointer to the buffer where the formatted string will be stored.
    - `len`: The maximum number of bytes to be written to the buffer, including the null terminator.
    - `fmt`: A format string that specifies how to format the data.
    - `ap`: A `va_list` of arguments to be formatted according to the format string.
- **Control Flow**:
    - Check if the provided length `len` is greater than `INT_MAX`; if so, call `fatalx` to handle the error.
    - Use `vsnprintf` to format the string according to `fmt` and `ap`, storing the result in `str`.
    - Check if the result of `vsnprintf` is negative or greater than or equal to `len`; if so, call `fatalx` to handle the overflow error.
    - Return the number of characters that would have been written if `len` had been sufficiently large, not counting the terminating null byte.
- **Output**:
    - The function returns the number of characters that would have been written to the buffer, excluding the null terminator, or calls `fatalx` if an error occurs.


