# Purpose
This C source code file is part of the `tmux` project, a terminal multiplexer that allows users to switch between several programs in one terminal, detach them, and reattach them to a different terminal. The file primarily handles the client-side operations of `tmux`, focusing on establishing and managing the connection between a `tmux` client and server. It includes functions for connecting to the server, handling client signals, sending identification messages, and managing client exit scenarios. The code is structured to handle various client states, such as waiting for server readiness and being attached to a session, and it processes messages from the server accordingly.

Key components of the file include functions for acquiring a server lock ([`client_get_lock`](#client_get_lock)), connecting to the server ([`client_connect`](#client_connect)), and executing commands in a shell ([`client_exec`](#client_exec)). The file also defines several static variables to manage client state and exit reasons, such as `client_exitreason` and `client_exitval`. The code is designed to be part of a larger application, as indicated by its inclusion of headers and its use of external functions and structures like `proc_start` and `proc_send`. It does not define a public API but rather implements internal client-side logic for `tmux`. The file is integral to the client-server communication mechanism within `tmux`, ensuring that client operations are correctly synchronized with the server's state and actions.
# Imports and Dependencies

---
- `sys/types.h`
- `sys/socket.h`
- `sys/uio.h`
- `sys/un.h`
- `sys/wait.h`
- `sys/file.h`
- `errno.h`
- `fcntl.h`
- `signal.h`
- `stdlib.h`
- `string.h`
- `unistd.h`
- `tmux.h`


# Global Variables

---
### client_proc
- **Type**: `struct tmuxproc*`
- **Description**: The `client_proc` variable is a static pointer to a `tmuxproc` structure, which is likely used to represent a client process within the tmux application. This structure is part of the internal process management system of tmux, handling client-specific operations and state.
- **Use**: `client_proc` is used to manage and interact with the client process in the tmux application, including starting, signaling, and exiting the client process.


---
### client_peer
- **Type**: `struct tmuxpeer*`
- **Description**: The `client_peer` is a static global variable that is a pointer to a `struct tmuxpeer`. This structure is likely used to represent a communication peer in the tmux client-server architecture, facilitating message passing between the client and server components of tmux.
- **Use**: This variable is used to manage and facilitate communication between the client and server in the tmux application.


---
### client_flags
- **Type**: `uint64_t`
- **Description**: The `client_flags` variable is a 64-bit unsigned integer used to store various flags related to the client's state and behavior. These flags are used to control the client's operations, such as whether to start a server or handle specific client modes.
- **Use**: This variable is used throughout the client code to check and set different operational flags that influence the client's behavior and interaction with the server.


---
### client_suspended
- **Type**: `int`
- **Description**: The `client_suspended` variable is a static integer that indicates whether the client process is currently suspended. It is used to track the suspension state of the client, which can be affected by signals such as SIGTSTP.
- **Use**: This variable is used to determine if the client should handle certain signals differently when it is in a suspended state.


---
### client_exitreason
- **Type**: `enum`
- **Description**: The `client_exitreason` is a static enumeration variable that represents the reason for a client's exit in a client-server application. It includes various exit states such as detached, lost TTY, terminated, and others. The initial value is set to `CLIENT_EXIT_NONE`, indicating no exit reason at the start.
- **Use**: This variable is used to track and determine the specific reason for a client's exit, which can be used for logging or handling exit scenarios appropriately.


---
### client_exitflag
- **Type**: `int`
- **Description**: The `client_exitflag` is a static integer variable used to indicate whether the client should exit. It is part of the client process management in the tmux client code.
- **Use**: This variable is used to signal the client to exit when certain conditions are met, such as receiving an exit message or losing connection to the server.


---
### client_exitval
- **Type**: `int`
- **Description**: The `client_exitval` is a static integer variable that holds the exit value for the client process. It is used to determine the exit status of the client when it terminates.
- **Use**: This variable is used to store the exit code that will be returned when the client process exits.


---
### client_exittype
- **Type**: `enum msgtype`
- **Description**: The `client_exittype` is a static global variable of type `enum msgtype`. It is used to store the type of message that indicates the client's exit condition.
- **Use**: This variable is used to determine the specific exit action or message type when the client process is terminating.


---
### client_exitsession
- **Type**: `const char*`
- **Description**: The `client_exitsession` is a static constant character pointer that holds the name of the session from which the client has detached. It is used to store session information when a client detaches from a session, particularly in scenarios where the client exit reason involves detachment.
- **Use**: This variable is used to store the session name when the client detaches, allowing the program to provide a meaningful exit message.


---
### client_exitmessage
- **Type**: `char*`
- **Description**: The `client_exitmessage` is a static global variable of type `char*` that holds a pointer to a string. This string is used to store a custom exit message for the client when a specific exit reason, `CLIENT_EXIT_MESSAGE_PROVIDED`, is triggered.
- **Use**: This variable is used to store and retrieve a custom exit message when the client exits with the `CLIENT_EXIT_MESSAGE_PROVIDED` reason.


---
### client_execshell
- **Type**: `const char*`
- **Description**: The `client_execshell` is a static constant pointer to a character string, which is intended to store the path or name of the shell to be executed by the client. It is defined at the global scope, indicating that it is accessible throughout the file where it is declared.
- **Use**: This variable is used to store the shell command that the client will execute when the `client_exec` function is called.


---
### client_execcmd
- **Type**: `const char*`
- **Description**: The `client_execcmd` is a static constant character pointer that holds a command string to be executed by the client. It is defined at the top level of the file, making it a global variable within the file scope.
- **Use**: This variable is used to store the command that the client will execute, particularly when the client is set to run a specific command instead of exiting.


---
### client_attached
- **Type**: `int`
- **Description**: The `client_attached` variable is a static integer that indicates whether the client is currently attached to a session. It is initialized to zero, which typically represents a false or unattached state.
- **Use**: This variable is used to track the attachment status of the client, influencing behavior such as signal handling and exit message display.


---
### client_files
- **Type**: `struct client_files`
- **Description**: The `client_files` variable is a static instance of a `struct client_files`, initialized using the `RB_INITIALIZER` macro. This suggests that `client_files` is likely part of a red-black tree data structure, which is commonly used for managing ordered data efficiently.
- **Use**: The `client_files` variable is used to manage file operations, such as reading and writing, within the client process, as indicated by its use in functions like `file_write_left`, `file_read_open`, and `file_write_open`.


---
### client_exit_message
- **Type**: `static const char*`
- **Description**: The `client_exit_message` is a static function that returns a constant character pointer. It is used to generate a human-readable message based on the client's exit reason.
- **Use**: This function is used to provide a descriptive exit message for the client based on the `client_exitreason` enumeration.


# Functions

---
### client_get_lock<!-- {{#callable:client_get_lock}} -->
The `client_get_lock` function attempts to acquire an exclusive lock on a specified lock file, blocking if necessary, and returns a file descriptor or an error code.
- **Inputs**:
    - `lockfile`: A string representing the path to the lock file that the function will attempt to lock.
- **Control Flow**:
    - Log the path of the lock file for debugging purposes.
    - Attempt to open the lock file with write permissions, creating it if it does not exist, and log an error if this fails, returning -1.
    - Try to acquire an exclusive, non-blocking lock on the file descriptor; if this fails, log the error and check if the error is not EAGAIN, returning the file descriptor if so.
    - If the error is EAGAIN, indicating the lock is held by another process, enter a loop to block until the lock can be acquired, ignoring interruptions by signals (EINTR).
    - Close the file descriptor and return -2 if the lock was initially held by another process and is now acquired.
    - Log a success message if the lock is acquired without blocking and return the file descriptor.
- **Output**:
    - The function returns the file descriptor of the locked file on success, -1 if the file cannot be opened, or -2 if the lock was initially held by another process and is now acquired.


---
### client_connect<!-- {{#callable:client_connect}} -->
The `client_connect` function attempts to establish a UNIX socket connection to a server, optionally starting the server if it is not running and the appropriate flags are set.
- **Inputs**:
    - `base`: A pointer to an `event_base` structure used for event handling.
    - `path`: A string representing the file system path to the UNIX socket.
    - `flags`: A 64-bit unsigned integer representing various flags that control the behavior of the connection, such as whether to start the server if it is not running.
- **Control Flow**:
    - Initialize a `sockaddr_un` structure and copy the socket path into it, returning -1 if the path is too long.
    - Attempt to create a socket and connect to the server at the specified path.
    - If the connection fails due to `ECONNREFUSED` or `ENOENT`, check the flags to determine if the server should be started.
    - If the server should be started, attempt to acquire a lock on a lockfile to ensure only one server is started, retrying the connection if necessary.
    - If the lock is acquired and the server is started, retry the connection.
    - If the connection is successful, set the socket to non-blocking mode and return the file descriptor.
    - If any step fails, clean up resources and return -1.
- **Output**:
    - Returns a file descriptor for the connected socket on success, or -1 on failure.
- **Functions called**:
    - [`strlcpy`](compat/strlcpy.c.driver.md#strlcpy)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)
    - [`client_get_lock`](#client_get_lock)
    - [`server_start`](server.c.driver.md#server_start)
    - [`setblocking`](tmux.c.driver.md#setblocking)


---
### client_exit_message<!-- {{#callable:client_exit_message}} -->
The `client_exit_message` function returns a string message describing the reason for a client's exit based on the `client_exitreason` variable.
- **Inputs**:
    - None
- **Control Flow**:
    - The function uses a static character array `msg` to store messages that require formatting.
    - A switch statement is used to determine the message to return based on the value of `client_exitreason`.
    - For `CLIENT_EXIT_DETACHED` and `CLIENT_EXIT_DETACHED_HUP`, if `client_exitsession` is not NULL, a formatted message including the session name is returned; otherwise, a generic message is returned.
    - For other exit reasons, predefined string messages are returned directly.
    - If `client_exitreason` is `CLIENT_EXIT_MESSAGE_PROVIDED`, the function returns the `client_exitmessage`.
    - If none of the cases match, the function returns "unknown reason".
- **Output**:
    - The function returns a constant character pointer to a string describing the client's exit reason.
- **Functions called**:
    - [`xsnprintf`](xmalloc.c.driver.md#xsnprintf)


---
### client_exit<!-- {{#callable:client_exit}} -->
The `client_exit` function checks if there are any pending file writes and exits the client process if none are left.
- **Inputs**:
    - None
- **Control Flow**:
    - The function calls [`file_write_left`](file.c.driver.md#file_write_left) with `client_files` to check if there are any pending file writes.
    - If [`file_write_left`](file.c.driver.md#file_write_left) returns false (indicating no pending writes), it calls [`proc_exit`](proc.c.driver.md#proc_exit) with `client_proc` to exit the client process.
- **Output**:
    - The function does not return any value; it may terminate the client process if there are no pending file writes.
- **Functions called**:
    - [`file_write_left`](file.c.driver.md#file_write_left)
    - [`proc_exit`](proc.c.driver.md#proc_exit)


---
### client_main<!-- {{#callable:client_main}} -->
The `client_main` function initializes and manages a client process that connects to a server, processes commands, and handles client-server communication and control flow.
- **Inputs**:
    - `base`: A pointer to an `event_base` structure used for event handling.
    - `argc`: An integer representing the number of command-line arguments.
    - `argv`: An array of strings representing the command-line arguments.
    - `flags`: A 64-bit unsigned integer representing various flags that control the client's behavior.
    - `feat`: An integer representing feature flags or capabilities of the client.
- **Control Flow**:
    - Determine the initial command type (shell or command) and set the appropriate message type and flags.
    - Parse command-line arguments to check for the start server flag and adjust flags accordingly.
    - Initialize the client process structure and set up signal handling.
    - Attempt to connect to the server using a socket; if unsuccessful, handle errors and possibly start the server.
    - Save current working directory, terminal name, and terminal type for later use.
    - Use the `pledge` system call to restrict the process's capabilities for security purposes.
    - Load terminal capabilities if the standard input is a terminal and handle any errors.
    - Free unused global resources and close file descriptors that are not needed by the client.
    - If control mode is enabled, configure terminal settings for raw input/output mode.
    - Send identification messages to the server, including terminal and environment information.
    - Prepare and send the initial command to the server, handling errors if the command is too long.
    - Enter the main processing loop to handle client-server communication.
    - If the client is set to execute a command upon exit, execute it using the specified shell and command.
    - Restore standard input/output streams to blocking mode before exiting.
    - Handle client exit scenarios, including printing exit messages and sending signals to the parent process if necessary.
    - Return the client's exit value.
- **Output**:
    - The function returns an integer exit value, which indicates the result of the client's execution.
- **Functions called**:
    - [`args_from_vector`](arguments.c.driver.md#args_from_vector)
    - [`cmd_list_any_have`](cmd.c.driver.md#cmd_list_any_have)
    - [`cmd_list_free`](cmd.c.driver.md#cmd_list_free)
    - [`args_free_values`](arguments.c.driver.md#args_free_values)
    - [`proc_start`](proc.c.driver.md#proc_start)
    - [`proc_set_signals`](proc.c.driver.md#proc_set_signals)
    - [`systemd_activated`](compat/systemd.c.driver.md#systemd_activated)
    - [`server_start`](server.c.driver.md#server_start)
    - [`client_connect`](#client_connect)
    - [`proc_add_peer`](proc.c.driver.md#proc_add_peer)
    - [`find_cwd`](tmux.c.driver.md#find_cwd)
    - [`find_home`](tmux.c.driver.md#find_home)
    - [`tty_term_read_list`](tty-term.c.driver.md#tty_term_read_list)
    - [`options_free`](options.c.driver.md#options_free)
    - [`environ_free`](environ.c.driver.md#environ_free)
    - [`cfmakeraw`](compat/cfmakeraw.c.driver.md#cfmakeraw)
    - [`client_send_identify`](#client_send_identify)
    - [`tty_term_free_list`](tty-term.c.driver.md#tty_term_free_list)
    - [`proc_flush_peer`](proc.c.driver.md#proc_flush_peer)
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`cmd_pack_argv`](cmd.c.driver.md#cmd_pack_argv)
    - [`proc_send`](proc.c.driver.md#proc_send)
    - [`proc_loop`](proc.c.driver.md#proc_loop)
    - [`client_exec`](#client_exec)
    - [`setblocking`](tmux.c.driver.md#setblocking)
    - [`client_exit_message`](#client_exit_message)
    - [`getline`](compat/getline.c.driver.md#getline)


---
### client_send_identify<!-- {{#callable:client_send_identify}} -->
The `client_send_identify` function sends various identification messages from the client to the server, including terminal and environment information.
- **Inputs**:
    - `ttynam`: The name of the terminal device.
    - `termname`: The name of the terminal type.
    - `caps`: An array of strings representing terminal capabilities.
    - `ncaps`: The number of terminal capabilities in the `caps` array.
    - `cwd`: The current working directory of the client.
    - `feat`: An integer representing client features.
- **Control Flow**:
    - Initialize local variables including a pointer to environment variables and file descriptors.
    - Send the client's flags to the server using [`proc_send`](proc.c.driver.md#proc_send) with the message type `MSG_IDENTIFY_LONGFLAGS`.
    - Send the terminal name and features to the server using [`proc_send`](proc.c.driver.md#proc_send) with the message types `MSG_IDENTIFY_TERM` and `MSG_IDENTIFY_FEATURES`.
    - Send the terminal device name and current working directory to the server using [`proc_send`](proc.c.driver.md#proc_send) with the message types `MSG_IDENTIFY_TTYNAME` and `MSG_IDENTIFY_CWD`.
    - Iterate over the `caps` array and send each terminal capability to the server using [`proc_send`](proc.c.driver.md#proc_send) with the message type `MSG_IDENTIFY_TERMINFO`.
    - Duplicate the standard input and output file descriptors and send them to the server using [`proc_send`](proc.c.driver.md#proc_send) with the message types `MSG_IDENTIFY_STDIN` and `MSG_IDENTIFY_STDOUT`.
    - Retrieve the process ID and send it to the server using [`proc_send`](proc.c.driver.md#proc_send) with the message type `MSG_IDENTIFY_CLIENTPID`.
    - Iterate over the environment variables and send each to the server using [`proc_send`](proc.c.driver.md#proc_send) with the message type `MSG_IDENTIFY_ENVIRON`, ensuring the message size does not exceed a maximum limit.
    - Send a final message to indicate completion using [`proc_send`](proc.c.driver.md#proc_send) with the message type `MSG_IDENTIFY_DONE`.
- **Output**:
    - The function does not return a value; it communicates with the server by sending messages.
- **Functions called**:
    - [`proc_send`](proc.c.driver.md#proc_send)


---
### client_exec<!-- {{#callable:client_exec}} -->
The `client_exec` function executes a shell command using a specified shell, setting up the environment and handling necessary process and signal configurations.
- **Inputs**:
    - `shell`: A string representing the path to the shell executable to be used for executing the command.
    - `shellcmd`: A string representing the command to be executed by the shell.
- **Control Flow**:
    - Logs the shell and command being executed for debugging purposes.
    - Determines the shell's argument vector using [`shell_argv0`](tmux.c.driver.md#shell_argv0) and sets the `SHELL` environment variable to the specified shell.
    - Clears signals for the client process using [`proc_clear_signals`](proc.c.driver.md#proc_clear_signals).
    - Sets the standard input, output, and error streams to blocking mode using [`setblocking`](tmux.c.driver.md#setblocking).
    - Closes all file descriptors starting from `STDERR_FILENO + 1` using `closefrom`.
    - Attempts to execute the command using `execl` with the shell, its argument vector, the `-c` option, and the command string.
    - If `execl` fails, it calls `fatal` to handle the error.
- **Output**:
    - This function does not return; it either successfully executes the command or terminates the process with an error if execution fails.
- **Functions called**:
    - [`shell_argv0`](tmux.c.driver.md#shell_argv0)
    - [`setenv`](compat/setenv.c.driver.md#setenv)
    - [`proc_clear_signals`](proc.c.driver.md#proc_clear_signals)
    - [`setblocking`](tmux.c.driver.md#setblocking)


---
### client_signal<!-- {{#callable:client_signal}} -->
The `client_signal` function handles various signals received by the client process, performing actions such as cleaning up child processes, exiting, or sending messages to a peer process based on the signal type.
- **Inputs**:
    - `sig`: An integer representing the signal number that the function is handling.
- **Control Flow**:
    - Log the received signal using `log_debug`.
    - If the signal is `SIGCHLD`, enter a loop to clean up any terminated child processes using `waitpid` until no more child processes are left or an error occurs.
    - If the signal is `SIGTERM` or `SIGHUP` and the client is not attached, call [`proc_exit`](proc.c.driver.md#proc_exit) to terminate the client process.
    - If the client is attached, handle the signal based on its type:
    - For `SIGHUP`, set the exit reason to `CLIENT_EXIT_LOST_TTY`, set the exit value to 1, and send an `MSG_EXITING` message to the peer.
    - For `SIGTERM`, if the client is not suspended, set the exit reason to `CLIENT_EXIT_TERMINATED`, set the exit value to 1, and send an `MSG_EXITING` message to the peer.
    - For `SIGWINCH`, send an `MSG_RESIZE` message to the peer.
    - For `SIGCONT`, ignore `SIGTSTP` using `sigaction`, send an `MSG_WAKEUP` message to the peer, and set `client_suspended` to 0.
- **Output**:
    - The function does not return a value; it performs actions based on the received signal.
- **Functions called**:
    - [`proc_exit`](proc.c.driver.md#proc_exit)
    - [`proc_send`](proc.c.driver.md#proc_send)


---
### client_file_check_cb<!-- {{#callable:client_file_check_cb}} -->
The `client_file_check_cb` function checks if the client should exit based on the `client_exitflag`.
- **Inputs**:
    - `c`: A pointer to a `client` structure, which is unused in this function.
    - `path`: A constant character pointer representing the file path, which is unused in this function.
    - `error`: An integer representing an error code, which is unused in this function.
    - `closed`: An integer indicating if the file is closed, which is unused in this function.
    - `buffer`: A pointer to an `evbuffer` structure, which is unused in this function.
    - `data`: A void pointer to additional data, which is unused in this function.
- **Control Flow**:
    - The function checks if the global variable `client_exitflag` is set.
    - If `client_exitflag` is true, it calls the `client_exit()` function to handle the client exit process.
- **Output**:
    - This function does not return any value.
- **Functions called**:
    - [`client_exit`](#client_exit)


---
### client_dispatch<!-- {{#callable:client_dispatch}} -->
The `client_dispatch` function handles incoming messages for a client, determining the appropriate action based on the client's attachment state and the message content.
- **Inputs**:
    - `imsg`: A pointer to an `imsg` structure representing the incoming message to be processed.
    - `arg`: An unused pointer argument, typically used for passing additional data, but not utilized in this function.
- **Control Flow**:
    - Check if `imsg` is NULL; if so, set the client exit reason to `CLIENT_EXIT_LOST_SERVER`, set the exit value to 1, and call [`proc_exit`](proc.c.driver.md#proc_exit) to terminate the client process.
    - If the client is attached (`client_attached` is true), call [`client_dispatch_attached`](#client_dispatch_attached) with the `imsg` to handle the message in the attached state.
    - If the client is not attached, call [`client_dispatch_wait`](#client_dispatch_wait) with the `imsg` to handle the message in the waiting state.
- **Output**:
    - The function does not return a value; it performs actions based on the message and client state, potentially altering global state or triggering process termination.
- **Functions called**:
    - [`proc_exit`](proc.c.driver.md#proc_exit)
    - [`client_dispatch_attached`](#client_dispatch_attached)
    - [`client_dispatch_wait`](#client_dispatch_wait)


---
### client_dispatch_exit_message<!-- {{#callable:client_dispatch_exit_message}} -->
The `client_dispatch_exit_message` function processes an exit message by extracting an exit value and an optional exit message from the provided data.
- **Inputs**:
    - `data`: A pointer to a character array containing the exit message data.
    - `datalen`: The length of the data array, indicating the size of the exit message data.
- **Control Flow**:
    - Check if the data length is less than the size of an integer and not zero, and if so, terminate the program with an error message.
    - If the data length is at least the size of an integer, copy the integer value from the data into the `retval` variable and assign it to `client_exitval`.
    - If the data length is greater than the size of an integer, adjust the data pointer and length to exclude the integer, allocate memory for the exit message, copy the message into `client_exitmessage`, and set the last character to a null terminator.
    - Set the `client_exitreason` to `CLIENT_EXIT_MESSAGE_PROVIDED` if an exit message is provided.
- **Output**:
    - The function does not return a value but updates global variables `client_exitval`, `client_exitmessage`, and `client_exitreason` based on the processed exit message data.
- **Functions called**:
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)


---
### client_dispatch_wait<!-- {{#callable:client_dispatch_wait}} -->
The `client_dispatch_wait` function processes incoming inter-process messages (imsg) while the client is in a waiting state, handling various message types to manage client-server interactions and state transitions.
- **Inputs**:
    - `imsg`: A pointer to an `imsg` structure representing the incoming inter-process message to be processed.
- **Control Flow**:
    - Check if the `pledge_applied` flag is not set, and if not, apply a security pledge to restrict the process's capabilities and set the flag.
    - Extract the data and data length from the `imsg` structure.
    - Use a switch statement to handle different message types based on `imsg->hdr.type`.
    - For `MSG_EXIT` and `MSG_SHUTDOWN`, call [`client_dispatch_exit_message`](#client_dispatch_exit_message), set `client_exitflag`, and call [`client_exit`](#client_exit).
    - For `MSG_READY`, verify data length, set `client_attached`, and send a `MSG_RESIZE` message.
    - For `MSG_VERSION`, verify data length, print a protocol version mismatch error, set `client_exitval`, and exit the process.
    - For `MSG_FLAGS`, verify data length, update `client_flags`, and log the new flags.
    - For `MSG_SHELL`, verify data length, execute the shell command, and exit.
    - For `MSG_DETACH` and `MSG_DETACHKILL`, send a `MSG_EXITING` message.
    - For `MSG_EXITED`, exit the process.
    - For `MSG_READ_OPEN`, `MSG_READ_CANCEL`, `MSG_WRITE_OPEN`, `MSG_WRITE`, and `MSG_WRITE_CLOSE`, call respective file operation functions with appropriate parameters.
    - For `MSG_OLDSTDERR`, `MSG_OLDSTDIN`, and `MSG_OLDSTDOUT`, print an error about server version and exit the process.
- **Output**:
    - The function does not return a value; it performs actions based on the message type, such as updating client state, sending messages, or exiting the process.
- **Functions called**:
    - [`client_dispatch_exit_message`](#client_dispatch_exit_message)
    - [`client_exit`](#client_exit)
    - [`proc_send`](proc.c.driver.md#proc_send)
    - [`proc_exit`](proc.c.driver.md#proc_exit)
    - [`client_exec`](#client_exec)
    - [`file_read_open`](file.c.driver.md#file_read_open)
    - [`file_read_cancel`](file.c.driver.md#file_read_cancel)
    - [`file_write_open`](file.c.driver.md#file_write_open)
    - [`file_write_data`](file.c.driver.md#file_write_data)
    - [`file_write_close`](file.c.driver.md#file_write_close)


---
### client_dispatch_attached<!-- {{#callable:client_dispatch_attached}} -->
The `client_dispatch_attached` function processes various message types received by a client in an attached state, executing corresponding actions based on the message type.
- **Inputs**:
    - `imsg`: A pointer to an `imsg` structure containing the message data and header information to be processed.
- **Control Flow**:
    - Extracts the data and data length from the `imsg` structure.
    - Switches on the message type specified in `imsg->hdr.type`.
    - For `MSG_FLAGS`, verifies data length, updates `client_flags`, and logs the new flags.
    - For `MSG_DETACH` and `MSG_DETACHKILL`, verifies data integrity, sets exit session and type, determines exit reason, and sends an `MSG_EXITING` message.
    - For `MSG_EXEC`, verifies data integrity, extracts command and shell, sets exit type, and sends an `MSG_EXITING` message.
    - For `MSG_EXIT`, calls [`client_dispatch_exit_message`](#client_dispatch_exit_message) to handle exit data, sets exit reason if not already set, and sends an `MSG_EXITING` message.
    - For `MSG_EXITED`, verifies data length and calls [`proc_exit`](proc.c.driver.md#proc_exit) to terminate the process.
    - For `MSG_SHUTDOWN`, verifies data length, sends an `MSG_EXITING` message, sets exit reason, and exit value.
    - For `MSG_SUSPEND`, verifies data length, sets up default signal handler for `SIGTSTP`, marks client as suspended, and sends `SIGTSTP` to itself.
    - For `MSG_LOCK`, verifies data integrity, executes the lock command using `system`, and sends an `MSG_UNLOCK` message.
- **Output**:
    - The function does not return a value; it performs actions based on the message type, such as updating client state, sending messages, or terminating the process.
- **Functions called**:
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`proc_send`](proc.c.driver.md#proc_send)
    - [`client_dispatch_exit_message`](#client_dispatch_exit_message)
    - [`proc_exit`](proc.c.driver.md#proc_exit)


