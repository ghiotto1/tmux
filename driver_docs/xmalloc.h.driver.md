# Purpose
This C header file, `xmalloc.h`, provides declarations for a set of memory allocation and string manipulation functions that enhance standard library functions by incorporating error checking and handling. The functions, such as [`xmalloc`](#xmalloc), [`xcalloc`](#xcalloc), [`xrealloc`](#xrealloc), and their variants, are designed to never return a failure; instead, they invoke a fatal error handler if an error occurs, ensuring robust memory management. Additionally, the file includes safer string duplication and formatted output functions like [`xstrdup`](#xstrdup), [`xasprintf`](#xasprintf), and [`xsnprintf`](#xsnprintf), which include attributes for format checking and non-null constraints to prevent common programming errors. This header is part of a larger software project, likely related to secure communications, as indicated by the author's note on usage and compatibility with the SSH protocol.
# Global Variables

---
### xmalloc
- **Type**: `function`
- **Description**: `xmalloc` is a function that allocates memory of a specified size and checks the result to ensure it is successful. If the allocation fails, it calls a fatal error handler instead of returning a null pointer.
- **Use**: This function is used to safely allocate memory, ensuring that the program does not proceed with a null pointer in case of allocation failure.


---
### xcalloc
- **Type**: `function pointer`
- **Description**: `xcalloc` is a function pointer that represents a custom implementation of the standard `calloc` function. It is designed to allocate memory for an array of elements, each of a specified size, and initializes all bytes in the allocated storage to zero.
- **Use**: This function is used to allocate and zero-initialize an array, ensuring that memory allocation errors are handled by calling a fatal error function if allocation fails.


---
### xrealloc
- **Type**: `function pointer`
- **Description**: `xrealloc` is a function pointer that points to a function designed to reallocate memory blocks. It takes a pointer to the existing memory block and a size_t value representing the new size of the memory block.
- **Use**: This function is used to resize an existing memory allocation, ensuring that the operation never fails by calling a fatal error handler if an error occurs.


---
### xreallocarray
- **Type**: `function pointer`
- **Description**: `xreallocarray` is a function pointer that points to a function designed to reallocate memory for an array. It takes three parameters: a pointer to the existing memory block, and two `size_t` values representing the number of elements and the size of each element, respectively.
- **Use**: This function is used to resize an existing memory block for an array, ensuring that the new size is sufficient to hold the specified number of elements of the given size.


---
### xrecallocarray
- **Type**: `function pointer`
- **Description**: `xrecallocarray` is a function pointer that points to a function designed to reallocate memory for an array, with the added functionality of zeroing out the newly allocated memory. It takes four parameters: a pointer to the existing memory block, the number of elements in the new array, the size of each element, and the size of the old array. This function is part of a set of memory management utilities that ensure memory allocation operations do not fail silently.
- **Use**: This function is used to safely resize an array in memory, ensuring that any newly allocated space is initialized to zero.


---
### xstrdup
- **Type**: `function`
- **Description**: `xstrdup` is a function that duplicates a given C-style string by allocating memory for the new string and copying the content of the original string into it. It is a safer version of the standard `strdup` function, as it is part of a set of memory allocation functions that check their results and handle errors by calling a fatal error function if necessary.
- **Use**: This function is used to create a duplicate of a string with error checking to ensure memory allocation is successful.


---
### xstrndup
- **Type**: `function`
- **Description**: `xstrndup` is a function that duplicates a specified number of characters from a given string, returning a pointer to the newly allocated string. It is similar to `strndup`, but is part of a custom memory management library that ensures memory allocation errors are handled gracefully.
- **Use**: This function is used to safely duplicate a portion of a string, ensuring that memory allocation is successful or handled appropriately.


# Function Declarations (Public API)

---
### xmalloc<!-- {{#callable_declaration:xmalloc}} -->
Allocates memory and terminates the program on failure.
- **Description**: This function allocates a block of memory of the specified size and returns a pointer to the beginning of the block. It is designed to be used in situations where memory allocation failure is considered fatal, as it will terminate the program if the allocation fails or if a zero size is requested. This function is useful in applications where memory allocation must succeed and the program cannot continue without the requested memory. It should be used with caution in environments where program termination is undesirable.
- **Inputs**:
    - `size`: The number of bytes to allocate. Must be greater than zero. If zero is passed, the function will terminate the program.
- **Output**: A pointer to the allocated memory block. The program will terminate if allocation fails.
- **See also**: [`xmalloc`](xmalloc.c.driver.md#xmalloc)  (Implementation)


---
### xcalloc<!-- {{#callable_declaration:xcalloc}} -->
Allocates and zeroes memory for an array.
- **Description**: This function allocates memory for an array of elements, each of a specified size, and initializes all bytes in the allocated storage to zero. It is intended for use when you need a block of memory that is guaranteed to be initialized to zero. The function will terminate the program if the allocation fails or if either the number of elements or the size of each element is zero, ensuring that it never returns a null pointer.
- **Inputs**:
    - `nmemb`: The number of elements to allocate. Must be greater than zero; otherwise, the program will terminate.
    - `size`: The size of each element. Must be greater than zero; otherwise, the program will terminate.
- **Output**: A pointer to the allocated memory, which is initialized to zero.
- **See also**: [`xcalloc`](xmalloc.c.driver.md#xcalloc)  (Implementation)


---
### xrealloc<!-- {{#callable_declaration:xrealloc}} -->
Reallocate memory to a new size, ensuring success or terminating the program.
- **Description**: Use this function to change the size of a previously allocated memory block. It is particularly useful when you need to expand or shrink a memory allocation without losing the existing data. This function guarantees that the reallocation will succeed; if it cannot, it will terminate the program. This behavior makes it suitable for applications where memory allocation failures are considered fatal and should not be handled by the caller. Ensure that the pointer provided was obtained from a compatible memory allocation function.
- **Inputs**:
    - `ptr`: A pointer to the memory block to be reallocated. This must be a pointer previously obtained from a compatible memory allocation function, such as xmalloc or xrealloc. It can be NULL, in which case the function behaves like xmalloc.
    - `size`: The new size for the memory block in bytes. It must be a positive value. If the size is zero, the behavior is implementation-defined, but typically it may free the memory block.
- **Output**: Returns a pointer to the newly allocated memory block, which is suitably aligned for any kind of variable. The contents of the memory block are unchanged up to the lesser of the new and old sizes. If the function fails to allocate the requested memory, it will terminate the program.
- **See also**: [`xrealloc`](xmalloc.c.driver.md#xrealloc)  (Implementation)


---
### xreallocarray<!-- {{#callable_declaration:xreallocarray}} -->
Reallocate memory for an array, ensuring allocation success.
- **Description**: This function reallocates memory for an array with a specified number of elements and element size, ensuring that the allocation is successful. It is used when you need to resize an existing memory block pointed to by `ptr` to accommodate `nmemb` elements of `size` bytes each. If either `nmemb` or `size` is zero, the function will terminate the program with an error message. Similarly, if the memory allocation fails, the function will also terminate the program with an error message. This function is suitable for use in scenarios where memory allocation failures are considered critical and should not be handled by the caller.
- **Inputs**:
    - `ptr`: A pointer to the memory block to be reallocated. It can be NULL, in which case the function behaves like `xmalloc` for the specified size.
    - `nmemb`: The number of elements to allocate. Must be greater than zero; otherwise, the function will terminate the program.
    - `size`: The size of each element in bytes. Must be greater than zero; otherwise, the function will terminate the program.
- **Output**: A pointer to the newly allocated memory block, which is guaranteed to be non-NULL.
- **See also**: [`xreallocarray`](xmalloc.c.driver.md#xreallocarray)  (Implementation)


---
### xrecallocarray<!-- {{#callable_declaration:xrecallocarray}} -->
Reallocate and zero-initialize an array with error checking.
- **Description**: This function reallocates an array to a new size, ensuring that any newly allocated memory is zero-initialized. It is designed to be used when you need to change the size of an existing memory block while maintaining the contents of the original block and zeroing any additional memory. The function will terminate the program if the allocation fails or if either the number of elements or the size of each element is zero, ensuring that it never returns a null pointer. This function is suitable for use in environments where memory allocation failures are considered fatal errors.
- **Inputs**:
    - `ptr`: A pointer to the memory block to be reallocated. This pointer must have been previously allocated by a compatible memory allocation function. The caller retains ownership and must ensure it is not null unless both `oldnmemb` and `nmemb` are zero.
    - `oldnmemb`: The number of elements in the existing memory block. Must be a non-negative value.
    - `nmemb`: The desired number of elements in the new memory block. Must be greater than zero, otherwise the function will terminate the program.
    - `size`: The size of each element in bytes. Must be greater than zero, otherwise the function will terminate the program.
- **Output**: A pointer to the newly allocated memory block, which is zero-initialized beyond the original size. The function will not return a null pointer.
- **See also**: [`xrecallocarray`](xmalloc.c.driver.md#xrecallocarray)  (Implementation)


---
### xstrdup<!-- {{#callable_declaration:xstrdup}} -->
Duplicates a string with error handling.
- **Description**: This function creates a duplicate of the given string, ensuring that memory allocation is successful. It should be used when a copy of a string is needed and the program cannot continue if memory allocation fails. The function will terminate the program with an error message if it cannot allocate the necessary memory, so it is suitable for use in applications where such a failure is considered fatal.
- **Inputs**:
    - `str`: A pointer to a null-terminated string that is to be duplicated. Must not be null, as passing a null pointer will result in undefined behavior. The caller retains ownership of the original string.
- **Output**: Returns a pointer to the newly allocated string, which is a duplicate of the input string. The caller is responsible for freeing this memory when it is no longer needed.
- **See also**: [`xstrdup`](xmalloc.c.driver.md#xstrdup)  (Implementation)


---
### xstrndup<!-- {{#callable_declaration:xstrndup}} -->
Duplicates a string up to a specified maximum length, ensuring successful allocation.
- **Description**: This function creates a duplicate of the input string, copying at most 'maxlen' characters, and ensures that memory allocation is successful. It is useful when you need a copy of a string with a limited length, and you want to avoid handling allocation failures manually. The function will terminate the program if memory allocation fails, so it should be used in contexts where such behavior is acceptable. The resulting string is null-terminated, and the caller is responsible for freeing the allocated memory.
- **Inputs**:
    - `str`: The input string to be duplicated. Must not be null, as the function does not handle null pointers and will result in undefined behavior if null.
    - `maxlen`: The maximum number of characters to duplicate from the input string. Must be a non-negative value. If 'maxlen' is greater than the length of 'str', the entire string is duplicated.
- **Output**: Returns a pointer to the newly allocated string, which is a duplicate of the input string up to 'maxlen' characters. The caller is responsible for freeing this memory.
- **See also**: [`xstrndup`](xmalloc.c.driver.md#xstrndup)  (Implementation)


---
### xasprintf<!-- {{#callable_declaration:xasprintf}} -->
Formats a string and allocates memory for it.
- **Description**: This function formats a string according to the specified format string and additional arguments, allocating memory for the resulting string. It is useful when you need a formatted string but do not know the required buffer size in advance. The function ensures that memory allocation is successful, and it will not return a failure. It must be called with a valid format string, and the caller is responsible for freeing the allocated memory when it is no longer needed.
- **Inputs**:
    - `ret`: A pointer to a char pointer where the address of the allocated string will be stored. Must not be null. The caller is responsible for freeing the allocated memory.
    - `fmt`: A format string as in printf. Must not be null. It specifies how subsequent arguments are converted for output.
    - `...`: Additional arguments to be formatted according to the format string. The number and types of these arguments must match the format specifiers in the format string.
- **Output**: Returns the number of characters printed, excluding the null byte used to end output to strings.
- **See also**: [`xasprintf`](xmalloc.c.driver.md#xasprintf)  (Implementation)


---
### xvasprintf<!-- {{#callable_declaration:xvasprintf}} -->
Formats a string and allocates memory for it, ensuring successful allocation.
- **Description**: This function formats a string according to the specified format string and variable argument list, allocating memory for the resulting string. It is used when you need to create a formatted string with dynamic content, and it ensures that memory allocation is successful by terminating the program if an error occurs. This function is particularly useful in environments where memory allocation failures must be handled by aborting the operation. It should be called with a valid format string and a corresponding list of arguments.
- **Inputs**:
    - `ret`: A pointer to a char pointer where the address of the allocated string will be stored. Must not be null. The caller is responsible for freeing the allocated memory.
    - `fmt`: A format string that specifies how to format the variable arguments. Must not be null.
    - `ap`: A va_list representing the variable arguments to be formatted according to the format string. Must be properly initialized before calling this function.
- **Output**: Returns the number of characters in the formatted string, excluding the null terminator. If memory allocation fails, the function does not return, as it terminates the program.
- **See also**: [`xvasprintf`](xmalloc.c.driver.md#xvasprintf)  (Implementation)


---
### xsnprintf<!-- {{#callable_declaration:xsnprintf}} -->
Formats a string and stores it in a buffer with size checking.
- **Description**: This function formats a string according to the specified format string and additional arguments, storing the result in the provided buffer. It is used when you need to safely format a string with variable content, ensuring that the buffer is not overflowed. The function must be called with a valid format string and a buffer that is large enough to hold the resulting formatted string, including the null terminator. It is important to ensure that the buffer size is correctly specified to prevent truncation of the output.
- **Inputs**:
    - `str`: A pointer to the buffer where the formatted string will be stored. Must not be null and should have enough space to hold the formatted string and a null terminator.
    - `len`: The size of the buffer pointed to by str. Must be greater than zero to ensure space for the null terminator.
    - `fmt`: A format string that specifies how to format the additional arguments. Must not be null and should follow printf-style formatting.
    - `...`: Additional arguments that are formatted according to the format string. The number and types of these arguments must match the format specifiers in fmt.
- **Output**: Returns the number of characters that would have been written if the buffer had been sufficiently large, not counting the terminating null character. If the output was truncated due to the buffer size, the return value is the number of characters (excluding the null terminator) that would have been written.
- **See also**: [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)  (Implementation)


---
### xvsnprintf<!-- {{#callable_declaration:xvsnprintf}} -->
Formats a string into a buffer using a variable argument list.
- **Description**: This function formats a string according to a specified format and stores the result in a provided buffer. It is used when you have a variable argument list that needs to be formatted into a string. The function ensures that the formatted string does not exceed the specified buffer length, and it will terminate the program if an overflow is detected or if the buffer length exceeds INT_MAX. This function should be used when you need a safe alternative to vsnprintf that checks for buffer overflows and handles them by terminating the program.
- **Inputs**:
    - `str`: A pointer to the buffer where the formatted string will be stored. The buffer must be large enough to hold the resulting string and the null terminator. The caller retains ownership and must ensure the buffer is valid.
    - `len`: The maximum number of bytes to write to the buffer, including the null terminator. Must not exceed INT_MAX. If len is greater than INT_MAX, the function will terminate the program.
    - `fmt`: A format string that specifies how to format the variable argument list. Must not be null. The format string follows the same specifications as printf.
    - `ap`: A va_list representing the variable arguments to be formatted. The caller is responsible for initializing and managing the va_list.
- **Output**: Returns the number of characters that would have been written if len had been sufficiently large, not counting the terminating null character. If the return value is negative or greater than or equal to len, the function will terminate the program.
- **See also**: [`xvsnprintf`](xmalloc.c.driver.md#xvsnprintf)  (Implementation)


