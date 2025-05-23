# Purpose
This C source code file is part of the `tmux` project, a terminal multiplexer that allows users to switch between several programs in one terminal, detach them, and reattach them to a different terminal. The file primarily handles the client-side operations of `tmux`, focusing on establishing and managing the connection between a `tmux` client and server. It includes functions for connecting to the server, sending identification and command messages, handling signals, and managing the client's lifecycle and exit conditions.

The code defines several static variables and functions that manage the client's state and interactions with the server. Key components include functions for obtaining a lock to start the server ([`client_get_lock`](#client_get_lock)), connecting to the server ([`client_connect`](#client_connect)), and sending identification messages ([`client_send_identify`](#client_send_identify)). The file also includes a main loop ([`client_main`](#client_main)) that processes command-line arguments, sets up the client environment, and manages the communication with the server. Additionally, it handles various signals and dispatches messages received from the server, ensuring proper client behavior and exit procedures. This file is integral to the `tmux` client functionality, providing a robust interface for client-server communication and process management.
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
- **Description**: The `client_proc` variable is a static pointer to a `struct tmuxproc`, which represents a client process structure in the tmux client code. This structure is used to manage the lifecycle and operations of a client process, including starting, signaling, and exiting the process.
- **Use**: `client_proc` is used to store and manage the client process structure, facilitating operations such as process creation, signal handling, and process termination within the tmux client.


---
### client_peer
- **Type**: `struct tmuxpeer*`
- **Description**: The `client_peer` is a pointer to a `tmuxpeer` structure, which is used to represent a communication peer in the tmux client-server architecture. It is declared as a static global variable, indicating that it is intended for use within the file scope only.
- **Use**: This variable is used to manage and facilitate communication between the client and server processes in the tmux application.


---
### client_flags
- **Type**: `uint64_t`
- **Description**: The `client_flags` variable is a 64-bit unsigned integer used to store various flags related to the client's state and behavior. These flags are used to control the client's operations and interactions with the server, such as whether to start the server, control mode settings, and other client-specific options.
- **Use**: This variable is used to store and manage client-specific flags that dictate the client's behavior and interaction with the server.


---
### client_suspended
- **Type**: `int`
- **Description**: The `client_suspended` variable is a static integer that indicates whether the client process is currently suspended. It is used to track the suspension state of the client, which can be affected by signals such as SIGTSTP.
- **Use**: This variable is used to determine if the client should handle certain signals differently when it is in a suspended state.


---
### client_exitreason
- **Type**: `enum`
- **Description**: The `client_exitreason` is a static enumeration variable that represents the reason for a client's exit in a client-server application. It includes various exit states such as `CLIENT_EXIT_NONE`, `CLIENT_EXIT_DETACHED`, `CLIENT_EXIT_TERMINATED`, and others, each indicating a specific exit condition. The variable is initialized to `CLIENT_EXIT_NONE`, indicating no exit reason initially.
- **Use**: This variable is used to track and determine the specific reason for a client's exit, which can be used for logging, debugging, or handling different exit scenarios in the application.


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
- **Description**: The `client_exittype` is a static global variable of type `enum msgtype`. It is used to store the type of message that indicates the reason for the client's exit.
- **Use**: This variable is used to determine the specific message type associated with the client's exit process.


---
### client_exitsession
- **Type**: `const char*`
- **Description**: `client_exitsession` is a static constant character pointer that holds the name of the session from which the client has detached. It is used to store session information when a client detaches from a session, particularly in scenarios where the client exit reason involves detachment.
- **Use**: This variable is used to store the session name when the client detaches, allowing the program to provide a meaningful exit message.


---
### client_exitmessage
- **Type**: `char*`
- **Description**: The `client_exitmessage` is a static global variable of type `char*` that holds a pointer to a string. This string is used to store a custom exit message for the client when a specific exit reason, `CLIENT_EXIT_MESSAGE_PROVIDED`, is triggered.
- **Use**: This variable is used to store and retrieve a custom exit message when the client exits with the `CLIENT_EXIT_MESSAGE_PROVIDED` reason.


---
### client_execshell
- **Type**: `const char*`
- **Description**: `client_execshell` is a static constant pointer to a character string, which is used to store the path or name of the shell to be executed by the client. It is defined at the global scope, indicating that it is intended to be used throughout the file where it is declared.
- **Use**: This variable is used to store the shell command that the client will execute when the `client_exec` function is called.


---
### client_execcmd
- **Type**: `const char*`
- **Description**: The `client_execcmd` is a static constant character pointer that holds the command to be executed by the client. It is used in conjunction with `client_execshell` to run a command in a shell environment.
- **Use**: This variable is used to store the command string that the client will execute when the `client_exec` function is called.


---
### client_attached
- **Type**: `int`
- **Description**: The `client_attached` variable is a static integer that indicates whether the client is currently attached to a session. It is initialized to zero, which typically represents a false or unattached state.
- **Use**: This variable is used to track the attachment status of the client, influencing behavior such as signal handling and exit message display.


---
### client_files
- **Type**: `struct client_files`
- **Description**: The `client_files` variable is a static instance of a `struct client_files`, initialized using the `RB_INITIALIZER` macro. This suggests that `client_files` is likely part of a red-black tree data structure, which is commonly used for managing ordered data efficiently.
- **Use**: This variable is used to manage file operations, such as reading and writing, within the client context of the application.


---
### client_exit_message
- **Type**: `function`
- **Description**: The `client_exit_message` is a static function that returns a constant character pointer. It is used to generate a human-readable message based on the client's exit reason, which is determined by the `client_exitreason` variable.
- **Use**: This function is used to provide a descriptive exit message for the client based on the exit reason.


# Functions

---
### client_get_lock<!-- {{#callable:client_get_lock}} -->
The `client_get_lock` function attempts to acquire an exclusive lock on a specified lock file, blocking if necessary, and returns a file descriptor or error code based on the outcome.
- **Inputs**:
    - `lockfile`: A string representing the path to the lock file that the function will attempt to lock.
- **Control Flow**:
    - Log the path of the lock file for debugging purposes.
    - Attempt to open the lock file with write and create permissions; if it fails, log the error and return -1.
    - Try to acquire an exclusive, non-blocking lock on the file; if it fails, log the error.
    - If the error is not EAGAIN, return the file descriptor; otherwise, enter a loop to acquire a blocking lock, ignoring interruptions.
    - If the blocking lock is acquired, close the file descriptor and return -2 to indicate the need to retry.
    - Log success if the lock is acquired and return the file descriptor.
- **Output**:
    - The function returns an integer: a file descriptor if the lock is successfully acquired, -1 if opening the file fails, or -2 if the lock is held by another process and the caller should retry.


---
### client_connect<!-- {{#callable:client_connect}} -->
The `client_connect` function attempts to establish a connection to a server using a UNIX domain socket, with options to start the server if it is not running and handle locking mechanisms to prevent race conditions.
- **Inputs**:
    - `base`: A pointer to an `event_base` structure used for event handling.
    - `path`: A string representing the file system path to the UNIX domain socket.
    - `flags`: A 64-bit unsigned integer representing various flags that control the behavior of the connection, such as whether to start the server if not running.
- **Control Flow**:
    - Initialize a `sockaddr_un` structure and copy the socket path into it, checking for path length errors.
    - Attempt to create a socket and connect to the server using the specified path.
    - If the connection fails due to specific errors (e.g., `ECONNREFUSED`, `ENOENT`), check the flags to determine if the server should be started.
    - If the server is to be started, acquire a lock using a lock file to prevent multiple clients from starting the server simultaneously.
    - Retry the connection attempt after acquiring the lock, and if successful, start the server if necessary.
    - Set the socket to non-blocking mode and return the file descriptor if successful.
    - Handle failure cases by cleaning up resources and returning an error code.
- **Output**:
    - Returns a file descriptor for the connected socket on success, or -1 on failure.
- **Functions called**:
    - [`client_get_lock`](#client_get_lock)


---
### client_exit_message<!-- {{#callable:client_exit_message}} -->
The `client_exit_message` function returns a string message describing the reason for a client's exit based on the `client_exitreason` variable.
- **Inputs**:
    - None
- **Control Flow**:
    - A static character array `msg` of size 256 is declared to hold the exit message.
    - A switch statement is used to determine the exit reason based on the `client_exitreason` variable.
    - For `CLIENT_EXIT_NONE`, no action is taken and the function proceeds to return "unknown reason".
    - For `CLIENT_EXIT_DETACHED`, if `client_exitsession` is not NULL, a formatted message including the session name is returned; otherwise, "detached" is returned.
    - For `CLIENT_EXIT_DETACHED_HUP`, similar logic to `CLIENT_EXIT_DETACHED` is applied, but the message includes "and SIGHUP".
    - For other exit reasons like `CLIENT_EXIT_LOST_TTY`, `CLIENT_EXIT_TERMINATED`, `CLIENT_EXIT_LOST_SERVER`, `CLIENT_EXIT_EXITED`, `CLIENT_EXIT_SERVER_EXITED`, and `CLIENT_EXIT_MESSAGE_PROVIDED`, predefined messages are returned.
    - If none of the cases match, "unknown reason" is returned.
- **Output**:
    - The function returns a constant character pointer to a string describing the client's exit reason.


---
### client_exit<!-- {{#callable:client_exit}} -->
The `client_exit` function checks if there are any pending file writes and exits the client process if none are left.
- **Inputs**:
    - None
- **Control Flow**:
    - The function checks if there are any pending file writes using `file_write_left(&client_files)`.
    - If there are no pending writes (i.e., `file_write_left` returns false), it calls `proc_exit(client_proc)` to exit the client process.
- **Output**:
    - The function does not return any value; it may cause the client process to exit if there are no pending file writes.


---
### client_main<!-- {{#callable:client_main}} -->
The `client_main` function initializes and manages a client process that connects to a server, processes commands, and handles client-server communication and control flow.
- **Inputs**:
    - `base`: A pointer to an `event_base` structure used for event handling.
    - `argc`: An integer representing the number of command-line arguments.
    - `argv`: An array of strings representing the command-line arguments.
    - `flags`: A 64-bit unsigned integer representing various flags that control client behavior.
    - `feat`: An integer representing feature flags or capabilities.
- **Control Flow**:
    - Determine the initial command type (MSG_SHELL or MSG_COMMAND) based on the presence of a shell command or command-line arguments.
    - If command-line arguments are present, parse them to check for the start server flag and update flags accordingly.
    - Initialize the client process structure and set up signal handling.
    - Attempt to connect to the server using a socket; if connection fails, handle errors and possibly start the server.
    - Save current working directory, terminal name, and terminal type for later use.
    - Use the `pledge` system call to restrict the process's capabilities for security purposes.
    - Load terminal capabilities if the standard input is a terminal and handle any errors.
    - Free unused global resources and close file descriptors that are not needed in the client.
    - If control mode is enabled, configure terminal settings for raw input/output.
    - Send identification messages to the server with terminal and environment information.
    - If the initial command is MSG_COMMAND, prepare and send the command to the server; if MSG_SHELL, send a shell message.
    - Enter the main processing loop to handle client-server communication.
    - If the client is set to execute a command upon exit, restore terminal settings and execute the command.
    - Restore standard input/output/error streams to blocking mode.
    - Handle client exit by printing exit messages, sending signals to parent processes if necessary, and cleaning up resources.
    - Return the client exit value.
- **Output**:
    - The function returns an integer exit value indicating the result of the client process execution.
- **Functions called**:
    - [`client_connect`](#client_connect)
    - [`client_send_identify`](#client_send_identify)
    - [`client_exec`](#client_exec)
    - [`client_exit_message`](#client_exit_message)


---
### client_send_identify<!-- {{#callable:client_send_identify}} -->
The `client_send_identify` function sends various identification messages from the client to the server, including terminal and environment information, to establish the client's identity and capabilities.
- **Inputs**:
    - `ttynam`: The name of the terminal device.
    - `termname`: The name of the terminal type.
    - `caps`: An array of strings representing terminal capabilities.
    - `ncaps`: The number of terminal capabilities in the `caps` array.
    - `cwd`: The current working directory of the client.
    - `feat`: An integer representing feature flags or capabilities of the client.
- **Control Flow**:
    - Initialize local variables including a flags variable set to `client_flags` and retrieve the process ID using `getpid()`.
    - Send the `MSG_IDENTIFY_LONGFLAGS` message twice with the `client_flags` to the server using `proc_send`.
    - Send the `MSG_IDENTIFY_TERM` message with the terminal name and the `MSG_IDENTIFY_FEATURES` message with the feature flags to the server.
    - Send the `MSG_IDENTIFY_TTYNAME` message with the terminal name and the `MSG_IDENTIFY_CWD` message with the current working directory to the server.
    - Iterate over the `caps` array and send each terminal capability as a `MSG_IDENTIFY_TERMINFO` message to the server.
    - Duplicate the standard input and output file descriptors and send them as `MSG_IDENTIFY_STDIN` and `MSG_IDENTIFY_STDOUT` messages, respectively.
    - Send the process ID as a `MSG_IDENTIFY_CLIENTPID` message to the server.
    - Iterate over the environment variables and send each as a `MSG_IDENTIFY_ENVIRON` message to the server, ensuring the message size does not exceed a maximum limit.
    - Send a `MSG_IDENTIFY_DONE` message to indicate the completion of the identification process.
- **Output**:
    - The function does not return a value; it communicates with the server by sending a series of messages to identify the client.


---
### client_exec<!-- {{#callable:client_exec}} -->
The `client_exec` function executes a shell command using a specified shell, setting up the environment and handling necessary process and signal configurations.
- **Inputs**:
    - `shell`: A string representing the path to the shell executable to be used for executing the command.
    - `shellcmd`: A string representing the command to be executed by the shell.
- **Control Flow**:
    - Log the shell and command being executed for debugging purposes.
    - Determine the shell's argument vector using `shell_argv0` and set the `SHELL` environment variable to the specified shell.
    - Clear any signals for the client process using `proc_clear_signals`.
    - Set the standard input, output, and error streams to blocking mode using `setblocking`.
    - Close all file descriptors starting from `STDERR_FILENO + 1` using `closefrom`.
    - Execute the shell with the command using `execl`, passing the shell, its argument vector, the `-c` option, and the command to be executed.
    - If `execl` fails, call `fatal` to handle the error.
- **Output**:
    - The function does not return; it either successfully executes the command or terminates the process with an error if `execl` fails.


---
### client_signal<!-- {{#callable:client_signal}} -->
The `client_signal` function handles various signals received by the client process, performing actions such as cleaning up child processes, exiting, or sending messages to a peer process based on the signal type.
- **Inputs**:
    - `sig`: An integer representing the signal number that the function is handling.
- **Control Flow**:
    - Log the received signal using `log_debug`.
    - If the signal is `SIGCHLD`, enter a loop to clean up any terminated child processes using `waitpid`.
    - If the client is not attached and the signal is `SIGTERM` or `SIGHUP`, call `proc_exit` to terminate the client process.
    - If the client is attached, handle different signals using a switch statement:
    - For `SIGHUP`, set the exit reason to `CLIENT_EXIT_LOST_TTY`, set the exit value to 1, and send an `MSG_EXITING` message to the peer.
    - For `SIGTERM`, if the client is not suspended, set the exit reason to `CLIENT_EXIT_TERMINATED`, set the exit value to 1, and send an `MSG_EXITING` message to the peer.
    - For `SIGWINCH`, send an `MSG_RESIZE` message to the peer.
    - For `SIGCONT`, ignore `SIGTSTP` using `sigaction`, send an `MSG_WAKEUP` message to the peer, and set `client_suspended` to 0.
- **Output**:
    - The function does not return a value; it performs actions based on the signal received, such as cleaning up child processes, exiting the client, or sending messages to a peer process.


---
### client_file_check_cb<!-- {{#callable:client_file_check_cb}} -->
The `client_file_check_cb` function checks if the client exit flag is set and calls [`client_exit`](#client_exit) if it is.
- **Inputs**:
    - `c`: A pointer to a client structure, marked as unused.
    - `path`: A constant character pointer representing the file path, marked as unused.
    - `error`: An integer representing an error code, marked as unused.
    - `closed`: An integer indicating if the file is closed, marked as unused.
    - `buffer`: A pointer to an evbuffer structure, marked as unused.
    - `data`: A void pointer to additional data, marked as unused.
- **Control Flow**:
    - Check if the global variable `client_exitflag` is set.
    - If `client_exitflag` is set, call the [`client_exit`](#client_exit) function.
- **Output**:
    - The function does not return any value (void function).
- **Functions called**:
    - [`client_exit`](#client_exit)


---
### client_dispatch<!-- {{#callable:client_dispatch}} -->
The `client_dispatch` function handles incoming messages for a client, determining the appropriate action based on the client's attachment status and the message content.
- **Inputs**:
    - `imsg`: A pointer to an `imsg` structure representing the incoming message to be processed.
    - `arg`: An unused pointer argument, typically used for passing additional data to the function.
- **Control Flow**:
    - Check if the `imsg` pointer is NULL, indicating a lost server connection; if so, set the exit reason to `CLIENT_EXIT_LOST_SERVER`, set the exit value to 1, and call `proc_exit` to terminate the client process.
    - If the client is attached (`client_attached` is true), call [`client_dispatch_attached`](#client_dispatch_attached) to handle the message in the attached state.
    - If the client is not attached, call [`client_dispatch_wait`](#client_dispatch_wait) to handle the message in the waiting state.
- **Output**:
    - The function does not return a value; it performs actions based on the message content and client state, potentially altering global state or terminating the process.
- **Functions called**:
    - [`client_dispatch_attached`](#client_dispatch_attached)
    - [`client_dispatch_wait`](#client_dispatch_wait)


---
### client_dispatch_exit_message<!-- {{#callable:client_dispatch_exit_message}} -->
The `client_dispatch_exit_message` function processes an exit message by extracting an exit value and an optional exit message from the provided data.
- **Inputs**:
    - `data`: A pointer to a character array containing the exit message data.
    - `datalen`: The length of the data array in bytes.
- **Control Flow**:
    - Check if the data length is less than the size of an integer and not zero, and if so, terminate the program with an error message.
    - If the data length is at least the size of an integer, copy the integer value from the data into the `retval` variable and assign it to `client_exitval`.
    - If the data length is greater than the size of an integer, adjust the data pointer and length to skip the integer, allocate memory for the exit message, copy the message into the allocated memory, null-terminate it, and set `client_exitreason` to `CLIENT_EXIT_MESSAGE_PROVIDED`.
- **Output**:
    - The function does not return a value but sets global variables `client_exitval`, `client_exitmessage`, and `client_exitreason` based on the processed data.


---
### client_dispatch_wait<!-- {{#callable:client_dispatch_wait}} -->
The `client_dispatch_wait` function processes incoming inter-process messages (imsg) while the client is in a waiting state, handling various message types to manage client-server interactions and state transitions.
- **Inputs**:
    - `imsg`: A pointer to an `imsg` structure representing the incoming inter-process message to be processed.
- **Control Flow**:
    - Check if the pledge has been applied; if not, apply it to restrict the process's capabilities and set `pledge_applied` to 1.
    - Extract the data and data length from the `imsg` structure.
    - Use a switch statement to handle different message types based on `imsg->hdr.type`.
    - For `MSG_EXIT` and `MSG_SHUTDOWN`, call [`client_dispatch_exit_message`](#client_dispatch_exit_message), set `client_exitflag` to 1, and call [`client_exit`](#client_exit).
    - For `MSG_READY`, verify the data length, set `client_attached` to 1, and send a `MSG_RESIZE` message to the server.
    - For `MSG_VERSION`, verify the data length, print a protocol version mismatch error, set `client_exitval` to 1, and exit the process.
    - For `MSG_FLAGS`, verify the data length, update `client_flags`, and log the new flags.
    - For `MSG_SHELL`, verify the data length and null-termination, then execute the shell command using [`client_exec`](#client_exec).
    - For `MSG_DETACH` and `MSG_DETACHKILL`, send a `MSG_EXITING` message to the server.
    - For `MSG_EXITED`, exit the process.
    - For `MSG_READ_OPEN`, `MSG_READ_CANCEL`, `MSG_WRITE_OPEN`, `MSG_WRITE`, and `MSG_WRITE_CLOSE`, call the respective file operation functions with appropriate parameters.
    - For `MSG_OLDSTDERR`, `MSG_OLDSTDIN`, and `MSG_OLDSTDOUT`, print an error message indicating the server version is too old and exit the process.
- **Output**:
    - The function does not return a value; it performs actions based on the message type, such as updating client state, sending messages, or exiting the process.
- **Functions called**:
    - [`client_dispatch_exit_message`](#client_dispatch_exit_message)
    - [`client_exit`](#client_exit)
    - [`client_exec`](#client_exec)


---
### client_dispatch_attached<!-- {{#callable:client_dispatch_attached}} -->
The `client_dispatch_attached` function processes various inter-process messages (imsg) received by a client in an attached state, executing actions based on the message type.
- **Inputs**:
    - `imsg`: A pointer to an `imsg` structure containing the message data and header information.
- **Control Flow**:
    - Extracts the data and data length from the `imsg` structure.
    - Uses a switch statement to handle different message types based on `imsg->hdr.type`.
    - For `MSG_FLAGS`, it verifies the data length and updates `client_flags`.
    - For `MSG_DETACH` and `MSG_DETACHKILL`, it checks the data validity, sets exit session and type, and sends an exit message.
    - For `MSG_EXEC`, it validates the data, sets execution command and shell, and sends an exit message.
    - For `MSG_EXIT`, it calls [`client_dispatch_exit_message`](#client_dispatch_exit_message) and sends an exit message if necessary.
    - For `MSG_EXITED`, it checks data length and calls `proc_exit`.
    - For `MSG_SHUTDOWN`, it verifies data length, sends an exit message, and sets exit reason and value.
    - For `MSG_SUSPEND`, it verifies data length, sets up a signal handler, marks the client as suspended, and sends a stop signal to itself.
    - For `MSG_LOCK`, it checks data validity, executes the lock command, and sends an unlock message.
- **Output**:
    - The function does not return a value; it performs actions such as updating global variables, sending messages, or executing system commands based on the message type.
- **Functions called**:
    - [`client_dispatch_exit_message`](#client_dispatch_exit_message)


# Function Declarations (Public API)

---
### client_exec<!-- {{#callable_declaration:client_exec}} -->
Executes a shell command using the specified shell.
- **Description**: Use this function to execute a command in a specified shell environment. It sets up the environment by setting the SHELL environment variable and ensures that standard input, output, and error streams are in blocking mode. This function is intended to be called when a client needs to execute a command directly in a shell, typically after other client operations have been completed. It does not return, as it replaces the current process image with a new process image using the specified shell and command.
- **Inputs**:
    - `shell`: A string representing the path to the shell executable. Must not be null. The caller is responsible for ensuring the shell path is valid and executable.
    - `shellcmd`: A string representing the command to be executed by the shell. Must not be null. The caller is responsible for ensuring the command is valid and appropriate for the specified shell.
- **Output**: None
- **See also**: [`client_exec`](#client_exec)  (Implementation)


---
### client_get_lock<!-- {{#callable_declaration:client_get_lock}} -->
Acquire a lock on a specified lock file.
- **Description**: This function attempts to acquire an exclusive lock on a specified lock file, which is used to coordinate server start operations among clients. It should be called when a client needs to ensure that no other client is starting the server simultaneously. If the lock is already held, the function will block until the lock is released and return -2, indicating that the caller should retry. If the lock cannot be acquired due to an error other than the lock being held, the function returns -1, allowing the caller to proceed with server start regardless. This function must be used with a valid lock file path and is intended for use in environments where file locking is supported.
- **Inputs**:
    - `lockfile`: A string representing the path to the lock file. It must not be null, and the caller retains ownership. The function will attempt to create the file if it does not exist. If the file cannot be opened, the function returns -1.
- **Output**: Returns a file descriptor for the lock file if the lock is successfully acquired. Returns -1 if the file cannot be opened or another error occurs. Returns -2 if the lock is held by another process, indicating the caller should retry.
- **See also**: [`client_get_lock`](#client_get_lock)  (Implementation)


---
### client_connect<!-- {{#callable_declaration:client_connect}} -->
Connects a client to a server using a UNIX domain socket.
- **Description**: This function attempts to establish a connection to a server using a UNIX domain socket specified by the path. It should be called when a client needs to connect to a server, potentially starting the server if it is not already running and the appropriate flags are set. The function handles retries and locking mechanisms to ensure that only one server is started. It is important to ensure that the path is valid and does not exceed the maximum length for UNIX domain socket paths. The function returns a file descriptor for the socket on success or -1 on failure, with errno set to indicate the error.
- **Inputs**:
    - `base`: A pointer to an event_base structure used for event handling. Must not be null.
    - `path`: A string representing the file system path to the UNIX domain socket. Must not be null and should not exceed the maximum path length for UNIX domain sockets.
    - `flags`: A 64-bit unsigned integer representing flags that control the behavior of the connection, such as whether to start the server if it is not running.
- **Output**: Returns a file descriptor for the connected socket on success, or -1 on failure with errno set to indicate the error.
- **See also**: [`client_connect`](#client_connect)  (Implementation)


---
### client_send_identify<!-- {{#callable_declaration:client_send_identify}} -->
Send identification information to the server.
- **Description**: This function is used to send various identification details from the client to the server, such as terminal names, capabilities, current working directory, and feature flags. It should be called when a client needs to establish its identity with the server. The function assumes that the provided strings are valid and non-null, and it duplicates file descriptors for standard input and output to send them to the server. It also sends the client's process ID and environment variables, handling any that exceed a maximum message size by skipping them.
- **Inputs**:
    - `ttynam`: The name of the terminal TTY. Must be a valid, non-null string.
    - `termname`: The name of the terminal. Must be a valid, non-null string.
    - `caps`: An array of strings representing terminal capabilities. Each string must be valid and non-null. The caller retains ownership of the array and its contents.
    - `ncaps`: The number of terminal capabilities in the 'caps' array. Must be a non-negative integer.
    - `cwd`: The current working directory. Must be a valid, non-null string.
    - `feat`: An integer representing feature flags. The specific meaning of each bit is determined by the application.
- **Output**: None
- **See also**: [`client_send_identify`](#client_send_identify)  (Implementation)


---
### client_signal<!-- {{#callable_declaration:client_signal}} -->
Handles client signals for process management and communication.
- **Description**: This function is designed to handle various signals received by a client process, managing child processes and communicating with a server or peer process as necessary. It should be used in a client application that needs to respond to signals such as SIGCHLD, SIGHUP, SIGTERM, SIGWINCH, and SIGCONT. The function ensures proper cleanup and communication with the server when the client is detached or terminated. It is important to ensure that the client process is correctly initialized and that signal handlers are set up to call this function.
- **Inputs**:
    - `sig`: The signal number to handle. Must be a valid signal such as SIGCHLD, SIGHUP, SIGTERM, SIGWINCH, or SIGCONT. Invalid signals may lead to undefined behavior.
- **Output**: None
- **See also**: [`client_signal`](#client_signal)  (Implementation)


---
### client_dispatch<!-- {{#callable_declaration:client_dispatch}} -->
Dispatches incoming messages to the appropriate client handler.
- **Description**: This function is used to handle incoming messages for a client, determining the appropriate action based on the client's current state. It should be called whenever a new message is received. If the message is null, indicating a lost connection, the function will set an exit reason and terminate the client process. If the client is attached, it dispatches the message to the attached handler; otherwise, it uses the wait handler. This function assumes that the client has been initialized and is in a valid state to process messages.
- **Inputs**:
    - `imsg`: A pointer to an imsg structure representing the incoming message. Must not be null unless indicating a lost connection.
    - `arg`: An unused parameter, typically set to null. It is ignored by the function.
- **Output**: None
- **See also**: [`client_dispatch`](#client_dispatch)  (Implementation)


---
### client_dispatch_attached<!-- {{#callable_declaration:client_dispatch_attached}} -->
Handles messages for an attached client session.
- **Description**: This function processes incoming messages for a client that is currently attached to a session. It handles various message types, such as flags updates, detachment requests, execution commands, and shutdown signals. The function is responsible for updating client state, sending appropriate responses, and managing session exits or suspensions. It should be called when a message is received for an attached client, ensuring that the message data is correctly formatted and the client is in an attached state.
- **Inputs**:
    - `imsg`: A pointer to an `imsg` structure containing the message data and header. The `imsg` must not be null, and its `hdr` field should contain a valid message type. The function expects the `data` field to be properly formatted according to the message type, and it will handle invalid data by terminating the process with an error.
- **Output**: None
- **See also**: [`client_dispatch_attached`](#client_dispatch_attached)  (Implementation)


---
### client_dispatch_wait<!-- {{#callable_declaration:client_dispatch_wait}} -->
Handles incoming messages from the server while in the wait state.
- **Description**: This function processes messages received from the server when the client is in a waiting state, before it is fully attached. It handles various message types, such as exit, shutdown, ready, version, flags, shell, detach, and file operations, and performs appropriate actions based on the message type. This function should be called whenever a new message is received from the server while the client is in the wait state. It ensures that the client transitions to the attached state when a MSG_READY message is received and manages resources and state changes accordingly.
- **Inputs**:
    - `imsg`: A pointer to an 'imsg' structure representing the incoming message from the server. Must not be null. The function expects the message to be well-formed and will handle specific message types as defined in the protocol.
- **Output**: None
- **See also**: [`client_dispatch_wait`](#client_dispatch_wait)  (Implementation)


---
### client_exit_message<!-- {{#callable_declaration:client_exit_message}} -->
Returns a message describing the client's exit reason.
- **Description**: This function provides a human-readable message that describes the reason for the client's exit. It should be called to obtain a textual explanation of why a client session ended. The function handles various exit scenarios, such as detachment, termination, or server exit, and returns a corresponding message. If a specific session is involved in the exit, the session name is included in the message. The function assumes that the exit reason has been set prior to its invocation.
- **Inputs**:
    - None
- **Output**: A constant character pointer to a string describing the client's exit reason.
- **See also**: [`client_exit_message`](#client_exit_message)  (Implementation)


