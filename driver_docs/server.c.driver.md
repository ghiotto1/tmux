# Purpose
This C source code file is part of the server-side implementation of the `tmux` terminal multiplexer. It provides the core server functionalities necessary for managing client connections, handling signals, and maintaining the state of sessions and windows. The file includes functions for creating and managing a server socket, handling client connections, and processing various server events. It also includes signal handlers for managing child processes and server shutdown procedures. The code is structured around event-driven programming, utilizing the `libevent` library to manage asynchronous events such as client connections and periodic tasks.

The file defines several key components, including structures for managing clients, sessions, and windows, as well as functions for setting and clearing marked panes within a session. It also includes mechanisms for logging messages and managing server state transitions, such as starting and stopping the server, and handling child process signals. The server is designed to run as a daemon, with the ability to fork and manage multiple client connections simultaneously. The code is intended to be part of a larger application, as indicated by the inclusion of the "tmux.h" header, which suggests that it is part of the `tmux` codebase. The file does not define a public API but rather implements internal server logic for the `tmux` application.
# Imports and Dependencies

---
- `sys/types.h`
- `sys/ioctl.h`
- `sys/socket.h`
- `sys/stat.h`
- `sys/un.h`
- `sys/wait.h`
- `errno.h`
- `fcntl.h`
- `signal.h`
- `stdio.h`
- `stdlib.h`
- `string.h`
- `termios.h`
- `time.h`
- `unistd.h`
- `tmux.h`


# Global Variables

---
### clients
- **Type**: `struct clients`
- **Description**: The `clients` variable is a global instance of the `struct clients` data structure. This structure is likely used to manage or store information about multiple client connections in the server application.
- **Use**: The `clients` variable is used to maintain a list or collection of client connections, allowing the server to manage and interact with connected clients.


---
### server_proc
- **Type**: `struct tmuxproc*`
- **Description**: The `server_proc` is a pointer to a `struct tmuxproc`, which is likely a data structure used to manage or represent a server process within the tmux application. This structure is part of the main server functions and is initialized when the server starts.
- **Use**: This variable is used to store and manage the server process information, allowing the server to perform operations such as signal handling and process management.


---
### server_fd
- **Type**: `int`
- **Description**: The `server_fd` is a static integer variable initialized to -1, which typically indicates an invalid or uninitialized file descriptor. It is used to store the file descriptor for the server socket in a server application.
- **Use**: This variable is used to manage the server socket's file descriptor, allowing the server to accept incoming client connections.


---
### server_client_flags
- **Type**: `uint64_t`
- **Description**: The `server_client_flags` is a global variable of type `uint64_t` used to store flags related to the server's client configuration. It is a 64-bit unsigned integer, allowing it to hold a large number of potential flag combinations.
- **Use**: This variable is used to store and manage various client-related flags that influence server behavior, such as socket creation and client handling.


---
### server_exit
- **Type**: `int`
- **Description**: The `server_exit` variable is a static integer that serves as a flag to indicate whether the server should exit its main loop and terminate its operations. It is used within the server's control flow to determine when to initiate the shutdown process.
- **Use**: This variable is used to control the server's main loop, allowing it to exit when set to a non-zero value, typically in response to signals like SIGINT or SIGTERM.


---
### server_ev_accept
- **Type**: `struct event`
- **Description**: The `server_ev_accept` is a static global variable of type `struct event`. It is used to manage and handle events related to accepting new connections on the server socket.
- **Use**: This variable is used to set up and manage the event loop for accepting incoming client connections on the server.


---
### server_ev_tidy
- **Type**: `struct event`
- **Description**: The `server_ev_tidy` is a static global variable of type `struct event`. It is used to manage a periodic event in the server, specifically for tidying up resources every hour.
- **Use**: This variable is used to schedule and manage the periodic execution of the `server_tidy_event` function, which performs cleanup tasks.


---
### marked_pane
- **Type**: `struct cmd_find_state`
- **Description**: The `marked_pane` is a global variable of type `struct cmd_find_state`. It is used to store the state of a marked pane within the tmux server, including references to the session, window link, window, and window pane that are currently marked.
- **Use**: This variable is used to track and manage the state of a marked pane, allowing operations to be performed on it, such as setting, clearing, and checking if a pane is marked.


---
### message_next
- **Type**: `u_int`
- **Description**: The `message_next` variable is a static unsigned integer that serves as a counter for message entries. It is used to assign a unique number to each message added to the `message_log`. This ensures that each message can be uniquely identified and managed within the system.
- **Use**: `message_next` is incremented each time a new message is added to the `message_log` to provide a unique identifier for the message.


---
### message_log
- **Type**: `struct message_list`
- **Description**: The `message_log` is a global variable of type `struct message_list`. It is used to maintain a list of messages, likely for logging or debugging purposes, within the server application.
- **Use**: This variable is used to store and manage a list of messages, with new messages being added to the tail of the list and old messages being removed when a limit is reached.


---
### current_time
- **Type**: `time_t`
- **Description**: The `current_time` variable is a global variable of type `time_t`, which is typically used to represent calendar time. It is used to store the current time as returned by the `time()` function.
- **Use**: This variable is updated in the `server_loop` function to hold the current time, likely for time-based operations or logging purposes.


# Functions

---
### server_set_marked<!-- {{#callable:server_set_marked}} -->
The `server_set_marked` function sets the global `marked_pane` state to the specified session, winlink, and window pane.
- **Inputs**:
    - `s`: A pointer to a `session` structure representing the session to be marked.
    - `wl`: A pointer to a `winlink` structure representing the winlink to be marked.
    - `wp`: A pointer to a `window_pane` structure representing the window pane to be marked.
- **Control Flow**:
    - The function begins by clearing the current state of `marked_pane` using `cmd_find_clear_state` with a flag value of 0.
    - It then assigns the provided session `s` to `marked_pane.s`.
    - The provided winlink `wl` is assigned to `marked_pane.wl`.
    - The window associated with the provided winlink `wl` is assigned to `marked_pane.w`.
    - Finally, the provided window pane `wp` is assigned to `marked_pane.wp`.
- **Output**:
    - The function does not return any value; it modifies the global `marked_pane` state.


---
### server_clear_marked<!-- {{#callable:server_clear_marked}} -->
The `server_clear_marked` function clears the state of the marked pane by resetting it using the `cmd_find_clear_state` function.
- **Inputs**:
    - None
- **Control Flow**:
    - The function calls `cmd_find_clear_state` with the `marked_pane` structure and a flag value of `0` to reset the state of the marked pane.
- **Output**:
    - The function does not return any value; it performs an action to clear the marked pane state.


---
### server_is_marked<!-- {{#callable:server_is_marked}} -->
The `server_is_marked` function checks if a given session, winlink, and window pane are the currently marked pane.
- **Inputs**:
    - `s`: A pointer to a `session` structure representing the session to check.
    - `wl`: A pointer to a `winlink` structure representing the winlink to check.
    - `wp`: A pointer to a `window_pane` structure representing the window pane to check.
- **Control Flow**:
    - Check if any of the input pointers (`s`, `wl`, `wp`) are NULL; if so, return 0.
    - Compare the input session `s` with the session in `marked_pane`; if they are not equal, return 0.
    - Compare the input winlink `wl` with the winlink in `marked_pane`; if they are not equal, return 0.
    - Compare the input window pane `wp` with the window pane in `marked_pane`; if they are not equal, return 0.
    - Call `server_check_marked()` to verify if the marked pane is still valid and return its result.
- **Output**:
    - Returns 1 if the given session, winlink, and window pane match the marked pane and it is valid; otherwise, returns 0.
- **Functions called**:
    - [`server_check_marked`](#server_check_marked)


---
### server_check_marked<!-- {{#callable:server_check_marked}} -->
The `server_check_marked` function checks if the currently marked pane is still valid.
- **Inputs**:
    - None
- **Control Flow**:
    - The function calls `cmd_find_valid_state` with a reference to `marked_pane`.
    - It returns the result of the `cmd_find_valid_state` function call.
- **Output**:
    - The function returns an integer indicating whether the marked pane is valid (non-zero) or not (zero).


---
### server_create_socket<!-- {{#callable:server_create_socket}} -->
The `server_create_socket` function creates a UNIX domain socket, binds it to a specified path, and sets it to listen for incoming connections, handling errors and setting appropriate permissions based on flags.
- **Inputs**:
    - `flags`: A 64-bit unsigned integer that specifies options for socket creation, such as whether to use default socket permissions.
    - `cause`: A pointer to a character pointer where an error message will be stored if socket creation fails.
- **Control Flow**:
    - Initialize a `sockaddr_un` structure and set its family to `AF_UNIX`.
    - Copy the socket path to `sa.sun_path` and check for path length errors.
    - Unlink any existing socket file at the path to avoid conflicts.
    - Create a UNIX domain socket with `socket(AF_UNIX, SOCK_STREAM, 0)` and handle errors.
    - Set the umask based on the `flags` to control socket file permissions.
    - Bind the socket to the address specified in `sa` and handle errors, restoring the umask afterwards.
    - Set the socket to listen for incoming connections with a backlog of 128 and handle errors.
    - Set the socket to non-blocking mode using `setblocking(fd, 0)`.
    - Return the file descriptor of the created socket on success.
    - On failure, if `cause` is not NULL, format an error message and store it in `cause`, then return -1.
- **Output**:
    - Returns the file descriptor of the created socket on success, or -1 on failure, with an optional error message stored in `cause`.


---
### server_tidy_event<!-- {{#callable:server_tidy_event}} -->
The `server_tidy_event` function performs periodic maintenance tasks such as tidying up jobs and optionally trimming memory, then schedules itself to run again in one hour.
- **Inputs**:
    - `fd`: An unused integer representing a file descriptor.
    - `events`: An unused short integer representing event flags.
    - `data`: An unused pointer to additional data.
- **Control Flow**:
    - Initialize a `struct timeval` with a 1-hour interval.
    - Record the current time using `get_timer()`.
    - Call `format_tidy_jobs()` to perform job tidying operations.
    - If the `HAVE_MALLOC_TRIM` macro is defined, call `malloc_trim(0)` to release free memory back to the system.
    - Log the time taken for the tidy operation using `log_debug()`.
    - Schedule the `server_tidy_event` to run again in one hour using `evtimer_add()`.
- **Output**:
    - The function does not return any value; it performs maintenance tasks and schedules itself to run again.


---
### server_start<!-- {{#callable:server_start}} -->
The `server_start` function initializes and starts a server process, handling signal blocking, event reinitialization, socket creation, and client management.
- **Inputs**:
    - `client`: A pointer to a `tmuxproc` structure representing the client process.
    - `flags`: A 64-bit unsigned integer representing various flags for server configuration.
    - `base`: A pointer to an `event_base` structure used for event management.
    - `lockfd`: An integer file descriptor for a lock file, used to ensure single instance execution.
    - `lockfile`: A string representing the path to the lock file.
- **Control Flow**:
    - Block all signals using `sigfillset` and `sigprocmask` to ensure a clean environment for server initialization.
    - Check if the `CLIENT_NOFORK` flag is not set; if not, fork the process and daemonize it using `proc_fork_and_daemon`.
    - Clear any signals set on the client process using `proc_clear_signals`.
    - Reinitialize the event base with `event_reinit` and start the server process using `proc_start`.
    - Set up signal handlers for the server process using `proc_set_signals`.
    - Restore the original signal mask using `sigprocmask`.
    - Initialize various server components such as key bindings, UTF-8 width cache, and data structures for windows, clients, and sessions.
    - Create a server socket using either `systemd_create_socket` or [`server_create_socket`](#server_create_socket) based on the system configuration.
    - If socket creation is successful, update the socket permissions with [`server_update_socket`](#server_update_socket).
    - If the `CLIENT_NOFORK` flag is not set, create a server client using `server_client_create`; otherwise, set the `exit-empty` option to 0.
    - If a lock file descriptor is provided, unlink and free the lock file, then close the descriptor.
    - If there is an error cause, handle it by setting the client's exit message or printing the error and exiting.
    - Set up a periodic tidy event using `evtimer_set` and `evtimer_add` to clean up resources every hour.
    - Initialize access control lists with `server_acl_init`.
    - Add an accept event for incoming connections using [`server_add_accept`](#server_add_accept).
    - Enter the server's main processing loop with `proc_loop`, which handles client requests and server operations.
    - Clean up by killing all jobs and saving the status prompt history before exiting.
- **Output**:
    - The function does not return a value; it exits the process after initializing and starting the server.
- **Functions called**:
    - [`server_create_socket`](#server_create_socket)
    - [`server_update_socket`](#server_update_socket)
    - [`server_add_accept`](#server_add_accept)


---
### server_loop<!-- {{#callable:server_loop}} -->
The `server_loop` function manages the main event loop of a server, processing client commands and determining when the server should exit.
- **Inputs**:
    - None
- **Control Flow**:
    - Initialize `current_time` with the current time.
    - Enter a do-while loop that continues as long as there are items to process.
    - Call `cmdq_next` for each client that is identified, accumulating the number of items processed.
    - After processing all items, call `server_client_loop` to handle client-specific operations.
    - Check if the server should exit based on the `exit-empty` and `exit-unattached` options and the presence of active sessions or attached clients.
    - If no clients are attached and no jobs are running, return 1 to indicate the server should exit.
    - Otherwise, return 0 to continue running.
- **Output**:
    - Returns 1 if the server should exit, otherwise returns 0 to continue running.


---
### server_send_exit<!-- {{#callable:server_send_exit}} -->
The `server_send_exit` function initiates the shutdown process for a server by marking all clients for exit and destroying all sessions.
- **Inputs**:
    - None
- **Control Flow**:
    - Call `cmd_wait_for_flush()` to ensure any pending commands are processed before proceeding.
    - Iterate over all clients using `TAILQ_FOREACH_SAFE` to safely traverse and modify the client list.
    - For each client, check if it is suspended using the `CLIENT_SUSPENDED` flag; if so, call `server_client_lost(c)` to handle the lost client.
    - If the client is not suspended, set the `CLIENT_EXIT` flag and mark the exit type as `CLIENT_EXIT_SHUTDOWN`.
    - Set the client's session to `NULL` to detach it from any session.
    - Iterate over all sessions using `RB_FOREACH_SAFE` to safely traverse and modify the session tree.
    - For each session, call `session_destroy(s, 1, __func__)` to destroy the session and clean up resources.
- **Output**:
    - The function does not return any value; it performs actions to prepare the server for shutdown by marking clients for exit and destroying sessions.


---
### server_update_socket<!-- {{#callable:server_update_socket}} -->
The `server_update_socket` function updates the execute permissions of a server socket based on whether any sessions are currently attached.
- **Inputs**:
    - None
- **Control Flow**:
    - Initialize a static variable `last` to -1 and local variables `n` and `mode`.
    - Iterate over all sessions using `RB_FOREACH` to check if any session is attached; if found, increment `n` and break the loop.
    - If the current number of attached sessions `n` is different from `last`, update `last` to `n`.
    - Use `stat` to get the current permissions of the socket file at `socket_path`; if it fails, return immediately.
    - Extract the current permissions of the socket file into `mode`.
    - If there are attached sessions (`n != 0`), add execute permissions for user, group, and others if read permissions are present.
    - If there are no attached sessions (`n == 0`), remove execute permissions for user, group, and others.
    - Apply the updated permissions to the socket file using `chmod`.
- **Output**:
    - The function does not return any value; it modifies the file permissions of the socket file at `socket_path`.


---
### server_accept<!-- {{#callable:server_accept}} -->
The `server_accept` function handles incoming client connections on a server socket, creating a new client instance for each accepted connection and managing access control.
- **Inputs**:
    - `fd`: The file descriptor of the server socket that is ready to accept a new connection.
    - `events`: A bitmask of event flags indicating the type of event that occurred, specifically checking for read events.
    - `data`: Unused parameter, typically used for passing additional data to the callback function.
- **Control Flow**:
    - The function begins by calling `server_add_accept(0)` to set up the server to accept new connections.
    - It checks if the event is a read event using the `EV_READ` flag; if not, it returns immediately.
    - The function attempts to accept a new connection using the `accept` system call, storing the new file descriptor in `newfd`.
    - If `accept` fails, it checks for specific error conditions (e.g., `EAGAIN`, `EINTR`, `ECONNABORTED`, `ENFILE`, `EMFILE`) and handles them accordingly, either by returning or by setting a timeout for the next accept attempt.
    - If the server is set to exit (`server_exit` is true), it closes the new file descriptor and returns.
    - A new client is created using `server_client_create(newfd)`.
    - The function checks if the client is allowed to join using `server_acl_join(c)`; if not, it sets an exit message and flags the client for exit.
- **Output**:
    - The function does not return a value; it manages client connections and modifies server state as needed.
- **Functions called**:
    - [`server_add_accept`](#server_add_accept)


---
### server_add_accept<!-- {{#callable:server_add_accept}} -->
The `server_add_accept` function sets up an event to accept incoming connections on a server socket, with an optional timeout for backoff when file descriptors are low.
- **Inputs**:
    - `timeout`: An integer representing the timeout duration in seconds; if zero, the event is set for reading, otherwise for timeout.
- **Control Flow**:
    - Initialize a `timeval` structure with the given timeout value.
    - Check if `server_fd` is valid (not -1); if invalid, exit the function.
    - If the `server_ev_accept` event is already initialized, delete it to prevent duplicate events.
    - If the timeout is zero, set the event to trigger on read events (`EV_READ`) and add it without a timeout.
    - If the timeout is non-zero, set the event to trigger on timeout events (`EV_TIMEOUT`) and add it with the specified timeout.
- **Output**:
    - The function does not return a value; it sets up an event for accepting connections on the server socket.


---
### server_signal<!-- {{#callable:server_signal}} -->
The `server_signal` function handles various signals received by the server, executing specific actions based on the signal type.
- **Inputs**:
    - `sig`: An integer representing the signal number that the server has received.
- **Control Flow**:
    - Log the received signal using `log_debug` and `strsignal` to convert the signal number to a human-readable string.
    - Use a switch statement to handle different signals:
    - For `SIGINT` and `SIGTERM`, set `server_exit` to 1 and call [`server_send_exit`](#server_send_exit) to initiate server shutdown.
    - For `SIGCHLD`, call [`server_child_signal`](#server_child_signal) to handle child process signals.
    - For `SIGUSR1`, delete the current accept event, create a new server socket, close the old socket, update the server socket, and add a new accept event.
    - For `SIGUSR2`, toggle the logging state of the server process using `proc_toggle_log`.
- **Output**:
    - The function does not return a value; it performs actions based on the received signal.
- **Functions called**:
    - [`server_send_exit`](#server_send_exit)
    - [`server_child_signal`](#server_child_signal)
    - [`server_create_socket`](#server_create_socket)
    - [`server_update_socket`](#server_update_socket)
    - [`server_add_accept`](#server_add_accept)


---
### server_child_signal<!-- {{#callable:server_child_signal}} -->
The `server_child_signal` function handles the SIGCHLD signal by checking the status of child processes and invoking appropriate handlers for stopped or exited children.
- **Inputs**:
    - None
- **Control Flow**:
    - The function enters an infinite loop to continuously check for child process status changes.
    - It calls `waitpid` with `WAIT_ANY`, `&status`, and `WNOHANG|WUNTRACED` to non-blockingly check for any child process status changes.
    - If `waitpid` returns -1 and `errno` is `ECHILD`, it means there are no child processes to wait for, so the function returns.
    - If `waitpid` returns 0, it means there are no child processes that have changed state, so the function returns.
    - If a child process is stopped (`WIFSTOPPED(status)`), it calls [`server_child_stopped`](#server_child_stopped) with the child's PID and status.
    - If a child process has exited or was terminated by a signal (`WIFEXITED(status)` or `WIFSIGNALED(status)`), it calls [`server_child_exited`](#server_child_exited) with the child's PID and status.
- **Output**:
    - The function does not return any value; it performs actions based on the status of child processes.
- **Functions called**:
    - [`server_child_stopped`](#server_child_stopped)
    - [`server_child_exited`](#server_child_exited)


---
### server_child_exited<!-- {{#callable:server_child_exited}} -->
The `server_child_exited` function handles the cleanup and status update of a window pane when a child process exits.
- **Inputs**:
    - `pid`: The process ID of the child process that has exited.
    - `status`: The exit status of the child process.
- **Control Flow**:
    - Iterate over all windows using `RB_FOREACH_SAFE` to safely traverse the red-black tree of windows.
    - For each window, iterate over its panes using `TAILQ_FOREACH`.
    - Check if the pane's process ID matches the given `pid`.
    - If a match is found, update the pane's status with the given `status` and set the `PANE_STATUSREADY` flag.
    - Log a debug message indicating the pane has exited and set the `PANE_EXITED` flag.
    - Check if the pane is ready to be destroyed using `window_pane_destroy_ready` and if so, call `server_destroy_pane` to destroy it.
    - Break out of the loop once the matching pane is found and processed.
    - Call `job_check_died` to handle any additional cleanup or checks for the exited job.
- **Output**:
    - The function does not return a value; it performs updates and cleanup on the window pane associated with the exited child process.


---
### server_child_stopped<!-- {{#callable:server_child_stopped}} -->
The `server_child_stopped` function handles the event when a child process is stopped, specifically resuming the process if it was stopped by signals other than SIGTTIN or SIGTTOU.
- **Inputs**:
    - `pid`: The process ID of the child process that has stopped.
    - `status`: The status code returned by the stopped child process, which includes information about the signal that stopped the process.
- **Control Flow**:
    - Check if the stopping signal is SIGTTIN or SIGTTOU; if so, return immediately without further action.
    - Iterate over all windows and their panes to find the pane with a matching process ID (pid).
    - If a matching pane is found, attempt to continue the process group of the stopped process using `killpg`; if that fails, use `kill` to send SIGCONT to the process.
    - Call `job_check_died` to handle any additional cleanup or checks for the stopped process.
- **Output**:
    - The function does not return a value; it performs actions to resume a stopped child process and checks for any job-related cleanup.


---
### server_add_message<!-- {{#callable:server_add_message}} -->
The `server_add_message` function formats a message, logs it, and manages a message log with a specified limit.
- **Inputs**:
    - `fmt`: A format string for the message to be added.
    - `...`: A variable number of arguments to be formatted according to the format string.
- **Control Flow**:
    - Initialize a variable argument list and format the message string using `xvasprintf`.
    - Log the formatted message using `log_debug`.
    - Allocate memory for a new `message_entry` structure and initialize it with the current time, a unique message number, and the formatted message string.
    - Insert the new message entry at the tail of the `message_log` queue.
    - Retrieve the message limit from global options using `options_get_number`.
    - Iterate over the `message_log` queue to remove and free messages that exceed the message limit, ensuring the log does not grow beyond the specified limit.
- **Output**:
    - The function does not return a value; it modifies the global `message_log` by adding a new message and potentially removing old ones.


# Function Declarations (Public API)

---
### server_loop<!-- {{#callable_declaration:server_loop}} -->
Runs the main server loop and determines if the server should exit.
- **Description**: This function executes the main loop for the server, processing client commands and managing server state. It should be called to maintain server operations, handling client interactions and checking conditions for server shutdown. The function evaluates whether the server should continue running based on client activity and configuration options. It returns a status indicating whether the server should exit, considering factors like active sessions, attached clients, and running jobs.
- **Inputs**:
    - None
- **Output**: Returns 1 if the server should exit, otherwise returns 0.
- **See also**: [`server_loop`](#server_loop)  (Implementation)


---
### server_send_exit<!-- {{#callable_declaration:server_send_exit}} -->
Initiates server shutdown by marking clients for exit and destroying sessions.
- **Description**: This function is used to initiate the shutdown process of the server by marking all connected clients for exit and destroying all active sessions. It should be called when the server needs to be gracefully terminated, ensuring that all clients are properly notified and all sessions are cleaned up. This function does not take any parameters and does not return a value. It is expected to be called in a context where the server is ready to shut down, and it handles both suspended and active clients appropriately.
- **Inputs**:
    - None
- **Output**: None
- **See also**: [`server_send_exit`](#server_send_exit)  (Implementation)


---
### server_accept<!-- {{#callable_declaration:server_accept}} -->
Handles incoming client connections on a server socket.
- **Description**: This function is used to accept new client connections on a server socket when there is data to be read. It should be called in response to an event indicating that the server socket is ready for reading. The function will attempt to accept a new connection and create a client object for it. If the server is in the process of exiting, the new connection will be immediately closed. The function handles various error conditions, such as resource exhaustion, by backing off and retrying later. It is important to ensure that the server is not in an exit state before calling this function.
- **Inputs**:
    - `fd`: The file descriptor of the server socket. It must be a valid, open socket descriptor.
    - `events`: A bitmask of event flags indicating the type of event that occurred. Must include EV_READ to proceed with accepting a connection.
    - `data`: Unused parameter. Can be NULL or any value, as it is not utilized in the function.
- **Output**: None
- **See also**: [`server_accept`](#server_accept)  (Implementation)


---
### server_signal<!-- {{#callable_declaration:server_signal}} -->
Handle server signals and perform appropriate actions.
- **Description**: This function is designed to handle various signals received by the server and execute corresponding actions. It should be used within a signal handling context to manage server behavior in response to signals such as SIGINT, SIGTERM, SIGCHLD, SIGUSR1, and SIGUSR2. The function modifies server state, manages child processes, and updates server socket configurations as needed. It is crucial to ensure that the server is properly initialized and running before invoking this function, as it relies on server-specific global variables and states.
- **Inputs**:
    - `sig`: The signal number to handle. Must be a valid signal such as SIGINT, SIGTERM, SIGCHLD, SIGUSR1, or SIGUSR2. The function does not handle invalid signal numbers explicitly.
- **Output**: None
- **See also**: [`server_signal`](#server_signal)  (Implementation)


---
### server_child_signal<!-- {{#callable_declaration:server_child_signal}} -->
Handle termination and state changes of child processes.
- **Description**: This function is used to handle signals related to child processes, specifically SIGCHLD, which indicates that a child process has stopped or terminated. It should be called in response to a SIGCHLD signal to ensure that the server properly manages child processes, cleaning up resources and handling state changes as necessary. The function will repeatedly call waitpid to check the status of child processes, and it will handle stopped or exited processes by invoking appropriate handlers. It is important to ensure that this function is called in a context where it can safely handle these signals, typically within a signal handler or a similar asynchronous context.
- **Inputs**:
    - None
- **Output**: None
- **See also**: [`server_child_signal`](#server_child_signal)  (Implementation)


---
### server_child_exited<!-- {{#callable_declaration:server_child_exited}} -->
Handles the termination of a child process.
- **Description**: This function is used to handle the event when a child process exits. It updates the status of the associated window pane, marks it as exited, and checks if the pane is ready to be destroyed. If so, it proceeds to destroy the pane. This function should be called in response to a SIGCHLD signal when a child process has terminated. It ensures that resources associated with the terminated process are properly cleaned up and that any necessary logging is performed.
- **Inputs**:
    - `pid`: The process ID of the child process that has exited. It must be a valid process ID that corresponds to a child process managed by the server.
    - `status`: The exit status of the child process. This is typically obtained from a waitpid call and contains information about how the process terminated.
- **Output**: None
- **See also**: [`server_child_exited`](#server_child_exited)  (Implementation)


---
### server_child_stopped<!-- {{#callable_declaration:server_child_stopped}} -->
Handles a stopped child process by resuming it if necessary.
- **Description**: This function is used to handle a child process that has stopped, typically due to a signal. It checks if the stop signal is SIGTTIN or SIGTTOU, in which case it does nothing. For other stop signals, it attempts to resume the process by sending a SIGCONT signal to the process group. This function should be called when a child process is detected to have stopped, and it ensures that the process can continue running if it was stopped for reasons other than terminal input/output signals.
- **Inputs**:
    - `pid`: The process ID of the stopped child process. It must be a valid process ID of a child process that the caller is managing.
    - `status`: The status code returned by the waitpid function, which includes information about the stop signal. It must be a valid status code indicating that the process has stopped.
- **Output**: None
- **See also**: [`server_child_stopped`](#server_child_stopped)  (Implementation)


