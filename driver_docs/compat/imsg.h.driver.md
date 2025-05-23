# Purpose
The provided code is a C header file, `imsg.h`, which is part of the OpenBSD project. It defines a set of structures and functions for inter-process communication (IPC) using message buffers. The primary purpose of this file is to facilitate the creation, manipulation, and transmission of messages between processes in a structured and efficient manner. The file includes definitions for several key structures, such as `ibuf`, `imsgbuf`, `imsg_hdr`, and `imsg`, which are used to manage message buffers, message headers, and the messages themselves. These structures are essential for handling the data and metadata associated with each message, including its type, length, and the process identifiers involved.

The header file also declares a comprehensive set of functions for operating on these structures, providing functionality to open, add to, reserve, seek, set, get, and free message buffers. Additionally, it includes functions for composing and forwarding messages, as well as reading from and writing to message buffers. This collection of functions and structures forms a public API that can be used by other parts of the OpenBSD system or external programs to implement robust IPC mechanisms. The file is designed to be included in other C source files, allowing developers to leverage its functionality for efficient and reliable message-based communication between processes.
# Imports and Dependencies

---
- `sys/types.h`


# Global Variables

---
### ibuf_open
- **Type**: `function`
- **Description**: The `ibuf_open` function is a global function that returns a pointer to a `struct ibuf`. It takes a single parameter of type `size_t`, which likely specifies the initial size or capacity of the buffer to be opened.
- **Use**: This function is used to create and initialize a new `ibuf` structure, which is a buffer used for managing data in the context of inter-process communication.


---
### ibuf_dynamic
- **Type**: `function pointer`
- **Description**: `ibuf_dynamic` is a function pointer that returns a pointer to a `struct ibuf`. It takes two `size_t` parameters, which likely specify the initial size and maximum size for the dynamic buffer being created.
- **Use**: This function is used to dynamically allocate and initialize an `ibuf` structure with specified size parameters.


---
### ibuf_reserve
- **Type**: `void*`
- **Description**: The `ibuf_reserve` function is a global function that reserves a specified amount of space in an `ibuf` structure. It returns a pointer to the reserved space within the buffer.
- **Use**: This function is used to allocate space within an `ibuf` for future data storage, ensuring that the buffer has enough capacity for the specified size.


---
### ibuf_seek
- **Type**: `void*`
- **Description**: The `ibuf_seek` function is a global function that returns a pointer to a specific position within an `ibuf` structure. It takes three parameters: a pointer to an `ibuf` structure, and two `size_t` values representing the offset and length to seek within the buffer.
- **Use**: This function is used to navigate to a specific position within an `ibuf` buffer, allowing for data manipulation or retrieval at that position.


---
### ibuf_data
- **Type**: `function`
- **Description**: The `ibuf_data` function is a utility function that returns a pointer to the data buffer within a given `ibuf` structure. The `ibuf` structure is used to manage a buffer of data, typically for inter-process communication or message handling.
- **Use**: This function is used to access the raw data stored in an `ibuf` structure.


---
### ibuf_get_string
- **Type**: `char *`
- **Description**: The `ibuf_get_string` function is a global function that returns a pointer to a character string extracted from a given `ibuf` structure. It takes a pointer to an `ibuf` and a size parameter as its arguments.
- **Use**: This function is used to retrieve a string from a buffer encapsulated within an `ibuf` structure.


---
### msgbuf_new
- **Type**: `function`
- **Description**: The `msgbuf_new` function is a global function that returns a pointer to a `struct msgbuf`. It is likely used to allocate and initialize a new message buffer structure, which is part of a messaging or inter-process communication system.
- **Use**: This function is used to create and initialize a new `msgbuf` structure for handling message buffers.


---
### msgbuf_new_reader
- **Type**: `struct msgbuf *`
- **Description**: The `msgbuf_new_reader` is a function that returns a pointer to a `struct msgbuf`. It is designed to create a new message buffer reader with a specified size and a custom read function. The function takes three parameters: a size for the buffer, a pointer to a function that processes the buffer, and a void pointer for additional data.
- **Use**: This function is used to initialize and return a new message buffer reader with custom read functionality.


---
### msgbuf_get
- **Type**: `function`
- **Description**: The `msgbuf_get` function is a global function that returns a pointer to an `ibuf` structure. It takes a pointer to a `msgbuf` structure as its parameter. This function is likely used to retrieve a buffer from a message buffer structure, facilitating operations on message data.
- **Use**: This function is used to obtain an `ibuf` from a `msgbuf`, enabling further manipulation or processing of message data.


---
### imsg_create
- **Type**: `function`
- **Description**: The `imsg_create` function is a part of the inter-process messaging system in OpenBSD, designed to create a new message buffer (`ibuf`) for sending messages between processes. It takes parameters including a pointer to an `imsgbuf` structure, message type, peer ID, process ID, and the size of the message to be created. This function is essential for initializing a message buffer that can be populated with data and sent to another process.
- **Use**: This function is used to initialize a new message buffer for inter-process communication, setting up the necessary structure to hold the message data.


# Data Structures

---
### ibuf
- **Type**: `struct`
- **Members**:
    - `entry`: A TAILQ_ENTRY structure for linking ibuf structures in a tail queue.
    - `buf`: A pointer to an unsigned char array that holds the buffer data.
    - `size`: The current size of the data in the buffer.
    - `max`: The maximum capacity of the buffer.
    - `wpos`: The current write position in the buffer.
    - `rpos`: The current read position in the buffer.
    - `fd`: An integer representing a file descriptor associated with the buffer.
- **Description**: The 'ibuf' structure is a buffer management data structure used to handle dynamic data storage and retrieval in a queue-like manner. It includes fields for managing the buffer's data, size, and read/write positions, as well as a file descriptor for I/O operations. The 'entry' field allows 'ibuf' instances to be linked in a tail queue, facilitating efficient management of multiple buffers.


---
### imsgbuf
- **Type**: `struct`
- **Members**:
    - `w`: A pointer to a struct msgbuf, used for message buffering.
    - `pid`: The process ID associated with the message buffer.
    - `maxsize`: The maximum size of the message buffer.
    - `fd`: The file descriptor used for communication.
    - `flags`: Flags used to control the behavior of the message buffer.
- **Description**: The `imsgbuf` structure is used to manage inter-process communication by buffering messages. It contains a pointer to a `msgbuf` structure for message handling, a process ID to identify the associated process, a maximum size to limit the buffer capacity, a file descriptor for communication, and flags to control its operation. This structure is integral to the message passing mechanism in systems that require efficient and organized communication between processes.


---
### imsg_hdr
- **Type**: `struct`
- **Members**:
    - `type`: A 32-bit unsigned integer representing the message type.
    - `len`: A 32-bit unsigned integer indicating the length of the message.
    - `peerid`: A 32-bit unsigned integer used to identify the peer.
    - `pid`: A 32-bit unsigned integer representing the process ID associated with the message.
- **Description**: The `imsg_hdr` structure is a header used in inter-process communication to encapsulate metadata about a message. It includes fields for the message type, length, peer identifier, and process ID, which are essential for managing and routing messages between processes in a system. This structure is part of a larger messaging framework that facilitates communication between different components of a software system.


---
### imsg
- **Type**: `struct`
- **Members**:
    - `hdr`: A header of type `struct imsg_hdr` that contains metadata about the message.
    - `data`: A pointer to the data associated with the message.
    - `buf`: A pointer to a buffer of type `struct ibuf` that holds the message data.
- **Description**: The `imsg` structure is used to represent inter-process messages in the OpenBSD operating system. It encapsulates a message header (`imsg_hdr`) that contains metadata such as type, length, peer ID, and process ID, a pointer to the actual message data, and a buffer (`ibuf`) that manages the message's data storage. This structure is part of a messaging system that facilitates communication between different processes, allowing them to exchange data efficiently.


# Function Declarations (Public API)

---
### ibuf_open<!-- {{#callable_declaration:ibuf_open}} -->
Allocates and initializes a new input buffer.
- **Description**: This function is used to create a new input buffer of a specified length. It should be called when a new buffer is needed for reading data. The length parameter determines the initial size of the buffer; if it is greater than zero, memory for the buffer is allocated. If memory allocation fails at any point, the function will return `NULL`. It is important to ensure that the caller handles the returned pointer appropriately, including freeing it when it is no longer needed.
- **Inputs**:
    - `len`: The desired length of the buffer to be allocated. Must be a non-negative value. If `len` is zero, the buffer will be created with no allocated space for data. The caller retains ownership of the returned buffer and is responsible for freeing it when done.
- **Output**: Returns a pointer to the newly allocated `ibuf` structure, or `NULL` if memory allocation fails.
- **See also**: [`ibuf_open`](imsg-buffer.c.driver.md#ibuf_open)  (Implementation)


---
### ibuf_dynamic<!-- {{#callable_declaration:ibuf_dynamic}} -->
Allocates a dynamic input buffer.
- **Description**: This function is used to create a dynamic input buffer with a specified initial length and a maximum size. It should be called when a new buffer is needed for data storage, ensuring that the maximum size is greater than or equal to the initial length. If the provided maximum size is zero or less than the initial length, the function will return `NULL` and set `errno` to `EINVAL`. The caller is responsible for managing the memory of the returned buffer, which must be freed when no longer needed.
- **Inputs**:
    - `len`: The initial length of the buffer to allocate. Must be greater than or equal to 0.
    - `max`: The maximum size of the buffer. Must be greater than 0 and greater than or equal to `len`.
- **Output**: Returns a pointer to the newly allocated `ibuf` structure on success, or `NULL` if the allocation fails or if the input parameters are invalid.
- **See also**: [`ibuf_dynamic`](imsg-buffer.c.driver.md#ibuf_dynamic)  (Implementation)


---
### ibuf_add<!-- {{#callable_declaration:ibuf_add}} -->
Adds data to a buffer.
- **Description**: This function is used to append a specified amount of data to a buffer. It should be called after the buffer has been properly initialized. If the buffer does not have enough space to accommodate the new data, the function will return an error. It is important to ensure that the `data` pointer is not null and that the `len` parameter is a positive value, as invalid values may lead to undefined behavior.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer to which data will be added. Must not be null.
    - `data`: A pointer to the data to be added to the buffer. Must not be null.
    - `len`: The number of bytes to add from the data pointer. Must be a positive value.
- **Output**: Returns 0 on success, or -1 if there is an error, such as insufficient space in the buffer.
- **See also**: [`ibuf_add`](imsg-buffer.c.driver.md#ibuf_add)  (Implementation)


---
### ibuf_add_ibuf<!-- {{#callable_declaration:ibuf_add_ibuf}} -->
Adds the contents of one `ibuf` to another.
- **Description**: This function is used to append the data from one `ibuf` structure to another. It is typically called when you want to combine multiple buffers into a single buffer. The source buffer must not be null, and the destination buffer must be properly initialized. If the source buffer is empty, the function will have no effect on the destination buffer. It is important to ensure that the destination buffer has enough capacity to accommodate the additional data; otherwise, the behavior is undefined.
- **Inputs**:
    - `buf`: A pointer to the destination `ibuf` where data will be added. Must not be null and should be initialized properly.
    - `from`: A pointer to the source `ibuf` whose contents will be added to `buf`. Must not be null. If this buffer is empty, no changes will be made to `buf`.
- **Output**: Returns 0 on success, or a negative value if an error occurs, such as insufficient space in the destination buffer.
- **See also**: [`ibuf_add_ibuf`](imsg-buffer.c.driver.md#ibuf_add_ibuf)  (Implementation)


---
### ibuf_add_zero<!-- {{#callable_declaration:ibuf_add_zero}} -->
Adds a specified number of zero bytes to a buffer.
- **Description**: This function is used to append a block of zeroed bytes to an `ibuf` structure, which is typically used for message buffering. It should be called after ensuring that the buffer has been properly initialized and can accommodate the additional data. If the buffer cannot reserve the requested space, the function will return an error code. It is important to handle the return value appropriately to ensure that the operation was successful.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure where the zero bytes will be added. Must not be null and should be properly initialized before calling this function.
    - `len`: The number of zero bytes to add to the buffer. Must be a positive value; passing zero may lead to undefined behavior.
- **Output**: Returns 0 on success, indicating that the zero bytes were successfully added. Returns -1 if there was an error reserving space in the buffer.
- **See also**: [`ibuf_add_zero`](imsg-buffer.c.driver.md#ibuf_add_zero)  (Implementation)


---
### ibuf_add_n8<!-- {{#callable_declaration:ibuf_add_n8}} -->
Adds an 8-bit unsigned integer to a buffer.
- **Description**: This function is used to add an 8-bit unsigned integer value to a specified buffer. It should be called when there is a need to store a small integer value in the buffer, ensuring that the value does not exceed the maximum limit for an 8-bit integer. The function will return an error if the provided value is greater than 255, and it is important to check the return value to handle any potential errors appropriately.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer to which the value will be added. Must not be null.
    - `value`: The 8-bit unsigned integer value to be added, which must be in the range of 0 to 255. If the value exceeds this range, the function will set errno to EINVAL and return -1.
- **Output**: Returns 0 on success, or -1 if an error occurs due to an invalid value.
- **See also**: [`ibuf_add_n8`](imsg-buffer.c.driver.md#ibuf_add_n8)  (Implementation)


---
### ibuf_add_n16<!-- {{#callable_declaration:ibuf_add_n16}} -->
Adds a 16-bit unsigned integer to a buffer.
- **Description**: This function is used to add a 16-bit unsigned integer value to a specified buffer. It should be called when there is a need to store a 16-bit integer representation in the buffer, typically after the buffer has been initialized. The function checks if the provided value exceeds the maximum allowable value for a 16-bit unsigned integer; if it does, the function sets the `errno` to `EINVAL` and returns -1 to indicate an error. It is important to ensure that the buffer is properly allocated and initialized before calling this function.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer to which the value will be added. Must not be null and should be properly initialized before use.
    - `value`: The 64-bit unsigned integer value to be added, which must be in the range of 0 to 65535 (inclusive). If the value exceeds this range, the function will return -1 and set `errno` to `EINVAL`.
- **Output**: Returns 0 on success, indicating that the value was successfully added to the buffer. Returns -1 on failure, with `errno` set to indicate the error.
- **See also**: [`ibuf_add_n16`](imsg-buffer.c.driver.md#ibuf_add_n16)  (Implementation)


---
### ibuf_add_n32<!-- {{#callable_declaration:ibuf_add_n32}} -->
Adds a 32-bit unsigned integer to a buffer.
- **Description**: This function is used to add a 32-bit unsigned integer value to a specified buffer. It should be called when there is a need to store a 32-bit integer in a buffer, typically after the buffer has been initialized. The function checks if the provided value exceeds the maximum allowable value for a 32-bit unsigned integer; if it does, it sets the `errno` to `EINVAL` and returns -1 to indicate an error. Otherwise, it converts the value to big-endian format and adds it to the buffer.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer to which the value will be added. Must not be null and should be properly initialized before calling this function.
    - `value`: The 64-bit unsigned integer value to be added, which must be less than or equal to UINT32_MAX (4,294,967,295). If the value exceeds this limit, the function will return -1 and set `errno` to `EINVAL`.
- **Output**: Returns 0 on success, indicating that the value was successfully added to the buffer. Returns -1 on failure, with `errno` set to indicate the error.
- **See also**: [`ibuf_add_n32`](imsg-buffer.c.driver.md#ibuf_add_n32)  (Implementation)


---
### ibuf_add_n64<!-- {{#callable_declaration:ibuf_add_n64}} -->
Adds a 64-bit integer to a buffer.
- **Description**: This function is used to append a 64-bit integer value to a specified buffer. It is important to ensure that the buffer has been properly initialized before calling this function. The value is converted to big-endian format before being added to the buffer. If the buffer is full or if there is an error during the addition, the function will return an error code. Therefore, it is advisable to check the return value to confirm that the operation was successful.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer to which the value will be added. This pointer must not be null and the buffer must be initialized prior to calling this function.
    - `value`: The 64-bit integer value to be added to the buffer. This value can be any valid 64-bit unsigned integer.
- **Output**: Returns 0 on success, or a negative error code if the addition fails.
- **See also**: [`ibuf_add_n64`](imsg-buffer.c.driver.md#ibuf_add_n64)  (Implementation)


---
### ibuf_add_h16<!-- {{#callable_declaration:ibuf_add_h16}} -->
Adds a 16-bit unsigned integer to a buffer.
- **Description**: This function is used to add a 16-bit unsigned integer value to a specified buffer. It should be called when there is a need to store a small integer value in a buffer, typically for message passing or data serialization. The value must be within the range of 0 to 65535 (inclusive), as values outside this range will result in an error. If the provided value exceeds this limit, the function sets the `errno` to `EINVAL` and returns -1 to indicate failure. It is important to ensure that the buffer is properly initialized before calling this function.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer to which the value will be added. This pointer must not be null and must point to a valid, initialized buffer.
    - `value`: The 64-bit unsigned integer value to be added, which must be in the range of 0 to 65535. If the value exceeds UINT16_MAX, the function will set `errno` to `EINVAL` and return -1.
- **Output**: Returns 0 on success, indicating that the value was successfully added to the buffer. If an error occurs due to an invalid value, it returns -1.
- **See also**: [`ibuf_add_h16`](imsg-buffer.c.driver.md#ibuf_add_h16)  (Implementation)


---
### ibuf_add_h32<!-- {{#callable_declaration:ibuf_add_h32}} -->
Adds a 32-bit unsigned integer to a buffer.
- **Description**: This function is used to add a 32-bit unsigned integer value to a specified buffer. It should be called when you need to store a value in the buffer, ensuring that the value does not exceed the maximum limit for a 32-bit unsigned integer. If the provided value is greater than `UINT32_MAX`, the function will set the `errno` to `EINVAL` and return -1 to indicate an error. It is important to ensure that the buffer has been properly initialized before calling this function.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer to which the value will be added. Must not be null and should be properly initialized.
    - `value`: The 64-bit unsigned integer value to be added, which must be less than or equal to `UINT32_MAX`. If the value exceeds this limit, the function will return -1 and set `errno` to `EINVAL`.
- **Output**: Returns 0 on success, indicating that the value was successfully added to the buffer. Returns -1 on error, with `errno` set to indicate the specific error.
- **See also**: [`ibuf_add_h32`](imsg-buffer.c.driver.md#ibuf_add_h32)  (Implementation)


---
### ibuf_add_h64<!-- {{#callable_declaration:ibuf_add_h64}} -->
Adds a 64-bit value to the buffer.
- **Description**: This function is used to append a 64-bit unsigned integer value to a buffer represented by the `ibuf` structure. It should be called after the buffer has been properly initialized and opened. The function will handle the addition of the value to the buffer, ensuring that the buffer's internal state is updated accordingly. If the buffer is full or if any other error occurs during the addition, the function will return an error code, allowing the caller to handle the situation appropriately.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure where the value will be added. This must not be null and should point to a valid, initialized buffer.
    - `value`: The 64-bit unsigned integer value to be added to the buffer. This value is passed by value, and the function will copy it into the buffer.
- **Output**: Returns 0 on success, or a negative error code if the addition fails.
- **See also**: [`ibuf_add_h64`](imsg-buffer.c.driver.md#ibuf_add_h64)  (Implementation)


---
### ibuf_reserve<!-- {{#callable_declaration:ibuf_reserve}} -->
Reserves space in a buffer.
- **Description**: This function is used to reserve a specified amount of space in a buffer for future data writing. It should be called when there is a need to ensure that enough space is available in the buffer before writing data. The function checks if the requested length can be accommodated without exceeding the buffer's maximum size. If the buffer is currently at its limit, it attempts to resize the buffer. It is important to note that this function cannot be used with buffers that are marked as stack buffers. If the reservation is successful, the function returns a pointer to the reserved space; otherwise, it returns `NULL` and sets the appropriate error code.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer. Must not be null.
    - `len`: The number of bytes to reserve in the buffer. Must be a positive value that does not cause the total size to exceed `SIZE_MAX`.
- **Output**: Returns a pointer to the reserved space in the buffer if successful; otherwise, returns `NULL`.
- **See also**: [`ibuf_reserve`](imsg-buffer.c.driver.md#ibuf_reserve)  (Implementation)


---
### ibuf_seek<!-- {{#callable_declaration:ibuf_seek}} -->
Seeks to a specific position in the input buffer.
- **Description**: This function is used to obtain a pointer to a specific location within an input buffer, defined by the parameters `pos` and `len`. It should be called when you need to access a portion of the buffer, ensuring that the specified range is valid. The function checks that the requested position and length do not exceed the bounds of the buffer, and if they do, it sets `errno` to `ERANGE` and returns `NULL`. Therefore, it is important to ensure that the buffer has been properly initialized and contains sufficient data before calling this function.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the input buffer. Must not be null and should point to a valid buffer that has been initialized.
    - `pos`: The starting position within the buffer from which to seek. Must be less than the size of the buffer.
    - `len`: The length of the data to seek. Must not cause the total requested range (pos + len) to exceed the size of the buffer.
- **Output**: Returns a pointer to the location in the buffer at `rpos + pos` if the seek is successful; otherwise, returns `NULL` if the specified range is invalid.
- **See also**: [`ibuf_seek`](imsg-buffer.c.driver.md#ibuf_seek)  (Implementation)


---
### ibuf_set<!-- {{#callable_declaration:ibuf_set}} -->
Sets data in a specified position of a buffer.
- **Description**: This function is used to write a specified amount of data into a buffer at a given position. It should be called when the buffer has been properly initialized and is ready for data manipulation. The function checks if the specified position is valid and if the buffer can accommodate the data length. If the position is invalid or the buffer is insufficient, the function will return an error. It is important to ensure that the data being written is valid and that the length does not exceed the buffer's capacity.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer where data will be written. Must not be null.
    - `pos`: The position in the buffer where the data will be set. This must be within the bounds of the buffer's current size.
    - `data`: A pointer to the data to be written into the buffer. Must not be null.
    - `len`: The number of bytes to write from the data pointer. This must be a positive value and should not exceed the remaining space in the buffer from the specified position.
- **Output**: Returns 0 on success, indicating that the data was successfully written to the buffer. Returns -1 if there was an error, such as an invalid position or insufficient space in the buffer.
- **See also**: [`ibuf_set`](imsg-buffer.c.driver.md#ibuf_set)  (Implementation)


---
### ibuf_set_n8<!-- {{#callable_declaration:ibuf_set_n8}} -->
Sets an 8-bit unsigned integer at a specified position in a buffer.
- **Description**: This function is used to set an 8-bit unsigned integer value at a specified position within a buffer. It should be called when you need to modify the contents of the buffer, ensuring that the position is valid and the value is within the acceptable range for an 8-bit integer. If the provided value exceeds the maximum allowable value for an 8-bit integer (255), the function will return an error. It is important to ensure that the buffer has been properly initialized and that the position specified is within the bounds of the buffer.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer where the value will be set. Must not be null.
    - `pos`: The position in the buffer where the value will be set. This must be a valid index within the bounds of the buffer.
    - `value`: The 64-bit unsigned integer value to be set as an 8-bit integer. Must be in the range of 0 to 255. If the value exceeds this range, the function will return an error.
- **Output**: Returns 0 on success, or -1 if an error occurs due to an invalid value.
- **See also**: [`ibuf_set_n8`](imsg-buffer.c.driver.md#ibuf_set_n8)  (Implementation)


---
### ibuf_set_n16<!-- {{#callable_declaration:ibuf_set_n16}} -->
Sets a 16-bit unsigned integer at a specified position in a buffer.
- **Description**: This function is used to write a 16-bit unsigned integer value into a specified position within a buffer. It should be called when you need to update a specific location in the buffer with a new value. The value must be within the range of a 16-bit unsigned integer (0 to 65535); if the value exceeds this range, the function will return an error. It is important to ensure that the buffer has been properly initialized and that the specified position is valid before calling this function.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer where the value will be set. Must not be null.
    - `pos`: The position in the buffer where the 16-bit value will be written. This must be a valid position within the bounds of the buffer.
    - `value`: The 16-bit unsigned integer value to be set. Must be in the range of 0 to 65535. If the value exceeds this range, the function will return an error.
- **Output**: Returns 0 on success, or -1 if an error occurs due to an invalid value.
- **See also**: [`ibuf_set_n16`](imsg-buffer.c.driver.md#ibuf_set_n16)  (Implementation)


---
### ibuf_set_n32<!-- {{#callable_declaration:ibuf_set_n32}} -->
Sets a 32-bit unsigned integer value in the specified buffer.
- **Description**: This function is used to write a 32-bit unsigned integer value into a specified position within a buffer. It should be called when you need to update a specific location in the buffer with a new value. The value must be within the range of a 32-bit unsigned integer; if it exceeds this range, the function will return an error. It is important to ensure that the buffer has been properly initialized and that the specified position is valid before calling this function.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer where the value will be set. Must not be null.
    - `pos`: The position in the buffer where the value will be written. This must be a valid position within the bounds of the buffer.
    - `value`: The 64-bit unsigned integer value to be set in the buffer. It must be less than or equal to UINT32_MAX; otherwise, the function will return an error.
- **Output**: Returns 0 on success, or -1 if an error occurs due to an invalid value.
- **See also**: [`ibuf_set_n32`](imsg-buffer.c.driver.md#ibuf_set_n32)  (Implementation)


---
### ibuf_set_n64<!-- {{#callable_declaration:ibuf_set_n64}} -->
Sets a 64-bit integer value in the specified buffer.
- **Description**: This function is used to set a 64-bit integer value at a specified position within a buffer. It is important to ensure that the buffer has been properly initialized and that the position specified is within the bounds of the buffer's allocated size. If the position is invalid, the function may return an error. This function should be called when there is a need to update a specific location in the buffer with a new 64-bit value.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer where the value will be set. Must not be null.
    - `pos`: The position in the buffer where the 64-bit value will be written. This must be a valid index within the bounds of the buffer.
    - `value`: The 64-bit integer value to be set in the buffer. This value will be converted to big-endian format before being written.
- **Output**: Returns 0 on success, or a negative value if an error occurs.
- **See also**: [`ibuf_set_n64`](imsg-buffer.c.driver.md#ibuf_set_n64)  (Implementation)


---
### ibuf_set_h16<!-- {{#callable_declaration:ibuf_set_h16}} -->
Sets a 16-bit unsigned integer at a specified position in a buffer.
- **Description**: This function is used to write a 16-bit unsigned integer value into a specified position within a buffer. It should be called when you need to update a specific location in the buffer with a new value. The value must be within the range of 0 to 65535 (inclusive), as values outside this range will result in an error. If the provided position is valid and the value is acceptable, the function will update the buffer accordingly. It is important to ensure that the buffer has been properly initialized before calling this function.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer where the value will be set. Must not be null.
    - `pos`: The position in the buffer where the 16-bit value will be written. This must be a valid position within the bounds of the buffer.
    - `value`: The 16-bit unsigned integer value to be set. Valid values are in the range of 0 to 65535. If the value exceeds this range, the function will return an error.
- **Output**: Returns 0 on success, or -1 if an error occurs due to an invalid value.
- **See also**: [`ibuf_set_h16`](imsg-buffer.c.driver.md#ibuf_set_h16)  (Implementation)


---
### ibuf_set_h32<!-- {{#callable_declaration:ibuf_set_h32}} -->
Sets a 32-bit value at a specified position in a buffer.
- **Description**: This function is used to write a 32-bit unsigned integer value into a specified position within a buffer. It should be called when you need to update or set a specific value in the buffer, ensuring that the value does not exceed the maximum limit for a 32-bit unsigned integer. The function will return an error if the provided value is greater than `UINT32_MAX`, and it is important to ensure that the buffer has been properly initialized before calling this function.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer where the value will be set. Must not be null.
    - `pos`: The position in the buffer where the 32-bit value will be written. This should be within the bounds of the buffer's size.
    - `value`: The 64-bit unsigned integer value to be set. It must be less than or equal to `UINT32_MAX`. If the value exceeds this limit, the function will return an error.
- **Output**: Returns 0 on success, or -1 if an error occurs due to an invalid value.
- **See also**: [`ibuf_set_h32`](imsg-buffer.c.driver.md#ibuf_set_h32)  (Implementation)


---
### ibuf_set_h64<!-- {{#callable_declaration:ibuf_set_h64}} -->
Sets a 64-bit value at a specified position in the buffer.
- **Description**: This function is used to write a 64-bit integer value into a specified position within a buffer. It should be called when the buffer has been properly initialized and is ready for data manipulation. The position specified must be valid and within the bounds of the buffer's current size; otherwise, the function may fail. It is important to ensure that the buffer is not null before calling this function to avoid undefined behavior.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer where the value will be set. Must not be null.
    - `pos`: The position in the buffer where the 64-bit value will be written. This must be a valid index within the current size of the buffer.
    - `value`: The 64-bit integer value to be written into the buffer. This is passed by reference.
- **Output**: Returns 0 on success, or a negative value on failure, indicating an error occurred during the operation.
- **See also**: [`ibuf_set_h64`](imsg-buffer.c.driver.md#ibuf_set_h64)  (Implementation)


---
### ibuf_data<!-- {{#callable_declaration:ibuf_data}} -->
Returns a pointer to the data in the input buffer.
- **Description**: This function is used to access the data stored in an `ibuf` structure, starting from the current read position. It should be called after ensuring that the `ibuf` has been properly initialized and contains data. The returned pointer allows the caller to read or manipulate the data directly. It is important to note that the caller must ensure that the `ibuf` is not null and that the read position is within the bounds of the buffer to avoid undefined behavior.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure that contains the data to be accessed. Must not be null. The caller retains ownership of the `ibuf`. If the `buf` is invalid or if the read position exceeds the buffer size, the behavior is undefined.
- **Output**: Returns a pointer to the data in the `ibuf` starting from the current read position. If the input buffer is valid, this pointer can be used to access the data directly.
- **See also**: [`ibuf_data`](imsg-buffer.c.driver.md#ibuf_data)  (Implementation)


---
### ibuf_size<!-- {{#callable_declaration:ibuf_size}} -->
Returns the size of the input buffer.
- **Description**: This function is used to determine the number of bytes available in the specified buffer for reading. It should be called when you need to know how much data can be read from the buffer, typically after ensuring that the buffer has been properly initialized and populated. The function calculates the size by subtracting the read position from the write position, thus providing the current size of the data in the buffer. It is important to ensure that the input buffer is not null before calling this function to avoid undefined behavior.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure representing the buffer. Must not be null; passing a null pointer will result in undefined behavior.
- **Output**: Returns the size of the data currently available in the buffer as a `size_t` value.
- **See also**: [`ibuf_size`](imsg-buffer.c.driver.md#ibuf_size)  (Implementation)


---
### ibuf_left<!-- {{#callable_declaration:ibuf_left}} -->
Returns the number of bytes left in the buffer.
- **Description**: This function is used to determine how many bytes are available for writing in a buffer. It should be called after the buffer has been properly initialized and opened. If the buffer is a stack buffer, it will always return zero, indicating that no space is available. This function is useful for managing buffer space before adding new data.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure representing the buffer. Must not be null. If the buffer is a stack buffer (indicated by `fd` being equal to `IBUF_FD_MARK_ON_STACK`), the function will return 0.
- **Output**: Returns the number of bytes left in the buffer that can be written to, which is calculated as the difference between the maximum size and the current write position.
- **See also**: [`ibuf_left`](imsg-buffer.c.driver.md#ibuf_left)  (Implementation)


---
### ibuf_truncate<!-- {{#callable_declaration:ibuf_truncate}} -->
Truncates the buffer to a specified length.
- **Description**: This function is used to adjust the size of a buffer to a specified length. It should be called when you need to reduce the size of the buffer's writable data. The function checks if the current size of the buffer is greater than or equal to the specified length; if so, it updates the write position accordingly. If the buffer is a stack-allocated buffer, truncation is only allowed if the new length is less than the current size, otherwise an error is returned. If the buffer is not stack-allocated and the new length is greater than the current size, it will add zero bytes to the buffer to accommodate the new size.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure representing the buffer to be truncated. Must not be null.
    - `len`: The new length to truncate the buffer to. Must be less than or equal to the current size of the buffer for stack-allocated buffers. If the buffer is not stack-allocated, this can be greater than the current size, in which case zero bytes will be added.
- **Output**: Returns 0 on success if the buffer was truncated successfully. Returns -1 if truncation is not allowed due to the buffer being stack-allocated and the new length being greater than the current size, or if an error occurs during the process.
- **See also**: [`ibuf_truncate`](imsg-buffer.c.driver.md#ibuf_truncate)  (Implementation)


---
### ibuf_rewind<!-- {{#callable_declaration:ibuf_rewind}} -->
Resets the read position of a buffer.
- **Description**: This function is used to reset the read position of a buffer to the beginning, allowing for re-reading of the data from the start. It should be called when you need to process the data in the buffer again after having read it previously. Ensure that the buffer is properly initialized before calling this function, as calling it on an uninitialized or invalid buffer may lead to undefined behavior.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure representing the buffer. Must not be null and should point to a valid, initialized buffer. The function does not take ownership of the buffer.
- **Output**: None
- **See also**: [`ibuf_rewind`](imsg-buffer.c.driver.md#ibuf_rewind)  (Implementation)


---
### ibuf_close<!-- {{#callable_declaration:ibuf_close}} -->
Closes an input buffer and enqueues it to a message buffer.
- **Description**: This function is intended to be called when an input buffer is no longer needed and should be properly closed. It enqueues the specified `ibuf` to the provided `msgbuf`, allowing for further processing or cleanup. It is important to ensure that both `msgbuf` and `buf` are valid and properly initialized before calling this function. There are no specific return values or error codes, but passing null pointers may lead to undefined behavior.
- **Inputs**:
    - `msgbuf`: A pointer to a `struct msgbuf` that represents the message buffer to which the input buffer will be enqueued. Must not be null.
    - `buf`: A pointer to a `struct ibuf` that represents the input buffer to be closed. Must not be null.
- **Output**: None
- **See also**: [`ibuf_close`](imsg-buffer.c.driver.md#ibuf_close)  (Implementation)


---
### ibuf_from_buffer<!-- {{#callable_declaration:ibuf_from_buffer}} -->
Initializes an `ibuf` structure from a buffer.
- **Description**: This function is used to set up an `ibuf` structure with a specified buffer and its length. It should be called when you need to create an `ibuf` that references existing data rather than allocating new memory. The function clears the `ibuf` structure before setting its fields, ensuring that it starts in a known state. It is important to ensure that the `data` pointer is valid and that the `len` parameter accurately reflects the size of the data being referenced.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure that will be initialized. Must not be null.
    - `data`: A pointer to the data buffer that the `ibuf` will reference. Must not be null.
    - `len`: The length of the data buffer in bytes. Must be greater than zero.
- **Output**: None
- **See also**: [`ibuf_from_buffer`](imsg-buffer.c.driver.md#ibuf_from_buffer)  (Implementation)


---
### ibuf_from_ibuf<!-- {{#callable_declaration:ibuf_from_ibuf}} -->
Copies data from one `ibuf` to another.
- **Description**: This function is used to copy the contents of one `ibuf` structure into another. It is essential to ensure that the destination buffer is properly initialized before calling this function. The source buffer must not be null, and it should contain valid data. If the source buffer is empty or invalid, the behavior is undefined, so it is crucial to validate the input before invoking this function.
- **Inputs**:
    - `buf`: A pointer to the destination `ibuf` structure where data will be copied. This must be a valid, initialized `ibuf` and must not be null.
    - `from`: A pointer to the source `ibuf` structure from which data will be copied. This must be a valid, initialized `ibuf` and must not be null.
- **Output**: None
- **See also**: [`ibuf_from_ibuf`](imsg-buffer.c.driver.md#ibuf_from_ibuf)  (Implementation)


---
### ibuf_get<!-- {{#callable_declaration:ibuf_get}} -->
Retrieves data from an input buffer.
- **Description**: This function is used to read a specified number of bytes from an input buffer into a provided data buffer. It should be called when there is a need to extract data from the `ibuf` structure, and it is essential that the input buffer contains at least the requested number of bytes. If the buffer does not have enough data, the function will set an error code and return -1. The caller must ensure that the `ibuf` has been properly initialized and contains sufficient data before invoking this function.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure from which data will be read. This must not be null and should point to a valid, initialized buffer.
    - `data`: A pointer to the destination buffer where the data will be copied. This must not be null and should have enough allocated space to hold the specified number of bytes.
    - `len`: The number of bytes to read from the input buffer. This must be greater than zero and less than or equal to the size of the data available in the `ibuf`. If `len` exceeds the available data, the function will set an error and return -1.
- **Output**: Returns 0 on success, indicating that the data has been successfully copied to the destination buffer. If an error occurs due to insufficient data in the input buffer, it returns -1.
- **See also**: [`ibuf_get`](imsg-buffer.c.driver.md#ibuf_get)  (Implementation)


---
### ibuf_get_ibuf<!-- {{#callable_declaration:ibuf_get_ibuf}} -->
Retrieves a specified length of data from a buffer.
- **Description**: This function is used to extract a specified number of bytes from a given input buffer into a new buffer. It should be called when there is a need to read a specific amount of data from the input buffer, ensuring that the requested length does not exceed the available data. If the input buffer does not contain enough data, an error is indicated, and the function will return -1. The caller must ensure that the `new` buffer is properly allocated before calling this function.
- **Inputs**:
    - `buf`: A pointer to the source `ibuf` structure from which data will be read. Must not be null. The function checks if the available data in `buf` is less than `len` and returns an error if so.
    - `len`: The number of bytes to read from the input buffer. Must be less than or equal to the size of the data currently available in `buf`. If `len` exceeds the available data, the function will return an error.
    - `new`: A pointer to the destination `ibuf` structure where the data will be stored. This buffer must be allocated by the caller and should be large enough to hold the specified length of data.
- **Output**: Returns 0 on success, indicating that the data has been successfully retrieved and stored in the `new` buffer. Returns -1 if there is not enough data in the `buf` to satisfy the request, and sets `errno` to indicate the error.
- **See also**: [`ibuf_get_ibuf`](imsg-buffer.c.driver.md#ibuf_get_ibuf)  (Implementation)


---
### ibuf_get_n8<!-- {{#callable_declaration:ibuf_get_n8}} -->
Retrieves an 8-bit unsigned integer from a buffer.
- **Description**: This function is used to extract an 8-bit unsigned integer from a specified buffer. It should be called when there is a need to read a single byte value from the buffer, ensuring that the buffer has been properly initialized and contains sufficient data. The caller must ensure that the buffer is not empty and that the provided pointer for the output value is valid. If the buffer does not contain enough data, the function may return an error.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure from which the value will be read. This buffer must be initialized and contain data. It must not be null.
    - `value`: A pointer to a `uint8_t` variable where the retrieved value will be stored. This pointer must not be null, and the caller retains ownership of the memory.
- **Output**: Returns an integer indicating the success or failure of the operation. A return value of 0 indicates success, while a negative value indicates an error, such as insufficient data in the buffer.
- **See also**: [`ibuf_get_n8`](imsg-buffer.c.driver.md#ibuf_get_n8)  (Implementation)


---
### ibuf_get_n16<!-- {{#callable_declaration:ibuf_get_n16}} -->
Retrieves a 16-bit unsigned integer from a buffer.
- **Description**: This function is used to extract a 16-bit unsigned integer from a specified buffer. It should be called when there is a need to read a value from the buffer that has been previously populated. The function expects that the buffer has been initialized and contains sufficient data to read a 16-bit value. If the buffer does not contain enough data, the function will return an error code. The retrieved value is converted from big-endian to host byte order before being returned.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure from which the value will be read. Must not be null and should point to a valid buffer that has been initialized and contains enough data.
    - `value`: A pointer to a `uint16_t` variable where the retrieved value will be stored. Must not be null; the function will write the retrieved value into this variable.
- **Output**: Returns 0 on success, or a negative error code if the operation fails.
- **See also**: [`ibuf_get_n16`](imsg-buffer.c.driver.md#ibuf_get_n16)  (Implementation)


---
### ibuf_get_n32<!-- {{#callable_declaration:ibuf_get_n32}} -->
Retrieves a 32-bit unsigned integer from a buffer.
- **Description**: This function is used to extract a 32-bit unsigned integer from a specified buffer. It should be called when there is a need to read a 32-bit value that has been stored in the buffer, ensuring that the buffer has been properly initialized and contains sufficient data. The function will convert the value from big-endian to host byte order. It is important to ensure that the buffer is not null and that it contains enough data to read a 32-bit integer; otherwise, the function may return an error.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure from which the 32-bit integer will be read. This buffer must not be null and should contain enough data to read a 32-bit integer (4 bytes). If the buffer does not contain sufficient data, the function will return an error.
    - `value`: A pointer to a `uint32_t` variable where the retrieved value will be stored. This pointer must not be null, and the caller retains ownership of the variable. The function will write the retrieved value into this variable.
- **Output**: Returns 0 on success, indicating that the value was successfully retrieved and converted. A negative value indicates an error occurred during the read operation.
- **See also**: [`ibuf_get_n32`](imsg-buffer.c.driver.md#ibuf_get_n32)  (Implementation)


---
### ibuf_get_n64<!-- {{#callable_declaration:ibuf_get_n64}} -->
Retrieves a 64-bit unsigned integer from a buffer.
- **Description**: This function is used to extract a 64-bit unsigned integer value from a specified buffer. It should be called when there is a need to read a 64-bit integer that has been previously stored in the buffer. The function expects that the buffer has been properly initialized and contains sufficient data to read the requested value. If the buffer does not contain enough data, the function will return an error code. The value retrieved will be converted from big-endian to host byte order before being returned.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure from which the 64-bit integer will be read. This buffer must be initialized and contain enough data to read a 64-bit integer. If the buffer is null or does not have sufficient data, the function will return an error.
    - `value`: A pointer to a `uint64_t` variable where the retrieved value will be stored. This pointer must not be null, and the caller retains ownership of the variable. If the function fails, the value pointed to by this parameter may remain unchanged.
- **Output**: Returns an integer indicating the success or failure of the operation. A return value of 0 indicates success, while a negative value indicates an error occurred during the read operation.
- **See also**: [`ibuf_get_n64`](imsg-buffer.c.driver.md#ibuf_get_n64)  (Implementation)


---
### ibuf_get_h16<!-- {{#callable_declaration:ibuf_get_h16}} -->
Retrieves a 16-bit unsigned integer from a buffer.
- **Description**: This function is used to extract a 16-bit unsigned integer value from a specified buffer. It should be called when there is a need to read data that has been previously written to the buffer in a compatible format. The buffer must be properly initialized and contain sufficient data to read the requested value. If the buffer does not contain enough data, the function will return an error code, indicating that the read operation could not be completed.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure from which the value will be read. This buffer must be initialized and contain enough data to read a 16-bit value. It must not be null.
    - `value`: A pointer to a `uint16_t` variable where the retrieved value will be stored. This pointer must not be null, and the caller retains ownership of the variable.
- **Output**: Returns 0 on success, or a negative error code if the read operation fails.
- **See also**: [`ibuf_get_h16`](imsg-buffer.c.driver.md#ibuf_get_h16)  (Implementation)


---
### ibuf_get_h32<!-- {{#callable_declaration:ibuf_get_h32}} -->
Retrieves a 32-bit unsigned integer from a buffer.
- **Description**: This function is used to extract a 32-bit unsigned integer value from a specified buffer. It should be called when there is a need to read data from the buffer that has been previously populated. The caller must ensure that the buffer contains enough data to read a 32-bit integer; otherwise, the function may fail. It is important to handle the return value properly to check for successful retrieval of the data.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure from which the value will be read. This must not be null and should point to a valid buffer that has been initialized and populated with data.
    - `value`: A pointer to a `uint32_t` variable where the retrieved value will be stored. This must not be null, and the caller retains ownership of this variable.
- **Output**: Returns 0 on success, indicating that the value was successfully retrieved. A non-zero value indicates an error, such as insufficient data in the buffer.
- **See also**: [`ibuf_get_h32`](imsg-buffer.c.driver.md#ibuf_get_h32)  (Implementation)


---
### ibuf_get_h64<!-- {{#callable_declaration:ibuf_get_h64}} -->
Retrieves a 64-bit value from a buffer.
- **Description**: This function is used to extract a 64-bit integer from a specified buffer. It should be called when there is a need to read a 64-bit value that has been previously added to the buffer. The buffer must be properly initialized and contain sufficient data to read the requested value. If the buffer does not contain enough data, the function will return an error code, indicating that the read operation could not be completed.
- **Inputs**:
    - `buf`: A pointer to the `ibuf` structure from which the 64-bit value will be read. This buffer must be initialized and contain enough data to read a 64-bit integer. If the buffer is null or does not have sufficient data, the function will return an error.
    - `value`: A pointer to a `uint64_t` variable where the retrieved value will be stored. This pointer must not be null, as the function will write the retrieved value to this location.
- **Output**: Returns an integer indicating the success or failure of the operation. A return value of 0 indicates success, while a negative value indicates an error occurred during the read operation.
- **See also**: [`ibuf_get_h64`](imsg-buffer.c.driver.md#ibuf_get_h64)  (Implementation)


---
### ibuf_get_string<!-- {{#callable_declaration:ibuf_get_string}} -->
Retrieves a string from the input buffer.
- **Description**: This function is used to extract a string of a specified length from an `ibuf` structure, which represents an input buffer. It should be called when there is a need to read a string from the buffer, ensuring that the requested length does not exceed the available data. The function checks if the buffer contains enough data; if not, it sets an error code and returns `NULL`. If the string is successfully retrieved, the buffer's read position is updated accordingly. It is important to manage the memory of the returned string, as it is dynamically allocated and must be freed by the caller.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure from which the string will be read. Must not be null. The function expects that the buffer has been properly initialized and contains data.
    - `len`: The length of the string to retrieve. Must be a positive value. If the length exceeds the available data in the buffer, the function will return `NULL` and set an error code.
- **Output**: Returns a pointer to a newly allocated string containing the requested data from the buffer. If the operation fails, it returns `NULL`.
- **See also**: [`ibuf_get_string`](imsg-buffer.c.driver.md#ibuf_get_string)  (Implementation)


---
### ibuf_skip<!-- {{#callable_declaration:ibuf_skip}} -->
Skips a specified number of bytes in the input buffer.
- **Description**: This function is used to advance the read position in an `ibuf` structure by a specified length. It should be called when you want to skip over data that has already been processed or is no longer needed. Before calling, ensure that the buffer has enough data to skip; otherwise, an error will be set. If the specified length exceeds the available data in the buffer, the function will fail and return an error.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure representing the input buffer. Must not be null. The buffer should be properly initialized and contain data. If the buffer does not have enough data to skip the specified length, the function will return an error.
    - `len`: The number of bytes to skip in the buffer. This value must be less than or equal to the size of the data currently in the buffer. If it exceeds the available data, the function will fail.
- **Output**: Returns 0 on success, indicating that the read position has been successfully advanced. Returns -1 on failure, and sets `errno` to indicate the error.
- **See also**: [`ibuf_skip`](imsg-buffer.c.driver.md#ibuf_skip)  (Implementation)


---
### ibuf_free<!-- {{#callable_declaration:ibuf_free}} -->
Frees the resources associated with an `ibuf` structure.
- **Description**: This function should be called to release the memory and file descriptor associated with an `ibuf` structure when it is no longer needed. It is important to ensure that the `ibuf` pointer is not null before calling this function. If the `ibuf` was allocated on the stack, the function will abort to prevent undefined behavior. Additionally, if the file descriptor within the `ibuf` is valid (non-negative), it will be closed before freeing the buffer memory. This function does not return any value.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure that needs to be freed. Must not be null. If the pointer points to an `ibuf` that was allocated on the stack, the function will abort. If the `fd` field of the `ibuf` is valid (greater than or equal to 0), the associated file descriptor will be closed.
- **Output**: None
- **See also**: [`ibuf_free`](imsg-buffer.c.driver.md#ibuf_free)  (Implementation)


---
### ibuf_fd_avail<!-- {{#callable_declaration:ibuf_fd_avail}} -->
Checks if a file descriptor is available.
- **Description**: This function is used to determine if the file descriptor associated with the given `ibuf` structure is valid and available for use. It should be called after the `ibuf` has been properly initialized. The function returns a non-zero value if the file descriptor is greater than or equal to zero, indicating that it is available; otherwise, it returns zero. This can be useful for checking the readiness of the buffer for I/O operations.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure that contains the file descriptor to be checked. This pointer must not be null. If the `fd` field of the `ibuf` structure is less than zero, the function will return zero, indicating that the file descriptor is not available.
- **Output**: Returns a non-zero value if the file descriptor is available (i.e., greater than or equal to zero), or zero if it is not available.
- **See also**: [`ibuf_fd_avail`](imsg-buffer.c.driver.md#ibuf_fd_avail)  (Implementation)


---
### ibuf_fd_get<!-- {{#callable_declaration:ibuf_fd_get}} -->
Retrieves and clears the file descriptor from the buffer.
- **Description**: This function is used to obtain the file descriptor associated with a given `ibuf` structure. It should be called when the file descriptor is needed for operations such as reading or writing. After calling this function, the file descriptor in the `ibuf` is set to -1, indicating that it has been retrieved and is no longer valid in the context of the `ibuf`. It is important to note that if the file descriptor is negative, which indicates it is for internal use, the function will return -1 without modifying the `ibuf`. Therefore, this function should only be called when the `ibuf` has been properly initialized and contains a valid file descriptor.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure that contains the file descriptor to be retrieved. This pointer must not be null. If the `fd` field of the `ibuf` is negative, the function will return -1 without modifying the `ibuf`.
- **Output**: Returns the file descriptor from the `ibuf` if it is valid; otherwise, returns -1 if the file descriptor is negative.
- **See also**: [`ibuf_fd_get`](imsg-buffer.c.driver.md#ibuf_fd_get)  (Implementation)


---
### ibuf_fd_set<!-- {{#callable_declaration:ibuf_fd_set}} -->
Sets the file descriptor for the given buffer.
- **Description**: This function is used to assign a file descriptor to an `ibuf` structure. It should be called when you need to associate a file descriptor with a buffer, typically after the buffer has been initialized. If the buffer's current file descriptor is valid, it will be closed before the new descriptor is set. It is important to ensure that the buffer does not reside on the stack, as this will lead to an abort. The function does not return a value.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure that represents the buffer. Must not be null. The buffer must be properly initialized before calling this function.
    - `fd`: An integer representing the file descriptor to be set. It should be a valid file descriptor (non-negative). If the value is negative, the file descriptor in the buffer will be set to -1.
- **Output**: None
- **See also**: [`ibuf_fd_set`](imsg-buffer.c.driver.md#ibuf_fd_set)  (Implementation)


---
### msgbuf_new<!-- {{#callable_declaration:msgbuf_new}} -->
Creates a new message buffer.
- **Description**: This function allocates and initializes a new `msgbuf` structure, which is used for managing message buffers in the system. It should be called when a new message buffer is needed, and it is essential to ensure that the caller properly frees the allocated buffer using `msgbuf_free` when it is no longer needed to avoid memory leaks. If memory allocation fails, the function will return `NULL`, so the caller should check for this condition before proceeding.
- **Inputs**:
    - `None`: This function does not take any parameters.
- **Output**: Returns a pointer to the newly created `msgbuf` structure, or `NULL` if memory allocation fails.
- **See also**: [`msgbuf_new`](imsg-buffer.c.driver.md#msgbuf_new)  (Implementation)


---
### msgbuf_new_reader<!-- {{#callable_declaration:msgbuf_new_reader}} -->
Creates a new message buffer reader.
- **Description**: This function is used to create a new message buffer reader, which is essential for reading messages with a specified header size. It must be called with a valid header size that is greater than zero and does not exceed half of the maximum read size. If the provided header size is invalid, the function will return `NULL` and set the `errno` to `EINVAL`. Additionally, it requires a function pointer for reading the header and an argument to be passed to that function. The caller is responsible for managing the memory of the returned message buffer, which should be freed when no longer needed.
- **Inputs**:
    - `hdrsz`: Specifies the size of the message header in bytes. Must be greater than 0 and less than or equal to IBUF_READ_SIZE / 2. If invalid, the function returns NULL and sets errno to EINVAL.
    - `readhdr`: A pointer to a function that reads the message header. This function must be compatible with the expected signature. Must not be NULL.
    - `arg`: A pointer to user-defined data that will be passed to the header reading function. Can be NULL if no additional data is needed.
- **Output**: Returns a pointer to a newly allocated `msgbuf` structure if successful, or NULL if an error occurs, such as invalid header size or memory allocation failure.
- **See also**: [`msgbuf_new_reader`](imsg-buffer.c.driver.md#msgbuf_new_reader)  (Implementation)


---
### msgbuf_free<!-- {{#callable_declaration:msgbuf_free}} -->
Frees the resources associated with a message buffer.
- **Description**: This function is intended to release the memory and resources allocated for a `msgbuf` structure. It should be called when the message buffer is no longer needed, typically after all operations on it have been completed. The function checks if the provided `msgbuf` pointer is `NULL` and safely returns without performing any action in that case. It is important to ensure that the `msgbuf` has been properly initialized before calling this function to avoid undefined behavior.
- **Inputs**:
    - `msgbuf`: A pointer to the `msgbuf` structure to be freed. This pointer must not be null; if it is null, the function will simply return without any action. The caller is responsible for ensuring that the `msgbuf` is no longer in use before calling this function.
- **Output**: This function does not return a value and does not mutate any inputs.
- **See also**: [`msgbuf_free`](imsg-buffer.c.driver.md#msgbuf_free)  (Implementation)


---
### msgbuf_clear<!-- {{#callable_declaration:msgbuf_clear}} -->
Clears all messages from a message buffer.
- **Description**: This function is used to reset a message buffer by removing all queued messages and freeing associated resources. It should be called when you want to clear the contents of a `msgbuf` instance, typically before reusing it or when cleaning up after processing messages. It is important to ensure that the `msgbuf` has been properly initialized before calling this function. After execution, the message buffer will be empty, and any previously allocated resources will be released.
- **Inputs**:
    - `msgbuf`: A pointer to a `struct msgbuf` that represents the message buffer to be cleared. This pointer must not be null and should point to a valid, initialized `msgbuf` instance. If the pointer is null or invalid, the behavior is undefined.
- **Output**: None
- **See also**: [`msgbuf_clear`](imsg-buffer.c.driver.md#msgbuf_clear)  (Implementation)


---
### msgbuf_queuelen<!-- {{#callable_declaration:msgbuf_queuelen}} -->
Returns the number of queued messages.
- **Description**: This function is used to retrieve the current number of messages that are queued in the specified `msgbuf`. It should be called when you need to check the status of the message queue, for instance, to determine if there are messages waiting to be processed. The caller must ensure that the `msgbuf` has been properly initialized before calling this function. If the `msgbuf` is null or uninitialized, the behavior is undefined.
- **Inputs**:
    - `msgbuf`: A pointer to a `msgbuf` structure representing the message buffer. Must not be null and should be properly initialized before use.
- **Output**: Returns a `uint32_t` value representing the number of messages currently queued in the `msgbuf`.
- **See also**: [`msgbuf_queuelen`](imsg-buffer.c.driver.md#msgbuf_queuelen)  (Implementation)


---
### ibuf_write<!-- {{#callable_declaration:ibuf_write}} -->
Writes data from a message buffer to a file descriptor.
- **Description**: This function is used to send data from a `msgbuf` structure to a specified file descriptor. It should be called when there is data queued in the message buffer that needs to be transmitted. The function will attempt to write all available data, and if the write operation is interrupted or cannot be completed due to resource limitations, it will handle these cases gracefully by returning 0 to indicate that no data was sent. It is important to ensure that the `msgbuf` is properly initialized and contains data before calling this function.
- **Inputs**:
    - `fd`: The file descriptor to which the data will be written. It must be a valid, open file descriptor. If the file descriptor is invalid or closed, the function will return -1.
    - `msgbuf`: A pointer to a `msgbuf` structure containing the data to be sent. This pointer must not be null, and the `msgbuf` should contain queued data. If the `msgbuf` is empty, the function will return 0 without performing any write operations.
- **Output**: Returns 0 on success if data was written or if there was nothing to write. Returns -1 on error, indicating that the write operation failed.
- **See also**: [`ibuf_write`](imsg-buffer.c.driver.md#ibuf_write)  (Implementation)


---
### msgbuf_write<!-- {{#callable_declaration:msgbuf_write}} -->
Writes a message buffer to a file descriptor.
- **Description**: This function is used to send messages contained in a `msgbuf` structure to a specified file descriptor. It should be called when there are messages queued in the `msgbuf` that need to be transmitted. The function handles the sending of the message and any associated file descriptors, ensuring that resources are properly managed. It is important to note that the function may block if the underlying socket is not ready for sending, and it will retry sending if interrupted. If the function encounters an error that is not recoverable, it will return -1.
- **Inputs**:
    - `fd`: The file descriptor to which the message buffer will be written. It must be a valid, open file descriptor. If the file descriptor is invalid, the function will return -1.
    - `msgbuf`: A pointer to a `msgbuf` structure containing the messages to be sent. This pointer must not be null, and the `msgbuf` should contain messages to be sent; if it is empty, the function will return 0 without sending anything.
- **Output**: Returns 0 on success if messages were sent, -1 on failure if an error occurred during sending.
- **See also**: [`msgbuf_write`](imsg-buffer.c.driver.md#msgbuf_write)  (Implementation)


---
### ibuf_read<!-- {{#callable_declaration:ibuf_read}} -->
Reads data from a file descriptor into a message buffer.
- **Description**: This function is used to read data from a specified file descriptor into a provided `msgbuf` structure. It should be called when there is a need to receive messages from a communication channel, such as a socket. Before calling, ensure that the `rbuf` member of the `msgbuf` structure is properly initialized and not null. If the read operation is interrupted or would block, the function handles these cases gracefully, allowing for retries or returning a specific status. The function modifies the `roff` member of the `msgbuf` to reflect the new read position, and it processes the newly arrived data.
- **Inputs**:
    - `fd`: The file descriptor from which to read data. It must be a valid, open file descriptor. If invalid, the function will return -1 and set errno to indicate the error.
    - `msgbuf`: A pointer to a `msgbuf` structure that must be initialized before use. The `rbuf` member of this structure must not be null. If it is null, the function will return -1 and set errno to EINVAL.
- **Output**: Returns the number of bytes processed if new data is read successfully, 0 if the connection is closed, or -1 if an error occurs.
- **See also**: [`ibuf_read`](imsg-buffer.c.driver.md#ibuf_read)  (Implementation)


---
### msgbuf_read<!-- {{#callable_declaration:msgbuf_read}} -->
Reads a message from a file descriptor.
- **Description**: This function is used to read messages from a specified file descriptor into a provided `msgbuf` structure. It must be called with a valid `msgbuf` that has been properly initialized, specifically ensuring that its `rbuf` member is not null. The function handles various conditions such as interrupted system calls and message size issues, retrying as necessary. If the connection is closed, it will return 0. The function may also pass a file descriptor through the message, which is managed internally. It is important to note that the function may modify the `roff` member of the `msgbuf` to reflect the new read position.
- **Inputs**:
    - `fd`: The file descriptor from which to read the message. It must be a valid, open file descriptor. If the descriptor is invalid or an error occurs during reading, the function will return -1.
    - `msgbuf`: A pointer to a `struct msgbuf` that must be initialized and have a non-null `rbuf` member. The function will modify the `roff` member of this structure to indicate the new read position. If `msgbuf->rbuf` is null, the function will set errno to EINVAL and return -1.
- **Output**: Returns the number of bytes read into the `msgbuf` on success, 0 if the connection is closed, and -1 on error, with errno set to indicate the error.
- **See also**: [`msgbuf_read`](imsg-buffer.c.driver.md#msgbuf_read)  (Implementation)


---
### msgbuf_get<!-- {{#callable_declaration:msgbuf_get}} -->
Retrieves and removes the first `ibuf` from a `msgbuf`.
- **Description**: This function is used to obtain the first `ibuf` from a `msgbuf`, effectively removing it from the message buffer's list of buffers. It should be called when there is a need to process or handle the next available buffer in the message queue. The caller must ensure that the `msgbuf` has been properly initialized and contains at least one `ibuf` before calling this function, as it will return `NULL` if the buffer list is empty.
- **Inputs**:
    - `msgbuf`: A pointer to a `msgbuf` structure from which the first `ibuf` will be retrieved. This pointer must not be null and should point to a valid `msgbuf` that has been initialized and populated with `ibuf` entries. If the `msgbuf` is empty, the function will return `NULL`.
- **Output**: Returns a pointer to the first `ibuf` in the `msgbuf` if available; otherwise, it returns `NULL`.
- **See also**: [`msgbuf_get`](imsg-buffer.c.driver.md#msgbuf_get)  (Implementation)


---
### imsgbuf_init<!-- {{#callable_declaration:imsgbuf_init}} -->
Initializes an `imsgbuf` structure.
- **Description**: This function is used to initialize an `imsgbuf` structure, which is essential for message handling in inter-process communication. It must be called before any operations on the `imsgbuf`, and it sets up the necessary fields, including a message buffer reader and process ID. The function expects a valid file descriptor to be passed, which will be associated with the `imsgbuf`. If the initialization fails, it returns -1, indicating an error, while a successful initialization returns 0.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure that will be initialized. Must not be null.
    - `fd`: An integer representing a valid file descriptor for communication. This should be a non-negative integer; passing an invalid file descriptor may lead to initialization failure.
- **Output**: Returns 0 on successful initialization or -1 if an error occurs during the process.
- **See also**: [`imsgbuf_init`](imsg.c.driver.md#imsgbuf_init)  (Implementation)


---
### imsgbuf_allow_fdpass<!-- {{#callable_declaration:imsgbuf_allow_fdpass}} -->
Allows file descriptor passing in the message buffer.
- **Description**: This function is used to enable the passing of file descriptors through the message buffer. It should be called when the `imsgbuf` is initialized and before any messages are sent that require file descriptor passing. Enabling this feature allows the receiving end of the message to access file descriptors that are sent along with the message, which can be useful for inter-process communication. There are no side effects if called multiple times, as it simply sets a flag.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure that represents the message buffer. This pointer must not be null and should point to a valid, initialized `imsgbuf`. The function does not take ownership of the `imsgbuf`.
- **Output**: None
- **See also**: [`imsgbuf_allow_fdpass`](imsg.c.driver.md#imsgbuf_allow_fdpass)  (Implementation)


---
### imsgbuf_set_maxsize<!-- {{#callable_declaration:imsgbuf_set_maxsize}} -->
Sets the maximum size for an imsg buffer.
- **Description**: This function is used to define the maximum size of an `imsgbuf`, which is essential for managing message sizes in inter-process communication. It should be called after initializing the `imsgbuf` and before any messages are sent or received. The maximum size must be greater than or equal to the size of the message header and must not have certain reserved bits set. If the provided size is invalid, the function will return an error, and the `imsgbuf` will not be modified.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure that will have its maximum size set. Must not be null and should be properly initialized before calling this function.
    - `maxsize`: The maximum size to set for the `imsgbuf`. It must be greater than or equal to `IMSG_HEADER_SIZE` and must not have the `IMSG_FD_MARK` bit set. If the value is invalid, the function will return -1 and set `errno` to EINVAL.
- **Output**: Returns 0 on success, indicating that the maximum size has been set successfully. Returns -1 on failure, with `errno` set to indicate the error.
- **See also**: [`imsgbuf_set_maxsize`](imsg.c.driver.md#imsgbuf_set_maxsize)  (Implementation)


---
### imsgbuf_read<!-- {{#callable_declaration:imsgbuf_read}} -->
Reads messages from an inter-process message buffer.
- **Description**: This function is used to read messages from an `imsgbuf` structure, which represents a buffer for inter-process communication. It should be called when there are messages to be processed, and it will read data from the associated file descriptor. The function's behavior may vary depending on the flags set in the `imsgbuf`; specifically, if the `IMSG_ALLOW_FDPASS` flag is set, it will use a different reading mechanism. It is important to ensure that the `imsgbuf` has been properly initialized before calling this function.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure that contains the message buffer and associated metadata. This pointer must not be null and should point to a valid, initialized `imsgbuf`. If the `imsgbuf` is not properly initialized, the behavior of the function is undefined.
- **Output**: Returns an integer indicating the number of bytes read from the message buffer. A return value of 0 indicates that no more messages are available to read, while a negative value indicates an error occurred during the read operation.
- **See also**: [`imsgbuf_read`](imsg.c.driver.md#imsgbuf_read)  (Implementation)


---
### imsgbuf_write<!-- {{#callable_declaration:imsgbuf_write}} -->
Writes data from an `imsgbuf` to its associated file descriptor.
- **Description**: This function is used to send messages from an `imsgbuf` to a file descriptor, which is typically a socket or a pipe. It should be called when there is data ready to be sent, and it will handle the writing process based on the flags set in the `imsgbuf`. The function may be called multiple times, but it is important to ensure that the `imsgbuf` is properly initialized and that the file descriptor is valid. If the `IMSG_ALLOW_FDPASS` flag is set, it will use a specific writing method that allows for file descriptor passing; otherwise, it will use a different method. The function does not handle invalid `imsgbuf` pointers gracefully, so the caller must ensure that the pointer is valid before calling this function.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure that contains the message data and the file descriptor to which the data will be written. This pointer must not be null and must point to a valid `imsgbuf` that has been initialized. If the pointer is invalid or the `imsgbuf` is not properly set up, the behavior is undefined.
- **Output**: Returns an integer indicating the success or failure of the write operation. A return value of 0 typically indicates success, while a negative value indicates an error occurred during the write process.
- **See also**: [`imsgbuf_write`](imsg.c.driver.md#imsgbuf_write)  (Implementation)


---
### imsgbuf_flush<!-- {{#callable_declaration:imsgbuf_flush}} -->
Flushes all messages from the message buffer.
- **Description**: This function is used to ensure that all messages currently queued in the specified message buffer are written out. It should be called when you want to clear the message buffer by sending all pending messages. The function will repeatedly attempt to write messages until the queue is empty. If any write operation fails, the function will return an error code. It is important to ensure that the `imsgbuf` has been properly initialized before calling this function.
- **Inputs**:
    - `imsgbuf`: A pointer to the `imsgbuf` structure that represents the message buffer. This pointer must not be null and should point to a valid, initialized message buffer. If the buffer is not properly initialized, the behavior of the function is undefined.
- **Output**: Returns 0 on success if all messages were successfully flushed; returns -1 if an error occurred during the write operation.
- **See also**: [`imsgbuf_flush`](imsg.c.driver.md#imsgbuf_flush)  (Implementation)


---
### imsgbuf_clear<!-- {{#callable_declaration:imsgbuf_clear}} -->
Clears the message buffer.
- **Description**: This function is used to clear the contents of a message buffer associated with an `imsgbuf` structure. It should be called when the message buffer is no longer needed, or before reusing it for new messages. The function sets the write buffer pointer to `NULL`, effectively releasing any resources associated with the buffer. It is important to ensure that the `imsgbuf` has been properly initialized before calling this function.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure that contains the message buffer to be cleared. This pointer must not be null and should point to a valid `imsgbuf` that has been initialized. If the pointer is null or points to an uninitialized structure, the behavior is undefined.
- **Output**: None
- **See also**: [`imsgbuf_clear`](imsg.c.driver.md#imsgbuf_clear)  (Implementation)


---
### imsgbuf_queuelen<!-- {{#callable_declaration:imsgbuf_queuelen}} -->
Returns the number of messages in the queue.
- **Description**: This function is used to retrieve the current length of the message queue associated with a given `imsgbuf`. It should be called when you need to check how many messages are pending in the queue, which can be useful for monitoring or debugging purposes. Ensure that the `imsgbuf` has been properly initialized before calling this function to avoid undefined behavior.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure representing the message buffer. Must not be null and should be initialized before use. Passing a null pointer or an uninitialized `imsgbuf` may lead to undefined behavior.
- **Output**: Returns a `uint32_t` value representing the number of messages currently in the queue. The value will be zero if there are no messages.
- **See also**: [`imsgbuf_queuelen`](imsg.c.driver.md#imsgbuf_queuelen)  (Implementation)


---
### imsg_get<!-- {{#callable_declaration:imsg_get}} -->
Retrieves an `imsg` from the specified `imsgbuf`.
- **Description**: This function is used to extract an `imsg` from an `imsgbuf`, which is a message buffer that holds inter-process messages. It should be called when there is a need to read a message from the buffer, typically after ensuring that the buffer has been properly initialized and populated with messages. The function will fill the provided `imsg` structure with the message header and data. If the buffer is empty or an error occurs while reading, the function will return an appropriate value indicating the status of the operation.
- **Inputs**:
    - `imsgbuf`: A pointer to the `imsgbuf` structure from which the message will be retrieved. This must not be null and should be properly initialized before calling the function.
    - `imsg`: A pointer to an `imsg` structure where the retrieved message will be stored. This must not be null, and the caller retains ownership of this structure.
- **Output**: Returns the size of the retrieved message in bytes, including the header size, or -1 if an error occurs, or 0 if there are no messages available.
- **See also**: [`imsg_get`](imsg.c.driver.md#imsg_get)  (Implementation)


---
### imsg_get_ibuf<!-- {{#callable_declaration:imsg_get_ibuf}} -->
Retrieves the input buffer from an imsg structure.
- **Description**: This function is used to extract the input buffer from a given `imsg` structure. It should be called when there is a need to access the data contained in the buffer of the `imsg`. Before calling this function, ensure that the `imsg` has been properly initialized and that its buffer is not empty, as an empty buffer will result in an error. If the buffer is empty, the function sets the `errno` to `EBADMSG` and returns -1, indicating a failure.
- **Inputs**:
    - `imsg`: A pointer to an `imsg` structure from which the input buffer will be retrieved. Must not be null.
    - `ibuf`: A pointer to an `ibuf` structure where the retrieved buffer will be stored. Must not be null.
- **Output**: Returns 0 on success, or -1 if the input buffer is empty, in which case `errno` is set to `EBADMSG`.
- **See also**: [`imsg_get_ibuf`](imsg.c.driver.md#imsg_get_ibuf)  (Implementation)


---
### imsg_get_data<!-- {{#callable_declaration:imsg_get_data}} -->
Retrieves data from an `imsg` structure.
- **Description**: This function is used to extract data from an `imsg` structure into a provided buffer. It should be called when the size of the data to be retrieved is known and matches the expected length. The function will return an error if the specified length is zero or if the size of the data in the `imsg` does not match the provided length. It is important to ensure that the buffer provided for data retrieval is large enough to hold the specified length.
- **Inputs**:
    - `imsg`: Pointer to an `imsg` structure from which data will be retrieved. Must not be null.
    - `data`: Pointer to the buffer where the retrieved data will be stored. Must not be null.
    - `len`: The length of data to retrieve. Must be greater than zero and must match the size of the data in the `imsg`. If it is zero, an error will be set.
- **Output**: Returns the number of bytes retrieved on success, or -1 on error, with `errno` set to indicate the specific error.
- **See also**: [`imsg_get_data`](imsg.c.driver.md#imsg_get_data)  (Implementation)


---
### imsg_get_fd<!-- {{#callable_declaration:imsg_get_fd}} -->
Retrieves the file descriptor associated with an imsg.
- **Description**: This function is used to obtain the file descriptor from an `imsg` structure, which is typically necessary for communication purposes in inter-process messaging. It should be called after ensuring that the `imsg` structure has been properly initialized and populated. The function does not perform any validation on the `imsg` pointer itself, so it is the caller's responsibility to ensure that it is not null and points to a valid `imsg` structure. If the `imsg` structure is invalid, the behavior is undefined.
- **Inputs**:
    - `imsg`: A pointer to an `imsg` structure from which the file descriptor will be retrieved. Must not be null and should point to a valid `imsg` instance. Passing a null or invalid pointer may lead to undefined behavior.
- **Output**: Returns the file descriptor associated with the `imsg`. If the `imsg` is invalid, the return value is undefined.
- **See also**: [`imsg_get_fd`](imsg.c.driver.md#imsg_get_fd)  (Implementation)


---
### imsg_get_id<!-- {{#callable_declaration:imsg_get_id}} -->
Retrieves the peer ID from an imsg structure.
- **Description**: This function is used to obtain the `peerid` from an `imsg` structure, which represents a message in the inter-process communication system. It should be called after ensuring that the `imsg` structure has been properly initialized and populated with data. The function does not perform any validation on the input, so it is the caller's responsibility to ensure that the `imsg` pointer is valid and points to a properly initialized structure.
- **Inputs**:
    - `imsg`: A pointer to an `imsg` structure from which the peer ID will be retrieved. Must not be null and should point to a valid `imsg` that has been initialized. If the pointer is null or invalid, the behavior is undefined.
- **Output**: Returns the `peerid` field from the `imsg` structure, which is a 32-bit unsigned integer representing the ID of the peer.
- **See also**: [`imsg_get_id`](imsg.c.driver.md#imsg_get_id)  (Implementation)


---
### imsg_get_len<!-- {{#callable_declaration:imsg_get_len}} -->
Returns the length of the message buffer.
- **Description**: This function is used to retrieve the length of the message buffer associated with a given `imsg` structure. It should be called when you need to know how much data is currently stored in the buffer, which can be useful for processing or transmitting the message. Ensure that the `imsg` pointer passed to the function is valid and points to an initialized `imsg` structure; otherwise, the behavior is undefined.
- **Inputs**:
    - `imsg`: A pointer to an `imsg` structure that contains the message buffer. Must not be null and should point to a valid, initialized `imsg` object.
- **Output**: Returns the size of the message buffer in bytes.
- **See also**: [`imsg_get_len`](imsg.c.driver.md#imsg_get_len)  (Implementation)


---
### imsg_get_pid<!-- {{#callable_declaration:imsg_get_pid}} -->
Retrieves the process ID from an imsg structure.
- **Description**: This function is used to obtain the process ID associated with a given `imsg` structure. It should be called after ensuring that the `imsg` is properly initialized and populated. The function directly accesses the `pid` field from the `hdr` member of the `imsg` structure, which is expected to be valid. There are no side effects from calling this function, and it is safe to call as long as the `imsg` pointer is valid.
- **Inputs**:
    - `imsg`: A pointer to an `imsg` structure from which the process ID will be retrieved. This pointer must not be null and should point to a valid `imsg` structure that has been initialized. If the pointer is null, the behavior is undefined.
- **Output**: Returns the process ID (of type `pid_t`) stored in the `imsg` structure's header. This value represents the ID of the process that sent the message.
- **See also**: [`imsg_get_pid`](imsg.c.driver.md#imsg_get_pid)  (Implementation)


---
### imsg_get_type<!-- {{#callable_declaration:imsg_get_type}} -->
Retrieves the type of an inter-message.
- **Description**: This function is used to obtain the type of a given inter-message, which is essential for processing messages correctly based on their type. It should be called after an inter-message has been received and is valid. The function does not modify the input message and is safe to call as long as the provided pointer is not null.
- **Inputs**:
    - `imsg`: A pointer to an `imsg` structure that contains the message header. This pointer must not be null and should point to a valid `imsg` instance. If the pointer is null, the behavior is undefined.
- **Output**: Returns the type of the inter-message as a `uint32_t` value, which indicates the specific type of the message.
- **See also**: [`imsg_get_type`](imsg.c.driver.md#imsg_get_type)  (Implementation)


---
### imsg_forward<!-- {{#callable_declaration:imsg_forward}} -->
Forwards an `imsg` to the specified `imsgbuf`.
- **Description**: This function is used to forward an `imsg` to a message buffer, which is typically called when a message needs to be sent to another process. It should be invoked after the `imsg` has been properly initialized and populated with data. The function handles the message's buffer by rewinding it and skipping the header before creating a new message in the target buffer. If the message length is non-zero, it attempts to add the message's buffer to the newly created message. If any operation fails, it returns an error code, ensuring that the caller can handle such cases appropriately.
- **Inputs**:
    - `imsgbuf`: A pointer to the `imsgbuf` structure that represents the target message buffer. Must not be null.
    - `msg`: A pointer to the `imsg` structure that contains the message to be forwarded. Must not be null.
- **Output**: Returns 1 on success, indicating that the message was successfully forwarded. Returns -1 on failure, indicating an error occurred during the forwarding process.
- **See also**: [`imsg_forward`](imsg.c.driver.md#imsg_forward)  (Implementation)


---
### imsg_compose<!-- {{#callable_declaration:imsg_compose}} -->
Composes and sends an inter-process message.
- **Description**: This function is used to create and send a message to a specified inter-process communication buffer. It should be called after initializing the `imsgbuf` structure. The function takes various parameters to define the message type, identifier, process ID, file descriptor, and the data to be sent. If the message creation or data addition fails, the function will return -1, indicating an error. It is important to ensure that the `data` pointer is valid and that `datalen` does not exceed the maximum message size.
- **Inputs**:
    - `imsgbuf`: Pointer to the `imsgbuf` structure that represents the message buffer. Must not be null and should be initialized before calling this function.
    - `type`: A 32-bit unsigned integer representing the type of the message. Valid values depend on the application's message type definitions.
    - `id`: A 32-bit unsigned integer that serves as an identifier for the message. This can be any valid identifier as per the application's requirements.
    - `pid`: The process ID (`pid_t`) of the target process. This can be set to 0 to indicate that the message is sent to all processes.
    - `fd`: An integer representing a file descriptor. This can be set to -1 if no file descriptor is to be sent with the message.
    - `data`: A pointer to the data to be sent with the message. Must not be null if `datalen` is greater than 0.
    - `datalen`: The size of the data in bytes. Must be less than or equal to the maximum message size defined by the application.
- **Output**: Returns 1 on success, indicating that the message was successfully composed and sent. Returns -1 on failure, indicating an error occurred during message creation or data addition.
- **See also**: [`imsg_compose`](imsg.c.driver.md#imsg_compose)  (Implementation)


---
### imsg_composev<!-- {{#callable_declaration:imsg_composev}} -->
Composes an inter-message with variable-length data.
- **Description**: This function is used to create and send an inter-message composed of multiple data segments specified by an array of `iovec` structures. It should be called after initializing the `imsgbuf` structure and is typically used when sending messages that consist of multiple parts. The function calculates the total length of the data to be sent, creates a new message buffer, and adds each segment of data to this buffer. If any of the parameters are invalid, such as a null `imsgbuf` or an invalid `iovcnt`, the function will return an error. It is important to ensure that the `iovec` array is properly populated and that the file descriptor is valid before calling this function.
- **Inputs**:
    - `imsgbuf`: Pointer to the `imsgbuf` structure that represents the message buffer. Must not be null.
    - `type`: An unsigned 32-bit integer representing the type of the message. Valid values depend on the application's message type definitions.
    - `id`: An unsigned 32-bit integer representing the identifier for the message. This can be any valid identifier as per the application's requirements.
    - `pid`: A process identifier (`pid_t`) representing the sender's process ID. This can be set to 0 if not applicable.
    - `fd`: An integer file descriptor that may be associated with the message. This can be set to -1 if no file descriptor is needed.
    - `iov`: Pointer to an array of `iovec` structures that describe the data segments to be sent. Must not be null.
    - `iovcnt`: An integer representing the number of elements in the `iov` array. Must be greater than 0.
- **Output**: Returns 1 on success, or -1 on failure, indicating that the message was not composed successfully.
- **See also**: [`imsg_composev`](imsg.c.driver.md#imsg_composev)  (Implementation)


---
### imsg_compose_ibuf<!-- {{#callable_declaration:imsg_compose_ibuf}} -->
Composes an inter-message buffer.
- **Description**: This function is used to compose a message to be sent through an inter-message buffer. It should be called when you need to send a message encapsulated in a buffer, ensuring that the total size of the message does not exceed the maximum allowed size of the `imsgbuf`. The function will automatically handle the message header and the provided buffer, and it is important to ensure that the buffer is valid and its size is within the acceptable range. If the function fails due to an invalid buffer size or other issues, it will return -1 and set the appropriate error code.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure that represents the message buffer to which the message will be composed. Must not be null.
    - `type`: A `uint32_t` value representing the type of the message. Valid values depend on the application's message type definitions.
    - `id`: A `uint32_t` value representing the identifier of the peer to which the message is directed. This should be a valid identifier.
    - `pid`: A `pid_t` value representing the process ID associated with the message. If set to 0, the function will use the process ID from the `imsgbuf`.
    - `buf`: A pointer to an `ibuf` structure containing the message data to be sent. Must not be null and its size must be within the limits defined by `imsgbuf->maxsize`.
- **Output**: Returns 1 on success, indicating that the message has been successfully composed and sent. Returns -1 on failure, with the error code set appropriately.
- **See also**: [`imsg_compose_ibuf`](imsg.c.driver.md#imsg_compose_ibuf)  (Implementation)


---
### imsg_create<!-- {{#callable_declaration:imsg_create}} -->
Creates a new message buffer.
- **Description**: This function is used to create a new message buffer that can hold a message with a specified type, identifier, and optional process ID. It should be called when a new message needs to be prepared for sending through an `imsgbuf`. The `datalen` parameter specifies the size of the message data, which must not exceed the maximum size defined in the `imsgbuf`. If the total size of the message, including the header, exceeds this limit, the function will return `NULL` and set `errno` to `ERANGE`. The function also handles the case where the provided `pid` is zero by using the `pid` from the `imsgbuf` instead.
- **Inputs**:
    - `imsgbuf`: Pointer to an `imsgbuf` structure that holds the message buffer context. Must not be null.
    - `type`: An unsigned 32-bit integer representing the type of the message. Valid values depend on the application's message type definitions.
    - `id`: An unsigned 32-bit integer representing the identifier for the message. This is typically used to identify the sender or the intended recipient.
    - `pid`: A process ID (`pid_t`) that can be used to specify the sender's process. If set to zero, the function will use the `pid` from the `imsgbuf`.
    - `datalen`: A size_t value indicating the length of the message data. This value will be increased by the size of the message header. Must not exceed the maximum size of the `imsgbuf`.
- **Output**: Returns a pointer to a newly created `ibuf` structure that contains the message buffer. If the creation fails due to exceeding the maximum size or other errors, it returns `NULL`.
- **See also**: [`imsg_create`](imsg.c.driver.md#imsg_create)  (Implementation)


---
### imsg_add<!-- {{#callable_declaration:imsg_add}} -->
Adds data to an imsg buffer.
- **Description**: This function is used to append a specified amount of data to an `imsg` buffer. It should be called after the buffer has been properly initialized. If the `datalen` is zero, the function will return zero without modifying the buffer. If `datalen` is non-zero and the addition of data fails, the buffer will be freed, and the function will return -1. Therefore, it is important to check the return value to ensure that the data was added successfully.
- **Inputs**:
    - `msg`: A pointer to the `ibuf` structure representing the message buffer. Must not be null and should be properly initialized before calling this function.
    - `data`: A pointer to the data to be added to the buffer. This can be null if `datalen` is zero.
    - `datalen`: The size of the data to be added, in bytes. Must be non-negative. If zero, no data is added, and the function will return zero.
- **Output**: Returns the number of bytes added to the buffer, which will be equal to `datalen` if successful, or -1 if an error occurred during the addition.
- **See also**: [`imsg_add`](imsg.c.driver.md#imsg_add)  (Implementation)


---
### imsg_close<!-- {{#callable_declaration:imsg_close}} -->
Closes an imsg buffer.
- **Description**: This function is used to close an `imsg` buffer after it has been prepared for sending. It should be called when the message is ready to be sent, ensuring that the message length is correctly set before closing the buffer. It is important to ensure that the `imsgbuf` and `msg` parameters are valid and properly initialized before calling this function. The function handles the message length and any associated file descriptors, marking them appropriately.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure that represents the message buffer to which the message will be sent. Must not be null and should be properly initialized before use.
    - `msg`: A pointer to an `ibuf` structure that contains the message to be sent. Must not be null and should contain a valid message that has been prepared for sending.
- **Output**: None
- **See also**: [`imsg_close`](imsg.c.driver.md#imsg_close)  (Implementation)


---
### imsg_free<!-- {{#callable_declaration:imsg_free}} -->
Frees the resources associated with an `imsg` structure.
- **Description**: This function should be called to release the memory and resources allocated for an `imsg` structure when it is no longer needed. It is important to ensure that the `imsg` pointer passed to this function is valid and has been previously allocated. Calling this function on a null pointer or an uninitialized `imsg` may lead to undefined behavior. After calling this function, the caller should not attempt to access the `imsg` structure.
- **Inputs**:
    - `imsg`: A pointer to an `imsg` structure that needs to be freed. This pointer must not be null and should point to a valid `imsg` that was previously allocated. Passing an invalid pointer may result in undefined behavior.
- **Output**: This function does not return a value and does not modify any input parameters.
- **See also**: [`imsg_free`](imsg.c.driver.md#imsg_free)  (Implementation)


