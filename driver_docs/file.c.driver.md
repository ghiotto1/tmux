# Purpose
This C source code file is part of the tmux project and is responsible for handling inter-process communication (IPC) related to file operations between the client and server components of tmux. The code provides a set of functions to manage file streams, allowing both the client and server to create, read, write, and manage files in a synchronized manner. The primary technical components include the use of red-black trees to manage active file streams, event-driven callbacks for asynchronous operations, and message passing for communication between processes. The code defines a series of functions that facilitate the creation of file objects, manage their lifecycle, and handle data transfer operations, such as reading from and writing to files.

The file is not an executable but rather a C source file intended to be compiled as part of the tmux application. It does not define public APIs or external interfaces directly but provides internal functionality crucial for the operation of tmux's client-server architecture. The code is organized around the concept of client files, which are structures that represent active file streams, and it includes mechanisms for error handling, event-driven callbacks, and message-based communication to ensure robust and efficient file operations. The use of event buffers and the integration with the tmux event loop highlight the asynchronous nature of the operations, allowing tmux to handle file I/O without blocking other processes.
# Imports and Dependencies

---
- `sys/types.h`
- `errno.h`
- `fcntl.h`
- `stdio.h`
- `stdlib.h`
- `string.h`
- `unistd.h`
- `tmux.h`


# Global Variables

---
### file_next_stream
- **Type**: `int`
- **Description**: The `file_next_stream` is a static integer variable initialized to 3. It is used to keep track of the next available stream number for file operations within the program.
- **Use**: This variable is incremented each time a new file stream is created, ensuring unique stream identifiers for file operations.


# Functions

---
### file_get_path<!-- {{#callable:file_get_path}} -->
The `file_get_path` function constructs a full file path based on whether the provided file path is absolute or relative to the client's current working directory.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client for which the file path is being constructed.
    - `file`: A constant character pointer representing the file name or path to be resolved.
- **Control Flow**:
    - Check if the first character of the `file` string is a '/' to determine if it is an absolute path.
    - If `file` is an absolute path, duplicate the `file` string using [`xstrdup`](xmalloc.c.driver.md#xstrdup) and assign it to `path`.
    - If `file` is not an absolute path, construct the full path by concatenating the client's current working directory (obtained via [`server_client_get_cwd`](server-client.c.driver.md#server_client_get_cwd)) with the `file` string using [`xasprintf`](xmalloc.c.driver.md#xasprintf), and assign it to `path`.
    - Return the constructed `path`.
- **Output**:
    - A dynamically allocated string containing the full file path, which must be freed by the caller.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`server_client_get_cwd`](server-client.c.driver.md#server_client_get_cwd)


---
### file_cmp<!-- {{#callable:file_cmp}} -->
The `file_cmp` function compares two `client_file` structures based on their `stream` values and returns an integer indicating their relative order.
- **Inputs**:
    - `cf1`: A pointer to the first `client_file` structure to be compared.
    - `cf2`: A pointer to the second `client_file` structure to be compared.
- **Control Flow**:
    - The function first checks if the `stream` value of `cf1` is less than that of `cf2`; if so, it returns -1.
    - If the `stream` value of `cf1` is greater than that of `cf2`, it returns 1.
    - If neither of the above conditions is true, it returns 0, indicating that the `stream` values are equal.
- **Output**:
    - An integer: -1 if `cf1`'s stream is less than `cf2`'s, 1 if greater, and 0 if they are equal.


---
### file_create_with_peer<!-- {{#callable:file_create_with_peer}} -->
The function `file_create_with_peer` initializes and inserts a new `client_file` structure into a red-black tree, associating it with a given peer and callback.
- **Inputs**:
    - `peer`: A pointer to a `tmuxpeer` structure representing the peer (server) to which the file object will be associated.
    - `files`: A pointer to a `client_files` structure, which is a red-black tree where the new `client_file` will be inserted.
    - `stream`: An integer representing the stream identifier for the new file object.
    - `cb`: A callback function of type `client_file_cb` that will be called when the file is finished with.
    - `cbdata`: A pointer to user-defined data that will be passed to the callback function.
- **Control Flow**:
    - Allocate memory for a new `client_file` structure using [`xcalloc`](xmalloc.c.driver.md#xcalloc) and initialize its fields.
    - Set the `c` field to `NULL` and `references` to 1.
    - Assign the `stream` parameter to the `stream` field of the `client_file`.
    - Create a new event buffer using `evbuffer_new` and assign it to the `buffer` field; if allocation fails, terminate the program with `fatalx`.
    - Assign the callback function `cb` and callback data `cbdata` to the respective fields in the `client_file`.
    - Set the `peer` field to the provided `peer` argument and the `tree` field to the provided `files` argument.
    - Insert the new `client_file` into the `files` red-black tree using `RB_INSERT`.
    - Return the pointer to the newly created `client_file`.
- **Output**:
    - Returns a pointer to the newly created `client_file` structure.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)


---
### file_create_with_client<!-- {{#callable:file_create_with_client}} -->
The `file_create_with_client` function creates a new `client_file` object associated with a given client, initializes its properties, and inserts it into the client's file tree if applicable.
- **Inputs**:
    - `c`: A pointer to a `client` structure, representing the client with which the file is associated.
    - `stream`: An integer representing the stream identifier for the file.
    - `cb`: A callback function of type `client_file_cb` to be called when the file operation is complete.
    - `cbdata`: A pointer to user-defined data to be passed to the callback function.
- **Control Flow**:
    - Check if the client `c` is not NULL and if it is attached, set `c` to NULL.
    - Allocate memory for a new `client_file` structure and initialize its fields, including setting the client, stream, and callback information.
    - Create a new buffer for the file using `evbuffer_new` and handle memory allocation failure with a fatal error.
    - If the client `c` is not NULL, set the file's peer to the client's peer, assign the client's file tree to the file, insert the file into the client's file tree, and increment the client's reference count.
    - Return the newly created `client_file` object.
- **Output**:
    - Returns a pointer to the newly created `client_file` structure.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)


---
### file_free<!-- {{#callable:file_free}} -->
The `file_free` function deallocates a `client_file` structure and its associated resources when its reference count reaches zero.
- **Inputs**:
    - `cf`: A pointer to a `client_file` structure that is to be freed.
- **Control Flow**:
    - Decrement the reference count of the `client_file` structure pointed to by `cf`.
    - If the reference count is not zero, return immediately without freeing resources.
    - Free the `evbuffer` associated with the `client_file`.
    - Free the `path` string associated with the `client_file`.
    - If the `client_file` is part of a tree, remove it from the tree.
    - If the `client_file` is associated with a client, decrement the client's reference count.
    - Finally, free the `client_file` structure itself.
- **Output**:
    - The function does not return a value; it performs cleanup operations on the `client_file` structure and its resources.
- **Functions called**:
    - [`server_client_unref`](server-client.c.driver.md#server_client_unref)


---
### file_fire_done_cb<!-- {{#callable:file_fire_done_cb}} -->
The `file_fire_done_cb` function triggers a callback for a file operation if certain conditions are met and then frees the associated resources.
- **Inputs**:
    - `fd`: An unused integer representing a file descriptor.
    - `events`: An unused short representing event flags.
    - `arg`: A pointer to a `client_file` structure, which contains information about the file operation.
- **Control Flow**:
    - Retrieve the `client_file` structure from the `arg` parameter.
    - Retrieve the associated `client` structure from the `client_file`.
    - Check if the callback (`cb`) is not NULL and if the file is closed, or the client is NULL, or the client is not marked as dead.
    - If the conditions are met, invoke the callback function with the client, file path, error status, a flag set to 1, the buffer, and additional data.
    - Call [`file_free`](#file_free) to release the resources associated with the `client_file`.
- **Output**:
    - The function does not return a value; it performs operations based on the state of the `client_file` and `client` structures and frees resources.
- **Functions called**:
    - [`file_free`](#file_free)


---
### file_fire_done<!-- {{#callable:file_fire_done}} -->
The `file_fire_done` function schedules a one-time event to trigger the `file_fire_done_cb` callback for a given `client_file` structure.
- **Inputs**:
    - `cf`: A pointer to a `client_file` structure, representing the file for which the done callback should be fired.
- **Control Flow**:
    - The function calls `event_once` with parameters to set up a one-time event.
    - The event is configured to trigger the `file_fire_done_cb` callback function when the event occurs.
    - The event is set to trigger immediately with `EV_TIMEOUT` and does not require a file descriptor, hence `-1` is passed as the file descriptor.
- **Output**:
    - The function does not return any value; it schedules an event to trigger a callback.


---
### file_fire_read<!-- {{#callable:file_fire_read}} -->
The `file_fire_read` function triggers a read callback for a client file if a callback is defined.
- **Inputs**:
    - `cf`: A pointer to a `struct client_file` which contains information about the client file, including the callback function to be executed.
- **Control Flow**:
    - Check if the callback function `cf->cb` is not NULL.
    - If the callback exists, invoke it with the parameters: client `cf->c`, file path `cf->path`, error code `cf->error`, a flag set to 0, buffer `cf->buffer`, and additional data `cf->data`.
- **Output**:
    - The function does not return any value; it executes a callback if defined.


---
### file_can_print<!-- {{#callable:file_can_print}} -->
The `file_can_print` function checks if a client is eligible to print to a file based on its state and flags.
- **Inputs**:
    - `c`: A pointer to a `struct client` which represents the client whose print eligibility is being checked.
- **Control Flow**:
    - The function first checks if the client pointer `c` is NULL, indicating no client is provided.
    - It then checks if the client has the `CLIENT_ATTACHED` flag set, meaning the client is currently attached.
    - Next, it checks if the client has the `CLIENT_CONTROL` flag set, indicating the client is in control mode.
    - If any of these conditions are true, the function returns 0, indicating the client cannot print.
    - If none of these conditions are true, the function returns 1, indicating the client can print.
- **Output**:
    - The function returns an integer: 0 if the client cannot print, and 1 if the client can print.


---
### file_print<!-- {{#callable:file_print}} -->
The `file_print` function formats and prints a message to a file associated with a client using a variable argument list.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client to which the message will be printed.
    - `fmt`: A format string similar to those used in `printf` functions, specifying how the subsequent arguments are formatted.
    - `...`: A variable number of arguments that are formatted according to the format string `fmt`.
- **Control Flow**:
    - Initialize a `va_list` variable `ap` to handle the variable arguments.
    - Start processing the variable arguments using `va_start`, with `fmt` as the last fixed argument.
    - Call the [`file_vprint`](#file_vprint) function, passing the client `c`, format string `fmt`, and the initialized `va_list` `ap`.
    - End processing of the variable arguments using `va_end`.
- **Output**:
    - The function does not return a value; it performs its operation by printing the formatted message to the client's associated file.
- **Functions called**:
    - [`file_vprint`](#file_vprint)


---
### file_vprint<!-- {{#callable:file_vprint}} -->
The `file_vprint` function formats and prints a message to a client's file stream, creating the file if it doesn't exist, and sends a message to open the file for writing if necessary.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client to which the message will be printed.
    - `fmt`: A format string similar to those used in `printf` functions, specifying how the message should be formatted.
    - `ap`: A `va_list` containing the arguments to be formatted according to the format string `fmt`.
- **Control Flow**:
    - Check if the client `c` can print using [`file_can_print`](#file_can_print); if not, return immediately.
    - Initialize a `client_file` structure `find` with `stream` set to 1 to search for an existing file stream.
    - Use `RB_FIND` to search for an existing file stream in the client's file tree; if not found, create a new file stream with [`file_create_with_client`](#file_create_with_client).
    - If a new file stream is created, set its path to "-", format the message using `evbuffer_add_vprintf`, and send a `MSG_WRITE_OPEN` message to the client's peer to open the file for writing.
    - If an existing file stream is found, format the message using `evbuffer_add_vprintf` and push the data to the client using [`file_push`](#file_push).
- **Output**:
    - The function does not return a value; it performs operations on the client's file stream and sends messages to the client's peer.
- **Functions called**:
    - [`file_can_print`](#file_can_print)
    - [`file_create_with_client`](#file_create_with_client)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`proc_send`](proc.c.driver.md#proc_send)
    - [`file_push`](#file_push)


---
### file_print_buffer<!-- {{#callable:file_print_buffer}} -->
The `file_print_buffer` function writes a buffer of data to a file associated with a client, creating the file if it doesn't exist.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the data is being printed.
    - `data`: A pointer to the data buffer that needs to be written to the file.
    - `size`: The size of the data buffer in bytes.
- **Control Flow**:
    - Check if the client can print using [`file_can_print`](#file_can_print); if not, return immediately.
    - Initialize a `client_file` structure `find` with `stream` set to 1.
    - Attempt to find an existing file associated with the client using `RB_FIND` with the `find` structure.
    - If no file is found, create a new file with [`file_create_with_client`](#file_create_with_client), set its path to "-", and add the data to its buffer.
    - Send a `MSG_WRITE_OPEN` message to the client's peer to open the file for writing to standard output.
    - If a file is found, add the data to its buffer and call [`file_push`](#file_push) to handle further processing.
- **Output**:
    - The function does not return a value; it performs operations on the client's file system and sends messages to the client's peer.
- **Functions called**:
    - [`file_can_print`](#file_can_print)
    - [`file_create_with_client`](#file_create_with_client)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`proc_send`](proc.c.driver.md#proc_send)
    - [`file_push`](#file_push)


---
### file_error<!-- {{#callable:file_error}} -->
The `file_error` function logs an error message to a client's error stream, creating a new file object if necessary.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the error message is being logged.
    - `fmt`: A format string for the error message, similar to the format used in `printf`.
    - `...`: A variable number of arguments that correspond to the format specifiers in `fmt`.
- **Control Flow**:
    - Check if the client can print using [`file_can_print`](#file_can_print); if not, return immediately.
    - Initialize a variable argument list with `va_start`.
    - Set up a `client_file` structure to search for an existing file object with stream 2 in the client's file tree.
    - If no existing file object is found, create a new one with [`file_create_with_client`](#file_create_with_client), set its path to "-", and add the formatted error message to its buffer.
    - Send a `MSG_WRITE_OPEN` message to the client's peer to open the error stream if a new file object was created.
    - If an existing file object is found, add the formatted error message to its buffer and push the data using [`file_push`](#file_push).
    - End the variable argument list with `va_end`.
- **Output**:
    - The function does not return a value; it performs operations to log an error message to a client's error stream.
- **Functions called**:
    - [`file_can_print`](#file_can_print)
    - [`file_create_with_client`](#file_create_with_client)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`proc_send`](proc.c.driver.md#proc_send)
    - [`file_push`](#file_push)


---
### file_write<!-- {{#callable:file_write}} -->
The `file_write` function writes data to a specified file path, handling both direct file writing and inter-process communication for client-server scenarios.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client context for the file operation.
    - `path`: A string representing the file path where data should be written; '-' indicates standard output.
    - `flags`: An integer representing file open flags, such as `O_APPEND` for appending data.
    - `bdata`: A pointer to the data buffer that contains the data to be written to the file.
    - `bsize`: A size_t value indicating the size of the data buffer `bdata`.
    - `cb`: A callback function of type `client_file_cb` to be called when the file operation is complete.
    - `cbdata`: A pointer to user-defined data to be passed to the callback function `cb`.
- **Control Flow**:
    - Check if the path is '-', indicating standard output, and handle accordingly by setting the file descriptor to `STDOUT_FILENO` and checking client flags.
    - Create a `client_file` structure with the client context, stream, callback, and callback data.
    - If the client is attached or the path is '-', write data directly to the file using `fopen` and `fwrite`, handling errors appropriately.
    - If the client is not attached, add the data to the `evbuffer` and prepare a message for inter-process communication.
    - Allocate and populate a `msg_write_open` structure with stream, file descriptor, and flags, then send it using [`proc_send`](proc.c.driver.md#proc_send).
    - Handle errors such as file size exceeding limits or invalid operations, and call [`file_fire_done`](#file_fire_done) to finalize the operation.
- **Output**:
    - The function does not return a value but triggers a callback upon completion of the file operation, indicating success or error through the `client_file` structure.
- **Functions called**:
    - [`file_create_with_client`](#file_create_with_client)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`file_get_path`](#file_get_path)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`proc_send`](proc.c.driver.md#proc_send)
    - [`file_fire_done`](#file_fire_done)


---
### file_read<!-- {{#callable:file_read}} -->
The `file_read` function reads a file from a specified path into a client file structure, handling both local and remote file reading scenarios.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client requesting the file read.
    - `path`: A constant character pointer representing the path of the file to be read.
    - `cb`: A callback function of type `client_file_cb` to be called when the file read operation is complete.
    - `cbdata`: A pointer to user-defined data to be passed to the callback function.
- **Control Flow**:
    - Initialize a new client file structure `cf` with a unique stream identifier.
    - Check if the path is "-", indicating standard input, and handle accordingly by setting `fd` to `STDIN_FILENO`.
    - If the client is not suitable for reading from standard input, set an error and exit.
    - If the path is not "-", resolve the full file path and open the file for reading if the client is attached.
    - Read the file in chunks into a buffer and add it to the client file's buffer, handling errors such as memory allocation failure or I/O errors.
    - If the client is not attached, prepare a message to request the server to open the file for reading, ensuring the message size is within limits.
    - Send the message to the server and handle any errors in message sending.
    - If any errors occur during the process, call [`file_fire_done`](#file_fire_done) to handle cleanup and return `NULL`.
- **Output**:
    - Returns a pointer to a `client_file` structure if successful, or `NULL` if an error occurs during the file read operation.
- **Functions called**:
    - [`file_create_with_client`](#file_create_with_client)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`file_get_path`](#file_get_path)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`proc_send`](proc.c.driver.md#proc_send)
    - [`file_fire_done`](#file_fire_done)


---
### file_cancel<!-- {{#callable:file_cancel}} -->
The `file_cancel` function cancels an ongoing file read operation for a given client file by marking it as closed and sending a cancellation message to the peer.
- **Inputs**:
    - `cf`: A pointer to a `client_file` structure representing the file read operation to be canceled.
- **Control Flow**:
    - Log a debug message indicating the cancellation of the file read operation for the specified stream.
    - Check if the file is already marked as closed; if so, return immediately without further action.
    - Set the `closed` flag of the `client_file` structure to 1, indicating that the file is now closed.
    - Prepare a `msg_read_cancel` message structure with the stream identifier from the `client_file`.
    - Send the `MSG_READ_CANCEL` message to the peer associated with the `client_file` using the [`proc_send`](proc.c.driver.md#proc_send) function.
- **Output**:
    - The function does not return any value; it performs an action to cancel a file read operation.
- **Functions called**:
    - [`proc_send`](proc.c.driver.md#proc_send)


---
### file_push_cb<!-- {{#callable:file_push_cb}} -->
The `file_push_cb` function is a callback that attempts to push unwritten data to a client file if the client is not dead, and then frees the client file structure.
- **Inputs**:
    - `fd`: An unused integer representing a file descriptor.
    - `events`: An unused short representing event flags.
    - `arg`: A pointer to a `client_file` structure, representing the client file to be processed.
- **Control Flow**:
    - The function casts the `arg` parameter to a `client_file` pointer `cf`.
    - It checks if the client `cf->c` is not NULL and if the client is not marked as dead (`~cf->c->flags & CLIENT_DEAD`).
    - If the client is valid, it calls `file_push(cf)` to push any unwritten data to the client.
    - Finally, it calls `file_free(cf)` to free the client file structure.
- **Output**:
    - The function does not return any value; it performs operations on the `client_file` structure passed as an argument.
- **Functions called**:
    - [`file_push`](#file_push)
    - [`file_free`](#file_free)


---
### file_push<!-- {{#callable:file_push}} -->
The `file_push` function sends buffered data from a client file to a peer, handling partial sends and completion notifications.
- **Inputs**:
    - `cf`: A pointer to a `client_file` structure representing the file whose data is to be sent.
- **Control Flow**:
    - Allocate memory for a `msg_write_data` structure to hold the message data.
    - Calculate the amount of data left in the buffer using `EVBUFFER_LENGTH`.
    - Enter a loop that continues until all data is sent or an error occurs.
    - Determine the amount of data to send in this iteration, ensuring it does not exceed the maximum message size.
    - Reallocate memory for the message to accommodate the data to be sent.
    - Set the stream identifier in the message and copy the data from the buffer into the message.
    - Send the message using [`proc_send`](proc.c.driver.md#proc_send) and break the loop if sending fails.
    - Drain the sent data from the buffer using `evbuffer_drain`.
    - Log the amount of data sent and remaining.
    - If data remains in the buffer, increment the reference count and schedule a callback with `event_once`.
    - If no data remains and the stream is greater than 2, send a `MSG_WRITE_CLOSE` message and call [`file_fire_done`](#file_fire_done).
    - Free the allocated message memory.
- **Output**:
    - The function does not return a value, but it sends data to a peer and may schedule further events or send a close message.
- **Functions called**:
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`xrealloc`](xmalloc.c.driver.md#xrealloc)
    - [`proc_send`](proc.c.driver.md#proc_send)
    - [`file_fire_done`](#file_fire_done)


---
### file_write_left<!-- {{#callable:file_write_left}} -->
The `file_write_left` function checks if there are any files in the given `client_files` structure that still have data left to be written.
- **Inputs**:
    - `files`: A pointer to a `client_files` structure, which is a collection of `client_file` objects representing files being managed.
- **Control Flow**:
    - Initialize a `waiting` counter to zero.
    - Iterate over each `client_file` object in the `files` structure using the `RB_FOREACH` macro.
    - For each `client_file`, check if its `event` is not `NULL`.
    - If the `event` is not `NULL`, calculate the length of the data left to be written using `EVBUFFER_LENGTH` on the `output` buffer of the event.
    - If there is data left to be written (i.e., the length is not zero), increment the `waiting` counter and log the stream number and bytes left.
    - After iterating through all files, return whether any files have data left to write by checking if `waiting` is not zero.
- **Output**:
    - Returns a non-zero integer if there are files with data left to write, otherwise returns zero.


---
### file_write_error_callback<!-- {{#callable:file_write_error_callback}} -->
The `file_write_error_callback` function handles errors that occur during file write operations by cleaning up resources and invoking a callback if provided.
- **Inputs**:
    - `bev`: A pointer to a `bufferevent` structure, which is unused in this function.
    - `what`: A short integer indicating the type of event that occurred, which is unused in this function.
    - `arg`: A pointer to a `client_file` structure, representing the file context associated with the error.
- **Control Flow**:
    - Retrieve the `client_file` structure from the `arg` parameter.
    - Log a debug message indicating a write error for the file associated with the `client_file` structure's stream.
    - Free the `bufferevent` associated with the `client_file` and set its event pointer to NULL.
    - Close the file descriptor associated with the `client_file` and set it to -1.
    - If a callback function (`cb`) is set in the `client_file`, invoke it with NULL parameters and an error code of -1.
- **Output**:
    - This function does not return a value; it performs cleanup and invokes a callback if necessary.


---
### file_write_callback<!-- {{#callable:file_write_callback}} -->
The `file_write_callback` function handles the completion of a write operation for a client file, checking if the file is closed and if the output buffer is empty, and then performs cleanup operations.
- **Inputs**:
    - `bev`: A pointer to a `bufferevent` structure, which is unused in this function.
    - `arg`: A pointer to a `client_file` structure, representing the file being written to.
- **Control Flow**:
    - Log a debug message indicating a write check for the file associated with the given stream.
    - If the callback function `cf->cb` is not NULL, invoke it with NULL parameters and a data argument from the `client_file` structure.
    - Check if the file is marked as closed and if the output buffer length of the event is zero.
    - If both conditions are true, free the bufferevent, close the file descriptor, remove the file from the red-black tree, and free the `client_file` structure.
- **Output**:
    - The function does not return a value; it performs cleanup operations on the `client_file` structure if certain conditions are met.
- **Functions called**:
    - [`file_free`](#file_free)


---
### file_write_open<!-- {{#callable:file_write_open}} -->
The `file_write_open` function handles the opening of a file for writing, setting up necessary structures and events, and sending a response back to the peer indicating success or failure.
- **Inputs**:
    - `files`: A pointer to a `client_files` structure representing the collection of active file streams.
    - `peer`: A pointer to a `tmuxpeer` structure representing the peer to communicate with.
    - `imsg`: A pointer to an `imsg` structure containing the message data for the file write open request.
    - `allow_streams`: An integer flag indicating whether streams are allowed.
    - `close_received`: An integer flag indicating whether the file descriptor should be closed after use.
    - `cb`: A callback function of type `client_file_cb` to be called when the file operation is complete.
    - `cbdata`: A pointer to user-defined data to be passed to the callback function.
- **Control Flow**:
    - Extract the `msg_write_open` message from the `imsg` data and calculate its length.
    - Check if the message length is valid; if not, terminate with an error.
    - Determine the file path from the message data.
    - Log the attempt to open the file for writing.
    - Check if a file with the same stream already exists in the `files` tree; if so, set an error and prepare to reply.
    - Create a new `client_file` structure with the peer and add it to the `files` tree.
    - If the file is marked as closed, set an error and prepare to reply.
    - Attempt to open the file using the provided file descriptor or path, depending on the `allow_streams` flag.
    - If the file descriptor is invalid, set an error and prepare to reply.
    - Create a new `bufferevent` for the file descriptor to handle write events.
    - Enable the write event on the `bufferevent`.
    - Send a reply message to the peer with the stream number and any error encountered.
- **Output**:
    - The function sends a `MSG_WRITE_READY` message back to the peer, containing the stream number and an error code indicating success or failure of the file open operation.
- **Functions called**:
    - [`file_create_with_peer`](#file_create_with_peer)
    - [`proc_send`](proc.c.driver.md#proc_send)


---
### file_write_data<!-- {{#callable:file_write_data}} -->
The `file_write_data` function writes data from an inter-process message to a specified client file stream.
- **Inputs**:
    - `files`: A pointer to a `client_files` structure, which is a red-black tree containing active client file streams.
    - `imsg`: A pointer to an `imsg` structure, which contains the message data and header information for inter-process communication.
- **Control Flow**:
    - Extract the `msg_write_data` structure from the `imsg` data and calculate the message length excluding the header size.
    - Check if the message length is valid; if not, terminate the program with an error message.
    - Initialize a `client_file` structure to find the corresponding file stream using the stream number from the message.
    - Search for the client file in the red-black tree using `RB_FIND`; if not found, terminate the program with an error message.
    - Log the size of the data to be written and the stream number.
    - If the client file has an associated event, write the data to the buffer event using `bufferevent_write`.
- **Output**:
    - The function does not return a value; it writes data to a buffer event associated with a client file stream.


---
### file_write_close<!-- {{#callable:file_write_close}} -->
The `file_write_close` function handles the closing of a file stream by verifying the message size, finding the corresponding file, and freeing resources if the file's output buffer is empty.
- **Inputs**:
    - `files`: A pointer to a `client_files` structure, which is a red-black tree containing active file streams.
    - `imsg`: A pointer to an `imsg` structure, which contains the message data and header for the file close operation.
- **Control Flow**:
    - Extract the `msg_write_close` structure from the `imsg` data and calculate the message length.
    - Check if the message length matches the expected size of `msg_write_close`; if not, terminate with an error.
    - Initialize a `client_file` structure to search for the file stream in the `files` tree using `RB_FIND`.
    - If the file stream is not found, terminate with an error indicating an unknown stream number.
    - Log the closing of the file stream for debugging purposes.
    - Check if the file's event is NULL or if the output buffer length is zero.
    - If the event is not NULL, free the buffer event.
    - If the file descriptor is valid (not -1), close the file descriptor.
    - Remove the file from the `files` tree and free the file structure using [`file_free`](#file_free).
- **Output**:
    - The function does not return a value; it performs operations to close and clean up resources associated with a file stream.
- **Functions called**:
    - [`file_free`](#file_free)


---
### file_read_error_callback<!-- {{#callable:file_read_error_callback}} -->
The `file_read_error_callback` function handles errors during file reading by logging the error, sending a completion message, and cleaning up resources.
- **Inputs**:
    - `bev`: A pointer to the bufferevent structure, which is unused in this function.
    - `what`: A short integer indicating the type of event that occurred, which is unused in this function.
    - `arg`: A pointer to a `client_file` structure representing the file being read.
- **Control Flow**:
    - Retrieve the `client_file` structure from the `arg` parameter.
    - Log a debug message indicating a read error for the file associated with the stream in the `client_file`.
    - Initialize a `msg_read_done` structure with the stream number and an error code of 0.
    - Send a `MSG_READ_DONE` message to the peer associated with the `client_file` to indicate the read operation is complete.
    - Free the bufferevent associated with the `client_file`.
    - Close the file descriptor associated with the `client_file`.
    - Remove the `client_file` from the red-black tree of active files.
    - Free the `client_file` structure.
- **Output**:
    - The function does not return a value; it performs cleanup and sends a message to indicate the read operation is complete.
- **Functions called**:
    - [`proc_send`](proc.c.driver.md#proc_send)
    - [`file_free`](#file_free)


---
### file_read_callback<!-- {{#callable:file_read_callback}} -->
The `file_read_callback` function processes data from a buffer event, sending it in chunks to a peer until the buffer is empty.
- **Inputs**:
    - `bev`: A pointer to a bufferevent structure, which is unused in this function.
    - `arg`: A pointer to a `client_file` structure, representing the file being read.
- **Control Flow**:
    - Allocate memory for a `msg_read_data` structure using [`xmalloc`](xmalloc.c.driver.md#xmalloc).
    - Enter a loop that continues as long as there is data in the buffer.
    - Retrieve the data and its size from the buffer associated with the `client_file` structure.
    - If the buffer size is zero, break out of the loop.
    - Limit the buffer size to a maximum allowed size if necessary.
    - Log the amount of data read from the file.
    - Reallocate memory for the `msg_read_data` structure to accommodate the data size.
    - Set the stream identifier in the `msg_read_data` structure.
    - Copy the data from the buffer into the `msg_read_data` structure.
    - Send the data to the peer using [`proc_send`](proc.c.driver.md#proc_send).
    - Drain the processed data from the buffer.
    - Free the allocated `msg_read_data` structure.
- **Output**:
    - The function does not return a value; it sends data to a peer and logs the read operation.
- **Functions called**:
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`xrealloc`](xmalloc.c.driver.md#xrealloc)
    - [`proc_send`](proc.c.driver.md#proc_send)


---
### file_read_open<!-- {{#callable:file_read_open}} -->
The `file_read_open` function handles the opening of a file for reading, setting up necessary structures and events for asynchronous file operations, and sending a response back to the peer.
- **Inputs**:
    - `files`: A pointer to a `client_files` structure, which is a red-black tree containing active file streams.
    - `peer`: A pointer to a `tmuxpeer` structure representing the communication peer.
    - `imsg`: A pointer to an `imsg` structure containing the message data for the file read operation.
    - `allow_streams`: An integer flag indicating whether streams are allowed.
    - `close_received`: An integer flag indicating whether the file descriptor should be closed after use.
    - `cb`: A callback function of type `client_file_cb` to be called when the file operation is complete.
    - `cbdata`: A pointer to user-defined data to be passed to the callback function.
- **Control Flow**:
    - Extracts the `msg_read_open` message from the `imsg` data and calculates the message length.
    - Checks if the message length is valid; if not, it triggers a fatal error.
    - Determines the file path from the message data, defaulting to '-' if no path is provided.
    - Logs the attempt to open the file for reading with the specified stream and path.
    - Searches for an existing file stream in the `files` tree; if found, sets an error and jumps to the reply section.
    - Creates a new `client_file` structure with the peer, stream, callback, and callback data.
    - Checks if the file is already closed; if so, sets an error and jumps to the reply section.
    - Attempts to open the file using the specified path and flags, or duplicates the file descriptor if streams are allowed and the descriptor is valid.
    - If the file descriptor is invalid, sets an error and jumps to the reply section.
    - Creates a new `bufferevent` for the file descriptor to handle asynchronous reading, enabling read events.
    - If any error occurs, sends a `MSG_READ_DONE` message back to the peer with the error code.
- **Output**:
    - The function does not return a value but sends a `MSG_READ_DONE` message to the peer with the stream number and error code if an error occurs.
- **Functions called**:
    - [`file_create_with_peer`](#file_create_with_peer)
    - [`proc_send`](proc.c.driver.md#proc_send)


---
### file_read_cancel<!-- {{#callable:file_read_cancel}} -->
The `file_read_cancel` function cancels an ongoing file read operation by identifying the file stream and invoking an error callback.
- **Inputs**:
    - `files`: A pointer to a `client_files` structure, which is a red-black tree containing active file streams.
    - `imsg`: A pointer to an `imsg` structure, which contains the message data and header for the read cancel operation.
- **Control Flow**:
    - Extract the `msg_read_cancel` structure from the `imsg` data.
    - Calculate the message length by subtracting the IMSG header size from the total message length.
    - Check if the message length matches the expected size of `msg_read_cancel`; if not, terminate with an error.
    - Initialize a `client_file` structure `find` with the stream number from the `msg_read_cancel` message.
    - Search for the corresponding `client_file` in the `files` tree using `RB_FIND`; if not found, terminate with an error.
    - Log a debug message indicating the cancellation of the file stream.
    - Invoke [`file_read_error_callback`](#file_read_error_callback) to handle the cancellation and cleanup.
- **Output**:
    - The function does not return a value; it performs operations to cancel a file read and handle any necessary cleanup.
- **Functions called**:
    - [`file_read_error_callback`](#file_read_error_callback)


---
### file_write_ready<!-- {{#callable:file_write_ready}} -->
The `file_write_ready` function processes a write-ready message for a file stream, checking for errors and either marking the file as done or pushing data to be written.
- **Inputs**:
    - `files`: A pointer to a `client_files` structure, which is a red-black tree containing active file streams.
    - `imsg`: A pointer to an `imsg` structure, which contains the message data and header, including the write-ready message.
- **Control Flow**:
    - Extract the `msg_write_ready` structure from the `imsg` data and calculate the message length excluding the header.
    - Check if the message length matches the expected size of `msg_write_ready`; if not, terminate the program with an error.
    - Initialize a `client_file` structure `find` with the stream from the message and attempt to find the corresponding file in the `files` tree using `RB_FIND`.
    - If the file is not found, return immediately.
    - If the message indicates an error, set the error in the `client_file` structure and call [`file_fire_done`](#file_fire_done) to handle completion.
    - If there is no error, call [`file_push`](#file_push) to continue writing data to the file.
- **Output**:
    - The function does not return a value; it modifies the state of the `client_file` structure associated with the stream, either marking it as done or pushing data to be written.
- **Functions called**:
    - [`file_fire_done`](#file_fire_done)
    - [`file_push`](#file_push)


---
### file_read_data<!-- {{#callable:file_read_data}} -->
The `file_read_data` function processes incoming read data messages for a specific file stream, adding the data to the corresponding client file buffer if no errors are present.
- **Inputs**:
    - `files`: A pointer to a `client_files` structure, which is a red-black tree containing all active client file streams.
    - `imsg`: A pointer to an `imsg` structure, which contains the message header and data for the read operation.
- **Control Flow**:
    - Extract the `msg_read_data` structure from the `imsg` data and calculate the message length excluding the header size.
    - Check if the message length is less than the size of `msg_read_data`; if so, terminate with an error using `fatalx`.
    - Initialize a `client_file` structure `find` with the stream from the message and attempt to find the corresponding client file in the `files` tree using `RB_FIND`.
    - If the client file is not found, return immediately.
    - Log the number of bytes read for the file stream.
    - If there is no error and the file is not closed, attempt to add the data to the file's buffer using `evbuffer_add`.
    - If adding to the buffer fails, set the file's error to `ENOMEM` and call [`file_fire_done`](#file_fire_done) to handle the error.
    - If successful, call [`file_fire_read`](#file_fire_read) to process the read data.
- **Output**:
    - The function does not return a value but modifies the state of the client file, potentially logging errors or triggering callbacks if data is successfully read or if an error occurs.
- **Functions called**:
    - [`file_fire_done`](#file_fire_done)
    - [`file_fire_read`](#file_fire_read)


---
### file_read_done<!-- {{#callable:file_read_done}} -->
The `file_read_done` function processes the completion of a file read operation by updating the corresponding client file's error status and triggering a completion event.
- **Inputs**:
    - `files`: A pointer to a `client_files` structure, which is a red-black tree containing all active client file entries.
    - `imsg`: A pointer to an `imsg` structure, which contains the message data and header related to the file read operation.
- **Control Flow**:
    - Extract the `msg_read_done` structure from the `imsg` data.
    - Calculate the message length by subtracting the IMSG header size from the total message length.
    - Check if the message length matches the expected size of `msg_read_done`; if not, terminate the program with an error.
    - Initialize a `client_file` structure `find` with the stream from the `msg_read_done` message.
    - Search for the corresponding `client_file` in the `files` tree using `RB_FIND`; if not found, return immediately.
    - Log a debug message indicating the completion of the file read for the found stream.
    - Update the `error` field of the found `client_file` with the error from the `msg_read_done` message.
    - Call [`file_fire_done`](#file_fire_done) to trigger the completion event for the `client_file`.
- **Output**:
    - The function does not return a value; it updates the state of the `client_file` and triggers a completion event.
- **Functions called**:
    - [`file_fire_done`](#file_fire_done)


