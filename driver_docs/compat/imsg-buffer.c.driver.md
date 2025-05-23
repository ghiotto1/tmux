# Purpose
The provided C source code file, `imsg-buffer.c`, is part of the OpenBSD project and is designed to handle inter-process communication (IPC) through message buffers. This file implements a set of functions for managing input and output buffers (`ibuf`) and message buffers (`msgbuf`). The primary purpose of this code is to facilitate the reading, writing, and management of data buffers, which are used to send and receive messages between processes, potentially over network sockets. The code includes functions for creating, manipulating, and freeing these buffers, as well as for reading from and writing to file descriptors using these buffers.

The code defines a variety of functions to handle buffer operations, such as reserving space, adding data, setting data at specific positions, and retrieving data from buffers. It also includes functions for handling network byte order conversions, which are crucial for ensuring data integrity across different systems. The `msgbuf` structure is used to manage a queue of `ibuf` structures, allowing for efficient message handling. The file provides a comprehensive API for buffer management, including functions for reading and writing data with support for file descriptor passing, which is essential for IPC. This code is intended to be used as part of a larger system where reliable and efficient message passing is required, such as in network daemons or other system-level applications.
# Imports and Dependencies

---
- `sys/types.h`
- `sys/socket.h`
- `sys/uio.h`
- `arpa/inet.h`
- `limits.h`
- `errno.h`
- `stdint.h`
- `stdlib.h`
- `string.h`
- `unistd.h`
- `compat.h`
- `imsg.h`


# Data Structures

---
### msgbuf
- **Type**: `struct`
- **Members**:
    - `bufs`: A tail queue head for storing output buffers of type `ibuf`.
    - `rbufs`: A tail queue head for storing input buffers of type `ibuf`.
    - `queued`: A 32-bit unsigned integer representing the number of queued messages.
    - `rbuf`: A pointer to a character buffer used for reading data.
    - `rpmsg`: A pointer to an `ibuf` structure representing the current message being processed.
    - `readhdr`: A function pointer for reading the header of a message from an `ibuf`.
    - `rarg`: A void pointer to additional arguments for the `readhdr` function.
    - `roff`: A size_t value indicating the current offset in the read buffer.
    - `hdrsize`: A size_t value representing the size of the message header.
- **Description**: The `msgbuf` structure is designed to manage message buffers for input and output operations, particularly in network communication contexts. It maintains separate tail queues for input (`rbufs`) and output (`bufs`) buffers, both of which are of type `ibuf`. The structure tracks the number of queued messages with the `queued` member and uses `rbuf` as a temporary buffer for reading data. The `rpmsg` pointer holds the current message being processed, while `readhdr` is a function pointer used to read message headers, with `rarg` providing additional arguments for this function. The `roff` and `hdrsize` members help manage the reading process by indicating the current offset and the size of the message header, respectively.


# Functions

---
### ibuf_open<!-- {{#callable:ibuf_open}} -->
The `ibuf_open` function allocates and initializes an `ibuf` structure for managing a buffer.
- **Inputs**:
    - `len`: The size of the buffer to allocate, specified in bytes.
- **Control Flow**:
    - The function begins by attempting to allocate memory for an `ibuf` structure using `calloc`.
    - If the allocation fails, it returns `NULL`.
    - If `len` is greater than 0, it attempts to allocate memory for the buffer itself.
    - If this second allocation fails, it frees the previously allocated `ibuf` structure and returns `NULL`.
    - If both allocations succeed, it initializes the `size` and `max` fields of the `ibuf` structure with the value of `len` and sets the file descriptor `fd` to -1.
    - Finally, it returns a pointer to the initialized `ibuf` structure.
- **Output**:
    - The function returns a pointer to the newly allocated and initialized `ibuf` structure, or `NULL` if any allocation fails.


---
### ibuf_dynamic<!-- {{#callable:ibuf_dynamic}} -->
The `ibuf_dynamic` function allocates and initializes a dynamic input buffer with specified length and maximum size.
- **Inputs**:
    - `len`: The initial length of the buffer to allocate.
    - `max`: The maximum size that the buffer can grow to.
- **Control Flow**:
    - Check if `max` is zero or less than `len`, and if so, set `errno` to EINVAL and return NULL.
    - Allocate memory for a new `struct ibuf` using `calloc`, and if allocation fails, return NULL.
    - If `len` is greater than zero, allocate memory for the buffer (`buf->buf`) of size `len`, and if this allocation fails, free the previously allocated `buf` and return NULL.
    - Set the `size` of the buffer to `len`, the `max` to the provided `max`, and initialize the file descriptor `fd` to -1.
    - Return the pointer to the newly created and initialized `struct ibuf`.
- **Output**:
    - Returns a pointer to the newly allocated `struct ibuf` if successful, or NULL if an error occurred.


---
### ibuf_reserve<!-- {{#callable:ibuf_reserve}} -->
The `ibuf_reserve` function reserves a specified amount of space in an `ibuf` structure, potentially reallocating the buffer if necessary.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure that represents the buffer in which space is to be reserved.
    - `len`: The size in bytes to reserve in the buffer.
- **Control Flow**:
    - Check if the requested length `len` would cause an overflow when added to the current write position `buf->wpos`. If so, set `errno` to `ERANGE` and return NULL.
    - Check if the buffer is marked as a stack buffer (`IBUF_FD_MARK_ON_STACK`). If it is, set `errno` to `EINVAL` and return NULL, as stack buffers cannot be resized.
    - If the current write position plus the requested length exceeds the current buffer size, check if the new size exceeds the maximum allowed size (`buf->max`). If it does, set `errno` to `ERANGE` and return NULL.
    - Attempt to reallocate the buffer to accommodate the new size. If `realloc` fails, return NULL.
    - Zero out the newly allocated space in the buffer to ensure it is clean.
    - Update the buffer's size and write position, and return a pointer to the newly reserved space.
- **Output**:
    - Returns a pointer to the reserved space in the buffer if successful, or NULL if an error occurred.


---
### ibuf_add<!-- {{#callable:ibuf_add}} -->
The `ibuf_add` function adds a specified amount of data to a buffer.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer to which data will be added.
    - `data`: A pointer to the data that needs to be added to the buffer.
    - `len`: The size in bytes of the data to be added.
- **Control Flow**:
    - The function first attempts to reserve space in the buffer by calling [`ibuf_reserve`](#ibuf_reserve) with the specified length.
    - If [`ibuf_reserve`](#ibuf_reserve) returns NULL, indicating that there is not enough space, the function returns -1 to signal an error.
    - If space is successfully reserved, the function copies the data from the provided pointer into the reserved space using `memcpy`.
    - Finally, the function returns 0 to indicate success.
- **Output**:
    - The function returns 0 on success and -1 on failure, with the failure reason indicated by the global `errno` variable.
- **Functions called**:
    - [`ibuf_reserve`](#ibuf_reserve)


---
### ibuf_add_ibuf<!-- {{#callable:ibuf_add_ibuf}} -->
The `ibuf_add_ibuf` function appends the contents of one `ibuf` structure to another.
- **Inputs**:
    - `buf`: A pointer to the destination `ibuf` structure where data will be added.
    - `from`: A pointer to the source `ibuf` structure whose data will be appended to the destination.
- **Control Flow**:
    - The function calls `ibuf_data(from)` to retrieve a pointer to the data contained in the source `ibuf`.
    - It then calls `ibuf_size(from)` to get the size of the data in the source `ibuf`.
    - Finally, it invokes [`ibuf_add`](#ibuf_add) to append the data from the source `ibuf` to the destination `ibuf`.
- **Output**:
    - The function returns the result of the [`ibuf_add`](#ibuf_add) function, which is an integer indicating success (0) or failure (-1).
- **Functions called**:
    - [`ibuf_add`](#ibuf_add)
    - [`ibuf_data`](#ibuf_data)
    - [`ibuf_size`](#ibuf_size)


---
### ibuf_add_n8<!-- {{#callable:ibuf_add_n8}} -->
The `ibuf_add_n8` function adds an 8-bit unsigned integer to a buffer.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure where the data will be added.
    - `value`: A 64-bit unsigned integer representing the value to be added, which must be within the range of an 8-bit unsigned integer.
- **Control Flow**:
    - The function first checks if the provided `value` exceeds `UINT8_MAX` (255).
    - If the value is too large, it sets `errno` to `EINVAL` and returns -1 to indicate an error.
    - If the value is valid, it casts the value to a `uint8_t` and stores it in the variable `v`.
    - Finally, it calls the [`ibuf_add`](#ibuf_add) function to add the byte-sized value to the buffer.
- **Output**:
    - The function returns 0 on success, indicating that the value was successfully added to the buffer, or -1 on failure if the value was out of range.
- **Functions called**:
    - [`ibuf_add`](#ibuf_add)


---
### ibuf_add_n16<!-- {{#callable:ibuf_add_n16}} -->
The `ibuf_add_n16` function adds a 16-bit unsigned integer value to a buffer after converting it to big-endian format.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure where the data will be added.
    - `value`: A 64-bit unsigned integer that represents the value to be added, which must be less than or equal to UINT16_MAX.
- **Control Flow**:
    - The function first checks if the input `value` exceeds `UINT16_MAX`. If it does, it sets `errno` to EINVAL and returns -1.
    - If the value is valid, it converts the value to big-endian format using `htobe16` and stores it in a local variable `v`.
    - Finally, it calls the [`ibuf_add`](#ibuf_add) function to add the converted value to the buffer.
- **Output**:
    - The function returns 0 on success, indicating that the value was successfully added to the buffer, or -1 on failure if the input value was invalid.
- **Functions called**:
    - [`ibuf_add`](#ibuf_add)


---
### ibuf_add_n32<!-- {{#callable:ibuf_add_n32}} -->
The `ibuf_add_n32` function adds a 32-bit unsigned integer value to a buffer after converting it to big-endian format.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure where the data will be added.
    - `value`: A 64-bit unsigned integer value that is to be added to the buffer.
- **Control Flow**:
    - Check if the input `value` exceeds `UINT32_MAX`; if it does, set `errno` to `EINVAL` and return -1.
    - Convert the `value` to big-endian format using `htobe32` and store it in a local variable `v`.
    - Call the [`ibuf_add`](#ibuf_add) function to add the converted value `v` to the buffer `buf`.
- **Output**:
    - Returns 0 on success, or -1 if an error occurs (e.g., if the input value is too large).
- **Functions called**:
    - [`ibuf_add`](#ibuf_add)


---
### ibuf_add_n64<!-- {{#callable:ibuf_add_n64}} -->
Adds a 64-bit integer value to a buffer after converting it to big-endian format.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure where the value will be added.
    - `value`: The 64-bit unsigned integer value to be added to the buffer.
- **Control Flow**:
    - The function first converts the input `value` from host byte order to big-endian byte order using `htobe64`.
    - It then calls the [`ibuf_add`](#ibuf_add) function, passing the buffer, a pointer to the converted value, and the size of the value.
- **Output**:
    - Returns the result of the [`ibuf_add`](#ibuf_add) function, which indicates success (0) or failure (-1).
- **Functions called**:
    - [`ibuf_add`](#ibuf_add)


---
### ibuf_add_h16<!-- {{#callable:ibuf_add_h16}} -->
The `ibuf_add_h16` function adds a 16-bit unsigned integer value to a buffer after validating its size.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure where the 16-bit value will be added.
    - `value`: A 64-bit unsigned integer that represents the value to be added, which must be less than or equal to UINT16_MAX.
- **Control Flow**:
    - The function first checks if the input `value` exceeds `UINT16_MAX`.
    - If the value is too large, it sets `errno` to EINVAL and returns -1.
    - If the value is valid, it casts the value to a `uint16_t` and stores it in `v`.
    - Finally, it calls the [`ibuf_add`](#ibuf_add) function to add the 16-bit value to the buffer.
- **Output**:
    - The function returns 0 on success, indicating that the value was successfully added to the buffer, or -1 on failure if the value was invalid.
- **Functions called**:
    - [`ibuf_add`](#ibuf_add)


---
### ibuf_add_h32<!-- {{#callable:ibuf_add_h32}} -->
The `ibuf_add_h32` function adds a 32-bit unsigned integer value to a buffer after validating its size.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure where the data will be added.
    - `value`: A 64-bit unsigned integer value that is to be added to the buffer, which must be less than or equal to UINT32_MAX.
- **Control Flow**:
    - The function first checks if the `value` exceeds `UINT32_MAX`. If it does, it sets `errno` to EINVAL and returns -1.
    - If the value is valid, it casts the `value` to a 32-bit unsigned integer (`v`).
    - The function then calls [`ibuf_add`](#ibuf_add) to add the 32-bit value to the buffer, passing the buffer, the address of `v`, and the size of `v`.
- **Output**:
    - The function returns 0 on success, indicating that the value was successfully added to the buffer, or -1 on failure if the value was invalid.
- **Functions called**:
    - [`ibuf_add`](#ibuf_add)


---
### ibuf_add_h64<!-- {{#callable:ibuf_add_h64}} -->
The `ibuf_add_h64` function adds a 64-bit unsigned integer value to a buffer.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer to which the value will be added.
    - `value`: A 64-bit unsigned integer value that will be added to the buffer.
- **Control Flow**:
    - The function calls [`ibuf_add`](#ibuf_add) with the buffer, a pointer to the value, and the size of the value (which is 8 bytes for a 64-bit integer).
    - The [`ibuf_add`](#ibuf_add) function handles the actual addition of the value to the buffer.
- **Output**:
    - The function returns the result of the [`ibuf_add`](#ibuf_add) call, which is an integer indicating success (0) or failure (-1).
- **Functions called**:
    - [`ibuf_add`](#ibuf_add)


---
### ibuf_add_zero<!-- {{#callable:ibuf_add_zero}} -->
The `ibuf_add_zero` function reserves a specified amount of space in an `ibuf` structure and initializes that space to zero.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure where the zeroed space will be added.
    - `len`: The size in bytes of the space to be reserved and initialized to zero.
- **Control Flow**:
    - The function first attempts to reserve `len` bytes in the `ibuf` structure by calling [`ibuf_reserve`](#ibuf_reserve).
    - If [`ibuf_reserve`](#ibuf_reserve) returns NULL, indicating that the reservation failed, the function returns -1.
    - If the reservation is successful, the function uses `memset` to fill the reserved space with zeros.
    - Finally, the function returns 0 to indicate success.
- **Output**:
    - The function returns 0 on success and -1 on failure, with the failure indicated by an error set in `errno`.
- **Functions called**:
    - [`ibuf_reserve`](#ibuf_reserve)


---
### ibuf_seek<!-- {{#callable:ibuf_seek}} -->
The `ibuf_seek` function calculates a pointer to a specific position within a buffer, ensuring that the requested position and length are within valid bounds.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer from which to seek.
    - `pos`: The position within the buffer from which to start seeking.
    - `len`: The length of the data to be accessed from the specified position.
- **Control Flow**:
    - The function first checks if the requested position and length are within the valid range of the buffer.
    - If the checks fail, it sets `errno` to `ERANGE` and returns `NULL`.
    - If the checks pass, it calculates the address by adding the read position (`rpos`) and the specified position (`pos`) to the buffer's base address.
- **Output**:
    - Returns a pointer to the calculated position within the buffer if successful, or `NULL` if the requested position and length are out of bounds.
- **Functions called**:
    - [`ibuf_size`](#ibuf_size)


---
### ibuf_set<!-- {{#callable:ibuf_set}} -->
The `ibuf_set` function sets a specified portion of an `ibuf` buffer with given data.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure that represents the buffer to be modified.
    - `pos`: The position in the buffer where the data should be written.
    - `data`: A pointer to the data that will be copied into the buffer.
    - `len`: The number of bytes to copy from the data into the buffer.
- **Control Flow**:
    - The function first calls [`ibuf_seek`](#ibuf_seek) to find the correct position in the buffer for writing the data.
    - If [`ibuf_seek`](#ibuf_seek) returns NULL, indicating an error in seeking, the function returns -1.
    - If seeking is successful, the function uses `memcpy` to copy the specified length of data from the `data` pointer to the calculated position in the buffer.
    - Finally, the function returns 0 to indicate success.
- **Output**:
    - The function returns 0 on success, or -1 if an error occurs during the seek operation.
- **Functions called**:
    - [`ibuf_seek`](#ibuf_seek)


---
### ibuf_set_n8<!-- {{#callable:ibuf_set_n8}} -->
Sets an 8-bit unsigned integer value at a specified position in a buffer.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer where the value will be set.
    - `pos`: The position in the buffer where the 8-bit value will be written.
    - `value`: The 64-bit unsigned integer value to be set, which must be within the range of an 8-bit unsigned integer.
- **Control Flow**:
    - Checks if the provided `value` exceeds `UINT8_MAX` (255). If it does, sets `errno` to `EINVAL` and returns -1.
    - If the value is valid, it casts the value to a `uint8_t` and stores it in the variable `v`.
    - Calls the [`ibuf_set`](#ibuf_set) function to write the byte `v` to the specified position `pos` in the buffer `buf`.
- **Output**:
    - Returns 0 on success, or -1 if an error occurred (e.g., if the value is out of range).
- **Functions called**:
    - [`ibuf_set`](#ibuf_set)


---
### ibuf_set_n16<!-- {{#callable:ibuf_set_n16}} -->
Sets a 16-bit unsigned integer value at a specified position in a buffer.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer where the value will be set.
    - `pos`: The position in the buffer where the 16-bit value should be written.
    - `value`: The 64-bit unsigned integer value to be converted to 16-bit and set in the buffer.
- **Control Flow**:
    - Checks if the provided `value` exceeds the maximum value for a 16-bit unsigned integer (UINT16_MAX).
    - If the value is too large, sets `errno` to EINVAL and returns -1.
    - Converts the value from host byte order to big-endian format using `htobe16`.
    - Calls the [`ibuf_set`](#ibuf_set) function to write the converted value into the buffer at the specified position.
- **Output**:
    - Returns 0 on success, or -1 if an error occurred (e.g., if the value is out of range).
- **Functions called**:
    - [`ibuf_set`](#ibuf_set)


---
### ibuf_set_n32<!-- {{#callable:ibuf_set_n32}} -->
Sets a 32-bit unsigned integer value at a specified position in a buffer.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer where the value will be set.
    - `pos`: The position in the buffer where the 32-bit value should be written.
    - `value`: The 64-bit unsigned integer value to be set, which must be within the range of a 32-bit unsigned integer.
- **Control Flow**:
    - Checks if the provided `value` exceeds `UINT32_MAX`; if it does, sets `errno` to `EINVAL` and returns -1.
    - Converts the `value` from host byte order to big-endian format using `htobe32`.
    - Calls the [`ibuf_set`](#ibuf_set) function to write the converted value into the buffer at the specified position.
- **Output**:
    - Returns 0 on success, or -1 if an error occurred (e.g., if the value is out of range).
- **Functions called**:
    - [`ibuf_set`](#ibuf_set)


---
### ibuf_set_n64<!-- {{#callable:ibuf_set_n64}} -->
The `ibuf_set_n64` function sets a 64-bit unsigned integer value at a specified position in a buffer after converting it to big-endian format.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer where the value will be set.
    - `pos`: A size_t value indicating the position in the buffer where the 64-bit value should be written.
    - `value`: A 64-bit unsigned integer value that will be converted to big-endian format before being written to the buffer.
- **Control Flow**:
    - The function first converts the input `value` from host byte order to big-endian byte order using the `htobe64` macro.
    - It then calls the [`ibuf_set`](#ibuf_set) function, passing the buffer, the specified position, a pointer to the converted value, and the size of the value.
- **Output**:
    - The function returns the result of the [`ibuf_set`](#ibuf_set) function, which is an integer indicating success (0) or failure (-1).
- **Functions called**:
    - [`ibuf_set`](#ibuf_set)


---
### ibuf_set_h16<!-- {{#callable:ibuf_set_h16}} -->
Sets a 16-bit unsigned integer value at a specified position in a buffer.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer where the value will be set.
    - `pos`: The position in the buffer where the 16-bit value will be written.
    - `value`: The 64-bit unsigned integer value to be set, which must be within the range of a 16-bit unsigned integer.
- **Control Flow**:
    - Checks if the provided `value` exceeds the maximum value for a 16-bit unsigned integer (UINT16_MAX).
    - If the value is invalid, sets `errno` to EINVAL and returns -1.
    - Casts the valid `value` to a 16-bit unsigned integer (`uint16_t`).
    - Calls the [`ibuf_set`](#ibuf_set) function to write the 16-bit value to the specified position in the buffer.
- **Output**:
    - Returns 0 on success, or -1 if an error occurred (e.g., if the value is out of range).
- **Functions called**:
    - [`ibuf_set`](#ibuf_set)


---
### ibuf_set_h32<!-- {{#callable:ibuf_set_h32}} -->
Sets a 32-bit unsigned integer value at a specified position in a buffer.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer where the value will be set.
    - `pos`: The position in the buffer where the 32-bit value will be written.
    - `value`: The 64-bit unsigned integer value to be set, which will be truncated to 32 bits.
- **Control Flow**:
    - Checks if the provided `value` exceeds the maximum value for a 32-bit unsigned integer (`UINT32_MAX`).
    - If the value is too large, sets `errno` to `EINVAL` and returns -1.
    - Truncates the `value` to a 32-bit unsigned integer and stores it in `v`.
    - Calls the [`ibuf_set`](#ibuf_set) function to write `v` into the buffer at the specified position.
- **Output**:
    - Returns 0 on success, or -1 if an error occurred (e.g., if the value is too large).
- **Functions called**:
    - [`ibuf_set`](#ibuf_set)


---
### ibuf_set_h64<!-- {{#callable:ibuf_set_h64}} -->
Sets a 64-bit value at a specified position in an `ibuf` buffer.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure where the value will be set.
    - `pos`: The position in the buffer where the 64-bit value will be written.
    - `value`: The 64-bit unsigned integer value to be set in the buffer.
- **Control Flow**:
    - Calls the [`ibuf_set`](#ibuf_set) function to write the 64-bit value to the specified position in the buffer.
    - The [`ibuf_set`](#ibuf_set) function handles the actual memory operations and checks for errors.
- **Output**:
    - Returns the result of the [`ibuf_set`](#ibuf_set) function, which indicates success (0) or failure (-1).
- **Functions called**:
    - [`ibuf_set`](#ibuf_set)


---
### ibuf_data<!-- {{#callable:ibuf_data}} -->
The `ibuf_data` function returns a pointer to the current read position in the buffer of an `ibuf` structure.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure from which the data is to be accessed.
- **Control Flow**:
    - The function directly accesses the `buf` member of the `ibuf` structure.
    - It calculates the address by adding the `rpos` (read position) to the base address of the buffer (`buf->buf`).
    - The resulting pointer is returned without any additional checks or conditions.
- **Output**:
    - The function outputs a pointer to the data in the buffer starting from the current read position.


---
### ibuf_size<!-- {{#callable:ibuf_size}} -->
Calculates the size of the data in an `ibuf` structure.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure whose size is to be calculated.
- **Control Flow**:
    - The function computes the size by subtracting the read position (`rpos`) from the write position (`wpos`) of the `ibuf` structure.
    - It directly returns the result of this subtraction.
- **Output**:
    - Returns the size of the data currently available in the buffer, which is the difference between the write position and the read position.


---
### ibuf_left<!-- {{#callable:ibuf_left}} -->
The `ibuf_left` function calculates the remaining writable space in an input buffer.
- **Inputs**:
    - `buf`: A pointer to a `struct ibuf` that represents the input buffer.
- **Control Flow**:
    - The function first checks if the buffer's file descriptor (`fd`) is marked as being on the stack using the constant `IBUF_FD_MARK_ON_STACK`.
    - If the buffer is on the stack, it returns 0, indicating no space is left.
    - If the buffer is not on the stack, it calculates the remaining space by subtracting the current write position (`wpos`) from the maximum size of the buffer (`max`).
- **Output**:
    - Returns the number of bytes left that can be written to the buffer, or 0 if the buffer is on the stack.


---
### ibuf_truncate<!-- {{#callable:ibuf_truncate}} -->
The `ibuf_truncate` function adjusts the write position of an `ibuf` structure to truncate its content to a specified length.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure that is to be truncated.
    - `len`: The new length to which the buffer should be truncated.
- **Control Flow**:
    - The function first checks if the current size of the buffer is greater than or equal to the specified length.
    - If the current size is sufficient, it sets the write position (`wpos`) to the read position (`rpos`) plus the new length and returns 0.
    - If the buffer is marked as being on the stack (indicated by `IBUF_FD_MARK_ON_STACK`), it sets `errno` to `ERANGE` and returns -1, as truncation is not allowed for stack buffers.
    - If the buffer size is less than the specified length and it is not a stack buffer, it calls [`ibuf_add_zero`](#ibuf_add_zero) to add zero bytes to the buffer to reach the desired length.
- **Output**:
    - The function returns 0 on success, indicating that the buffer has been truncated successfully, or -1 on failure, with `errno` set to indicate the error.
- **Functions called**:
    - [`ibuf_size`](#ibuf_size)
    - [`ibuf_add_zero`](#ibuf_add_zero)


---
### ibuf_rewind<!-- {{#callable:ibuf_rewind}} -->
Resets the read position of the `ibuf` structure to the beginning.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure that represents the input buffer whose read position is to be reset.
- **Control Flow**:
    - The function directly accesses the `rpos` member of the `ibuf` structure.
    - It sets `buf->rpos` to 0, effectively rewinding the read position.
- **Output**:
    - The function does not return a value; it modifies the state of the `ibuf` structure in place.


---
### ibuf_close<!-- {{#callable:ibuf_close}} -->
The `ibuf_close` function enqueues a buffer into a message buffer for later processing.
- **Inputs**:
    - `msgbuf`: A pointer to a `struct msgbuf` that represents the message buffer to which the `ibuf` will be added.
    - `buf`: A pointer to a `struct ibuf` that represents the input buffer to be enqueued.
- **Control Flow**:
    - The function calls [`msgbuf_enqueue`](#msgbuf_enqueue) with the provided `msgbuf` and `buf` as arguments.
    - The [`msgbuf_enqueue`](#msgbuf_enqueue) function inserts the `buf` into the tail of the `msgbuf`'s queue and increments the queued count.
- **Output**:
    - The function does not return a value; it modifies the state of the `msgbuf` by adding the `buf` to its queue.
- **Functions called**:
    - [`msgbuf_enqueue`](#msgbuf_enqueue)


---
### ibuf_from_buffer<!-- {{#callable:ibuf_from_buffer}} -->
Initializes an `ibuf` structure from a given data buffer.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure that will be initialized.
    - `data`: A pointer to the data buffer that will be assigned to the `ibuf`.
    - `len`: The length of the data buffer, which will be set as the size and write position of the `ibuf`.
- **Control Flow**:
    - The function starts by zeroing out the memory of the `ibuf` structure pointed to by `buf` using `memset`.
    - It then assigns the `data` pointer to the `buf->buf` field of the `ibuf`.
    - The `size` and `wpos` fields of the `ibuf` are set to the value of `len`.
    - Finally, it sets the `fd` field of the `ibuf` to a constant value indicating that the buffer is on the stack.
- **Output**:
    - The function does not return a value; it modifies the `ibuf` structure in place.


---
### ibuf_from_ibuf<!-- {{#callable:ibuf_from_ibuf}} -->
The `ibuf_from_ibuf` function initializes a destination `ibuf` structure using the data and size from a source `ibuf`.
- **Inputs**:
    - `buf`: A pointer to the destination `ibuf` structure that will be initialized.
    - `from`: A pointer to the source `ibuf` structure from which data will be copied.
- **Control Flow**:
    - The function calls [`ibuf_from_buffer`](#ibuf_from_buffer), passing the destination buffer, the data from the source buffer, and the size of the source buffer.
    - The [`ibuf_data`](#ibuf_data) function retrieves the data pointer from the source `ibuf`.
    - The [`ibuf_size`](#ibuf_size) function retrieves the size of the data in the source `ibuf`.
- **Output**:
    - The function does not return a value; it modifies the destination `ibuf` in place to reference the data from the source `ibuf`.
- **Functions called**:
    - [`ibuf_from_buffer`](#ibuf_from_buffer)
    - [`ibuf_data`](#ibuf_data)
    - [`ibuf_size`](#ibuf_size)


---
### ibuf_get<!-- {{#callable:ibuf_get}} -->
The `ibuf_get` function retrieves a specified number of bytes from an input buffer into a provided data buffer.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure from which data is to be read.
    - `data`: A pointer to the memory location where the retrieved data will be stored.
    - `len`: The number of bytes to read from the input buffer.
- **Control Flow**:
    - The function first checks if the size of the data available in the `buf` is less than `len`.
    - If there is insufficient data, it sets `errno` to `EBADMSG` and returns -1.
    - If sufficient data is available, it copies `len` bytes from the buffer's current read position to the `data` pointer using `memcpy`.
    - The read position (`rpos`) of the buffer is then incremented by `len`.
    - Finally, the function returns 0 to indicate success.
- **Output**:
    - The function returns 0 on success, or -1 if there is an error, with `errno` set to indicate the specific error.
- **Functions called**:
    - [`ibuf_size`](#ibuf_size)
    - [`ibuf_data`](#ibuf_data)


---
### ibuf_get_ibuf<!-- {{#callable:ibuf_get_ibuf}} -->
The `ibuf_get_ibuf` function retrieves a specified length of data from an input buffer and stores it in a new buffer.
- **Inputs**:
    - `buf`: A pointer to the source `ibuf` structure from which data is to be read.
    - `len`: The number of bytes to read from the source buffer.
    - `new`: A pointer to the destination `ibuf` structure where the read data will be stored.
- **Control Flow**:
    - The function first checks if the size of the source buffer (`buf`) is less than the specified length (`len`).
    - If the size is insufficient, it sets the `errno` to `EBADMSG` and returns -1 to indicate an error.
    - If the size is sufficient, it calls [`ibuf_from_buffer`](#ibuf_from_buffer) to initialize the `new` buffer with the data from the source buffer.
    - Finally, it updates the read position (`rpos`) of the source buffer by incrementing it by `len` and returns 0 to indicate success.
- **Output**:
    - The function returns 0 on success, or -1 on failure, with `errno` set to indicate the error.
- **Functions called**:
    - [`ibuf_size`](#ibuf_size)
    - [`ibuf_from_buffer`](#ibuf_from_buffer)
    - [`ibuf_data`](#ibuf_data)


---
### ibuf_get_h16<!-- {{#callable:ibuf_get_h16}} -->
Retrieves a 16-bit unsigned integer from the specified `ibuf` buffer.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure from which the value is to be read.
    - `value`: A pointer to a `uint16_t` variable where the retrieved value will be stored.
- **Control Flow**:
    - Calls the [`ibuf_get`](#ibuf_get) function to read data from the `buf` into the memory location pointed to by `value`.
    - The size of the data to read is determined by the size of `*value`, which is 2 bytes for a 16-bit integer.
- **Output**:
    - Returns 0 on success, or -1 if there is an error (e.g., if there is not enough data in the buffer).
- **Functions called**:
    - [`ibuf_get`](#ibuf_get)


---
### ibuf_get_h32<!-- {{#callable:ibuf_get_h32}} -->
Retrieves a 32-bit unsigned integer from the specified `ibuf` buffer.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure from which the value will be read.
    - `value`: A pointer to a `uint32_t` variable where the retrieved value will be stored.
- **Control Flow**:
    - Calls the [`ibuf_get`](#ibuf_get) function to read data from the `buf` into the memory location pointed to by `value`.
    - The size of the data to be read is determined by the size of the `value` variable.
- **Output**:
    - Returns 0 on success, or -1 if there is an error during the read operation, such as insufficient data in the buffer.
- **Functions called**:
    - [`ibuf_get`](#ibuf_get)


---
### ibuf_get_h64<!-- {{#callable:ibuf_get_h64}} -->
Retrieves a 64-bit unsigned integer from the specified `ibuf` buffer.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure from which the 64-bit value will be read.
    - `value`: A pointer to a `uint64_t` variable where the retrieved value will be stored.
- **Control Flow**:
    - Calls the [`ibuf_get`](#ibuf_get) function to read data from the `buf` into the memory location pointed to by `value`.
    - The size of the data to be read is determined by the size of the `uint64_t` type.
- **Output**:
    - Returns 0 on success, or -1 if there is an error during the read operation, with the error code set in `errno`.
- **Functions called**:
    - [`ibuf_get`](#ibuf_get)


---
### ibuf_get_n8<!-- {{#callable:ibuf_get_n8}} -->
Retrieves an 8-bit unsigned integer from the specified input buffer.
- **Inputs**:
    - `buf`: A pointer to a `struct ibuf` that represents the input buffer from which the value will be read.
    - `value`: A pointer to a `uint8_t` variable where the retrieved 8-bit unsigned integer will be stored.
- **Control Flow**:
    - Calls the [`ibuf_get`](#ibuf_get) function to read data from the buffer.
    - Passes the size of the `uint8_t` type to [`ibuf_get`](#ibuf_get) to ensure the correct amount of data is read.
- **Output**:
    - Returns 0 on success, or -1 if an error occurs during the read operation, with the error code set in `errno`.
- **Functions called**:
    - [`ibuf_get`](#ibuf_get)


---
### ibuf_get_n16<!-- {{#callable:ibuf_get_n16}} -->
The `ibuf_get_n16` function retrieves a 16-bit unsigned integer from a buffer and converts it from big-endian to host byte order.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure from which the 16-bit value will be read.
    - `value`: A pointer to a `uint16_t` variable where the retrieved value will be stored.
- **Control Flow**:
    - Calls the [`ibuf_get`](#ibuf_get) function to read `sizeof(*value)` bytes from the buffer into the location pointed to by `value`.
    - If the read operation is successful, it converts the value from big-endian to host byte order using `be16toh`.
    - Returns the result of the [`ibuf_get`](#ibuf_get) function, which indicates success or failure.
- **Output**:
    - Returns an integer indicating the success (0) or failure (-1) of the read operation.
- **Functions called**:
    - [`ibuf_get`](#ibuf_get)


---
### ibuf_get_n32<!-- {{#callable:ibuf_get_n32}} -->
The `ibuf_get_n32` function retrieves a 32-bit unsigned integer from a buffer and converts it from big-endian to host byte order.
- **Inputs**:
    - `buf`: A pointer to a `struct ibuf` that represents the input buffer from which the value will be read.
    - `value`: A pointer to a `uint32_t` variable where the retrieved value will be stored.
- **Control Flow**:
    - The function first calls [`ibuf_get`](#ibuf_get) to read `sizeof(*value)` bytes from the `buf` into the memory location pointed to by `value`.
    - If the read operation is successful, it then converts the retrieved value from big-endian to host byte order using `be32toh`.
    - Finally, it returns the result of the [`ibuf_get`](#ibuf_get) call, which indicates success or failure.
- **Output**:
    - The function returns an integer indicating the success (0) or failure (-1) of the read operation.
- **Functions called**:
    - [`ibuf_get`](#ibuf_get)


---
### ibuf_get_n64<!-- {{#callable:ibuf_get_n64}} -->
Retrieves a 64-bit unsigned integer from a buffer and converts it from big-endian to host byte order.
- **Inputs**:
    - `buf`: A pointer to a `struct ibuf` that represents the input buffer from which the value will be read.
    - `value`: A pointer to a `uint64_t` variable where the retrieved value will be stored.
- **Control Flow**:
    - Calls the [`ibuf_get`](#ibuf_get) function to read the raw 64-bit value from the buffer.
    - If the read operation is successful, it converts the value from big-endian to host byte order using `be64toh`.
    - Returns the result of the [`ibuf_get`](#ibuf_get) function, indicating success or failure.
- **Output**:
    - Returns an integer indicating the success (0) or failure (-1) of the operation.
- **Functions called**:
    - [`ibuf_get`](#ibuf_get)


---
### ibuf_get_string<!-- {{#callable:ibuf_get_string}} -->
Retrieves a string of a specified length from an `ibuf` buffer.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure from which the string will be retrieved.
    - `len`: The length of the string to retrieve from the buffer.
- **Control Flow**:
    - Checks if the available size in the `ibuf` is less than the requested length; if so, sets `errno` to `EBADMSG` and returns NULL.
    - If sufficient data is available, it uses `strndup` to duplicate the string from the buffer's data.
    - If `strndup` fails, it returns NULL.
    - If successful, it increments the read position (`rpos`) of the buffer by the length of the string retrieved and returns the duplicated string.
- **Output**:
    - Returns a pointer to the duplicated string if successful, or NULL if there is an error.
- **Functions called**:
    - [`ibuf_size`](#ibuf_size)
    - [`ibuf_data`](#ibuf_data)


---
### ibuf_skip<!-- {{#callable:ibuf_skip}} -->
The `ibuf_skip` function advances the read position of an input buffer by a specified length.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the input buffer from which data is being skipped.
    - `len`: The number of bytes to skip from the current read position in the buffer.
- **Control Flow**:
    - The function first checks if the current size of the buffer (from the [`ibuf_size`](#ibuf_size) function) is less than the specified length to skip.
    - If the buffer size is insufficient, it sets the `errno` to `EBADMSG` and returns -1 to indicate an error.
    - If the buffer size is sufficient, it increments the read position (`rpos`) of the buffer by the specified length.
    - Finally, it returns 0 to indicate success.
- **Output**:
    - The function returns 0 on success, or -1 if there is an error due to insufficient data in the buffer, with `errno` set to `EBADMSG`.
- **Functions called**:
    - [`ibuf_size`](#ibuf_size)


---
### ibuf_free<!-- {{#callable:ibuf_free}} -->
The `ibuf_free` function deallocates memory associated with an `ibuf` structure.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure that needs to be freed.
- **Control Flow**:
    - Checks if the `buf` pointer is NULL; if so, the function returns immediately without doing anything.
    - Checks if the `fd` member of the `buf` is marked as `IBUF_FD_MARK_ON_STACK`, and if so, it calls `abort()` to terminate the program, preventing further harm.
    - If the `fd` is a valid file descriptor (greater than or equal to 0), it closes the file descriptor using `close(buf->fd)`.
    - Calls `freezero` to securely free the memory allocated for the buffer pointed to by `buf->buf`, ensuring that the memory is zeroed out before being freed.
    - Finally, it frees the `buf` structure itself using `free(buf)`.
- **Output**:
    - This function does not return a value; it performs cleanup operations on the provided `ibuf` structure.


---
### ibuf_fd_avail<!-- {{#callable:ibuf_fd_avail}} -->
The `ibuf_fd_avail` function checks if the file descriptor associated with the given `ibuf` structure is valid.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure that contains the file descriptor to be checked.
- **Control Flow**:
    - The function evaluates the condition `buf->fd >= 0`.
    - If the condition is true, it indicates that the file descriptor is valid (non-negative).
    - The function returns the result of this evaluation as an integer.
- **Output**:
    - Returns 1 if the file descriptor is valid (greater than or equal to 0), otherwise returns 0.


---
### ibuf_fd_get<!-- {{#callable:ibuf_fd_get}} -->
Retrieves and clears the file descriptor from the `ibuf` structure.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure from which the file descriptor is to be retrieved.
- **Control Flow**:
    - Checks if the file descriptor (`fd`) in the `ibuf` structure is negative, which indicates it is for internal use.
    - If the `fd` is negative, the function returns -1.
    - If the `fd` is valid (non-negative), it stores the current `fd` in a local variable, sets the `fd` in the `ibuf` structure to -1, and returns the stored `fd`.
- **Output**:
    - Returns the file descriptor if it is valid; otherwise, returns -1.


---
### ibuf_fd_set<!-- {{#callable:ibuf_fd_set}} -->
Sets the file descriptor for a given `ibuf` structure, ensuring proper resource management.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure that holds the file descriptor.
    - `fd`: An integer representing the file descriptor to be set; if negative, it indicates no file descriptor.
- **Control Flow**:
    - Checks if the `ibuf` is marked as living on the stack; if so, it calls `abort()` to prevent further issues.
    - If the current file descriptor in `buf` is valid (non-negative), it closes that file descriptor.
    - Sets the file descriptor in `buf` to -1 to indicate no valid file descriptor.
    - If the provided `fd` is non-negative, it assigns this value to `buf->fd`.
- **Output**:
    - The function does not return a value; it modifies the `ibuf` structure in place to update its file descriptor.


---
### msgbuf_new<!-- {{#callable:msgbuf_new}} -->
Creates and initializes a new `msgbuf` structure.
- **Inputs**:
    - None
- **Control Flow**:
    - Allocates memory for a new `msgbuf` structure using `calloc`.
    - Checks if the memory allocation was successful; if not, returns NULL.
    - Initializes the `queued` field of the `msgbuf` to 0.
    - Initializes the `bufs` and `rbufs` fields as empty tail queues using `TAILQ_INIT`.
    - Returns the pointer to the newly created and initialized `msgbuf`.
- **Output**:
    - Returns a pointer to the newly created `msgbuf` structure, or NULL if memory allocation fails.


---
### msgbuf_new_reader<!-- {{#callable:msgbuf_new_reader}} -->
Creates a new `msgbuf` reader with specified header size and a function to read headers.
- **Inputs**:
    - `hdrsz`: The size of the header to be read, which must be greater than 0 and less than or equal to half of `IBUF_READ_SIZE`.
    - `readhdr`: A pointer to a function that takes an `ibuf` and additional arguments to read the header.
    - `arg`: A pointer to user-defined data that will be passed to the `readhdr` function.
- **Control Flow**:
    - Checks if `hdrsz` is valid; if not, sets `errno` to `EINVAL` and returns NULL.
    - Attempts to allocate memory for a buffer of size `IBUF_READ_SIZE`; if allocation fails, returns NULL.
    - Calls `msgbuf_new()` to create a new `msgbuf`; if this fails, frees the allocated buffer and returns NULL.
    - Sets the `rbuf`, `hdrsize`, `readhdr`, and `rarg` fields of the `msgbuf` structure.
    - Returns the initialized `msgbuf` structure.
- **Output**:
    - Returns a pointer to a newly created `msgbuf` structure if successful, or NULL if an error occurs.
- **Functions called**:
    - [`msgbuf_new`](#msgbuf_new)


---
### msgbuf_free<!-- {{#callable:msgbuf_free}} -->
The `msgbuf_free` function deallocates memory associated with a `msgbuf` structure.
- **Inputs**:
    - `msgbuf`: A pointer to a `struct msgbuf` that is to be freed.
- **Control Flow**:
    - Checks if the `msgbuf` pointer is NULL; if it is, the function returns immediately without doing anything.
    - Calls [`msgbuf_clear`](#msgbuf_clear) to clear any data associated with the `msgbuf` before freeing.
    - Frees the memory allocated for the `rbuf` member of the `msgbuf` structure.
    - Finally, frees the memory allocated for the `msgbuf` structure itself.
- **Output**:
    - The function does not return a value; it performs memory deallocation.
- **Functions called**:
    - [`msgbuf_clear`](#msgbuf_clear)


---
### msgbuf_queuelen<!-- {{#callable:msgbuf_queuelen}} -->
The `msgbuf_queuelen` function returns the number of queued messages in a `msgbuf` structure.
- **Inputs**:
    - `msgbuf`: A pointer to a `struct msgbuf`, which contains information about the message buffer including the count of queued messages.
- **Control Flow**:
    - The function directly accesses the `queued` member of the `msgbuf` structure.
    - It returns the value of `msgbuf->queued` without any additional processing or checks.
- **Output**:
    - Returns a `uint32_t` representing the number of messages currently queued in the `msgbuf`.


---
### msgbuf_clear<!-- {{#callable:msgbuf_clear}} -->
Clears all buffers in a `msgbuf` structure, resetting its state.
- **Inputs**:
    - `msgbuf`: A pointer to a `msgbuf` structure that contains the buffers to be cleared.
- **Control Flow**:
    - The function first processes the write side by dequeuing all buffers from the `bufs` list until it is empty.
    - It sets the `queued` count to zero after clearing the write buffers.
    - Next, it processes the read side by removing each buffer from the `rbufs` list and freeing them until the list is empty.
    - Finally, it resets the read offset `roff` to zero and frees the `rpmsg` buffer, setting it to NULL.
- **Output**:
    - The function does not return a value; it modifies the state of the `msgbuf` structure directly.
- **Functions called**:
    - [`msgbuf_dequeue`](#msgbuf_dequeue)
    - [`ibuf_free`](#ibuf_free)


---
### msgbuf_get<!-- {{#callable:msgbuf_get}} -->
The `msgbuf_get` function retrieves and removes the first `ibuf` from the `rbufs` queue of a `msgbuf`.
- **Inputs**:
    - `msgbuf`: A pointer to a `msgbuf` structure from which the first `ibuf` is to be retrieved.
- **Control Flow**:
    - The function checks if the `rbufs` queue of the `msgbuf` is not empty by attempting to get the first `ibuf` using `TAILQ_FIRST`.
    - If an `ibuf` is found, it is removed from the `rbufs` queue using `TAILQ_REMOVE`.
    - The function then returns the retrieved `ibuf`, which may be NULL if the queue was empty.
- **Output**:
    - Returns a pointer to the retrieved `ibuf` if available, or NULL if the `rbufs` queue was empty.


---
### ibuf_write<!-- {{#callable:ibuf_write}} -->
The `ibuf_write` function writes data from a message buffer to a specified file descriptor.
- **Inputs**:
    - `fd`: An integer representing the file descriptor to which data will be written.
    - `msgbuf`: A pointer to a `msgbuf` structure containing the data to be written.
- **Control Flow**:
    - Initializes an array of `iovec` structures to hold the data to be written.
    - Iterates over the buffers in the `msgbuf` structure, populating the `iovec` array with pointers to the data and their lengths.
    - If no buffers are found, returns 0 indicating nothing was queued.
    - Attempts to write the data using `writev`, handling interruptions and specific errors (EINTR, EAGAIN, ENOBUFS) by retrying or returning appropriate error codes.
    - Drains the written data from the `msgbuf` based on the number of bytes successfully written.
- **Output**:
    - Returns 0 on success, -1 on failure (with errno set), or 0 if there was nothing to write.
- **Functions called**:
    - [`ibuf_data`](#ibuf_data)
    - [`ibuf_size`](#ibuf_size)
    - [`msgbuf_drain`](#msgbuf_drain)


---
### msgbuf_write<!-- {{#callable:msgbuf_write}} -->
The `msgbuf_write` function sends messages from a `msgbuf` structure to a specified file descriptor.
- **Inputs**:
    - `fd`: An integer representing the file descriptor to which the messages will be sent.
    - `msgbuf`: A pointer to a `msgbuf` structure containing the messages to be sent.
- **Control Flow**:
    - Initializes an array of `iovec` structures and other necessary variables.
    - Iterates through the buffers in the `msgbuf` to populate the `iovec` array with message data.
    - Checks if any buffers are available to send; if none, returns 0.
    - Sets up the `msghdr` structure for sending messages, including control message for file descriptor passing if applicable.
    - Attempts to send the message using `sendmsg`, retrying on interruption or specific errors.
    - If a file descriptor was sent, it is closed after sending.
    - Drains the sent messages from the `msgbuf` based on the number of bytes sent.
- **Output**:
    - Returns 0 on success, -1 on failure with errno set appropriately, or 0 if no messages were queued.
- **Functions called**:
    - [`ibuf_data`](#ibuf_data)
    - [`ibuf_size`](#ibuf_size)
    - [`msgbuf_drain`](#msgbuf_drain)


---
### ibuf_read_process<!-- {{#callable:ibuf_read_process}} -->
Processes incoming data from a file descriptor and populates a message buffer.
- **Inputs**:
    - `msgbuf`: A pointer to a `msgbuf` structure that holds the message buffer and related metadata.
    - `fd`: An integer file descriptor from which data is read.
- **Control Flow**:
    - Initializes a read buffer `rbuf` from the `msgbuf`'s internal buffer.
    - Enters a loop that continues as long as there is data in `rbuf`.
    - Checks if there is a message pointer `rpmsg`; if not, it attempts to read the header size from `rbuf`.
    - If the header is successfully read, it calls the `readhdr` function to populate `rpmsg`.
    - Determines the size of data to read based on the remaining size in `rpmsg` and `rbuf`.
    - Attempts to extract data from `rbuf` and add it to `rpmsg`.
    - If `rpmsg` is fully populated, it enqueues it for further processing and resets `rpmsg`.
    - After processing, if there is remaining data in `rbuf`, it moves that data to the start of the buffer.
    - Updates the read offset `roff` in `msgbuf` and closes the file descriptor if it is valid.
    - If any operation fails, it jumps to the fail label to handle cleanup.
- **Output**:
    - Returns 1 on successful processing of data, or -1 if an error occurs during the process.
- **Functions called**:
    - [`ibuf_from_buffer`](#ibuf_from_buffer)
    - [`ibuf_size`](#ibuf_size)
    - [`ibuf_data`](#ibuf_data)
    - [`ibuf_left`](#ibuf_left)
    - [`ibuf_get_ibuf`](#ibuf_get_ibuf)
    - [`ibuf_add_ibuf`](#ibuf_add_ibuf)
    - [`msgbuf_read_enqueue`](#msgbuf_read_enqueue)


---
### ibuf_read<!-- {{#callable:ibuf_read}} -->
The `ibuf_read` function reads data from a file descriptor into a message buffer.
- **Inputs**:
    - `fd`: An integer file descriptor from which data will be read.
    - `msgbuf`: A pointer to a `struct msgbuf` that contains the buffer where the read data will be stored.
- **Control Flow**:
    - Checks if the `rbuf` in `msgbuf` is NULL; if so, sets `errno` to EINVAL and returns -1.
    - Sets up an `iovec` structure to point to the buffer in `msgbuf` where data will be read.
    - Enters a loop to attempt reading data using `readv` until successful or an error occurs.
    - If `readv` returns -1 due to EINTR, it retries the read operation.
    - If `readv` returns -1 due to EAGAIN, it returns 1 to indicate that the operation should be retried later.
    - If `readv` returns 0, it indicates that the connection is closed, and the function returns 0.
    - Updates the read offset in `msgbuf` and calls [`ibuf_read_process`](#ibuf_read_process) to process the newly arrived data.
- **Output**:
    - Returns the number of bytes read, or -1 on error, or 0 if the connection is closed.
- **Functions called**:
    - [`ibuf_read_process`](#ibuf_read_process)


---
### msgbuf_read<!-- {{#callable:msgbuf_read}} -->
The `msgbuf_read` function reads messages from a socket and processes them into a message buffer.
- **Inputs**:
    - `fd`: An integer file descriptor representing the socket from which to read messages.
    - `msgbuf`: A pointer to a `msgbuf` structure that holds the message buffer where the read data will be stored.
- **Control Flow**:
    - Checks if the `rbuf` in `msgbuf` is NULL, returning an error if it is.
    - Initializes a `msghdr` structure and a control message buffer.
    - Sets up an `iovec` structure to point to the buffer where data will be read.
    - Enters a loop to call `recvmsg` to read data from the socket, handling interruptions and errors appropriately.
    - If data is received, it updates the read offset in `msgbuf` and processes any ancillary data (like file descriptors) received with the message.
    - Calls [`ibuf_read_process`](#ibuf_read_process) to handle the newly arrived data and potentially enqueue it for further processing.
- **Output**:
    - Returns the number of bytes processed, 0 if the connection is closed, or -1 on error.
- **Functions called**:
    - [`ibuf_read_process`](#ibuf_read_process)


---
### msgbuf_read_enqueue<!-- {{#callable:msgbuf_read_enqueue}} -->
The `msgbuf_read_enqueue` function adds a buffer to the end of the read buffers queue in a `msgbuf` structure.
- **Inputs**:
    - `msgbuf`: A pointer to a `struct msgbuf` that represents the message buffer to which the read buffer will be added.
    - `buf`: A pointer to a `struct ibuf` that represents the buffer to be enqueued.
- **Control Flow**:
    - The function first checks if the `buf` is marked as living on the stack by comparing its `fd` field to `IBUF_FD_MARK_ON_STACK`.
    - If `buf` is on the stack, the function calls `abort()` to terminate the program to prevent potential memory issues.
    - If the check passes, the function uses `TAILQ_INSERT_TAIL` to add `buf` to the end of the `rbufs` queue in the `msgbuf` structure.
- **Output**:
    - The function does not return a value; it modifies the state of the `msgbuf` by adding the `buf` to its read buffers queue.


---
### msgbuf_enqueue<!-- {{#callable:msgbuf_enqueue}} -->
The `msgbuf_enqueue` function adds a buffer to the end of a message buffer queue.
- **Inputs**:
    - `msgbuf`: A pointer to a `msgbuf` structure that represents the message buffer to which the buffer will be added.
    - `buf`: A pointer to an `ibuf` structure that represents the buffer to be enqueued.
- **Control Flow**:
    - Checks if the `buf` is marked as living on the stack by comparing its `fd` to `IBUF_FD_MARK_ON_STACK`.
    - If the check fails, the function calls `abort()` to terminate the program.
    - If the check passes, the `buf` is inserted at the tail of the `msgbuf`'s buffer queue using `TAILQ_INSERT_TAIL`.
    - The `queued` count in the `msgbuf` is incremented by one.
- **Output**:
    - The function does not return a value; it modifies the state of the `msgbuf` by adding the `buf` to its queue and updating the count of queued buffers.


---
### msgbuf_dequeue<!-- {{#callable:msgbuf_dequeue}} -->
The `msgbuf_dequeue` function removes a specified `ibuf` from a `msgbuf` and frees its memory.
- **Inputs**:
    - `msgbuf`: A pointer to a `struct msgbuf` which contains a queue of `ibuf` structures.
    - `buf`: A pointer to the `struct ibuf` that is to be removed from the `msgbuf`.
- **Control Flow**:
    - The function first removes the specified `buf` from the `msgbuf`'s queue using `TAILQ_REMOVE`.
    - It then decrements the `queued` count in the `msgbuf` structure to reflect the removal.
    - Finally, it calls [`ibuf_free`](#ibuf_free) to release the memory allocated for the `buf`.
- **Output**:
    - The function does not return a value; it modifies the state of the `msgbuf` and frees the memory of the `buf`.
- **Functions called**:
    - [`ibuf_free`](#ibuf_free)


---
### msgbuf_drain<!-- {{#callable:msgbuf_drain}} -->
`msgbuf_drain` removes data from the message buffer by adjusting the read position of the buffers based on the specified number of bytes.
- **Inputs**:
    - `msgbuf`: A pointer to a `msgbuf` structure that contains a queue of message buffers.
    - `n`: A size_t value representing the number of bytes to drain from the message buffers.
- **Control Flow**:
    - The function starts by initializing a pointer `buf` to the first buffer in the `msgbuf`'s queue.
    - It enters a loop that continues as long as there are buffers in the queue and `n` is greater than zero.
    - Within the loop, it retrieves the next buffer and checks if the remaining bytes `n` is greater than or equal to the size of the current buffer.
    - If `n` is sufficient to remove the entire buffer, it decrements `n` by the size of the buffer and dequeues the buffer from the `msgbuf`.
    - If `n` is less than the size of the buffer, it adjusts the read position (`rpos`) of the current buffer by `n` and sets `n` to zero, effectively draining the specified bytes.
- **Output**:
    - The function does not return a value; it modifies the state of the `msgbuf` by adjusting the read positions of the buffers and potentially removing buffers from the queue.
- **Functions called**:
    - [`ibuf_size`](#ibuf_size)
    - [`msgbuf_dequeue`](#msgbuf_dequeue)


# Function Declarations (Public API)

---
### msgbuf_read_enqueue<!-- {{#callable_declaration:msgbuf_read_enqueue}} -->
Enqueues a buffer for reading in a message buffer.
- **Description**: Use this function to add a buffer to the read queue of a message buffer, ensuring that the buffer is not marked as residing on the stack. This function is typically called when a buffer is ready to be processed for reading. It is important to ensure that the buffer is not marked with the special file descriptor value indicating it is on the stack, as this will cause the program to abort. This function does not return a value and is used for managing the internal queue of buffers to be read.
- **Inputs**:
    - `msgbuf`: A pointer to a 'struct msgbuf' that represents the message buffer where the buffer will be enqueued. Must not be null.
    - `buf`: A pointer to a 'struct ibuf' that represents the buffer to be enqueued. Must not be null and must not have its 'fd' field set to 'IBUF_FD_MARK_ON_STACK', as this will cause the function to abort.
- **Output**: None
- **See also**: [`msgbuf_read_enqueue`](#msgbuf_read_enqueue)  (Implementation)


---
### msgbuf_enqueue<!-- {{#callable_declaration:msgbuf_enqueue}} -->
Enqueues a buffer into a message buffer.
- **Description**: Use this function to add a buffer to the end of a message buffer's queue. This function should be called when you want to queue a buffer for processing or transmission. It is important to ensure that the buffer is not marked as living on the stack, as this will cause the function to abort. The function increments the queued count of the message buffer, indicating the number of buffers currently enqueued.
- **Inputs**:
    - `msgbuf`: A pointer to a `struct msgbuf` that represents the message buffer where the buffer will be enqueued. Must not be null.
    - `buf`: A pointer to a `struct ibuf` that represents the buffer to be enqueued. The buffer must not be marked as living on the stack (i.e., `buf->fd` must not be `IBUF_FD_MARK_ON_STACK`). The caller retains ownership of the buffer.
- **Output**: None
- **See also**: [`msgbuf_enqueue`](#msgbuf_enqueue)  (Implementation)


---
### msgbuf_dequeue<!-- {{#callable_declaration:msgbuf_dequeue}} -->
Removes a buffer from the message buffer queue.
- **Description**: This function is used to remove a specified buffer from the queue of a message buffer. It should be called when a buffer is no longer needed and should be deallocated. The function decreases the count of queued buffers and frees the memory associated with the buffer. It is important to ensure that the buffer being removed is part of the message buffer's queue to avoid undefined behavior.
- **Inputs**:
    - `msgbuf`: A pointer to the msgbuf structure from which the buffer will be removed. Must not be null and must be properly initialized.
    - `buf`: A pointer to the ibuf structure that is to be removed from the msgbuf's queue. Must not be null and must be part of the msgbuf's queue.
- **Output**: None
- **See also**: [`msgbuf_dequeue`](#msgbuf_dequeue)  (Implementation)


---
### msgbuf_drain<!-- {{#callable_declaration:msgbuf_drain}} -->
Removes a specified number of bytes from the message buffer.
- **Description**: This function is used to remove up to 'n' bytes from the message buffer's queue of buffers. It is typically called when data has been successfully processed or sent, and the corresponding space in the buffer can be freed. The function iterates over the buffers in the message buffer, removing entire buffers if 'n' is greater than or equal to the buffer's size, or partially draining a buffer if 'n' is smaller. It must be called with a valid 'msgbuf' that has been properly initialized and contains data to be drained.
- **Inputs**:
    - `msgbuf`: A pointer to a 'struct msgbuf' that contains the queue of buffers to be drained. Must not be null and should be properly initialized before calling this function.
    - `n`: The number of bytes to remove from the message buffer. Must be a non-negative value. If 'n' is larger than the total size of the buffers, all buffers will be removed.
- **Output**: None
- **See also**: [`msgbuf_drain`](#msgbuf_drain)  (Implementation)


