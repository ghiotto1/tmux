# Purpose
This C header file, `compat.h`, serves as a compatibility layer to ensure that a program can be compiled and run across different systems and environments, particularly those with varying levels of support for certain functions and libraries. It includes conditional inclusion of system headers and defines macros and functions that may be missing on some platforms, such as [`strlcpy`](#strlcpy), [`strlcat`](#strlcat), and [`clock_gettime`](#clock_gettime). The file also provides alternative implementations for functions and features that are not available in older or non-standard environments, such as `flock`, [`daemon`](#daemon), and [`getpeereid`](#getpeereid). Additionally, it includes compatibility definitions for various system constants and macros, ensuring that the program can handle differences in system capabilities and configurations. This approach allows the software to maintain portability and functionality across diverse operating systems, including those that do not natively support certain POSIX or BSD-specific features.
# Imports and Dependencies

---
- `sys/types.h`
- `sys/ioctl.h`
- `sys/uio.h`
- `fnmatch.h`
- `limits.h`
- `stdio.h`
- `termios.h`
- `wchar.h`
- `event2/event.h`
- `event2/event_compat.h`
- `event2/event_struct.h`
- `event2/buffer.h`
- `event2/buffer_compat.h`
- `event2/bufferevent.h`
- `event2/bufferevent_struct.h`
- `event2/bufferevent_compat.h`
- `event.h`
- `malloc.h`
- `utf8proc.h`
- `sys/filio.h`
- `err.h`
- `paths.h`
- `stdint.h`
- `inttypes.h`
- `sys/queue.h`
- `compat/queue.h`
- `sys/tree.h`
- `compat/tree.h`
- `bitstring.h`
- `compat/bitstring.h`
- `libutil.h`
- `pty.h`
- `util.h`
- `vis.h`
- `compat/vis.h`
- `imsg.h`
- `compat/imsg.h`


# Global Variables

---
### strcasestr
- **Type**: `function pointer`
- **Description**: The `strcasestr` variable is a function pointer that points to a function which searches for the first occurrence of a substring in a string, ignoring the case of the characters. It is defined as a global variable to provide compatibility for systems that do not have the `strcasestr` function natively available.
- **Use**: This variable is used to perform case-insensitive substring searches in strings.


---
### strsep
- **Type**: `function pointer`
- **Description**: The `strsep` variable is a function pointer to a function that takes two arguments: a pointer to a character pointer and a constant character pointer. It returns a character pointer. This function is used to split a string into tokens based on a set of delimiter characters.
- **Use**: This function is used to tokenize a string by modifying the input string and returning pointers to each token.


---
### strndup
- **Type**: `function pointer`
- **Description**: The `strndup` variable is a function pointer that points to a function which duplicates a specified number of characters from a given string. It takes two parameters: a constant character pointer to the source string and a size_t value indicating the number of characters to duplicate.
- **Use**: This function is used to create a new string that is a duplicate of the first 'n' characters of the given source string.


---
### memmem
- **Type**: `function pointer`
- **Description**: The `memmem` variable is a function pointer that represents a function used to locate a sequence of bytes within another sequence of bytes. It takes four parameters: two pointers to the memory areas to be searched and their respective sizes.
- **Use**: This function is used to find a specific byte sequence within a larger memory block.


---
### getprogname
- **Type**: `const char*`
- **Description**: The `getprogname` function is a global function that returns a constant character pointer. It is used to retrieve the name of the program currently being executed. This function is typically used in environments where the program name is needed for logging, error messages, or other identification purposes.
- **Use**: This function is used to obtain the name of the currently running program.


---
### fgetln
- **Type**: `char *`
- **Description**: The `fgetln` function is a global function that reads a line from a file stream and returns a pointer to the line read, along with its length. It is typically used to read lines from a file until EOF is reached.
- **Use**: This function is used to read lines from a file stream, returning a pointer to the line and its length.


---
### reallocarray
- **Type**: `function pointer`
- **Description**: The `reallocarray` is a function pointer that points to a function which reallocates memory for an array. It takes three parameters: a pointer to the existing memory block, and two `size_t` values representing the number of elements and the size of each element, respectively.
- **Use**: This function is used to dynamically resize an array in memory, ensuring that the total size does not overflow.


---
### recallocarray
- **Type**: `function pointer`
- **Description**: The `recallocarray` is a function pointer that points to a function which reallocates memory for an array, changing its size and initializing any newly allocated memory to zero. It takes four parameters: a pointer to the existing memory block, the old number of elements, the new number of elements, and the size of each element.
- **Use**: This function is used to resize an array while ensuring that any newly allocated memory is zeroed out.


---
### BSDopterr
- **Type**: `int`
- **Description**: `BSDopterr` is a global integer variable used in the context of command-line option parsing. It is part of a set of variables that mimic the behavior of the standard getopt function, but with a BSD-specific implementation.
- **Use**: `BSDopterr` is used to control whether error messages are printed by the BSDgetopt function when an unrecognized option is encountered.


---
### BSDoptind
- **Type**: `int`
- **Description**: `BSDoptind` is an external integer variable used in the context of command-line argument parsing. It is part of a set of variables that mimic the behavior of the standard `getopt` function, specifically for BSD systems.
- **Use**: `BSDoptind` is used to track the index of the next element to be processed in the command-line argument list.


---
### BSDoptopt
- **Type**: `int`
- **Description**: `BSDoptopt` is an external integer variable used in the context of command-line option parsing. It is part of a set of variables that mimic the behavior of the standard getopt function, but with a BSD-specific implementation.
- **Use**: This variable is used to store the last known option character that caused an error during option parsing.


---
### BSDoptreset
- **Type**: `int`
- **Description**: `BSDoptreset` is an external integer variable used in the context of command-line option parsing. It is part of a set of variables that are used to manage the state of option parsing functions, specifically in a BSD-compatible `getopt` implementation.
- **Use**: `BSDoptreset` is used to reset the state of the `getopt` function, allowing for re-parsing of command-line arguments.


---
### BSDoptarg
- **Type**: `char*`
- **Description**: `BSDoptarg` is a global variable of type `char*` that is used to store the argument value of an option processed by the `BSDgetopt` function. It is part of a set of variables that mimic the behavior of the standard `getopt` function, but with a BSD-specific implementation.
- **Use**: `BSDoptarg` is used to hold the argument value associated with an option when parsing command-line arguments using the `BSDgetopt` function.


# Function Declarations (Public API)

---
### err<!-- {{#callable_declaration:err}} -->
Prints an error message and terminates the program.
- **Description**: This function is used to report an error by printing a formatted message to the standard error stream, followed by a description of the current error based on the global `errno` variable. It then terminates the program with the specified exit status. This function should be called when a critical error occurs that requires the program to exit immediately. The function does not return, and any resources not explicitly released before calling this function will not be freed.
- **Inputs**:
    - `eval`: The exit status with which the program will terminate. It should be a non-negative integer.
    - `fmt`: A format string for the error message, similar to those used in `printf`. It can be `NULL` if no additional message is needed. If non-NULL, it must be a valid format string.
    - `...`: Additional arguments that correspond to the format specifiers in `fmt`. These are optional and only required if `fmt` contains format specifiers.
- **Output**: None
- **See also**: [`err`](compat/err.c.driver.md#err)  (Implementation)


---
### errx<!-- {{#callable_declaration:errx}} -->
Prints an error message and terminates the program.
- **Description**: Use this function to print a formatted error message to the standard error stream and terminate the program with a specified exit status. It is typically used to handle fatal errors where the program cannot continue execution. The function appends a newline to the error message and includes the program name at the beginning of the message. It is important to note that this function does not return, as it calls `exit` with the provided exit status.
- **Inputs**:
    - `eval`: The exit status with which the program will terminate. It should be a valid integer representing the desired exit code.
    - `fmt`: A format string for the error message, similar to `printf`. It can be `NULL`, in which case no additional message is printed beyond the program name and newline. The caller retains ownership of the string.
    - `...`: A variable number of arguments that correspond to the format specifiers in `fmt`. These are used to format the error message.
- **Output**: None
- **See also**: [`errx`](compat/err.c.driver.md#errx)  (Implementation)


---
### warn<!-- {{#callable_declaration:warn}} -->
Logs a warning message to the standard error stream.
- **Description**: This function is used to log a warning message to the standard error stream, typically to inform the user of a non-critical error or issue that has occurred. It prefixes the message with the program name and appends a description of the current error based on the global errno variable. The function can be called with a format string and additional arguments similar to printf, allowing for formatted output. If the format string is null, only the error description is printed. This function is useful for debugging and logging purposes in applications where error reporting is necessary.
- **Inputs**:
    - `fmt`: A format string similar to printf, which specifies how subsequent arguments are converted for output. It can be null, in which case only the error description is printed. The caller retains ownership of the string.
- **Output**: None
- **See also**: [`warn`](compat/err.c.driver.md#warn)  (Implementation)


---
### warnx<!-- {{#callable_declaration:warnx}} -->
Prints a formatted warning message to standard error.
- **Description**: Use this function to display a warning message to the standard error stream, typically for logging or debugging purposes. The message is prefixed with the program name and followed by a newline. It accepts a format string and a variable number of arguments similar to printf, allowing for formatted output. This function should be called when a non-fatal error or warning needs to be communicated to the user or logged. Ensure that the format string is not null to avoid undefined behavior.
- **Inputs**:
    - `fmt`: A format string similar to printf, which specifies how subsequent arguments are converted for output. It can be null, in which case no additional message is printed beyond the program name and newline.
- **Output**: None
- **See also**: [`warnx`](compat/err.c.driver.md#warnx)  (Implementation)


---
### explicit_bzero<!-- {{#callable_declaration:explicit_bzero}} -->
Zeroes out a specified memory buffer.
- **Description**: This function is used to securely zero out a memory buffer, ensuring that the data is overwritten and not optimized away by the compiler. It is particularly useful for clearing sensitive information, such as passwords or cryptographic keys, from memory. The function should be called when the data in the buffer is no longer needed and must be securely erased. It is important to ensure that the buffer is valid and that the length specified does not exceed the actual size of the buffer.
- **Inputs**:
    - `buf`: A pointer to the memory buffer to be zeroed. Must not be null, and the caller retains ownership of the buffer.
    - `len`: The number of bytes to zero out in the buffer. Must be a non-negative value and should not exceed the size of the buffer.
- **Output**: None
- **See also**: [`explicit_bzero`](compat/explicit_bzero.c.driver.md#explicit_bzero)  (Implementation)


---
### getdtablesize<!-- {{#callable_declaration:getdtablesize}} -->
Returns the maximum number of file descriptors that a process can open.
- **Description**: This function provides the maximum number of file descriptors that a process can have open at any given time. It is useful for applications that need to manage resources efficiently and ensure they do not exceed system limits. The function does not require any parameters and can be called at any point in the program to retrieve the current limit. It is important to note that this value can vary between different systems and configurations.
- **Inputs**:
    - None
- **Output**: The function returns an integer representing the maximum number of file descriptors that can be opened by a process.
- **See also**: [`getdtablesize`](compat/getdtablesize.c.driver.md#getdtablesize)  (Implementation)


---
### strcasestr<!-- {{#callable_declaration:strcasestr}} -->
Finds the first occurrence of a substring in a string, ignoring case.
- **Description**: This function searches for the first occurrence of the substring specified by `find` within the string `s`, without considering the case of the characters. It is useful when a case-insensitive search is required. The function returns a pointer to the beginning of the located substring, or NULL if the substring is not found. The search is case-insensitive, meaning 'A' and 'a' are considered equal. The function expects both `s` and `find` to be null-terminated strings. If `find` is an empty string, the function returns `s`. The behavior is undefined if either `s` or `find` is NULL.
- **Inputs**:
    - `s`: A pointer to the null-terminated string to be searched. Must not be NULL.
    - `find`: A pointer to the null-terminated substring to search for. Must not be NULL.
- **Output**: Returns a pointer to the first occurrence of the substring in the string, or NULL if not found.
- **See also**: [`strcasestr`](compat/strcasestr.c.driver.md#strcasestr)  (Implementation)


---
### strsep<!-- {{#callable_declaration:strsep}} -->
Splits a string into tokens based on delimiter characters.
- **Description**: Use this function to tokenize a string by replacing the first occurrence of any delimiter character in the string with a null character, effectively splitting the string into tokens. It updates the input string pointer to point to the next character after the delimiter, or sets it to NULL if the end of the string is reached. This function is useful for parsing strings where multiple delimiters are possible. It must be called with a valid pointer to a string pointer, and the string pointer must not be NULL. The function modifies the input string in place.
- **Inputs**:
    - `stringp`: A pointer to a pointer to the string to be tokenized. The string must be null-terminated, and the pointer must not be NULL. The function updates this pointer to point to the next token or NULL if no more tokens are found.
    - `delim`: A pointer to a null-terminated string containing delimiter characters. The function uses these characters to determine where to split the input string. This pointer must not be NULL.
- **Output**: Returns a pointer to the beginning of the next token, or NULL if there are no more tokens.
- **See also**: [`strsep`](compat/strsep.c.driver.md#strsep)  (Implementation)


---
### strtonum<!-- {{#callable_declaration:strtonum}} -->
Convert a string to a long long integer within specified bounds.
- **Description**: This function converts a string representation of a number into a long long integer, ensuring that the result falls within the specified minimum and maximum bounds. It is useful when you need to safely parse numeric input from strings while enforcing range constraints. If the conversion fails or the number is out of bounds, an error string is set, and the function returns zero. The function should be used when you need to handle numeric input robustly, with clear error reporting.
- **Inputs**:
    - `numstr`: A pointer to a null-terminated string containing the number to convert. Must not be null.
    - `minval`: The minimum allowable value for the conversion result. Must be less than or equal to maxval.
    - `maxval`: The maximum allowable value for the conversion result. Must be greater than or equal to minval.
    - `errstrp`: A pointer to a string pointer where an error message will be stored if an error occurs. Can be null if error messages are not needed.
- **Output**: Returns the converted long long integer if successful, or zero if an error occurs. If an error occurs, errstrp (if not null) will point to a string describing the error.
- **See also**: [`strtonum`](compat/strtonum.c.driver.md#strtonum)  (Implementation)


---
### strlcpy<!-- {{#callable_declaration:strlcpy}} -->
Copies a string to a destination buffer with size limitation.
- **Description**: This function is used to copy a string from a source to a destination buffer, ensuring that the destination buffer is not overflowed. It is particularly useful when working with fixed-size buffers to prevent buffer overflows. The function guarantees that the destination string is null-terminated if the size is greater than zero. It should be used when you need a safe alternative to `strcpy` that respects buffer boundaries. The function returns the total length of the string it tried to create, which allows the caller to detect truncation by comparing the return value to the size of the destination buffer.
- **Inputs**:
    - `dst`: A pointer to the destination buffer where the string will be copied. Must not be null and should have enough space to hold the copied string, including the null terminator.
    - `src`: A pointer to the source string to be copied. Must not be null.
    - `siz`: The total size of the destination buffer, including space for the null terminator. If this is zero, the destination buffer is not modified.
- **Output**: Returns the length of the source string. If this value is greater than or equal to `siz`, truncation occurred.
- **See also**: [`strlcpy`](compat/strlcpy.c.driver.md#strlcpy)  (Implementation)


---
### strlcat<!-- {{#callable_declaration:strlcat}} -->
Appends a source string to a destination buffer ensuring null-termination.
- **Description**: This function appends the null-terminated string from `src` to the end of the string in `dst`, ensuring that the total length of the resulting string does not exceed `siz` bytes, including the null terminator. It is useful for safely concatenating strings in a buffer with a known size limit, preventing buffer overflows. The function assumes that `dst` is a valid, null-terminated string and that `siz` is the total size of the buffer `dst` points to. If `siz` is less than or equal to the length of `dst`, no characters from `src` are appended, but the length of `src` plus the initial length of `dst` is returned. This function is typically used when you need to concatenate strings with a guarantee of null-termination and without exceeding buffer limits.
- **Inputs**:
    - `dst`: A pointer to the destination buffer where the source string will be appended. It must be a null-terminated string and have enough space to accommodate the concatenated result, including the null terminator.
    - `src`: A pointer to the null-terminated source string to be appended to `dst`. The function reads from this string until a null character is encountered.
    - `siz`: The total size of the destination buffer `dst`, including space for the null terminator. It must be greater than zero.
- **Output**: Returns the total length of the string it tried to create, which is the initial length of `dst` plus the length of `src`. If the return value is greater than or equal to `siz`, truncation occurred.
- **See also**: [`strlcat`](compat/strlcat.c.driver.md#strlcat)  (Implementation)


---
### strnlen<!-- {{#callable_declaration:strnlen}} -->
Determine the length of a string up to a maximum number of characters.
- **Description**: Use this function to find the length of a string, but limit the count to a specified maximum number of characters. This is useful when working with strings that may not be null-terminated within the given limit. The function will stop counting either when a null character is encountered or when the specified maximum length is reached, whichever comes first. It is important to ensure that the string pointer is not null before calling this function.
- **Inputs**:
    - `str`: A pointer to the string whose length is to be measured. The string must be null-terminated if it is shorter than maxlen. The pointer must not be null.
    - `maxlen`: The maximum number of characters to count in the string. It must be a non-negative value.
- **Output**: Returns the number of characters in the string, up to a maximum of maxlen. If the string is shorter than maxlen, the length of the string is returned.
- **See also**: [`strnlen`](compat/strnlen.c.driver.md#strnlen)  (Implementation)


---
### strndup<!-- {{#callable_declaration:strndup}} -->
Create a duplicate of a string with a maximum length.
- **Description**: This function is used to create a new, dynamically allocated copy of a string, with the copy being at most `maxlen` characters long. It is useful when you need a separate copy of a string that is limited to a specific length, ensuring that the new string is null-terminated. The function allocates memory for the new string, which the caller is responsible for freeing. If memory allocation fails, the function returns `NULL`. This function should be used when you need to safely duplicate a string with a length constraint.
- **Inputs**:
    - `str`: A pointer to the null-terminated string to be duplicated. Must not be null.
    - `maxlen`: The maximum number of characters to copy from `str`. Must be a non-negative value.
- **Output**: Returns a pointer to the newly allocated string, or `NULL` if memory allocation fails.
- **See also**: [`strndup`](compat/strndup.c.driver.md#strndup)  (Implementation)


---
### memmem<!-- {{#callable_declaration:memmem}} -->
Searches for a memory block within another memory block.
- **Description**: This function is used to locate the first occurrence of a memory block `s` within another memory block `l`. It is useful when you need to find a specific sequence of bytes within a larger data buffer. The function returns a pointer to the beginning of the located memory block within `l`, or `NULL` if the block is not found. It handles edge cases such as when the length of `s` is zero, in which case it returns the pointer to `l`, and when `s` is longer than `l`, returning `NULL`. This function does not modify the input memory blocks.
- **Inputs**:
    - `l`: A pointer to the memory block to be searched. Must not be null, and the length of this block is specified by `l_len`.
    - `l_len`: The length of the memory block `l` in bytes. Must be greater than or equal to zero.
    - `s`: A pointer to the memory block to search for within `l`. Must not be null, and the length of this block is specified by `s_len`.
    - `s_len`: The length of the memory block `s` in bytes. Must be greater than or equal to zero.
- **Output**: Returns a pointer to the first occurrence of the memory block `s` within `l`, or `NULL` if `s` is not found in `l`.
- **See also**: [`memmem`](compat/memmem.c.driver.md#memmem)  (Implementation)


---
### htonll<!-- {{#callable_declaration:htonll}} -->
Converts a 64-bit integer from host to network byte order.
- **Description**: Use this function to convert a 64-bit unsigned integer from the host's byte order to network byte order, which is big-endian. This is typically necessary when preparing data for transmission over a network to ensure consistent interpretation across different systems with varying byte orders. The function should be used when dealing with network protocols that require data to be in network byte order. It is important to ensure that the input value is a valid 64-bit unsigned integer.
- **Inputs**:
    - `v`: A 64-bit unsigned integer representing the value to be converted. The input must be a valid 64-bit unsigned integer, and the caller retains ownership of the value.
- **Output**: Returns the 64-bit unsigned integer converted to network byte order.
- **See also**: [`htonll`](compat/htonll.c.driver.md#htonll)  (Implementation)


---
### ntohll<!-- {{#callable_declaration:ntohll}} -->
Converts a 64-bit integer from network byte order to host byte order.
- **Description**: Use this function to convert a 64-bit integer from network byte order, which is big-endian, to the host's native byte order. This is typically necessary when receiving data over a network to ensure that multi-byte integers are interpreted correctly on different architectures. The function should be called whenever a 64-bit integer is received from a network source and needs to be processed on the host machine.
- **Inputs**:
    - `v`: A 64-bit unsigned integer representing a value in network byte order. The caller retains ownership of the value, and it must be a valid 64-bit integer.
- **Output**: Returns a 64-bit unsigned integer converted to the host's byte order.
- **See also**: [`ntohll`](compat/ntohll.c.driver.md#ntohll)  (Implementation)


---
### getpeereid<!-- {{#callable_declaration:getpeereid}} -->
Retrieve the effective user ID and group ID of the peer connected to a socket.
- **Description**: This function is used to obtain the effective user ID (UID) and group ID (GID) of the peer process connected to a given socket. It is typically called in network applications where identifying the peer's credentials is necessary for access control or logging purposes. The function must be called with a valid socket descriptor, and it writes the retrieved UID and GID to the provided pointers. If the function fails, it returns -1, and the values pointed to by `uid` and `gid` are not modified. This function should be used in environments where peer credential retrieval is supported.
- **Inputs**:
    - `s`: A valid socket file descriptor representing the connection to the peer. The socket must be in a state where peer credentials can be retrieved.
    - `uid`: A pointer to a uid_t variable where the effective user ID of the peer will be stored. Must not be null.
    - `gid`: A pointer to a gid_t variable where the effective group ID of the peer will be stored. Must not be null.
- **Output**: Returns 0 on success, with the peer's UID and GID written to the provided pointers. Returns -1 on failure, with no changes to the values pointed to by `uid` and `gid`.
- **See also**: [`getpeereid`](compat/getpeereid.c.driver.md#getpeereid)  (Implementation)


---
### daemon<!-- {{#callable_declaration:daemon}} -->
Run the program as a daemon process.
- **Description**: This function is used to run a program as a daemon process, which is a background process that detaches from the controlling terminal. It is typically used in server applications or long-running background tasks. The function forks the process, creates a new session, optionally changes the working directory to the root, and redirects standard input, output, and error to /dev/null. It should be called early in the program's execution, before any resources that should not be inherited by the daemon are allocated.
- **Inputs**:
    - `nochdir`: An integer flag that, if non-zero, prevents the function from changing the current working directory to the root directory ('/'). If zero, the working directory is changed to '/'. The caller retains ownership.
    - `noclose`: An integer flag that, if non-zero, prevents the function from redirecting standard input, output, and error to /dev/null. If zero, these file descriptors are redirected to /dev/null. The caller retains ownership.
- **Output**: Returns 0 on success. On failure, returns -1, indicating an error occurred during the fork or session creation process.
- **See also**: [`daemon`](compat/daemon.c.driver.md#daemon)  (Implementation)


---
### clock_gettime<!-- {{#callable_declaration:clock_gettime}} -->
Retrieve the current time and store it in a timespec structure.
- **Description**: Use this function to obtain the current time, which is stored in the provided timespec structure. This function is useful when you need a high-resolution timestamp. It must be called with a valid pointer to a timespec structure where the time will be stored. The function does not use the clock parameter, so any value can be passed for it.
- **Inputs**:
    - `clock`: An integer representing the clock type, which is unused in this function. Any value can be passed.
    - `ts`: A pointer to a struct timespec where the current time will be stored. Must not be null, as the function will write the time data to the location pointed to by this parameter.
- **Output**: Returns 0 on success, indicating that the current time has been successfully stored in the provided timespec structure.
- **See also**: [`clock_gettime`](compat/clock_gettime.c.driver.md#clock_gettime)  (Implementation)


---
### b64_ntop<!-- {{#callable_declaration:b64_ntop}} -->
Encodes binary data into a Base64 string.
- **Description**: Use this function to convert binary data into a Base64 encoded string, which is a common format for transmitting binary data over text-based protocols. The function requires a buffer to store the encoded output and ensures that the output is null-terminated. It is important to ensure that the target buffer is large enough to hold the encoded data, including the null terminator, to avoid truncation. If the target buffer is too small, the function returns an error.
- **Inputs**:
    - `src`: Pointer to the binary data to be encoded. Must not be null.
    - `srclength`: The length of the binary data in bytes. Must be greater than zero.
    - `target`: Pointer to the buffer where the Base64 encoded string will be stored. Must not be null and should be large enough to hold the encoded data plus a null terminator.
    - `targsize`: The size of the target buffer in bytes. Must be sufficient to store the encoded data and a null terminator.
- **Output**: Returns the length of the encoded Base64 string, excluding the null terminator, or -1 if the target buffer is too small.
- **See also**: [`b64_ntop`](compat/base64.c.driver.md#b64_ntop)  (Implementation)


---
### b64_pton<!-- {{#callable_declaration:b64_pton}} -->
Decodes a Base64 encoded string into binary data.
- **Description**: This function decodes a Base64 encoded string into its binary representation, storing the result in the provided target buffer. It is useful when you need to convert data encoded in Base64 back to its original binary form. The function expects a valid Base64 string and a sufficiently large target buffer to store the decoded data. It handles whitespace in the input string by ignoring it, and it requires proper padding with '=' characters if the input is not a multiple of 4 characters. The function returns the number of bytes written to the target buffer or -1 if an error occurs, such as encountering invalid characters or insufficient buffer size.
- **Inputs**:
    - `src`: A pointer to a null-terminated string containing the Base64 encoded data. The string must be valid and properly padded if necessary. Whitespace is ignored.
    - `target`: A pointer to a buffer where the decoded binary data will be stored. The buffer must be large enough to hold the decoded data. If null, the function will only calculate the required buffer size.
    - `targsize`: The size of the target buffer in bytes. It must be large enough to store the decoded data, otherwise the function will return -1.
- **Output**: Returns the number of bytes written to the target buffer, or -1 if an error occurs (e.g., invalid input or insufficient buffer size).
- **See also**: [`b64_pton`](compat/base64.c.driver.md#b64_pton)  (Implementation)


---
### getptmfd<!-- {{#callable_declaration:getptmfd}} -->
Retrieve a file descriptor for a pseudo-terminal master.
- **Description**: This function is used to obtain a file descriptor for a pseudo-terminal master, which is typically used in terminal emulation or process control applications. It is important to note that the function currently returns a constant value, which may not represent a valid file descriptor. Therefore, it should be used with caution and understanding of its limitations. This function does not require any initialization or setup before being called.
- **Inputs**:
    - None
- **Output**: Returns an integer representing a file descriptor, but currently returns a constant value (INT_MAX) which may not be a valid descriptor.
- **See also**: [`getptmfd`](compat/fdforkpty.c.driver.md#getptmfd)  (Implementation)


---
### fdforkpty<!-- {{#callable_declaration:fdforkpty}} -->
Creates a new pseudo-terminal and forks a child process.
- **Description**: This function is used to create a new pseudo-terminal and fork a child process, which is useful for terminal emulation or running a subprocess with a terminal interface. It should be called when a new pseudo-terminal is needed, and the caller wants to execute a child process in that terminal. The function requires valid pointers for the master file descriptor, terminal name, termios structure, and window size structure. It returns the process ID of the child process to the parent process, and zero to the child process. If an error occurs, it returns -1.
- **Inputs**:
    - `ptmfd`: This parameter is unused and can be ignored.
    - `master`: A pointer to an integer where the file descriptor for the master side of the pseudo-terminal will be stored. Must not be null.
    - `name`: A pointer to a character array where the name of the slave pseudo-terminal will be stored. Can be null if the name is not needed.
    - `tio`: A pointer to a termios structure that specifies the terminal attributes for the slave pseudo-terminal. Can be null if default attributes are acceptable.
    - `ws`: A pointer to a winsize structure that specifies the window size for the slave pseudo-terminal. Can be null if default size is acceptable.
- **Output**: Returns the process ID of the child process to the parent, zero to the child, or -1 on error.
- **See also**: [`fdforkpty`](compat/fdforkpty.c.driver.md#fdforkpty)  (Implementation)


---
### asprintf<!-- {{#callable_declaration:asprintf}} -->
Formats a string and allocates memory for it.
- **Description**: This function formats a string according to the specified format and additional arguments, allocating memory to store the resulting string. It is useful when you need a formatted string but do not know the required buffer size in advance. The function must be called with a valid format string and a pointer to a char pointer, which will be set to point to the newly allocated string. The caller is responsible for freeing the allocated memory. If memory allocation fails, the function returns a negative value and the pointer is set to NULL.
- **Inputs**:
    - `ret`: A pointer to a char pointer where the address of the allocated string will be stored. Must not be null. The caller is responsible for freeing the allocated memory.
    - `fmt`: A format string that specifies how to format the additional arguments. Must not be null and should follow printf-style formatting.
    - `...`: Additional arguments that are formatted according to the format string. The number and types of these arguments must match the format specifiers in the format string.
- **Output**: Returns the number of characters printed (excluding the null byte) on success, or a negative value if an error occurs.
- **See also**: [`asprintf`](compat/asprintf.c.driver.md#asprintf)  (Implementation)


---
### vasprintf<!-- {{#callable_declaration:vasprintf}} -->
Formats a string and allocates memory for it.
- **Description**: This function is used to format a string according to a specified format and a list of arguments, storing the result in a newly allocated buffer. It is useful when the size of the formatted string is not known in advance. The function allocates memory for the resulting string, which must be freed by the caller to avoid memory leaks. It should be called with a valid format string and a properly initialized va_list. If the function fails, it returns -1 and sets the output pointer to NULL.
- **Inputs**:
    - `ret`: A pointer to a char pointer where the address of the allocated buffer will be stored. Must not be null. The caller is responsible for freeing the allocated memory.
    - `fmt`: A format string that specifies how to format the output. Must not be null.
    - `ap`: A va_list containing the arguments to format according to the format string. Must be properly initialized before calling this function.
- **Output**: Returns the number of characters printed (excluding the null byte) on success, or -1 on failure.
- **See also**: [`vasprintf`](compat/asprintf.c.driver.md#vasprintf)  (Implementation)


---
### fgetln<!-- {{#callable_declaration:fgetln}} -->
Reads a line from a file stream into a buffer.
- **Description**: Use this function to read a line from the specified file stream, storing it in a dynamically managed buffer. It is suitable for reading lines of arbitrary length, as the buffer will be resized as needed. The function must be called with a valid file pointer and a non-null pointer to a size_t variable to store the length of the line. It returns a pointer to the buffer containing the line, or NULL if an error occurs or the end of the file is reached without reading any data. The buffer is managed internally and should not be freed by the caller.
- **Inputs**:
    - `fp`: A pointer to a FILE object that identifies the input stream. Must not be null. If null, the function sets errno to EINVAL and returns NULL.
    - `len`: A pointer to a size_t variable where the length of the read line will be stored. Must not be null. If null, the function sets errno to EINVAL and returns NULL.
- **Output**: Returns a pointer to a buffer containing the line read from the file, or NULL if an error occurs or the end of the file is reached without reading any data. The length of the line is stored in the variable pointed to by len.
- **See also**: [`fgetln`](compat/fgetln.c.driver.md#fgetln)  (Implementation)


---
### getline<!-- {{#callable_declaration:getline}} -->
Read a line from a file stream into a buffer.
- **Description**: This function reads an entire line from the specified file stream, storing the address of the buffer containing the text into the provided pointer. It is useful for reading lines of text from a file, handling memory allocation for the buffer as needed. The function must be called with a valid file stream, and the buffer pointer must be initialized to NULL if no buffer is allocated yet. The function will allocate or resize the buffer as necessary to fit the line, including the terminating newline character and a null byte. It is the caller's responsibility to free the buffer when it is no longer needed.
- **Inputs**:
    - `buf`: A pointer to a char pointer where the address of the buffer will be stored. Must not be null. If the buffer is not allocated, the pointer should be initialized to NULL.
    - `bufsiz`: A pointer to a size_t variable that holds the size of the buffer. Must not be null. The function will update this value if the buffer is reallocated.
    - `fp`: A pointer to a FILE object that identifies the input stream. Must not be null and should be opened for reading.
- **Output**: Returns the number of characters read, including the delimiter, or -1 on failure or end of file.
- **See also**: [`getline`](compat/getline.c.driver.md#getline)  (Implementation)


---
### setenv<!-- {{#callable_declaration:setenv}} -->
Sets an environment variable to a specified value.
- **Description**: Use this function to set or update the value of an environment variable in the current process's environment. The function constructs a new environment variable string in the format 'name=value' and adds it to the environment. It does not check the 'overwrite' parameter, so it will always attempt to set the variable regardless of whether it already exists. This function should be used when you need to ensure that a specific environment variable is set to a particular value for the duration of the process.
- **Inputs**:
    - `name`: The name of the environment variable to set. Must not be null and should be a valid string representing the variable name.
    - `value`: The value to assign to the environment variable. Must not be null and should be a valid string representing the variable value.
    - `overwrite`: This parameter is ignored by the function and has no effect on its behavior.
- **Output**: Returns 0 on success, or a non-zero value if an error occurs while setting the environment variable.
- **See also**: [`setenv`](compat/setenv.c.driver.md#setenv)  (Implementation)


---
### unsetenv<!-- {{#callable_declaration:unsetenv}} -->
Removes an environment variable.
- **Description**: Use this function to remove an environment variable specified by its name from the environment. It is typically used when you need to ensure that a particular environment variable is no longer set. The function must be called with a valid, non-null string representing the name of the environment variable to be removed. If the specified variable is not found, the function has no effect. This function does not return an error code, so it is assumed to always succeed in removing the variable if it exists.
- **Inputs**:
    - `name`: A pointer to a null-terminated string representing the name of the environment variable to be removed. The string must not be null, and it should be a valid environment variable name. If the name is not found in the environment, the function does nothing.
- **Output**: Returns 0 after attempting to remove the environment variable, regardless of whether the variable was found and removed.
- **See also**: [`unsetenv`](compat/setenv.c.driver.md#unsetenv)  (Implementation)


---
### cfmakeraw<!-- {{#callable_declaration:cfmakeraw}} -->
Configure a termios structure for raw input mode.
- **Description**: Use this function to set a termios structure to raw mode, which is useful for applications that require unprocessed input and output, such as terminal emulators or custom shell environments. In raw mode, input is available character by character, and no special processing is performed on input or output data. This function should be called with a valid termios structure, typically obtained from a previous call to tcgetattr, and is often followed by a call to tcsetattr to apply the changes to a terminal device.
- **Inputs**:
    - `tio`: A pointer to a termios structure that will be modified to configure raw mode. Must not be null. The caller retains ownership of the structure.
- **Output**: None
- **See also**: [`cfmakeraw`](compat/cfmakeraw.c.driver.md#cfmakeraw)  (Implementation)


---
### freezero<!-- {{#callable_declaration:freezero}} -->
Securely deallocate memory by zeroing it before freeing.
- **Description**: Use this function to securely deallocate memory by first zeroing out the specified number of bytes and then freeing the memory block. This is particularly useful when dealing with sensitive data that should not remain in memory after deallocation. The function should be called with a valid pointer and a size indicating the number of bytes to zero. If the pointer is NULL, the function does nothing, ensuring safe handling of null pointers.
- **Inputs**:
    - `ptr`: A pointer to the memory block to be zeroed and freed. Must not be null unless intentionally passing a null pointer, in which case the function does nothing.
    - `size`: The number of bytes to zero in the memory block pointed to by ptr. Should be a valid size for the allocated memory block.
- **Output**: None
- **See also**: [`freezero`](compat/freezero.c.driver.md#freezero)  (Implementation)


---
### reallocarray<!-- {{#callable_declaration:reallocarray}} -->
Reallocate memory for an array with overflow checking.
- **Description**: Use this function to resize a memory block for an array, ensuring that the total size does not overflow. It is particularly useful when you need to allocate memory for an array of elements, where the number of elements and the size of each element are specified separately. This function checks for potential overflow in the multiplication of these two values, which could otherwise lead to undefined behavior. If the multiplication would overflow, the function sets `errno` to `ENOMEM` and returns `NULL`. Otherwise, it behaves like `realloc`, resizing the memory block pointed to by `optr` to the new size `nmemb * size`. The function should be used when you need to safely handle large memory allocations for arrays.
- **Inputs**:
    - `optr`: A pointer to the memory block to be reallocated. It can be `NULL`, in which case the function behaves like `malloc`. The caller retains ownership of the memory.
    - `nmemb`: The number of elements in the array. Must be a non-negative value. If the multiplication of `nmemb` and `size` would overflow, the function returns `NULL`.
    - `size`: The size of each element in the array. Must be a non-negative value. If the multiplication of `nmemb` and `size` would overflow, the function returns `NULL`.
- **Output**: A pointer to the newly allocated memory block, or `NULL` if the allocation fails or if the multiplication of `nmemb` and `size` would overflow. If `NULL` is returned, `errno` is set to `ENOMEM`.
- **See also**: [`reallocarray`](compat/reallocarray.c.driver.md#reallocarray)  (Implementation)


---
### recallocarray<!-- {{#callable_declaration:recallocarray}} -->
Reallocate and zero-initialize an array.
- **Description**: This function reallocates a memory block for an array, changing its size from `oldnmemb` elements to `newnmemb` elements, each of size `size`. It is useful when you need to resize an array while ensuring that any newly allocated memory is zero-initialized. If `ptr` is NULL, it behaves like `calloc`. The function handles potential integer overflow in size calculations and sets `errno` appropriately on failure. It also securely clears the old memory block before freeing it.
- **Inputs**:
    - `ptr`: A pointer to the currently allocated memory block. If NULL, the function allocates a new block.
    - `oldnmemb`: The number of elements in the current memory block. Must be a valid size_t value.
    - `newnmemb`: The desired number of elements in the new memory block. Must be a valid size_t value.
    - `size`: The size of each element in bytes. Must be a valid size_t value.
- **Output**: Returns a pointer to the newly allocated memory block, or NULL if allocation fails.
- **See also**: [`recallocarray`](compat/recallocarray.c.driver.md#recallocarray)  (Implementation)


---
### systemd_activated<!-- {{#callable_declaration:systemd_activated}} -->
Checks if the program was started by systemd with socket activation.
- **Description**: This function determines whether the current process was launched by systemd using socket activation. It is useful in applications that need to adapt their behavior based on how they were started, particularly in environments where systemd is used to manage services. The function should be called early in the program's execution to decide on initialization paths that depend on socket activation. It returns a non-zero value if systemd socket activation is detected, otherwise it returns zero.
- **Inputs**:
    - None
- **Output**: Returns a non-zero integer if systemd socket activation is detected, otherwise returns zero.
- **See also**: [`systemd_activated`](compat/systemd.c.driver.md#systemd_activated)  (Implementation)


---
### systemd_create_socket<!-- {{#callable_declaration:systemd_create_socket}} -->
Creates a socket for systemd service activation.
- **Description**: This function is used to create a socket that can be used for systemd service activation. It should be called when a service is expected to be socket-activated by systemd. The function checks for existing file descriptors passed by systemd and validates them. If no valid socket is found, it attempts to create a new one using the provided flags. It is important to handle the case where the function returns an error, as indicated by a negative return value, and to check the 'cause' parameter for a descriptive error message if provided.
- **Inputs**:
    - `flags`: An integer representing the flags used for socket creation. The specific flags and their meanings are not detailed in the header, but they are used when creating a new socket if no systemd socket is available.
    - `cause`: A pointer to a character pointer where an error message will be stored if the function fails. This parameter can be NULL if the caller does not require an error message. If provided, the caller is responsible for freeing the memory allocated for the error message.
- **Output**: Returns a file descriptor for the socket on success, or -1 on failure. If -1 is returned, 'cause' may contain an error message if it is not NULL.
- **See also**: [`systemd_create_socket`](compat/systemd.c.driver.md#systemd_create_socket)  (Implementation)


---
### systemd_move_to_new_cgroup<!-- {{#callable_declaration:systemd_move_to_new_cgroup}} -->
Moves the current process to a new systemd cgroup.
- **Description**: This function attempts to move the current process into a new systemd cgroup by creating a transient unit. It is typically used in environments where process isolation and resource management are required, such as in containerized or multi-user systems. The function connects to the systemd session bus and issues a method call to start a transient unit, handling potential errors and timeouts. It must be called in a context where the systemd bus is accessible, and the caller should be prepared to handle error messages returned through the provided parameter.
- **Inputs**:
    - `cause`: A pointer to a char pointer where an error message will be stored if the function fails. The caller must ensure this pointer is valid and is responsible for freeing any allocated memory. If the function succeeds, this will remain unchanged.
- **Output**: Returns an integer indicating success (0) or failure (negative value). On failure, an error message is stored in the provided 'cause' parameter.
- **See also**: [`systemd_move_to_new_cgroup`](compat/systemd.c.driver.md#systemd_move_to_new_cgroup)  (Implementation)


---
### utf8proc_wcwidth<!-- {{#callable_declaration:utf8proc_wcwidth}} -->
Determines the display width of a wide character.
- **Description**: This function calculates the number of column positions required to display a given wide character in a terminal. It is useful when handling text that includes wide characters, such as those found in various international character sets. The function should be used when you need to align text output that includes wide characters. It handles special cases like private use characters by assigning them a width of 1. Ensure that the input character is valid and within the range of wide characters.
- **Inputs**:
    - `wc`: A wide character whose display width is to be determined. It must be a valid wide character; otherwise, the behavior is undefined.
- **Output**: Returns the number of column positions required to display the wide character, typically 0, 1, or 2.
- **See also**: [`utf8proc_wcwidth`](compat/utf8proc.c.driver.md#utf8proc_wcwidth)  (Implementation)


---
### utf8proc_mbtowc<!-- {{#callable_declaration:utf8proc_mbtowc}} -->
Converts a multibyte UTF-8 sequence to a wide character.
- **Description**: This function is used to convert a multibyte UTF-8 sequence into a wide character, storing the result in the location pointed to by `pwc`. It is useful when working with UTF-8 encoded text that needs to be processed as wide characters. The function should be called with a valid UTF-8 string and a buffer size that is sufficient to hold the multibyte sequence. If the input string is null, the function returns 0. The function returns the number of bytes processed or -1 if an error occurs, such as an invalid UTF-8 sequence.
- **Inputs**:
    - `pwc`: A pointer to a `wchar_t` where the resulting wide character will be stored. Must not be null. If the conversion fails, the value pointed to by `pwc` is set to -1.
    - `s`: A pointer to the UTF-8 encoded string to be converted. Must not be null. If null, the function returns 0.
    - `n`: The maximum number of bytes to read from the string `s`. Should be a positive number that does not exceed the length of `s`.
- **Output**: Returns the number of bytes processed from the input string `s`, or -1 if an error occurs. If `s` is null, returns 0.
- **See also**: [`utf8proc_mbtowc`](compat/utf8proc.c.driver.md#utf8proc_mbtowc)  (Implementation)


---
### utf8proc_wctomb<!-- {{#callable_declaration:utf8proc_wctomb}} -->
Converts a wide character to a UTF-8 encoded string.
- **Description**: This function is used to convert a single wide character to its UTF-8 encoded representation. It is useful when you need to handle wide characters and convert them to a format suitable for UTF-8 encoded text processing. The function should be called with a valid wide character and a buffer to store the resulting UTF-8 string. It is important to ensure that the buffer is large enough to hold the UTF-8 encoded character, which can be up to 4 bytes. The function will return an error if the wide character is invalid or if the buffer is null.
- **Inputs**:
    - `s`: A pointer to a buffer where the UTF-8 encoded character will be stored. Must not be null. The buffer should be large enough to hold up to 4 bytes.
    - `wc`: The wide character to be converted. It must be a valid Unicode code point. If the code point is invalid, the function returns an error.
- **Output**: Returns the number of bytes written to the buffer if successful, 0 if the buffer is null, or -1 if the wide character is invalid.
- **See also**: [`utf8proc_wctomb`](compat/utf8proc.c.driver.md#utf8proc_wctomb)  (Implementation)


