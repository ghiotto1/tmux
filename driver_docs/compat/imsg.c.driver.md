# Purpose
The provided C source code file is part of the OpenBSD project and implements an inter-process messaging (imsg) system. This system facilitates communication between different processes by providing a structured way to send and receive messages, potentially with file descriptors. The code defines a set of functions that manage message buffers, allowing for the initialization, reading, writing, and flushing of these buffers. It also includes functions for composing messages, either from raw data or from existing buffers, and for forwarding messages to other channels. The imsg system is designed to handle messages with headers that include metadata such as type, length, and process identifiers, ensuring that messages are correctly formatted and processed.

The file is not an executable but rather a library intended to be used by other parts of a software system that require inter-process communication. It provides a public API for managing message buffers and composing messages, which can be used by other programs or modules to implement communication protocols. The code includes mechanisms for handling file descriptor passing, which is a critical feature for many system-level applications that need to share resources between processes. The functions are designed to be robust, with error handling for various edge cases, such as invalid message sizes or failed memory allocations. Overall, this file is a crucial component for enabling efficient and reliable communication between processes in a Unix-like operating system environment.
# Imports and Dependencies

---
- `sys/types.h`
- `sys/socket.h`
- `sys/uio.h`
- `errno.h`
- `stddef.h`
- `stdlib.h`
- `string.h`
- `unistd.h`
- `compat.h`
- `imsg.h`


# Global Variables

---
### imsg_parse_hdr
- **Type**: `function pointer`
- **Description**: `imsg_parse_hdr` is a static function that parses the header of an inter-process message (imsg) from a buffer. It extracts the header information and prepares a new buffer for the message content, handling file descriptor passing if necessary.
- **Use**: This function is used internally to process incoming message headers and prepare them for further handling in the imsg system.


# Functions

---
### imsgbuf_init<!-- {{#callable:imsgbuf_init}} -->
The `imsgbuf_init` function initializes an `imsgbuf` structure for inter-process message handling by setting up a message buffer reader, storing the process ID, and configuring the file descriptor and maximum message size.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure that will be initialized.
    - `fd`: An integer representing the file descriptor to be associated with the `imsgbuf` for communication.
- **Control Flow**:
    - Call `msgbuf_new_reader` to create a new message buffer reader with a specified header size and parsing function, associating it with the `imsgbuf` structure.
    - Check if the message buffer reader creation was successful; if not, return -1 to indicate failure.
    - Set the `pid` field of the `imsgbuf` to the current process ID using `getpid()`.
    - Set the `maxsize` field of the `imsgbuf` to the constant `MAX_IMSGSIZE`.
    - Assign the provided file descriptor `fd` to the `fd` field of the `imsgbuf`.
    - Initialize the `flags` field of the `imsgbuf` to 0.
    - Return 0 to indicate successful initialization.
- **Output**:
    - Returns 0 on successful initialization, or -1 if the message buffer reader could not be created.


---
### imsgbuf_allow_fdpass<!-- {{#callable:imsgbuf_allow_fdpass}} -->
The `imsgbuf_allow_fdpass` function enables file descriptor passing for a given `imsgbuf` structure by setting a specific flag.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure, which is used for inter-process message passing.
- **Control Flow**:
    - The function accesses the `flags` field of the `imsgbuf` structure pointed to by the input argument.
    - It performs a bitwise OR operation on the `flags` field with the `IMSG_ALLOW_FDPASS` constant, effectively setting this flag.
- **Output**:
    - The function does not return any value; it modifies the `flags` field of the `imsgbuf` structure in place.


---
### imsgbuf_set_maxsize<!-- {{#callable:imsgbuf_set_maxsize}} -->
The `imsgbuf_set_maxsize` function sets the maximum size for messages in an `imsgbuf` structure, ensuring the size is valid and does not contain a file descriptor mark.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure whose maximum message size is to be set.
    - `maxsize`: A `uint32_t` value representing the desired maximum size for messages in the `imsgbuf`.
- **Control Flow**:
    - Check if `maxsize` is less than `IMSG_HEADER_SIZE` or if it has the `IMSG_FD_MARK` bit set.
    - If either condition is true, set `errno` to `EINVAL` and return `-1` to indicate an error.
    - If both conditions are false, set the `maxsize` field of the `imsgbuf` structure to the provided `maxsize` value.
    - Return `0` to indicate success.
- **Output**:
    - Returns `0` on success, or `-1` on failure with `errno` set to `EINVAL` if the `maxsize` is invalid.


---
### imsgbuf_read<!-- {{#callable:imsgbuf_read}} -->
The `imsgbuf_read` function reads data from a file descriptor into a message buffer, with optional file descriptor passing.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure, which contains the file descriptor and message buffer to read into.
- **Control Flow**:
    - Check if the `IMSG_ALLOW_FDPASS` flag is set in the `imsgbuf` structure's `flags` field.
    - If the flag is set, call `msgbuf_read` with the file descriptor and message buffer from `imsgbuf`.
    - If the flag is not set, call `ibuf_read` with the file descriptor and message buffer from `imsgbuf`.
- **Output**:
    - Returns the result of either `msgbuf_read` or `ibuf_read`, which is typically the number of bytes read or -1 on error.


---
### imsgbuf_write<!-- {{#callable:imsgbuf_write}} -->
The `imsgbuf_write` function writes data from an imsg buffer to a file descriptor, with optional file descriptor passing.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure, which contains the file descriptor and buffer to write from.
- **Control Flow**:
    - Check if the `IMSG_ALLOW_FDPASS` flag is set in the `imsgbuf` structure's `flags` field.
    - If the flag is set, call `msgbuf_write` with the file descriptor and buffer from `imsgbuf`.
    - If the flag is not set, call `ibuf_write` with the file descriptor and buffer from `imsgbuf`.
- **Output**:
    - Returns the result of either `msgbuf_write` or `ibuf_write`, which is typically the number of bytes written or -1 on error.


---
### imsgbuf_flush<!-- {{#callable:imsgbuf_flush}} -->
The `imsgbuf_flush` function attempts to write all queued messages in an `imsgbuf` structure to their destination, returning success or failure.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure, which contains the message buffer and associated metadata for inter-process communication.
- **Control Flow**:
    - The function enters a loop that continues as long as there are messages in the queue, determined by `imsgbuf_queuelen(imsgbuf) > 0`.
    - Within the loop, it calls `imsgbuf_write(imsgbuf)` to attempt to write the next message in the queue.
    - If `imsgbuf_write(imsgbuf)` returns -1, indicating a failure to write, the function immediately returns -1 to signal an error.
    - If all messages are successfully written, the loop exits and the function returns 0 to indicate success.
- **Output**:
    - The function returns 0 on successful flushing of all messages, or -1 if an error occurs during the write process.
- **Functions called**:
    - [`imsgbuf_queuelen`](#imsgbuf_queuelen)
    - [`imsgbuf_write`](#imsgbuf_write)


---
### imsgbuf_clear<!-- {{#callable:imsgbuf_clear}} -->
The `imsgbuf_clear` function releases the resources associated with the write buffer of an `imsgbuf` structure and sets the buffer pointer to NULL.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure whose write buffer is to be cleared.
- **Control Flow**:
    - Call the `msgbuf_free` function to release the resources associated with the write buffer `imsgbuf->w`.
    - Set the write buffer pointer `imsgbuf->w` to NULL to indicate that it no longer points to a valid buffer.
- **Output**:
    - The function does not return a value; it performs its operations directly on the `imsgbuf` structure passed to it.


---
### imsgbuf_queuelen<!-- {{#callable:imsgbuf_queuelen}} -->
The `imsgbuf_queuelen` function returns the length of the message queue associated with a given `imsgbuf` structure.
- **Inputs**:
    - `imsgbuf`: A pointer to a `struct imsgbuf`, which contains the message buffer whose queue length is to be determined.
- **Control Flow**:
    - The function calls `msgbuf_queuelen` with the `w` member of the `imsgbuf` structure, which represents the write buffer.
    - It directly returns the result of the `msgbuf_queuelen` function call.
- **Output**:
    - The function returns a `uint32_t` value representing the number of messages currently queued in the message buffer.


---
### imsg_get<!-- {{#callable:imsg_get}} -->
The `imsg_get` function retrieves a message from an imsg buffer and populates an imsg structure with the message data.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure, which represents the message buffer from which the message is to be retrieved.
    - `imsg`: A pointer to an `imsg` structure, which will be populated with the retrieved message data.
- **Control Flow**:
    - Attempt to retrieve a buffer from the message buffer `imsgbuf->w` using `msgbuf_get`; if unsuccessful, return 0.
    - Use `ibuf_get` to extract the message header from the buffer; if unsuccessful, return -1.
    - Check if the buffer has additional data using `ibuf_size`; if so, set `m.data` to the buffer's data, otherwise set it to NULL.
    - Assign the buffer to `m.buf` and clear the `IMSG_FD_MARK` from the message header length.
    - Copy the local `imsg` structure `m` to the provided `imsg` pointer.
    - Return the size of the buffer plus the size of the message header.
- **Output**:
    - Returns the total size of the message, including the header, or 0 if no message is available, or -1 on error.


---
### imsg_get_ibuf<!-- {{#callable:imsg_get_ibuf}} -->
The `imsg_get_ibuf` function retrieves an internal buffer from an imsg structure and stores it in a provided ibuf structure, returning an error if the buffer is empty.
- **Inputs**:
    - `imsg`: A pointer to a struct imsg, which contains a buffer from which data is to be retrieved.
    - `ibuf`: A pointer to a struct ibuf, where the retrieved buffer data will be stored.
- **Control Flow**:
    - Check if the size of the buffer in the imsg structure is zero.
    - If the buffer size is zero, set the errno to EBADMSG and return -1 to indicate an error.
    - If the buffer size is not zero, call the function `ibuf_get_ibuf` with the imsg buffer, its size, and the ibuf pointer to retrieve the buffer data.
- **Output**:
    - Returns 0 on success, or -1 if the imsg buffer is empty, with errno set to EBADMSG.


---
### imsg_get_data<!-- {{#callable:imsg_get_data}} -->
The `imsg_get_data` function retrieves data from an imsg buffer into a provided data buffer, ensuring the data length matches the expected size.
- **Inputs**:
    - `imsg`: A pointer to a struct imsg, which contains the buffer from which data is to be retrieved.
    - `data`: A pointer to a memory location where the retrieved data will be stored.
    - `len`: The expected length of the data to be retrieved from the imsg buffer.
- **Control Flow**:
    - Check if the provided length 'len' is zero; if so, set errno to EINVAL and return -1 indicating an error.
    - Check if the size of the buffer in 'imsg' does not match 'len'; if so, set errno to EBADMSG and return -1 indicating an error.
    - If both checks pass, call 'ibuf_get' to retrieve the data from 'imsg->buf' into 'data' with the specified 'len' and return the result.
- **Output**:
    - The function returns 0 on success, or -1 on failure, setting errno to indicate the error.


---
### imsg_get_fd<!-- {{#callable:imsg_get_fd}} -->
The `imsg_get_fd` function retrieves a file descriptor from a given inter-process message structure.
- **Inputs**:
    - `imsg`: A pointer to a struct imsg, which contains a buffer from which the file descriptor is to be retrieved.
- **Control Flow**:
    - The function calls `ibuf_fd_get` with the buffer from the `imsg` structure as its argument.
    - The result of `ibuf_fd_get` is returned directly as the output of the function.
- **Output**:
    - The function returns an integer representing the file descriptor obtained from the `imsg` buffer.


---
### imsg_get_id<!-- {{#callable:imsg_get_id}} -->
The `imsg_get_id` function retrieves the peer ID from the header of an inter-process message (imsg).
- **Inputs**:
    - `imsg`: A pointer to a struct `imsg`, which contains the message header and associated data.
- **Control Flow**:
    - Access the `hdr` member of the `imsg` structure.
    - Return the `peerid` field from the `hdr` member.
- **Output**:
    - The function returns a `uint32_t` value representing the peer ID of the message.


---
### imsg_get_len<!-- {{#callable:imsg_get_len}} -->
The `imsg_get_len` function returns the size of the buffer associated with a given inter-process message (imsg).
- **Inputs**:
    - `imsg`: A pointer to a struct imsg, which contains a buffer whose size is to be determined.
- **Control Flow**:
    - The function calls `ibuf_size` with the buffer from the `imsg` structure as an argument.
    - It returns the result of the `ibuf_size` function call, which is the size of the buffer.
- **Output**:
    - The function returns a `size_t` value representing the size of the buffer in the `imsg` structure.


---
### imsg_get_pid<!-- {{#callable:imsg_get_pid}} -->
The `imsg_get_pid` function retrieves the process ID (PID) from the header of an inter-process message (imsg).
- **Inputs**:
    - `imsg`: A pointer to a struct imsg, which contains the message header and associated data.
- **Control Flow**:
    - The function accesses the `hdr` field of the `imsg` structure, which is expected to be a struct containing a `pid` field.
    - It returns the value of the `pid` field from the `hdr` structure.
- **Output**:
    - The function returns a `pid_t` type, which is the process ID stored in the message header.


---
### imsg_get_type<!-- {{#callable:imsg_get_type}} -->
The `imsg_get_type` function retrieves the type of an inter-process message from its header.
- **Inputs**:
    - `imsg`: A pointer to a struct `imsg`, which contains the message header and associated data.
- **Control Flow**:
    - Access the `hdr` field of the `imsg` structure, which contains the message header.
    - Return the `type` field from the `hdr` structure, representing the message type.
- **Output**:
    - The function returns a `uint32_t` value representing the type of the message.


---
### imsg_compose<!-- {{#callable:imsg_compose}} -->
The `imsg_compose` function creates and sends an inter-process message with optional data and file descriptor.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure that manages the message buffer.
    - `type`: A 32-bit unsigned integer representing the message type.
    - `id`: A 32-bit unsigned integer representing the message identifier.
    - `pid`: A process ID (pid_t) associated with the message.
    - `fd`: An integer file descriptor to be associated with the message.
    - `data`: A pointer to the data to be included in the message.
    - `datalen`: The size of the data to be included in the message.
- **Control Flow**:
    - Attempt to create a new message buffer `wbuf` using [`imsg_create`](#imsg_create) with the provided parameters.
    - If [`imsg_create`](#imsg_create) returns NULL, indicating failure, return -1.
    - Add the provided data to the message buffer `wbuf` using [`imsg_add`](#imsg_add). If this fails, return -1.
    - Set the file descriptor for the message buffer `wbuf` using `ibuf_fd_set`.
    - Close the message buffer `wbuf` and finalize the message using [`imsg_close`](#imsg_close).
    - Return 1 to indicate success.
- **Output**:
    - Returns 1 on success, or -1 on failure.
- **Functions called**:
    - [`imsg_create`](#imsg_create)
    - [`imsg_add`](#imsg_add)
    - [`imsg_close`](#imsg_close)


---
### imsg_composev<!-- {{#callable:imsg_composev}} -->
The `imsg_composev` function constructs and sends a message with a vector of data buffers to a specified message buffer.
- **Inputs**:
    - `imsgbuf`: A pointer to the `imsgbuf` structure where the message will be composed and sent.
    - `type`: A 32-bit unsigned integer representing the type of the message.
    - `id`: A 32-bit unsigned integer representing the identifier of the message.
    - `pid`: A process ID (pid_t) associated with the message.
    - `fd`: An integer file descriptor to be associated with the message.
    - `iov`: A pointer to an array of `iovec` structures containing the data to be included in the message.
    - `iovcnt`: An integer representing the number of elements in the `iov` array.
- **Control Flow**:
    - Initialize `datalen` to 0 to store the total length of data from the `iovec` array.
    - Iterate over each `iovec` in the `iov` array to accumulate the total data length in `datalen`.
    - Call [`imsg_create`](#imsg_create) to create a new message buffer `wbuf` with the specified type, id, pid, and total data length; return -1 if creation fails.
    - Iterate over each `iovec` again to add its data to `wbuf` using [`imsg_add`](#imsg_add); return -1 if adding fails.
    - Set the file descriptor `fd` in `wbuf` using `ibuf_fd_set`.
    - Close the message by calling [`imsg_close`](#imsg_close) with `imsgbuf` and `wbuf`.
    - Return 1 to indicate success.
- **Output**:
    - Returns 1 on success, or -1 if an error occurs during message creation or data addition.
- **Functions called**:
    - [`imsg_create`](#imsg_create)
    - [`imsg_add`](#imsg_add)
    - [`imsg_close`](#imsg_close)


---
### imsg_compose_ibuf<!-- {{#callable:imsg_compose_ibuf}} -->
The `imsg_compose_ibuf` function composes and enqueues an inter-process message with a payload from a given buffer into an imsg buffer, without allowing file descriptor passing.
- **Inputs**:
    - `imsgbuf`: A pointer to the `imsgbuf` structure where the composed message will be enqueued.
    - `type`: A 32-bit unsigned integer representing the type of the message.
    - `id`: A 32-bit unsigned integer representing the peer ID associated with the message.
    - `pid`: A process ID (pid_t) that will be included in the message header; if zero, the pid from `imsgbuf` is used.
    - `buf`: A pointer to an `ibuf` structure containing the payload data to be included in the message.
- **Control Flow**:
    - Check if the total size of the buffer plus the header size exceeds the maximum allowed size in `imsgbuf`; if so, set `errno` to `ERANGE` and go to the fail label.
    - Initialize the message header with the provided type, calculated length, peer ID, and process ID (using `imsgbuf`'s pid if the provided pid is zero).
    - Attempt to open a new buffer for the message header; if unsuccessful, go to the fail label.
    - Add the header to the newly opened buffer; if unsuccessful, go to the fail label.
    - Close the header buffer and the payload buffer, enqueuing them into the `imsgbuf`'s write buffer.
    - Return 1 to indicate success.
    - In the fail label, save the current `errno`, free the payload and header buffers, restore `errno`, and return -1 to indicate failure.
- **Output**:
    - Returns 1 on success, or -1 on failure with `errno` set to indicate the error.
- **Functions called**:
    - [`imsg_add`](#imsg_add)


---
### imsg_forward<!-- {{#callable:imsg_forward}} -->
The `imsg_forward` function forwards an inter-process message to another channel, closing any attached file descriptor.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure representing the message buffer to which the message will be forwarded.
    - `msg`: A pointer to an `imsg` structure representing the message to be forwarded.
- **Control Flow**:
    - Rewinds the buffer of the message `msg` to the beginning.
    - Skips the header part of the message buffer to focus on the payload.
    - Calculates the size of the remaining message buffer after the header.
    - Creates a new message buffer `wbuf` in `imsgbuf` with the same type, peer ID, and PID as the original message, and the calculated payload length.
    - If the payload length is non-zero, attempts to add the payload from the original message buffer to the new buffer `wbuf`.
    - If adding the payload fails, frees the new buffer and returns -1.
    - Closes the message in `imsgbuf` by finalizing the buffer `wbuf`.
    - Returns 1 to indicate successful forwarding.
- **Output**:
    - Returns 1 on successful forwarding of the message, or -1 if an error occurs during the process.
- **Functions called**:
    - [`imsg_create`](#imsg_create)
    - [`imsg_close`](#imsg_close)


---
### imsg_create<!-- {{#callable:imsg_create}} -->
The `imsg_create` function initializes and returns a new dynamic buffer for inter-process message communication, including a header with specified parameters.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure that contains information about the message buffer, including the maximum size and process ID.
    - `type`: A 32-bit unsigned integer representing the type of the message.
    - `id`: A 32-bit unsigned integer representing the peer ID associated with the message.
    - `pid`: A process ID (pid_t) that identifies the process sending the message; if zero, the function uses the `pid` from `imsgbuf`.
    - `datalen`: A size_t value representing the length of the data to be included in the message, excluding the header size.
- **Control Flow**:
    - Add the size of the message header to `datalen` to account for the total message size.
    - Check if the total `datalen` exceeds the maximum size allowed by `imsgbuf`; if so, set `errno` to `ERANGE` and return `NULL`.
    - Initialize the `hdr` structure with the provided `type`, `id`, and `pid`; if `pid` is zero, use the `pid` from `imsgbuf`.
    - Attempt to create a dynamic buffer `wbuf` with the calculated `datalen` and the maximum size from `imsgbuf`; return `NULL` if unsuccessful.
    - Add the `hdr` to `wbuf` using [`imsg_add`](#imsg_add); return `NULL` if this operation fails.
    - Return the initialized buffer `wbuf`.
- **Output**:
    - Returns a pointer to a dynamically allocated `ibuf` structure containing the message header and space for the message data, or `NULL` if an error occurs.
- **Functions called**:
    - [`imsg_add`](#imsg_add)


---
### imsg_add<!-- {{#callable:imsg_add}} -->
The `imsg_add` function attempts to add data to a message buffer and returns the length of the data added or -1 on failure.
- **Inputs**:
    - `msg`: A pointer to an `ibuf` structure representing the message buffer to which data will be added.
    - `data`: A pointer to the data that needs to be added to the message buffer.
    - `datalen`: The size of the data to be added, in bytes.
- **Control Flow**:
    - Check if `datalen` is non-zero; if it is, attempt to add the data to the message buffer using `ibuf_add`.
    - If `ibuf_add` returns -1, indicating failure, free the message buffer using `ibuf_free` and return -1.
    - If the data is successfully added, return the length of the data added (`datalen`).
- **Output**:
    - The function returns the length of the data added to the message buffer (`datalen`) on success, or -1 if an error occurs during the addition of data.


---
### imsg_close<!-- {{#callable:imsg_close}} -->
The `imsg_close` function finalizes an inter-process message by setting its length and closing the buffer.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure, which represents the message buffer context.
    - `msg`: A pointer to an `ibuf` structure, which represents the message buffer to be closed.
- **Control Flow**:
    - Retrieve the size of the message buffer using `ibuf_size` and store it in `len`.
    - Check if a file descriptor is available in the message buffer using `ibuf_fd_avail`; if so, set the `IMSG_FD_MARK` flag in `len`.
    - Set the length of the message header in the buffer using `ibuf_set_h32`, specifying the offset for the length field.
    - Close the message buffer using `ibuf_close`, passing the write buffer from `imsgbuf` and the message buffer `msg`.
- **Output**:
    - The function does not return a value; it performs operations on the provided message buffer to finalize it for sending.


---
### imsg_free<!-- {{#callable:imsg_free}} -->
The `imsg_free` function releases the memory associated with the buffer of an inter-process message.
- **Inputs**:
    - `imsg`: A pointer to a `struct imsg` which contains a buffer that needs to be freed.
- **Control Flow**:
    - The function calls `ibuf_free` with the buffer (`imsg->buf`) from the `imsg` structure to release its allocated memory.
- **Output**:
    - The function does not return any value; it performs a cleanup operation on the provided `imsg` structure's buffer.


---
### imsg_parse_hdr<!-- {{#callable:imsg_parse_hdr}} -->
The `imsg_parse_hdr` function parses the header of an inter-process message from a buffer and prepares a new buffer for the message content, handling file descriptor passing if necessary.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure containing the message data to be parsed.
    - `arg`: A pointer to an `imsgbuf` structure, which provides context such as the maximum allowed message size.
    - `fd`: A pointer to an integer representing a file descriptor, which may be set if the message includes a file descriptor.
- **Control Flow**:
    - Retrieve the message header from the provided buffer using `ibuf_get` and store it in a local `imsg_hdr` structure.
    - Extract the message length from the header, masking out the file descriptor mark bit.
    - Check if the message length is valid (i.e., not smaller than the header size and not exceeding the maximum size allowed by `imsgbuf`).
    - If the length is invalid, set `errno` to `ERANGE` and return `NULL`.
    - Attempt to open a new buffer of the specified length using `ibuf_open`; return `NULL` if this fails.
    - If the original message length indicates a file descriptor is included (using the `IMSG_FD_MARK`), set the file descriptor in the new buffer and reset the input file descriptor to -1.
    - Return the newly created buffer.
- **Output**:
    - A pointer to a newly allocated `ibuf` structure containing the message content, or `NULL` if an error occurs.


# Function Declarations (Public API)

---
### imsg_parse_hdr<!-- {{#callable_declaration:imsg_parse_hdr}} -->
Parses an imsg header from a buffer and prepares a new buffer for the message.
- **Description**: This function is used to parse the header of an inter-process message (imsg) from a given buffer and prepare a new buffer for the message content. It should be called when a message header needs to be extracted and validated. The function checks if the message length is within valid bounds and handles file descriptor passing if indicated by the header. It returns a new buffer for the message content or NULL if an error occurs, such as an invalid message length or memory allocation failure.
- **Inputs**:
    - `buf`: A pointer to an ibuf structure containing the message data. Must not be null and should contain at least the size of an imsg header.
    - `arg`: A pointer to an imsgbuf structure, which provides context for parsing, including the maximum allowed message size. Must not be null.
    - `fd`: A pointer to an integer representing a file descriptor. If the message header indicates a file descriptor is present, it will be set in the new buffer and the original value will be set to -1. Must not be null.
- **Output**: Returns a pointer to a new ibuf structure containing the message content, or NULL if an error occurs.
- **See also**: [`imsg_parse_hdr`](#imsg_parse_hdr)  (Implementation)


