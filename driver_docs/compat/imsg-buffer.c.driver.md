# Purpose
The provided C source code file, `imsg-buffer.c`, is part of the OpenBSD project and is designed to handle inter-process communication (IPC) through message buffers. This file implements a set of functions for managing input and output buffers (`ibuf`) and message buffers (`msgbuf`). The primary functionality revolves around creating, manipulating, and transmitting data buffers, which are essential for efficient and reliable IPC. The code includes functions for opening, reserving, adding, setting, and retrieving data from buffers, as well as handling file descriptors associated with these buffers. It also provides mechanisms for reading from and writing to sockets using these buffers, which is crucial for network communication.

The file defines a comprehensive API for buffer management, including functions for dynamic buffer allocation, data serialization and deserialization, and buffer queue management. The `msgbuf` structure is used to manage a queue of `ibuf` structures, facilitating the reading and writing of messages over sockets. The code also includes functions for handling endian conversions, ensuring compatibility across different architectures. This file is intended to be part of a larger system, likely imported and used by other components that require IPC capabilities. It does not define a standalone executable but rather serves as a library providing essential services for message-based communication in networked applications.
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
    - `bufs`: A tail queue head for storing output buffers of type 'ibuf'.
    - `rbufs`: A tail queue head for storing input buffers of type 'ibuf'.
    - `queued`: A 32-bit unsigned integer representing the number of queued messages.
    - `rbuf`: A pointer to a character buffer used for reading data.
    - `rpmsg`: A pointer to an 'ibuf' structure representing the current message being processed.
    - `readhdr`: A function pointer for reading the header of a message from an 'ibuf'.
    - `rarg`: A void pointer to an argument passed to the 'readhdr' function.
    - `roff`: A size_t value indicating the current offset in the read buffer.
    - `hdrsize`: A size_t value representing the size of the message header.
- **Description**: The 'msgbuf' structure is designed to manage message buffers for input and output operations, particularly in network communication contexts. It maintains separate queues for input and output buffers, tracks the number of queued messages, and handles the reading and processing of message headers. The structure is equipped with function pointers and auxiliary data to facilitate flexible message handling and processing.


# Functions

---
### ibuf_open<!-- {{#callable:ibuf_open}} -->
The `ibuf_open` function allocates and initializes an `ibuf` structure with an optional buffer of a specified length.
- **Inputs**:
    - `len`: The size of the buffer to allocate, specified in bytes.
- **Control Flow**:
    - The function begins by attempting to allocate memory for an `ibuf` structure using `calloc`.
    - If memory allocation for the `ibuf` fails, it returns `NULL`.
    - If `len` is greater than 0, it attempts to allocate a buffer of size `len`.
    - If the buffer allocation fails, it frees the previously allocated `ibuf` and returns `NULL`.
    - The function sets the `size` and `max` fields of the `ibuf` to `len` and initializes the file descriptor `fd` to -1.
    - Finally, it returns a pointer to the initialized `ibuf` structure.
- **Output**:
    - Returns a pointer to the newly allocated and initialized `ibuf` structure, or `NULL` if memory allocation fails.


---
### ibuf_dynamic<!-- {{#callable:ibuf_dynamic}} -->
The `ibuf_dynamic` function allocates and initializes a dynamic input buffer with specified length and maximum size.
- **Inputs**:
    - `len`: The initial length of the buffer to allocate.
    - `max`: The maximum size that the buffer can grow to.
- **Control Flow**:
    - Checks if `max` is zero or less than `len`, setting `errno` to EINVAL and returning NULL if true.
    - Attempts to allocate memory for a new `ibuf` structure using `calloc`, returning NULL if allocation fails.
    - If `len` is greater than zero, attempts to allocate memory for the buffer of size `len`, freeing the `ibuf` structure if this allocation fails.
    - Sets the `size` and `max` fields of the `ibuf` structure to the provided `len` and `max` values, respectively, and initializes the `fd` field to -1.
    - Returns a pointer to the initialized `ibuf` structure.
- **Output**:
    - Returns a pointer to the newly allocated and initialized `ibuf` structure, or NULL if an error occurred.


---
### ibuf_reserve<!-- {{#callable:ibuf_reserve}} -->
The `ibuf_reserve` function reserves a specified amount of space in an `ibuf` structure, potentially reallocating memory if necessary.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure that represents the buffer in which space is to be reserved.
    - `len`: The size in bytes to reserve in the buffer.
- **Control Flow**:
    - The function first checks if the requested length exceeds the maximum allowable size based on the current write position.
    - It then checks if the buffer is marked as a stack buffer, which cannot be resized.
    - If the current write position plus the requested length exceeds the current buffer size, it checks if the buffer can grow beyond its maximum size.
    - If the buffer can grow, it reallocates memory for the buffer and initializes the newly allocated space to zero.
    - Finally, it updates the write position and returns a pointer to the newly reserved space.
- **Output**:
    - Returns a pointer to the reserved space in the buffer, or NULL if an error occurs, with the error code set in `errno`.


---
### ibuf_add<!-- {{#callable:ibuf_add}} -->
The `ibuf_add` function adds a specified amount of data to a buffer, reserving space as needed.
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
    - The function returns 0 on success, or -1 if there was an error during the operation.
- **Functions called**:
    - [`ibuf_reserve`](#ibuf_reserve)


---
### ibuf_add_ibuf<!-- {{#callable:ibuf_add_ibuf}} -->
The `ibuf_add_ibuf` function appends the contents of one `ibuf` structure to another.
- **Inputs**:
    - `buf`: A pointer to the destination `ibuf` structure where data will be added.
    - `from`: A pointer to the source `ibuf` structure whose data will be appended to the destination.
- **Control Flow**:
    - The function calls `ibuf_data(from)` to retrieve a pointer to the data contained in the `from` buffer.
    - It then calls `ibuf_size(from)` to get the size of the data in the `from` buffer.
    - Finally, it invokes [`ibuf_add`](#ibuf_add) to add the data from `from` to `buf`.
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
    - `value`: A 64-bit unsigned integer that is to be added to the buffer, which must be within the range of an 8-bit unsigned integer.
- **Control Flow**:
    - The function first checks if the provided `value` exceeds `UINT8_MAX` (255).
    - If the value is too large, it sets `errno` to `EINVAL` and returns -1 to indicate an error.
    - If the value is valid, it casts the value to an 8-bit unsigned integer (`uint8_t`) and stores it in `v`.
    - Finally, it calls the [`ibuf_add`](#ibuf_add) function to add the byte to the buffer.
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
    - The function first checks if the input `value` exceeds `UINT16_MAX`, and if so, sets `errno` to EINVAL and returns -1.
    - If the value is valid, it converts the value to big-endian format using `htobe16` and stores it in a local variable `v`.
    - Finally, it calls the [`ibuf_add`](#ibuf_add) function to add the converted value to the buffer.
- **Output**:
    - The function returns 0 on success, or -1 if an error occurs (e.g., if the input value is out of range).
- **Functions called**:
    - [`ibuf_add`](#ibuf_add)


---
### ibuf_add_n32<!-- {{#callable:ibuf_add_n32}} -->
The `ibuf_add_n32` function adds a 32-bit unsigned integer value to a buffer after converting it to big-endian format.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure where the data will be added.
    - `value`: A 64-bit unsigned integer value that is to be added to the buffer, which must not exceed UINT32_MAX.
- **Control Flow**:
    - Check if the input `value` exceeds UINT32_MAX; if it does, set `errno` to EINVAL and return -1.
    - Convert the `value` to big-endian format using `htobe32`.
    - Call the [`ibuf_add`](#ibuf_add) function to add the converted value to the buffer.
- **Output**:
    - Returns 0 on success, or -1 if an error occurs (e.g., if the value is too large).
- **Functions called**:
    - [`ibuf_add`](#ibuf_add)


---
### ibuf_add_n64<!-- {{#callable:ibuf_add_n64}} -->
The `ibuf_add_n64` function adds a 64-bit integer value to a buffer after converting it to big-endian format.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure where the value will be added.
    - `value`: A 64-bit unsigned integer value to be added to the buffer.
- **Control Flow**:
    - The function first converts the input `value` from host byte order to big-endian byte order using the `htobe64` macro.
    - It then calls the [`ibuf_add`](#ibuf_add) function, passing the buffer pointer, the address of the converted value, and the size of the value (8 bytes).
- **Output**:
    - The function returns the result of the [`ibuf_add`](#ibuf_add) function, which is typically 0 on success or -1 on failure.
- **Functions called**:
    - [`ibuf_add`](#ibuf_add)


---
### ibuf_add_h16<!-- {{#callable:ibuf_add_h16}} -->
The `ibuf_add_h16` function adds a 16-bit unsigned integer value to a buffer after validating its size.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure where the data will be added.
    - `value`: A 64-bit unsigned integer that is to be added to the buffer as a 16-bit value.
- **Control Flow**:
    - Check if the input `value` exceeds the maximum value for a 16-bit unsigned integer (UINT16_MAX).
    - If the value is too large, set `errno` to EINVAL and return -1 to indicate an error.
    - If the value is valid, cast it to a 16-bit unsigned integer and store it in `v`.
    - Call the [`ibuf_add`](#ibuf_add) function to add the 16-bit value to the buffer.
- **Output**:
    - Returns 0 on success, or -1 if an error occurred (e.g., if the input value is too large).
- **Functions called**:
    - [`ibuf_add`](#ibuf_add)


---
### ibuf_add_h32<!-- {{#callable:ibuf_add_h32}} -->
The `ibuf_add_h32` function adds a 32-bit unsigned integer value to a buffer after validating its size.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure where the data will be added.
    - `value`: A 64-bit unsigned integer that represents the value to be added, which must be less than or equal to UINT32_MAX.
- **Control Flow**:
    - The function first checks if the `value` exceeds `UINT32_MAX`.
    - If it does, it sets `errno` to `EINVAL` and returns -1 to indicate an error.
    - If the value is valid, it casts the `value` to a 32-bit unsigned integer and stores it in `v`.
    - Finally, it calls the [`ibuf_add`](#ibuf_add) function to add the 32-bit value to the buffer.
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
    - The function calls [`ibuf_add`](#ibuf_add) with the buffer, a pointer to the value, and the size of the value.
    - The [`ibuf_add`](#ibuf_add) function handles the actual addition of the value to the buffer.
- **Output**:
    - Returns the result of the [`ibuf_add`](#ibuf_add) function, which is typically 0 on success or -1 on failure.
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
    - The function returns 0 on success and -1 on failure, with the failure indicated by setting errno appropriately.
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
    - Returns a pointer to the start of the requested data within the buffer, or `NULL` if the request is out of bounds.
- **Functions called**:
    - [`ibuf_size`](#ibuf_size)


---
### ibuf_set<!-- {{#callable:ibuf_set}} -->
The `ibuf_set` function sets a specified portion of an `ibuf` buffer to a given data value.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure that represents the buffer to be modified.
    - `pos`: The position in the buffer where the data should be set.
    - `data`: A pointer to the data that will be copied into the buffer.
    - `len`: The length of the data to be copied into the buffer.
- **Control Flow**:
    - The function first calls [`ibuf_seek`](#ibuf_seek) to find the correct position in the buffer for writing the data.
    - If [`ibuf_seek`](#ibuf_seek) returns NULL, indicating an error in seeking, the function returns -1.
    - If the seek is successful, the function uses `memcpy` to copy the specified data into the buffer at the determined position.
    - Finally, the function returns 0 to indicate success.
- **Output**:
    - The function returns 0 on success and -1 on failure, indicating whether the data was successfully set in the buffer.
- **Functions called**:
    - [`ibuf_seek`](#ibuf_seek)


---
### ibuf_set_n8<!-- {{#callable:ibuf_set_n8}} -->
The `ibuf_set_n8` function sets an 8-bit unsigned integer value at a specified position in a buffer.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer where the value will be set.
    - `pos`: The position in the buffer where the 8-bit value will be written.
    - `value`: The 64-bit unsigned integer value to be set, which must be within the range of an 8-bit unsigned integer.
- **Control Flow**:
    - The function first checks if the provided `value` exceeds the maximum value for an 8-bit unsigned integer (255).
    - If the value is too large, it sets `errno` to EINVAL and returns -1 to indicate an error.
    - If the value is valid, it casts the value to a `uint8_t` and calls the [`ibuf_set`](#ibuf_set) function to write this value to the specified position in the buffer.
- **Output**:
    - The function returns 0 on success, indicating that the value was successfully set in the buffer, or -1 on failure if the value was out of range.
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
    - Checks if the provided `value` exceeds the maximum limit for a 16-bit unsigned integer (UINT16_MAX).
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
    - Checks if the provided `value` exceeds `UINT32_MAX`. If it does, sets `errno` to `EINVAL` and returns -1.
    - Converts the `value` from host byte order to big-endian format using `htobe32`.
    - Calls the [`ibuf_set`](#ibuf_set) function to write the converted value into the buffer at the specified position.
- **Output**:
    - Returns 0 on success, or -1 if an error occurred (e.g., if the value is out of range).
- **Functions called**:
    - [`ibuf_set`](#ibuf_set)


---
### ibuf_set_n64<!-- {{#callable:ibuf_set_n64}} -->
The `ibuf_set_n64` function sets a 64-bit integer value at a specified position in a buffer after converting it to big-endian format.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer where the value will be set.
    - `pos`: A size_t value indicating the position in the buffer where the 64-bit integer will be written.
    - `value`: A uint64_t value that will be converted to big-endian format before being written to the buffer.
- **Control Flow**:
    - The function first converts the input `value` from host byte order to big-endian byte order using the `htobe64` macro.
    - It then calls the [`ibuf_set`](#ibuf_set) function, passing the buffer, the specified position, a pointer to the converted value, and the size of the value.
- **Output**:
    - The function returns the result of the [`ibuf_set`](#ibuf_set) function, which indicates success (0) or failure (-1) in setting the value in the buffer.
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
    - If the value is too large, sets `errno` to EINVAL and returns -1.
    - If the value is valid, it is cast to a `uint16_t` and passed to the [`ibuf_set`](#ibuf_set) function along with the buffer and position.
- **Output**:
    - Returns 0 on success, indicating that the value was successfully set in the buffer; returns -1 on failure, with `errno` set to indicate the error.
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
    - Checks if the provided `value` exceeds the maximum value for a 32-bit unsigned integer (UINT32_MAX).
    - If the value is too large, sets `errno` to EINVAL and returns -1.
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
    - Calls the [`ibuf_set`](#ibuf_set) function to write the 64-bit value into the buffer at the specified position.
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
    - The function accesses the `buf` member of the `ibuf` structure to get the base pointer of the buffer.
    - It then adds the `rpos` member of the `ibuf` structure to this base pointer to calculate the current read position.
    - Finally, it returns the computed pointer.
- **Output**:
    - The function outputs a pointer to the data in the buffer starting from the current read position.


---
### ibuf_size<!-- {{#callable:ibuf_size}} -->
Calculates the size of the data in an `ibuf` structure by subtracting the read position from the write position.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure whose size is to be calculated.
- **Control Flow**:
    - The function accesses the `wpos` and `rpos` members of the `ibuf` structure.
    - It computes the difference between `wpos` (write position) and `rpos` (read position) to determine the size of the buffer.
- **Output**:
    - Returns the size of the data currently available in the buffer as a `size_t` value.


---
### ibuf_left<!-- {{#callable:ibuf_left}} -->
The `ibuf_left` function calculates the remaining writable space in an input buffer.
- **Inputs**:
    - `buf`: A pointer to a `struct ibuf` that represents the input buffer.
- **Control Flow**:
    - The function first checks if the buffer's file descriptor (`fd`) is marked as being on the stack.
    - If it is on the stack, the function returns 0, indicating no space is left.
    - If not, it calculates the remaining space by subtracting the current write position (`wpos`) from the maximum size (`max`) of the buffer.
- **Output**:
    - Returns the number of bytes left in the buffer that can be written to.


---
### ibuf_truncate<!-- {{#callable:ibuf_truncate}} -->
The `ibuf_truncate` function adjusts the write position of an `ibuf` structure to truncate its content to a specified length.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure that is to be truncated.
    - `len`: The new length to which the buffer should be truncated.
- **Control Flow**:
    - The function first checks if the current size of the buffer is greater than or equal to the specified length.
    - If the current size is sufficient, it sets the write position (`wpos`) to the read position (`rpos`) plus the new length and returns 0.
    - If the buffer is marked as a stack buffer (indicated by `IBUF_FD_MARK_ON_STACK`), it sets `errno` to `ERANGE` and returns -1, as truncation is not allowed.
    - If the current size is less than the specified length and the buffer is not a stack buffer, it calls [`ibuf_add_zero`](#ibuf_add_zero) to add zero bytes to the buffer to reach the desired length.
- **Output**:
    - The function returns 0 on success, indicating that the truncation was successful, or -1 on failure, with `errno` set appropriately.
- **Functions called**:
    - [`ibuf_size`](#ibuf_size)
    - [`ibuf_add_zero`](#ibuf_add_zero)


---
### ibuf_rewind<!-- {{#callable:ibuf_rewind}} -->
The `ibuf_rewind` function resets the read position of an input buffer to the beginning.
- **Inputs**:
    - `buf`: A pointer to a `struct ibuf` which represents the input buffer whose read position is to be reset.
- **Control Flow**:
    - The function directly accesses the `rpos` member of the `struct ibuf` pointed to by `buf`.
    - It sets `buf->rpos` to 0, effectively rewinding the read position of the buffer.
- **Output**:
    - The function does not return a value; it modifies the state of the input buffer in place.


---
### ibuf_close<!-- {{#callable:ibuf_close}} -->
The `ibuf_close` function enqueues a buffer into a message buffer for further processing.
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
    - It then assigns the `data` pointer to the `buf->buf` field of the `ibuf` structure.
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
    - Calls the [`ibuf_from_buffer`](#ibuf_from_buffer) function to set the `buf` with data and size obtained from the `from` buffer.
    - The data is accessed using `ibuf_data(from)` and the size is obtained using `ibuf_size(from)`.
- **Output**:
    - The function does not return a value; it modifies the `buf` in place to reference the data from `from`.
- **Functions called**:
    - [`ibuf_from_buffer`](#ibuf_from_buffer)
    - [`ibuf_data`](#ibuf_data)
    - [`ibuf_size`](#ibuf_size)


---
### ibuf_get<!-- {{#callable:ibuf_get}} -->
The `ibuf_get` function retrieves a specified number of bytes from an input buffer into a provided memory location.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure from which data is to be read.
    - `data`: A pointer to the memory location where the retrieved data will be stored.
    - `len`: The number of bytes to read from the buffer.
- **Control Flow**:
    - The function first checks if the available size of the buffer (obtained via [`ibuf_size`](#ibuf_size)) is less than the requested length (`len`).
    - If the available size is insufficient, it sets `errno` to `EBADMSG` and returns -1 to indicate an error.
    - If there is enough data, it uses `memcpy` to copy `len` bytes from the buffer's current read position (`rpos`) to the provided `data` pointer.
    - After copying, it updates the read position (`rpos`) of the buffer by incrementing it by `len`.
    - Finally, it returns 0 to indicate success.
- **Output**:
    - The function returns 0 on success, or -1 on failure, with `errno` set to indicate the error.
- **Functions called**:
    - [`ibuf_size`](#ibuf_size)
    - [`ibuf_data`](#ibuf_data)


---
### ibuf_get_ibuf<!-- {{#callable:ibuf_get_ibuf}} -->
Retrieves a specified length of data from an `ibuf` and populates a new `ibuf` with that data.
- **Inputs**:
    - `buf`: A pointer to the source `ibuf` from which data will be read.
    - `len`: The number of bytes to read from the `buf`.
    - `new`: A pointer to the destination `ibuf` where the read data will be stored.
- **Control Flow**:
    - Checks if the size of `buf` is less than `len`; if so, sets `errno` to `EBADMSG` and returns -1.
    - Calls [`ibuf_from_buffer`](#ibuf_from_buffer) to initialize `new` with the data from `buf` starting at the current read position.
    - Increments the read position (`rpos`) of `buf` by `len`.
    - Returns 0 to indicate success.
- **Output**:
    - Returns 0 on success, or -1 if there is an error (with `errno` set to indicate the error).
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
The `ibuf_get_h32` function retrieves a 32-bit unsigned integer from a buffer.
- **Inputs**:
    - `buf`: A pointer to a `struct ibuf` that represents the input buffer from which the value is to be read.
    - `value`: A pointer to a `uint32_t` variable where the retrieved 32-bit unsigned integer will be stored.
- **Control Flow**:
    - The function calls [`ibuf_get`](#ibuf_get), passing the buffer, the address of the `value`, and the size of the `value` type.
    - The [`ibuf_get`](#ibuf_get) function attempts to read the specified number of bytes from the buffer into the provided memory location.
- **Output**:
    - The function returns the result of the [`ibuf_get`](#ibuf_get) call, which indicates success (0) or failure (-1) in reading the data.
- **Functions called**:
    - [`ibuf_get`](#ibuf_get)


---
### ibuf_get_h64<!-- {{#callable:ibuf_get_h64}} -->
The `ibuf_get_h64` function retrieves a 64-bit value from an `ibuf` buffer.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure from which the 64-bit value will be read.
    - `value`: A pointer to a `uint64_t` variable where the retrieved value will be stored.
- **Control Flow**:
    - Calls the [`ibuf_get`](#ibuf_get) function to read data from the buffer.
    - Passes the size of the `uint64_t` type to [`ibuf_get`](#ibuf_get) to ensure the correct amount of data is read.
- **Output**:
    - Returns the result of the [`ibuf_get`](#ibuf_get) function, which indicates success (0) or failure (-1) in reading the value.
- **Functions called**:
    - [`ibuf_get`](#ibuf_get)


---
### ibuf_get_n8<!-- {{#callable:ibuf_get_n8}} -->
The `ibuf_get_n8` function retrieves an 8-bit unsigned integer from a buffer.
- **Inputs**:
    - `buf`: A pointer to a `struct ibuf` that represents the input buffer from which the value is to be read.
    - `value`: A pointer to a `uint8_t` variable where the retrieved 8-bit unsigned integer will be stored.
- **Control Flow**:
    - The function calls [`ibuf_get`](#ibuf_get), passing the buffer, the address of the `value`, and the size of the `value` (which is 1 byte for an 8-bit integer).
    - The [`ibuf_get`](#ibuf_get) function checks if there is enough data in the buffer to read the specified number of bytes.
    - If there is enough data, it copies the data from the buffer to the location pointed to by `value` and updates the read position in the buffer.
- **Output**:
    - The function returns 0 on success, or -1 if there is not enough data in the buffer to read the requested value, setting `errno` appropriately.
- **Functions called**:
    - [`ibuf_get`](#ibuf_get)


---
### ibuf_get_n16<!-- {{#callable:ibuf_get_n16}} -->
Retrieves a 16-bit unsigned integer from a buffer and converts it from big-endian to host byte order.
- **Inputs**:
    - `buf`: A pointer to a `struct ibuf` that represents the input buffer from which the value is to be read.
    - `value`: A pointer to a `uint16_t` variable where the retrieved value will be stored.
- **Control Flow**:
    - Calls the [`ibuf_get`](#ibuf_get) function to read the raw 16-bit value from the buffer into the memory location pointed to by `value`.
    - Converts the retrieved value from big-endian to host byte order using the `be16toh` function.
    - Returns the result of the [`ibuf_get`](#ibuf_get) function, which indicates success or failure.
- **Output**:
    - Returns 0 on success, or -1 if an error occurred during the read operation.
- **Functions called**:
    - [`ibuf_get`](#ibuf_get)


---
### ibuf_get_n32<!-- {{#callable:ibuf_get_n32}} -->
Retrieves a 32-bit unsigned integer from a buffer and converts it from big-endian to host byte order.
- **Inputs**:
    - `buf`: A pointer to a `struct ibuf` that represents the input buffer from which the value is to be read.
    - `value`: A pointer to a `uint32_t` variable where the retrieved value will be stored.
- **Control Flow**:
    - Calls the [`ibuf_get`](#ibuf_get) function to read the raw 32-bit value from the buffer into the memory location pointed to by `value`.
    - If the read operation is successful, the function converts the value from big-endian to host byte order using `be32toh`.
    - Returns the result of the [`ibuf_get`](#ibuf_get) function, which indicates success or failure of the read operation.
- **Output**:
    - Returns an integer indicating the success (0) or failure (-1) of the operation.
- **Functions called**:
    - [`ibuf_get`](#ibuf_get)


---
### ibuf_get_n64<!-- {{#callable:ibuf_get_n64}} -->
Retrieves a 64-bit unsigned integer from a buffer and converts it from big-endian to host byte order.
- **Inputs**:
    - `buf`: A pointer to a `struct ibuf` that represents the input buffer from which the value will be read.
    - `value`: A pointer to a `uint64_t` variable where the retrieved value will be stored after reading from the buffer.
- **Control Flow**:
    - Calls the [`ibuf_get`](#ibuf_get) function to read the raw 64-bit value from the buffer.
    - If the read operation is successful, it converts the value from big-endian to host byte order using `be64toh`.
    - Returns the result of the [`ibuf_get`](#ibuf_get) function, indicating success or failure.
- **Output**:
    - Returns an integer indicating the success (0) or failure (-1) of the read operation.
- **Functions called**:
    - [`ibuf_get`](#ibuf_get)


---
### ibuf_get_string<!-- {{#callable:ibuf_get_string}} -->
The `ibuf_get_string` function retrieves a string of a specified length from an input buffer.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure from which the string is to be retrieved.
    - `len`: The length of the string to retrieve from the buffer.
- **Control Flow**:
    - The function first checks if the size of the buffer is less than the specified length; if so, it sets `errno` to `EBADMSG` and returns `NULL`.
    - If the buffer has sufficient data, it uses [`strndup`](strndup.c.driver.md#strndup) to duplicate the string from the buffer's data up to the specified length.
    - If [`strndup`](strndup.c.driver.md#strndup) fails, it returns `NULL`.
    - If successful, it increments the read position (`rpos`) of the buffer by the length of the string retrieved and returns the duplicated string.
- **Output**:
    - The function returns a pointer to the duplicated string if successful, or `NULL` if there is an error.
- **Functions called**:
    - [`ibuf_size`](#ibuf_size)
    - [`strndup`](strndup.c.driver.md#strndup)
    - [`ibuf_data`](#ibuf_data)


---
### ibuf_skip<!-- {{#callable:ibuf_skip}} -->
The `ibuf_skip` function advances the read position of an input buffer by a specified length.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the input buffer from which data is to be skipped.
    - `len`: The number of bytes to skip in the buffer.
- **Control Flow**:
    - The function first checks if the current size of the buffer (obtained via [`ibuf_size`](#ibuf_size)) is less than the specified length to skip.
    - If the buffer size is insufficient, it sets the `errno` to `EBADMSG` and returns -1 to indicate an error.
    - If the buffer size is sufficient, it increments the read position (`rpos`) of the buffer by the specified length.
    - Finally, it returns 0 to indicate success.
- **Output**:
    - The function returns 0 on success, or -1 if there is an error due to insufficient data in the buffer.
- **Functions called**:
    - [`ibuf_size`](#ibuf_size)


---
### ibuf_free<!-- {{#callable:ibuf_free}} -->
Frees the memory associated with an `ibuf` structure.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure that is to be freed.
- **Control Flow**:
    - Checks if the `buf` pointer is NULL; if so, the function returns immediately.
    - Checks if the `fd` field of `buf` is marked as `IBUF_FD_MARK_ON_STACK`, and if so, calls `abort()` to terminate the program.
    - If `fd` is non-negative, it closes the file descriptor associated with `buf`.
    - Calls [`freezero`](freezero.c.driver.md#freezero) to securely free the memory allocated for the buffer in `buf`.
    - Finally, calls `free` to deallocate the `ibuf` structure itself.
- **Output**:
    - This function does not return a value; it performs cleanup and deallocates resources associated with the `ibuf` structure.
- **Functions called**:
    - [`freezero`](freezero.c.driver.md#freezero)


---
### ibuf_fd_avail<!-- {{#callable:ibuf_fd_avail}} -->
The `ibuf_fd_avail` function checks if the file descriptor associated with an `ibuf` structure is valid.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure that contains the file descriptor to be checked.
- **Control Flow**:
    - The function evaluates the condition `buf->fd >= 0` to determine if the file descriptor is valid.
    - If the condition is true, it indicates that the file descriptor is available (non-negative).
- **Output**:
    - The function returns an integer value: 1 if the file descriptor is valid (greater than or equal to 0), and 0 otherwise.


---
### ibuf_fd_get<!-- {{#callable:ibuf_fd_get}} -->
The `ibuf_fd_get` function retrieves and clears the file descriptor from an `ibuf` structure.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure from which the file descriptor is to be retrieved.
- **Control Flow**:
    - Checks if the file descriptor in the `ibuf` structure is negative, which indicates it is for internal use.
    - If the file descriptor is negative, the function returns -1.
    - If the file descriptor is valid (non-negative), it stores the value in a local variable, sets the file descriptor in the `ibuf` structure to -1, and returns the stored file descriptor.
- **Output**:
    - Returns the file descriptor from the `ibuf` structure if it is valid; otherwise, returns -1.


---
### ibuf_fd_set<!-- {{#callable:ibuf_fd_set}} -->
The `ibuf_fd_set` function sets a file descriptor for a given buffer, ensuring proper resource management.
- **Inputs**:
    - `buf`: A pointer to a `struct ibuf` that represents the buffer to which the file descriptor will be assigned.
    - `fd`: An integer representing the file descriptor to be set for the buffer.
- **Control Flow**:
    - Checks if the buffer's file descriptor indicates that it is on the stack; if so, it calls `abort()` to prevent further issues.
    - If the buffer's current file descriptor is valid (non-negative), it closes that file descriptor to free the associated resources.
    - Sets the buffer's file descriptor to -1 to indicate that it is currently not set.
    - If the provided file descriptor `fd` is valid (non-negative), it assigns this value to the buffer's file descriptor.
- **Output**:
    - The function does not return a value; it modifies the `fd` field of the `struct ibuf` directly.


---
### msgbuf_new<!-- {{#callable:msgbuf_new}} -->
Creates and initializes a new `msgbuf` structure.
- **Inputs**:
    - None
- **Control Flow**:
    - Allocates memory for a new `msgbuf` structure using `calloc`.
    - Checks if the memory allocation was successful; if not, returns `NULL`.
    - Initializes the `queued` field of the `msgbuf` to 0.
    - Initializes the `bufs` and `rbufs` fields as empty tail queues using `TAILQ_INIT`.
    - Returns a pointer to the newly created and initialized `msgbuf`.
- **Output**:
    - Returns a pointer to the newly created `msgbuf` structure, or `NULL` if memory allocation fails.


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
    - Sets the `rbuf`, `hdrsize`, `readhdr`, and `rarg` fields of the `msgbuf` structure with the provided values.
    - Returns the initialized `msgbuf` structure.
- **Output**:
    - Returns a pointer to the newly created `msgbuf` reader, or NULL if an error occurs.
- **Functions called**:
    - [`msgbuf_new`](#msgbuf_new)


---
### msgbuf_free<!-- {{#callable:msgbuf_free}} -->
The `msgbuf_free` function deallocates memory associated with a `msgbuf` structure.
- **Inputs**:
    - `msgbuf`: A pointer to a `struct msgbuf` that is to be freed.
- **Control Flow**:
    - Checks if the `msgbuf` pointer is NULL; if it is, the function returns immediately without doing anything.
    - Calls the [`msgbuf_clear`](#msgbuf_clear) function to clear any data associated with the `msgbuf` before freeing.
    - Frees the memory allocated for the `rbuf` member of the `msgbuf` structure.
    - Finally, frees the memory allocated for the `msgbuf` structure itself.
- **Output**:
    - This function does not return a value; it performs memory deallocation and cleanup.
- **Functions called**:
    - [`msgbuf_clear`](#msgbuf_clear)


---
### msgbuf_queuelen<!-- {{#callable:msgbuf_queuelen}} -->
Returns the number of queued messages in the `msgbuf`.
- **Inputs**:
    - `msgbuf`: A pointer to a `struct msgbuf` which contains the queued message count.
- **Control Flow**:
    - The function directly accesses the `queued` member of the `msgbuf` structure.
    - It returns the value of `msgbuf->queued` without any additional processing.
- **Output**:
    - Returns a `uint32_t` representing the number of messages currently queued in the `msgbuf`.


---
### msgbuf_clear<!-- {{#callable:msgbuf_clear}} -->
Clears all buffers in a `msgbuf` structure, resetting its state.
- **Inputs**:
    - `msgbuf`: A pointer to a `struct msgbuf` that contains the buffers to be cleared.
- **Control Flow**:
    - The function first processes the write side by dequeuing all buffers from the `bufs` list until it is empty.
    - It sets the `queued` count to zero after clearing the write buffers.
    - Next, it processes the read side by removing each buffer from the `rbufs` list and freeing them.
    - Finally, it resets the read offset `roff` to zero and frees the `rpmsg` buffer, setting it to NULL.
- **Output**:
    - The function does not return a value; it modifies the state of the `msgbuf` by clearing its buffers and resetting its properties.
- **Functions called**:
    - [`msgbuf_dequeue`](#msgbuf_dequeue)
    - [`ibuf_free`](#ibuf_free)


---
### msgbuf_get<!-- {{#callable:msgbuf_get}} -->
The `msgbuf_get` function retrieves and removes the first `ibuf` from the read buffer queue of a `msgbuf`.
- **Inputs**:
    - `msgbuf`: A pointer to a `msgbuf` structure from which the first `ibuf` is to be retrieved.
- **Control Flow**:
    - The function checks if the read buffer queue (`rbufs`) of the `msgbuf` is not empty by attempting to get the first `ibuf` using `TAILQ_FIRST`.
    - If an `ibuf` is found, it is removed from the queue using `TAILQ_REMOVE`.
    - The function then returns the retrieved `ibuf`, which may be NULL if the queue was empty.
- **Output**:
    - Returns a pointer to the retrieved `ibuf` if available, or NULL if the read buffer queue was empty.


---
### ibuf_write<!-- {{#callable:ibuf_write}} -->
The `ibuf_write` function writes data from a message buffer to a specified file descriptor using scatter-gather I/O.
- **Inputs**:
    - `fd`: An integer representing the file descriptor to which data will be written.
    - `msgbuf`: A pointer to a `msgbuf` structure containing the message buffers to be written.
- **Control Flow**:
    - Initializes an array of `iovec` structures to hold the data to be written.
    - Iterates over the linked list of `ibuf` structures in `msgbuf`, populating the `iovec` array until it reaches the maximum number of entries or the end of the list.
    - If no buffers are queued, the function returns 0 immediately.
    - Attempts to write the data using `writev`, handling interruptions and specific errors (EINTR, EAGAIN, ENOBUFS) by retrying or returning early.
    - After a successful write, it drains the written bytes from the `msgbuf`.
- **Output**:
    - Returns 0 on success, -1 on failure (with errno set), or 0 if there was nothing to write.
- **Functions called**:
    - [`ibuf_data`](#ibuf_data)
    - [`ibuf_size`](#ibuf_size)
    - [`msgbuf_drain`](#msgbuf_drain)


---
### msgbuf_write<!-- {{#callable:msgbuf_write}} -->
The `msgbuf_write` function sends messages from a `msgbuf` structure over a socket file descriptor.
- **Inputs**:
    - `fd`: An integer representing the file descriptor of the socket to which the message will be sent.
    - `msgbuf`: A pointer to a `msgbuf` structure containing the messages to be sent.
- **Control Flow**:
    - Initializes an array of `iovec` structures and other necessary variables.
    - Iterates through the buffers in the `msgbuf` to populate the `iovec` array with message data.
    - Checks if any buffers are available to send; if none, returns 0.
    - Sets up the `msghdr` structure for the message, including control message data if a file descriptor is present.
    - Attempts to send the message using `sendmsg`, retrying if interrupted by a signal.
    - If the send is successful, closes the file descriptor if it was sent, and drains the sent data from the `msgbuf`.
- **Output**:
    - Returns 0 on success, -1 on failure (with errno set), or 0 if there was nothing to send.
- **Functions called**:
    - [`ibuf_data`](#ibuf_data)
    - [`ibuf_size`](#ibuf_size)
    - [`msgbuf_drain`](#msgbuf_drain)


---
### ibuf_read_process<!-- {{#callable:ibuf_read_process}} -->
Processes incoming data from a file descriptor and manages message buffers.
- **Inputs**:
    - `msgbuf`: A pointer to a `msgbuf` structure that holds the message buffer and related metadata.
    - `fd`: An integer file descriptor from which data is read.
- **Control Flow**:
    - Initializes a temporary `ibuf` structure `rbuf` from the `msgbuf`'s read buffer.
    - Enters a loop that continues as long as there is data in `rbuf`.
    - Checks if `rpmsg` (the current message being processed) is NULL; if so, it verifies if enough data is available to read the header.
    - If enough data is available, it reads the header and calls the `readhdr` function to process it, storing the result in `rpmsg`.
    - Determines how much data to read from `rbuf` based on the remaining size of `rpmsg` and the available data in `rbuf`.
    - Reads data from `rbuf` into a temporary message buffer and adds it to `rpmsg`.
    - If `rpmsg` is fully populated, it enqueues it for further processing and resets `rpmsg` to NULL.
    - After processing, if there is remaining data in `rbuf`, it moves that data to the start of the buffer.
    - Updates the offset in `msgbuf` to reflect the new position in `rbuf`.
    - Closes the file descriptor if it is valid and returns success or failure based on the processing outcome.
- **Output**:
    - Returns 1 on successful processing of messages, or -1 on failure, indicating an error occurred during the process.
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
    - `msgbuf`: A pointer to a `msgbuf` structure that contains the buffer where the read data will be stored.
- **Control Flow**:
    - Checks if the `rbuf` in `msgbuf` is NULL; if so, sets `errno` to EINVAL and returns -1.
    - Sets up an `iovec` structure to point to the buffer in `msgbuf` where data will be read.
    - Enters a loop to attempt reading data using `readv` until successful or an error occurs.
    - If `readv` returns -1 due to EINTR, it retries the read operation.
    - If `readv` returns -1 due to EAGAIN, it returns 1 to indicate that the read should be retried later.
    - If `readv` returns 0, it indicates that the connection is closed, and the function returns 0.
    - Updates the read offset in `msgbuf` and calls [`ibuf_read_process`](#ibuf_read_process) to process the newly arrived data.
- **Output**:
    - Returns the number of bytes read, or -1 on error, or 0 if the connection is closed.
- **Functions called**:
    - [`ibuf_read_process`](#ibuf_read_process)


---
### msgbuf_read<!-- {{#callable:msgbuf_read}} -->
The `msgbuf_read` function reads messages from a socket into a message buffer.
- **Inputs**:
    - `fd`: An integer file descriptor representing the socket from which to read the message.
    - `msgbuf`: A pointer to a `msgbuf` structure that holds the message buffer where the read data will be stored.
- **Control Flow**:
    - Checks if the `rbuf` in `msgbuf` is NULL, returning an error if it is.
    - Initializes a `msghdr` structure and a control message buffer.
    - Sets up an `iovec` structure to point to the buffer in `msgbuf` where data will be read.
    - Enters a loop to call `recvmsg` to read data from the socket, handling interruptions and errors appropriately.
    - If data is received, it updates the read offset in `msgbuf` and processes any ancillary data (like file descriptors) received with the message.
    - Calls [`ibuf_read_process`](#ibuf_read_process) to handle the newly arrived data and potentially enqueue it for further processing.
- **Output**:
    - Returns the number of bytes read, 0 if the connection is closed, or -1 on error.
- **Functions called**:
    - [`ibuf_read_process`](#ibuf_read_process)


---
### msgbuf_read_enqueue<!-- {{#callable:msgbuf_read_enqueue}} -->
The `msgbuf_read_enqueue` function adds a buffer to the end of the read buffers queue in a message buffer.
- **Inputs**:
    - `msgbuf`: A pointer to a `struct msgbuf` that represents the message buffer to which the read buffer will be added.
    - `buf`: A pointer to a `struct ibuf` that represents the buffer to be enqueued.
- **Control Flow**:
    - The function first checks if the `fd` field of the `buf` is marked as `IBUF_FD_MARK_ON_STACK`, indicating that the buffer is on the stack.
    - If the buffer is on the stack, the function calls `abort()` to terminate the program to prevent potential memory issues.
    - If the buffer is not on the stack, it uses `TAILQ_INSERT_TAIL` to add the buffer to the end of the `rbufs` queue in the `msgbuf`.
- **Output**:
    - The function does not return a value; it modifies the state of the `msgbuf` by adding the `buf` to its read buffers queue.


---
### msgbuf_enqueue<!-- {{#callable:msgbuf_enqueue}} -->
The `msgbuf_enqueue` function adds a buffer to the end of a message buffer queue.
- **Inputs**:
    - `msgbuf`: A pointer to a `struct msgbuf` that represents the message buffer to which the buffer will be added.
    - `buf`: A pointer to a `struct ibuf` that represents the buffer to be enqueued.
- **Control Flow**:
    - The function first checks if the `buf` is marked as living on the stack by comparing its `fd` field to `IBUF_FD_MARK_ON_STACK`.
    - If the buffer is on the stack, the function calls `abort()` to terminate the program to prevent potential issues.
    - If the buffer is not on the stack, it is added to the end of the `msgbuf`'s queue using `TAILQ_INSERT_TAIL`.
    - Finally, the `queued` count in the `msgbuf` is incremented to reflect the addition of the new buffer.
- **Output**:
    - The function does not return a value; it modifies the state of the `msgbuf` by adding the `buf` to its queue and updating the count of queued buffers.


---
### msgbuf_dequeue<!-- {{#callable:msgbuf_dequeue}} -->
The `msgbuf_dequeue` function removes a specified `ibuf` from a `msgbuf` and frees its memory.
- **Inputs**:
    - `msgbuf`: A pointer to a `msgbuf` structure from which the buffer will be dequeued.
    - `buf`: A pointer to the `ibuf` structure that is to be removed from the `msgbuf`.
- **Control Flow**:
    - The function first removes the specified `ibuf` (`buf`) from the linked list of buffers in the `msgbuf` using `TAILQ_REMOVE`.
    - It then decrements the `queued` count in the `msgbuf` to reflect that one less buffer is queued.
    - Finally, it calls [`ibuf_free`](#ibuf_free) to release the memory allocated for the `ibuf`.
- **Output**:
    - The function does not return a value; it modifies the state of the `msgbuf` and frees the memory of the dequeued `ibuf`.
- **Functions called**:
    - [`ibuf_free`](#ibuf_free)


---
### msgbuf_drain<!-- {{#callable:msgbuf_drain}} -->
The `msgbuf_drain` function removes data from the message buffer by adjusting the read position of the buffers based on the specified number of bytes.
- **Inputs**:
    - `msgbuf`: A pointer to a `struct msgbuf` that represents the message buffer from which data will be drained.
    - `n`: A size_t value indicating the number of bytes to drain from the message buffer.
- **Control Flow**:
    - The function initializes a pointer `buf` to the first buffer in the message buffer's queue.
    - It enters a loop that continues as long as there are buffers in the queue and the number of bytes to drain (`n`) is greater than zero.
    - Within the loop, it retrieves the next buffer and checks if the remaining bytes to drain (`n`) is greater than or equal to the size of the current buffer.
    - If so, it subtracts the size of the current buffer from `n`, dequeues the buffer from the message buffer, and continues to the next buffer.
    - If `n` is less than the size of the current buffer, it adjusts the read position (`rpos`) of the current buffer by `n` and sets `n` to zero, effectively draining the specified bytes.
- **Output**:
    - The function does not return a value; it modifies the state of the `msgbuf` by adjusting the read positions of the buffers and potentially removing buffers from the queue.
- **Functions called**:
    - [`ibuf_size`](#ibuf_size)
    - [`msgbuf_dequeue`](#msgbuf_dequeue)


