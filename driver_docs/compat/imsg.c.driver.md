# Purpose
The provided C source code file is part of the OpenBSD project and implements an inter-process messaging (imsg) system. This system facilitates communication between different processes by providing a structured way to send and receive messages, potentially with file descriptors attached. The code defines a set of functions that manage message buffers, allowing for the initialization, reading, writing, and clearing of these buffers. It also includes functions for composing messages, either from raw data or from existing buffers, and for forwarding messages to other channels. The imsg system is designed to handle messages with headers that include metadata such as message type, peer ID, and process ID, which are crucial for identifying and routing messages correctly.

The file is not an executable but rather a library intended to be used by other parts of a system that require inter-process communication. It provides a public API through functions like [`imsgbuf_init`](#imsgbuf_init), [`imsg_compose`](#imsg_compose), and [`imsg_get`](#imsg_get), which are used to manage the lifecycle of message buffers and to facilitate the sending and receiving of messages. The code also supports advanced features such as file descriptor passing, which is controlled by specific flags. This functionality is critical in systems programming, where processes often need to share resources like sockets or files. The code is modular, with a clear separation of concerns, making it a robust solution for message-based communication in a Unix-like operating system environment.
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
- **Description**: `imsg_parse_hdr` is a static function that parses the header of an inter-process message (imsg) from a buffer. It extracts the header information and checks the validity of the message length, ensuring it is within acceptable bounds.
- **Use**: This function is used internally to parse and validate the header of a message in the imsg communication framework.


# Functions

---
### imsgbuf_init<!-- {{#callable:imsgbuf_init}} -->
The `imsgbuf_init` function initializes an `imsgbuf` structure for inter-process communication by setting up a message buffer reader and configuring various parameters.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure that will be initialized.
    - `fd`: An integer file descriptor that will be associated with the `imsgbuf` for reading and writing messages.
- **Control Flow**:
    - Call [`msgbuf_new_reader`](imsg-buffer.c.driver.md#msgbuf_new_reader) with `IMSG_HEADER_SIZE`, `imsg_parse_hdr`, and `imsgbuf` to initialize the message buffer reader and assign it to `imsgbuf->w`.
    - Check if `imsgbuf->w` is `NULL`, indicating failure to initialize the reader, and return `-1` if so.
    - Set `imsgbuf->pid` to the current process ID using `getpid()`.
    - Set `imsgbuf->maxsize` to `MAX_IMSGSIZE`, defining the maximum size for messages.
    - Assign the provided file descriptor `fd` to `imsgbuf->fd`.
    - Initialize `imsgbuf->flags` to `0`.
    - Return `0` to indicate successful initialization.
- **Output**:
    - Returns `0` on successful initialization, or `-1` if the message buffer reader could not be created.
- **Functions called**:
    - [`msgbuf_new_reader`](imsg-buffer.c.driver.md#msgbuf_new_reader)


---
### imsgbuf_allow_fdpass<!-- {{#callable:imsgbuf_allow_fdpass}} -->
The function `imsgbuf_allow_fdpass` enables file descriptor passing for a given `imsgbuf` structure by setting a specific flag.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure, which is used for inter-process message passing.
- **Control Flow**:
    - The function accesses the `flags` field of the `imsgbuf` structure pointed to by the input argument.
    - It performs a bitwise OR operation on the `flags` field with the constant `IMSG_ALLOW_FDPASS`, effectively setting this flag.
- **Output**:
    - The function does not return any value; it modifies the `flags` field of the `imsgbuf` structure in place.


---
### imsgbuf_set_maxsize<!-- {{#callable:imsgbuf_set_maxsize}} -->
The function `imsgbuf_set_maxsize` sets the maximum size for an imsg buffer, ensuring it meets certain constraints.
- **Inputs**:
    - `imsgbuf`: A pointer to a struct imsgbuf, which represents the message buffer whose maximum size is to be set.
    - `maxsize`: A uint32_t value representing the desired maximum size for the imsg buffer.
- **Control Flow**:
    - Check if the provided maxsize is less than IMSG_HEADER_SIZE or if it has the IMSG_FD_MARK bit set.
    - If either condition is true, set errno to EINVAL and return -1 to indicate an error.
    - If the conditions are not met, set the maxsize field of the imsgbuf structure to the provided maxsize value.
    - Return 0 to indicate success.
- **Output**:
    - Returns 0 on success, or -1 on failure with errno set to EINVAL if the maxsize is invalid.


---
### imsgbuf_read<!-- {{#callable:imsgbuf_read}} -->
The `imsgbuf_read` function reads data from a file descriptor into a message buffer, with optional file descriptor passing.
- **Inputs**:
    - `imsgbuf`: A pointer to a `struct imsgbuf` which contains the file descriptor and message buffer to read into, as well as flags indicating if file descriptor passing is allowed.
- **Control Flow**:
    - Check if the `IMSG_ALLOW_FDPASS` flag is set in the `imsgbuf` structure's flags.
    - If the flag is set, call [`msgbuf_read`](imsg-buffer.c.driver.md#msgbuf_read) with the file descriptor and message buffer from `imsgbuf`.
    - If the flag is not set, call [`ibuf_read`](imsg-buffer.c.driver.md#ibuf_read) with the file descriptor and message buffer from `imsgbuf`.
- **Output**:
    - The function returns the result of either [`msgbuf_read`](imsg-buffer.c.driver.md#msgbuf_read) or [`ibuf_read`](imsg-buffer.c.driver.md#ibuf_read), which is typically the number of bytes read or -1 on error.
- **Functions called**:
    - [`msgbuf_read`](imsg-buffer.c.driver.md#msgbuf_read)
    - [`ibuf_read`](imsg-buffer.c.driver.md#ibuf_read)


---
### imsgbuf_write<!-- {{#callable:imsgbuf_write}} -->
The `imsgbuf_write` function writes data from an imsg buffer to a file descriptor, with optional file descriptor passing.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure, which contains the file descriptor and buffer to write from, as well as flags indicating if file descriptor passing is allowed.
- **Control Flow**:
    - Check if the `IMSG_ALLOW_FDPASS` flag is set in the `imsgbuf` structure's flags.
    - If the flag is set, call [`msgbuf_write`](imsg-buffer.c.driver.md#msgbuf_write) with the file descriptor and buffer from `imsgbuf`.
    - If the flag is not set, call [`ibuf_write`](imsg-buffer.c.driver.md#ibuf_write) with the file descriptor and buffer from `imsgbuf`.
- **Output**:
    - Returns the result of either [`msgbuf_write`](imsg-buffer.c.driver.md#msgbuf_write) or [`ibuf_write`](imsg-buffer.c.driver.md#ibuf_write), which is typically the number of bytes written or -1 on error.
- **Functions called**:
    - [`msgbuf_write`](imsg-buffer.c.driver.md#msgbuf_write)
    - [`ibuf_write`](imsg-buffer.c.driver.md#ibuf_write)


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
    - The function returns 0 on success, indicating all messages were flushed, or -1 on failure, indicating an error occurred during writing.
- **Functions called**:
    - [`imsgbuf_queuelen`](#imsgbuf_queuelen)
    - [`imsgbuf_write`](#imsgbuf_write)


---
### imsgbuf_clear<!-- {{#callable:imsgbuf_clear}} -->
The `imsgbuf_clear` function frees the message buffer associated with an `imsgbuf` structure and sets its pointer to NULL.
- **Inputs**:
    - `imsgbuf`: A pointer to a `struct imsgbuf` which contains a message buffer to be cleared.
- **Control Flow**:
    - Call [`msgbuf_free`](imsg-buffer.c.driver.md#msgbuf_free) to free the message buffer pointed to by `imsgbuf->w`.
    - Set `imsgbuf->w` to NULL to indicate that the message buffer has been cleared.
- **Output**:
    - This function does not return a value; it performs an operation on the `imsgbuf` structure passed to it.
- **Functions called**:
    - [`msgbuf_free`](imsg-buffer.c.driver.md#msgbuf_free)


---
### imsgbuf_queuelen<!-- {{#callable:imsgbuf_queuelen}} -->
The `imsgbuf_queuelen` function returns the length of the message queue associated with a given `imsgbuf` structure.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure, which contains a message buffer (`w`) whose queue length is to be determined.
- **Control Flow**:
    - The function calls [`msgbuf_queuelen`](imsg-buffer.c.driver.md#msgbuf_queuelen) with the `w` member of the `imsgbuf` structure as its argument.
    - The result of [`msgbuf_queuelen`](imsg-buffer.c.driver.md#msgbuf_queuelen) is returned as the output of the function.
- **Output**:
    - The function returns a `uint32_t` value representing the length of the message queue in the `imsgbuf` structure.
- **Functions called**:
    - [`msgbuf_queuelen`](imsg-buffer.c.driver.md#msgbuf_queuelen)


---
### imsg_get<!-- {{#callable:imsg_get}} -->
The `imsg_get` function retrieves a message from an imsg buffer and populates an imsg structure with the message data.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure, which represents the buffer from which the message is to be retrieved.
    - `imsg`: A pointer to an `imsg` structure, which will be populated with the retrieved message data.
- **Control Flow**:
    - Attempt to retrieve a buffer from the message buffer `imsgbuf->w` using [`msgbuf_get`](imsg-buffer.c.driver.md#msgbuf_get); if unsuccessful, return 0.
    - Attempt to read the message header from the buffer using [`ibuf_get`](imsg-buffer.c.driver.md#ibuf_get); if unsuccessful, return -1.
    - Check if the buffer has additional data using [`ibuf_size`](imsg-buffer.c.driver.md#ibuf_size); if it does, set `m.data` to the buffer's data, otherwise set `m.data` to NULL.
    - Assign the buffer to `m.buf` and clear the `IMSG_FD_MARK` flag from the message header length.
    - Copy the local `imsg` structure `m` to the provided `imsg` pointer.
    - Return the size of the buffer plus the size of the message header.
- **Output**:
    - Returns the total size of the message, including the header, if successful; returns 0 if no message is available, or -1 if an error occurs.
- **Functions called**:
    - [`msgbuf_get`](imsg-buffer.c.driver.md#msgbuf_get)
    - [`ibuf_get`](imsg-buffer.c.driver.md#ibuf_get)
    - [`ibuf_size`](imsg-buffer.c.driver.md#ibuf_size)
    - [`ibuf_data`](imsg-buffer.c.driver.md#ibuf_data)


---
### imsg_get_ibuf<!-- {{#callable:imsg_get_ibuf}} -->
The `imsg_get_ibuf` function retrieves an internal buffer from an imsg structure and stores it in a provided ibuf structure, returning an error if the buffer is empty.
- **Inputs**:
    - `imsg`: A pointer to a struct imsg, which contains a buffer from which data is to be retrieved.
    - `ibuf`: A pointer to a struct ibuf, where the retrieved buffer data will be stored.
- **Control Flow**:
    - Check if the size of the buffer in the imsg structure is zero.
    - If the buffer size is zero, set the errno to EBADMSG and return -1 to indicate an error.
    - If the buffer size is not zero, call the function [`ibuf_get_ibuf`](imsg-buffer.c.driver.md#ibuf_get_ibuf) with the imsg buffer, its size, and the ibuf pointer to retrieve the buffer data.
- **Output**:
    - Returns 0 on success, or -1 if the imsg buffer is empty, with errno set to EBADMSG.
- **Functions called**:
    - [`ibuf_size`](imsg-buffer.c.driver.md#ibuf_size)
    - [`ibuf_get_ibuf`](imsg-buffer.c.driver.md#ibuf_get_ibuf)


---
### imsg_get_data<!-- {{#callable:imsg_get_data}} -->
The `imsg_get_data` function retrieves data from an imsg buffer into a provided data buffer, ensuring the data length matches the expected size.
- **Inputs**:
    - `imsg`: A pointer to a struct imsg, which contains the buffer from which data is to be retrieved.
    - `data`: A pointer to the destination buffer where the data will be copied.
    - `len`: The expected length of the data to be retrieved from the imsg buffer.
- **Control Flow**:
    - Check if the provided length `len` is zero; if so, set `errno` to `EINVAL` and return -1 to indicate an error.
    - Check if the size of the buffer in `imsg` does not match the provided `len`; if so, set `errno` to `EBADMSG` and return -1 to indicate an error.
    - If both checks pass, call [`ibuf_get`](imsg-buffer.c.driver.md#ibuf_get) to copy the data from the imsg buffer to the provided data buffer and return the result of this operation.
- **Output**:
    - The function returns 0 on success, or -1 on failure, setting `errno` to indicate the error type.
- **Functions called**:
    - [`ibuf_size`](imsg-buffer.c.driver.md#ibuf_size)
    - [`ibuf_get`](imsg-buffer.c.driver.md#ibuf_get)


---
### imsg_get_fd<!-- {{#callable:imsg_get_fd}} -->
The `imsg_get_fd` function retrieves a file descriptor from a given inter-process message structure.
- **Inputs**:
    - `imsg`: A pointer to an `imsg` structure from which the file descriptor is to be retrieved.
- **Control Flow**:
    - The function calls [`ibuf_fd_get`](imsg-buffer.c.driver.md#ibuf_fd_get) with the buffer from the `imsg` structure as its argument.
    - The result of [`ibuf_fd_get`](imsg-buffer.c.driver.md#ibuf_fd_get) is returned directly.
- **Output**:
    - The function returns an integer representing the file descriptor extracted from the `imsg` structure's buffer.
- **Functions called**:
    - [`ibuf_fd_get`](imsg-buffer.c.driver.md#ibuf_fd_get)


---
### imsg_get_id<!-- {{#callable:imsg_get_id}} -->
The `imsg_get_id` function retrieves the peer ID from the header of an inter-process message.
- **Inputs**:
    - `imsg`: A pointer to a `struct imsg` which contains the message header and associated data.
- **Control Flow**:
    - Access the `hdr` member of the `imsg` structure, which is of type `struct imsg_hdr`.
    - Return the `peerid` field from the `hdr` structure.
- **Output**:
    - The function returns a `uint32_t` value representing the peer ID of the message.


---
### imsg_get_len<!-- {{#callable:imsg_get_len}} -->
The `imsg_get_len` function returns the size of the buffer associated with a given inter-process message (imsg).
- **Inputs**:
    - `imsg`: A pointer to a struct imsg, which contains a buffer (buf) whose size is to be determined.
- **Control Flow**:
    - The function calls [`ibuf_size`](imsg-buffer.c.driver.md#ibuf_size) with the buffer `imsg->buf` as an argument.
    - The result of [`ibuf_size`](imsg-buffer.c.driver.md#ibuf_size) is returned as the output of the function.
- **Output**:
    - The function returns a `size_t` value representing the size of the buffer within the provided `imsg` structure.
- **Functions called**:
    - [`ibuf_size`](imsg-buffer.c.driver.md#ibuf_size)


---
### imsg_get_pid<!-- {{#callable:imsg_get_pid}} -->
The `imsg_get_pid` function retrieves the process ID (PID) from the header of an inter-process message (imsg).
- **Inputs**:
    - `imsg`: A pointer to a struct imsg, which contains the message header and associated data.
- **Control Flow**:
    - Access the `hdr` member of the `imsg` structure, which is assumed to be a struct containing message metadata.
    - Return the `pid` field from the `hdr` member, which represents the process ID associated with the message.
- **Output**:
    - The function returns a `pid_t` type, which is the process ID extracted from the message header.


---
### imsg_get_type<!-- {{#callable:imsg_get_type}} -->
The `imsg_get_type` function retrieves the type of an inter-process message from its header.
- **Inputs**:
    - `imsg`: A pointer to a struct `imsg`, which contains the message header and associated data.
- **Control Flow**:
    - Access the `hdr` field of the `imsg` structure, which is of type `imsg_hdr`.
    - Return the `type` field from the `hdr` structure.
- **Output**:
    - The function returns a `uint32_t` value representing the type of the message.


---
### imsg_compose<!-- {{#callable:imsg_compose}} -->
The `imsg_compose` function creates and sends an inter-process message with optional data and file descriptor.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure that manages the message buffer.
    - `type`: A 32-bit unsigned integer representing the message type.
    - `id`: A 32-bit unsigned integer representing the message identifier.
    - `pid`: A process identifier (PID) of type `pid_t` associated with the message.
    - `fd`: An integer representing a file descriptor to be associated with the message.
    - `data`: A pointer to the data to be included in the message.
    - `datalen`: A size_t value indicating the length of the data to be included in the message.
- **Control Flow**:
    - Attempt to create a new message buffer `wbuf` using [`imsg_create`](#imsg_create) with the provided parameters.
    - If [`imsg_create`](#imsg_create) returns NULL, indicating failure, return -1 to signal an error.
    - Add the provided data to the message buffer `wbuf` using [`imsg_add`](#imsg_add).
    - If [`imsg_add`](#imsg_add) fails, return -1 to signal an error.
    - Set the file descriptor for the message buffer `wbuf` using [`ibuf_fd_set`](imsg-buffer.c.driver.md#ibuf_fd_set).
    - Close the message buffer `wbuf` and finalize the message using [`imsg_close`](#imsg_close).
    - Return 1 to indicate successful message composition and sending.
- **Output**:
    - Returns 1 on success, or -1 on failure.
- **Functions called**:
    - [`imsg_create`](#imsg_create)
    - [`imsg_add`](#imsg_add)
    - [`ibuf_fd_set`](imsg-buffer.c.driver.md#ibuf_fd_set)
    - [`imsg_close`](#imsg_close)


---
### imsg_composev<!-- {{#callable:imsg_composev}} -->
The `imsg_composev` function constructs and sends a message with a vector of data buffers over an inter-process communication channel.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure that represents the message buffer to which the message will be added.
    - `type`: A 32-bit unsigned integer representing the type of the message.
    - `id`: A 32-bit unsigned integer representing the identifier of the message.
    - `pid`: A process ID (pid_t) representing the process ID associated with the message.
    - `fd`: An integer file descriptor that may be associated with the message.
    - `iov`: A pointer to an array of `iovec` structures, each containing a base pointer and length for a data buffer to be included in the message.
    - `iovcnt`: An integer representing the number of `iovec` structures in the `iov` array.
- **Control Flow**:
    - Initialize a variable `datalen` to zero to accumulate the total length of data from the `iovec` array.
    - Iterate over each `iovec` in the `iov` array, adding the length of each to `datalen`.
    - Call [`imsg_create`](#imsg_create) with the accumulated `datalen` to create a new message buffer `wbuf`; return -1 if creation fails.
    - Iterate over each `iovec` again, adding its data to `wbuf` using [`imsg_add`](#imsg_add); return -1 if adding fails.
    - Set the file descriptor for `wbuf` using [`ibuf_fd_set`](imsg-buffer.c.driver.md#ibuf_fd_set).
    - Finalize the message by calling [`imsg_close`](#imsg_close) with `imsgbuf` and `wbuf`.
    - Return 1 to indicate success.
- **Output**:
    - Returns 1 on success, or -1 if an error occurs during message creation or data addition.
- **Functions called**:
    - [`imsg_create`](#imsg_create)
    - [`imsg_add`](#imsg_add)
    - [`ibuf_fd_set`](imsg-buffer.c.driver.md#ibuf_fd_set)
    - [`imsg_close`](#imsg_close)


---
### imsg_compose_ibuf<!-- {{#callable:imsg_compose_ibuf}} -->
The `imsg_compose_ibuf` function composes and enqueues an inter-process message with a payload from a given buffer into a message buffer, without allowing file descriptor passing.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure that represents the message buffer where the composed message will be enqueued.
    - `type`: A 32-bit unsigned integer representing the type of the message to be composed.
    - `id`: A 32-bit unsigned integer representing the peer ID associated with the message.
    - `pid`: A process ID (pid_t) that identifies the process sending the message; if zero, the function uses the `pid` from `imsgbuf`.
    - `buf`: A pointer to an `ibuf` structure containing the payload data to be included in the message.
- **Control Flow**:
    - Check if the total size of the buffer plus the message header exceeds the maximum allowed size in `imsgbuf`; if so, set `errno` to `ERANGE` and go to the fail label.
    - Initialize the message header with the provided type, calculated length, peer ID, and process ID (using `imsgbuf`'s pid if the provided pid is zero).
    - Attempt to open a new buffer for the message header; if unsuccessful, go to the fail label.
    - Add the message header to the header buffer; if unsuccessful, go to the fail label.
    - Close the header buffer and the payload buffer, enqueuing them into the message buffer `imsgbuf->w`.
    - Return 1 to indicate success.
    - In the fail label, save the current `errno`, free the payload and header buffers, restore `errno`, and return -1 to indicate failure.
- **Output**:
    - Returns 1 on success, or -1 on failure with `errno` set to indicate the error.
- **Functions called**:
    - [`ibuf_size`](imsg-buffer.c.driver.md#ibuf_size)
    - [`ibuf_open`](imsg-buffer.c.driver.md#ibuf_open)
    - [`imsg_add`](#imsg_add)
    - [`ibuf_close`](imsg-buffer.c.driver.md#ibuf_close)
    - [`ibuf_free`](imsg-buffer.c.driver.md#ibuf_free)


---
### imsg_forward<!-- {{#callable:imsg_forward}} -->
The `imsg_forward` function forwards an inter-process message (imsg) to another communication channel, ensuring any attached file descriptor is closed.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure representing the target message buffer where the message will be forwarded.
    - `msg`: A pointer to an `imsg` structure containing the message to be forwarded.
- **Control Flow**:
    - Rewinds the buffer of the message `msg` to the beginning.
    - Skips the header part of the message buffer to focus on the payload.
    - Calculates the size of the remaining message payload.
    - Attempts to create a new message buffer `wbuf` in `imsgbuf` with the same type, peer ID, and PID as the original message, and the calculated payload length.
    - If the creation of `wbuf` fails, returns -1 indicating an error.
    - If the payload length is not zero, attempts to add the payload from the original message buffer to `wbuf`.
    - If adding the payload fails, frees `wbuf` and returns -1 indicating an error.
    - Closes the message in `imsgbuf` using `wbuf`, finalizing the forwarding process.
    - Returns 1 to indicate successful forwarding.
- **Output**:
    - Returns 1 on successful forwarding of the message, or -1 if an error occurs during the process.
- **Functions called**:
    - [`ibuf_rewind`](imsg-buffer.c.driver.md#ibuf_rewind)
    - [`ibuf_skip`](imsg-buffer.c.driver.md#ibuf_skip)
    - [`ibuf_size`](imsg-buffer.c.driver.md#ibuf_size)
    - [`imsg_create`](#imsg_create)
    - [`ibuf_add_ibuf`](imsg-buffer.c.driver.md#ibuf_add_ibuf)
    - [`ibuf_free`](imsg-buffer.c.driver.md#ibuf_free)
    - [`imsg_close`](#imsg_close)


---
### imsg_create<!-- {{#callable:imsg_create}} -->
The `imsg_create` function initializes and returns a new message buffer with a header for inter-process communication.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure that holds the message buffer state and configuration.
    - `type`: A 32-bit unsigned integer representing the message type.
    - `id`: A 32-bit unsigned integer representing the peer identifier.
    - `pid`: A process identifier (PID) for the message sender, or 0 to use the default PID from `imsgbuf`.
    - `datalen`: The size of the data payload to be included in the message, excluding the header size.
- **Control Flow**:
    - Calculate the total length of the message by adding the header size to `datalen`.
    - Check if the total message length exceeds the maximum allowed size in `imsgbuf`; if so, set `errno` to `ERANGE` and return `NULL`.
    - Initialize the message header with the provided `type`, `id`, and `pid` (or default to `imsgbuf->pid` if `pid` is 0).
    - Attempt to allocate a dynamic buffer `wbuf` for the message using [`ibuf_dynamic`](imsg-buffer.c.driver.md#ibuf_dynamic); return `NULL` if allocation fails.
    - Add the initialized header to the buffer `wbuf` using [`imsg_add`](#imsg_add); return `NULL` if this operation fails.
    - Return the initialized and populated buffer `wbuf`.
- **Output**:
    - Returns a pointer to the newly created `ibuf` structure containing the message header, or `NULL` if an error occurs.
- **Functions called**:
    - [`ibuf_dynamic`](imsg-buffer.c.driver.md#ibuf_dynamic)
    - [`imsg_add`](#imsg_add)


---
### imsg_add<!-- {{#callable:imsg_add}} -->
The `imsg_add` function appends data to a message buffer and handles errors by freeing the buffer if the addition fails.
- **Inputs**:
    - `msg`: A pointer to an `ibuf` structure representing the message buffer to which data will be added.
    - `data`: A pointer to the data that needs to be added to the message buffer.
    - `datalen`: The size of the data to be added, in bytes.
- **Control Flow**:
    - Check if `datalen` is non-zero to determine if there is data to add.
    - If `datalen` is non-zero, attempt to add the data to the message buffer using [`ibuf_add`](imsg-buffer.c.driver.md#ibuf_add).
    - If [`ibuf_add`](imsg-buffer.c.driver.md#ibuf_add) returns -1, indicating failure, free the message buffer using [`ibuf_free`](imsg-buffer.c.driver.md#ibuf_free) and return -1.
    - If the data is successfully added, return the length of the data added (`datalen`).
- **Output**:
    - Returns the length of the data added to the buffer if successful, or -1 if an error occurs.
- **Functions called**:
    - [`ibuf_add`](imsg-buffer.c.driver.md#ibuf_add)
    - [`ibuf_free`](imsg-buffer.c.driver.md#ibuf_free)


---
### imsg_close<!-- {{#callable:imsg_close}} -->
The `imsg_close` function finalizes an inter-process message by setting its length and closing the buffer.
- **Inputs**:
    - `imsgbuf`: A pointer to an `imsgbuf` structure, which represents the message buffer context.
    - `msg`: A pointer to an `ibuf` structure, which represents the message buffer to be closed.
- **Control Flow**:
    - Calculate the size of the message buffer using `ibuf_size(msg)` and store it in `len`.
    - Check if a file descriptor is available in the message buffer using `ibuf_fd_avail(msg)`.
    - If a file descriptor is available, set the `IMSG_FD_MARK` flag in `len`.
    - Set the calculated length in the message header at the offset of `len` using [`ibuf_set_h32`](imsg-buffer.c.driver.md#ibuf_set_h32).
    - Close the message buffer by calling [`ibuf_close`](imsg-buffer.c.driver.md#ibuf_close) with the write buffer of `imsgbuf` and the message buffer `msg`.
- **Output**:
    - The function does not return a value; it modifies the message buffer to finalize it for sending.
- **Functions called**:
    - [`ibuf_size`](imsg-buffer.c.driver.md#ibuf_size)
    - [`ibuf_fd_avail`](imsg-buffer.c.driver.md#ibuf_fd_avail)
    - [`ibuf_set_h32`](imsg-buffer.c.driver.md#ibuf_set_h32)
    - [`ibuf_close`](imsg-buffer.c.driver.md#ibuf_close)


---
### imsg_free<!-- {{#callable:imsg_free}} -->
The `imsg_free` function releases the memory associated with the buffer of an inter-process message.
- **Inputs**:
    - `imsg`: A pointer to a `struct imsg` which contains a buffer that needs to be freed.
- **Control Flow**:
    - The function calls [`ibuf_free`](imsg-buffer.c.driver.md#ibuf_free) with the buffer (`imsg->buf`) of the provided `imsg` structure to release the allocated memory.
- **Output**:
    - The function does not return any value.
- **Functions called**:
    - [`ibuf_free`](imsg-buffer.c.driver.md#ibuf_free)


---
### imsg_parse_hdr<!-- {{#callable:imsg_parse_hdr}} -->
The `imsg_parse_hdr` function parses the header of an inter-process message from a buffer and prepares a new buffer for the message content, handling file descriptor passing if necessary.
- **Inputs**:
    - `buf`: A pointer to an `ibuf` structure containing the message data to be parsed.
    - `arg`: A pointer to an `imsgbuf` structure, which provides context such as the maximum allowed message size.
    - `fd`: A pointer to an integer representing a file descriptor, which may be set if the message includes a file descriptor.
- **Control Flow**:
    - Attempt to read the message header from the provided buffer using [`ibuf_get`](imsg-buffer.c.driver.md#ibuf_get); return `NULL` if unsuccessful.
    - Extract the message length from the header, ignoring the file descriptor mark bit.
    - Check if the message length is valid (not smaller than the header size and not exceeding the maximum size); set `errno` to `ERANGE` and return `NULL` if invalid.
    - Open a new buffer of the specified length using [`ibuf_open`](imsg-buffer.c.driver.md#ibuf_open); return `NULL` if unsuccessful.
    - If the message length indicates a file descriptor is included, set the file descriptor in the new buffer and reset the input file descriptor to -1.
    - Return the newly opened buffer.
- **Output**:
    - Returns a pointer to a newly allocated `ibuf` structure containing the parsed message, or `NULL` if an error occurs.
- **Functions called**:
    - [`ibuf_get`](imsg-buffer.c.driver.md#ibuf_get)
    - [`ibuf_open`](imsg-buffer.c.driver.md#ibuf_open)
    - [`ibuf_fd_set`](imsg-buffer.c.driver.md#ibuf_fd_set)


