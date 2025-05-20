# Purpose
This C source code file is part of the server-side implementation for the `tmux` terminal multiplexer. It provides the core server functionalities necessary for managing client connections, handling signals, and maintaining the state of sessions and windows. The file includes functions for creating and managing a server socket, handling client connections, and processing server events. It also manages the lifecycle of child processes and handles signals such as `SIGCHLD` for child process termination and `SIGUSR1` for socket recreation. The server is designed to run in a loop, processing commands and managing client interactions, and it can operate in both foreground and daemon modes.

The code defines several key structures and functions that are central to the server's operation. These include the [`server_start`](#server_start) function, which initializes the server, sets up signal handlers, and enters the main event loop. The [`server_loop`](#server_loop) function is responsible for processing commands and managing client sessions. The file also includes mechanisms for marking and clearing panes, managing message logs, and handling server exit conditions. The use of UNIX domain sockets for client-server communication and the integration with system signals highlight the server's role in managing multiple client connections and ensuring robust session management. Overall, this file is a critical component of the `tmux` server, providing the necessary infrastructure to support its multi-client, multi-session capabilities.
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
- **Description**: The `clients` variable is a global instance of the `struct clients` data structure. This structure is likely used to manage and store information about connected clients in the server application. The exact fields and details of the `struct clients` are not provided in the given code, but it is typically used to maintain a list or collection of client connections.
- **Use**: The `clients` variable is used to keep track of all connected clients in the server, allowing the server to manage client connections and interactions.


---
### server_proc
- **Type**: `struct tmuxproc*`
- **Description**: The `server_proc` is a pointer to a `struct tmuxproc`, which represents a process structure used within the tmux server. This structure is likely used to manage and control the server process, including handling signals and managing the event loop.
- **Use**: This variable is used to store and manage the server process within the tmux application, facilitating process control and event handling.


---
### server_fd
- **Type**: `int`
- **Description**: The `server_fd` is a static integer variable initialized to -1, which typically indicates an invalid or uninitialized file descriptor. It is used to store the file descriptor for the server socket in a server application.
- **Use**: This variable is used to manage the server socket's file descriptor, allowing the server to accept incoming client connections.


---
### server_client_flags
- **Type**: `uint64_t`
- **Description**: The `server_client_flags` is a global variable of type `uint64_t` used to store flags related to the server's client operations. It is a 64-bit unsigned integer, allowing it to hold a large number of potential flag combinations.
- **Use**: This variable is used to store and manage various client-related flags within the server, influencing behavior such as socket creation and client handling.


---
### server_exit
- **Type**: `int`
- **Description**: The `server_exit` variable is a static integer that acts as a flag to indicate whether the server should exit its main loop and terminate. It is set to 1 when a termination signal (such as SIGINT or SIGTERM) is received, prompting the server to initiate its shutdown process.
- **Use**: This variable is used within the server loop to determine if the server should continue running or begin the shutdown process.


---
### server_ev_accept
- **Type**: `struct event`
- **Description**: The `server_ev_accept` is a static global variable of type `struct event`. It is used to manage and handle events related to accepting new connections on the server socket.
- **Use**: This variable is used to set up and manage the event for accepting new client connections in the server's event loop.


---
### server_ev_tidy
- **Type**: `struct event`
- **Description**: The `server_ev_tidy` is a static global variable of type `struct event`. It is used to manage a periodic event in the server, specifically for tidying up resources every hour.
- **Use**: This variable is used to schedule and manage the execution of the `server_tidy_event` function, which performs cleanup tasks at regular intervals.


---
### marked_pane
- **Type**: `struct cmd_find_state`
- **Description**: The `marked_pane` is a global variable of type `struct cmd_find_state`. It is used to store the state of a marked pane within the tmux server, including references to the session, window link, window, and window pane that are currently marked.
- **Use**: This variable is used to track and manage the marked pane in the tmux server, allowing operations to be performed on the marked pane.


---
### message_next
- **Type**: `u_int`
- **Description**: The `message_next` variable is a static unsigned integer that serves as a counter for message entries. It is used to assign a unique number to each message added to the message log.
- **Use**: This variable is incremented each time a new message is added to the `message_log` to ensure each message has a unique identifier.


---
### message_log
- **Type**: `struct message_list`
- **Description**: The `message_log` is a global variable of type `struct message_list`. It is used to maintain a list of messages that are logged by the server. This structure is likely a linked list or a similar data structure that allows for dynamic addition and removal of messages.
- **Use**: The `message_log` is used to store and manage server messages, allowing for logging and retrieval of messages as needed.


---
### current_time
- **Type**: `time_t`
- **Description**: The `current_time` variable is a global variable of type `time_t`, which is typically used to represent calendar time. It is used to store the current time as returned by the `time()` function.
- **Use**: This variable is updated in the `server_loop` function to keep track of the current time during the server's operation.


# Functions

---
### server_set_marked<!-- {{#callable:server_set_marked}} -->
The `server_set_marked` function sets the global `marked_pane` state to the specified session, winlink, and window pane.
- **Inputs**:
    - `s`: A pointer to a `session` structure representing the session to be marked.
    - `wl`: A pointer to a `winlink` structure representing the winlink to be marked.
    - `wp`: A pointer to a `window_pane` structure representing the window pane to be marked.
- **Control Flow**:
    - The function begins by clearing the current state of `marked_pane` using [`cmd_find_clear_state`](cmd-find.c.driver.md#cmd_find_clear_state) with a flag value of 0.
    - It then assigns the provided session `s` to `marked_pane.s`.
    - The provided winlink `wl` is assigned to `marked_pane.wl`.
    - The window associated with the provided winlink `wl` is assigned to `marked_pane.w`.
    - Finally, the provided window pane `wp` is assigned to `marked_pane.wp`.
- **Output**:
    - The function does not return a value; it modifies the global `marked_pane` state.
- **Functions called**:
    - [`cmd_find_clear_state`](cmd-find.c.driver.md#cmd_find_clear_state)


---
### server_clear_marked<!-- {{#callable:server_clear_marked}} -->
The `server_clear_marked` function clears the state of the marked pane by resetting it using the [`cmd_find_clear_state`](cmd-find.c.driver.md#cmd_find_clear_state) function.
- **Inputs**:
    - None
- **Control Flow**:
    - The function calls [`cmd_find_clear_state`](cmd-find.c.driver.md#cmd_find_clear_state) with the `marked_pane` structure and a flag value of `0` to reset the state of the marked pane.
- **Output**:
    - The function does not return any value; it performs an action to clear the marked pane state.
- **Functions called**:
    - [`cmd_find_clear_state`](cmd-find.c.driver.md#cmd_find_clear_state)


---
### server_is_marked<!-- {{#callable:server_is_marked}} -->
The `server_is_marked` function checks if a given session, winlink, and window pane are the currently marked pane in the server.
- **Inputs**:
    - `s`: A pointer to a `struct session` representing the session to check.
    - `wl`: A pointer to a `struct winlink` representing the winlink to check.
    - `wp`: A pointer to a `struct window_pane` representing the window pane to check.
- **Control Flow**:
    - Check if any of the input pointers (`s`, `wl`, `wp`) are `NULL`; if so, return `0` indicating the pane is not marked.
    - Compare the input session `s` with the session in `marked_pane`; if they do not match, return `0`.
    - Compare the input winlink `wl` with the winlink in `marked_pane`; if they do not match, return `0`.
    - Compare the input window pane `wp` with the window pane in `marked_pane`; if they do not match, return `0`.
    - Call `server_check_marked()` to verify if the marked pane is still valid and return its result.
- **Output**:
    - Returns an integer `1` if the given session, winlink, and window pane are the marked pane and it is valid, otherwise returns `0`.
- **Functions called**:
    - [`server_check_marked`](#server_check_marked)


---
### server_check_marked<!-- {{#callable:server_check_marked}} -->
The `server_check_marked` function checks if the currently marked pane is still valid.
- **Inputs**:
    - None
- **Control Flow**:
    - The function calls [`cmd_find_valid_state`](cmd-find.c.driver.md#cmd_find_valid_state) with the `marked_pane` structure as an argument.
    - It returns the result of the [`cmd_find_valid_state`](cmd-find.c.driver.md#cmd_find_valid_state) function call.
- **Output**:
    - The function returns an integer value indicating whether the marked pane is valid or not.
- **Functions called**:
    - [`cmd_find_valid_state`](cmd-find.c.driver.md#cmd_find_valid_state)


---
### server_create_socket<!-- {{#callable:server_create_socket}} -->
The `server_create_socket` function creates a UNIX domain socket, binds it to a specified path, and sets it to listen for incoming connections, handling errors and setting appropriate permissions based on flags.
- **Inputs**:
    - `flags`: A 64-bit unsigned integer that specifies options for socket creation, such as whether to use default socket permissions.
    - `cause`: A pointer to a character pointer where an error message can be stored if socket creation fails.
- **Control Flow**:
    - Initialize a `sockaddr_un` structure and set its family to `AF_UNIX`.
    - Copy the socket path to `sa.sun_path` and check for path length errors.
    - Unlink any existing socket file at the path to avoid conflicts.
    - Create a new UNIX domain socket with `socket()` and handle errors.
    - Set the umask based on the `flags` to control socket file permissions.
    - Bind the socket to the address specified in `sa` and handle errors.
    - Restore the previous umask value after binding.
    - Set the socket to listen for incoming connections with a backlog of 128 and handle errors.
    - Set the socket to non-blocking mode using `setblocking()`.
    - Return the file descriptor of the created socket on success.
    - On failure, format an error message into `cause` if it is not `NULL` and return `-1`.
- **Output**:
    - Returns the file descriptor of the created socket on success, or `-1` on failure, with an optional error message stored in `cause`.
- **Functions called**:
    - [`strlcpy`](compat/strlcpy.c.driver.md#strlcpy)
    - [`setblocking`](tmux.c.driver.md#setblocking)
    - [`xasprintf`](xmalloc.c.driver.md#xasprintf)


---
### server_tidy_event<!-- {{#callable:server_tidy_event}} -->
The `server_tidy_event` function performs periodic maintenance tasks such as tidying up jobs and optionally trimming memory, and then schedules itself to run again in one hour.
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
    - The function does not return any value; it performs its operations and schedules itself to run again.
- **Functions called**:
    - [`get_timer`](tmux.c.driver.md#get_timer)
    - [`format_tidy_jobs`](format.c.driver.md#format_tidy_jobs)


---
### server_start<!-- {{#callable:server_start}} -->
The `server_start` function initializes and starts a server process, handling signal blocking, event reinitialization, socket creation, and client management.
- **Inputs**:
    - `client`: A pointer to a `tmuxproc` structure representing the client process.
    - `flags`: A 64-bit unsigned integer representing various flags that control server behavior.
    - `base`: A pointer to an `event_base` structure used for event management.
    - `lockfd`: An integer file descriptor for a lock file, used to ensure single instance execution.
    - `lockfile`: A string representing the path to the lock file.
- **Control Flow**:
    - Block all signals using `sigfillset` and `sigprocmask` to ensure a clean environment for server initialization.
    - Check if the `CLIENT_NOFORK` flag is not set; if not, fork the process and daemonize it using [`proc_fork_and_daemon`](proc.c.driver.md#proc_fork_and_daemon).
    - Clear any signals for the client process using [`proc_clear_signals`](proc.c.driver.md#proc_clear_signals).
    - Reinitialize the event base with `event_reinit` and start the server process using [`proc_start`](proc.c.driver.md#proc_start).
    - Set up signal handlers for the server process using [`proc_set_signals`](proc.c.driver.md#proc_set_signals).
    - Restore the original signal mask using `sigprocmask`.
    - Initialize various server components such as key bindings, UTF-8 width cache, and data structures for windows, clients, and sessions.
    - Create a server socket using either [`systemd_create_socket`](compat/systemd.c.driver.md#systemd_create_socket) or [`server_create_socket`](#server_create_socket) based on the presence of systemd.
    - If the server socket is successfully created, update its permissions with [`server_update_socket`](#server_update_socket).
    - Create a client if not in no-fork mode, otherwise set the `exit-empty` option to 0.
    - Handle the lock file by unlinking, freeing, and closing it if `lockfd` is valid.
    - If there is an error cause, set the client's exit message or print it to stderr and exit.
    - Set up a periodic tidy event using `evtimer_set` and `evtimer_add`.
    - Initialize access control lists with [`server_acl_init`](server-acl.c.driver.md#server_acl_init).
    - Add an accept event for incoming connections using [`server_add_accept`](#server_add_accept).
    - Enter the server's main processing loop with [`proc_loop`](proc.c.driver.md#proc_loop).
    - Clean up by killing all jobs and saving the status prompt history before exiting.
- **Output**:
    - The function does not return a value; it exits the process with `exit(0)` after starting the server and entering the main loop.
- **Functions called**:
    - [`proc_fork_and_daemon`](proc.c.driver.md#proc_fork_and_daemon)
    - [`proc_clear_signals`](proc.c.driver.md#proc_clear_signals)
    - [`proc_start`](proc.c.driver.md#proc_start)
    - [`proc_set_signals`](proc.c.driver.md#proc_set_signals)
    - [`log_get_level`](log.c.driver.md#log_get_level)
    - [`tty_create_log`](tty.c.driver.md#tty_create_log)
    - [`input_key_build`](input-keys.c.driver.md#input_key_build)
    - [`utf8_update_width_cache`](utf8.c.driver.md#utf8_update_width_cache)
    - [`key_bindings_init`](key-bindings.c.driver.md#key_bindings_init)
    - [`systemd_create_socket`](compat/systemd.c.driver.md#systemd_create_socket)
    - [`server_create_socket`](#server_create_socket)
    - [`server_update_socket`](#server_update_socket)
    - [`server_client_create`](server-client.c.driver.md#server_client_create)
    - [`options_set_number`](options.c.driver.md#options_set_number)
    - [`server_acl_init`](server-acl.c.driver.md#server_acl_init)
    - [`server_add_accept`](#server_add_accept)
    - [`proc_loop`](proc.c.driver.md#proc_loop)
    - [`job_kill_all`](job.c.driver.md#job_kill_all)
    - [`status_prompt_save_history`](status.c.driver.md#status_prompt_save_history)


---
### server_loop<!-- {{#callable:server_loop}} -->
The `server_loop` function manages the main event loop of a server, processing client commands and determining when the server should exit.
- **Inputs**:
    - None
- **Control Flow**:
    - Initialize `current_time` with the current time.
    - Enter a do-while loop that continues as long as there are items to process.
    - Call [`cmdq_next`](cmd-queue.c.driver.md#cmdq_next) with `NULL` to process global commands and accumulate the number of items processed.
    - Iterate over each client in the `clients` list, checking if the client is identified, and if so, call [`cmdq_next`](cmd-queue.c.driver.md#cmdq_next) for that client to process its commands and add to the item count.
    - Exit the do-while loop when no more items are left to process.
    - Call [`server_client_loop`](server-client.c.driver.md#server_client_loop) to handle client-specific operations.
    - Check if the `exit-empty` option is not set and `server_exit` is false; if so, return 0 to continue running.
    - Check if the `exit-unattached` option is not set and there are active sessions; if so, return 0 to continue running.
    - Iterate over each client to check if any client is attached to a session; if so, return 0 to continue running.
    - Call [`cmd_wait_for_flush`](cmd-wait-for.c.driver.md#cmd_wait_for_flush) to ensure all client commands are processed before exiting.
    - Check if there are still clients in the queue; if so, return 0 to continue running.
    - Check if any jobs are still running; if so, return 0 to continue running.
    - Return 1 to indicate the server should exit.
- **Output**:
    - The function returns an integer: 0 if the server should continue running, or 1 if the server should exit.
- **Functions called**:
    - [`cmdq_next`](cmd-queue.c.driver.md#cmdq_next)
    - [`server_client_loop`](server-client.c.driver.md#server_client_loop)
    - [`options_get_number`](options.c.driver.md#options_get_number)
    - [`cmd_wait_for_flush`](cmd-wait-for.c.driver.md#cmd_wait_for_flush)
    - [`job_still_running`](job.c.driver.md#job_still_running)


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
- **Functions called**:
    - [`cmd_wait_for_flush`](cmd-wait-for.c.driver.md#cmd_wait_for_flush)
    - [`server_client_lost`](server-client.c.driver.md#server_client_lost)
    - [`session_destroy`](session.c.driver.md#session_destroy)


---
### server_update_socket<!-- {{#callable:server_update_socket}} -->
The `server_update_socket` function updates the execute permissions of a socket file based on whether there are any attached sessions.
- **Inputs**:
    - None
- **Control Flow**:
    - Initialize a static variable `last` to -1 and a local variable `n` to 0.
    - Iterate over all sessions using `RB_FOREACH` to check if any session is attached; if found, increment `n` and break the loop.
    - Compare `n` with `last`; if they are different, update `last` to `n`.
    - Use `stat` to get the current permissions of the socket file; if it fails, return immediately.
    - Extract the current permissions using `sb.st_mode & ACCESSPERMS`.
    - If `n` is not zero, add execute permissions for user, group, and others if read permissions are present.
    - If `n` is zero, remove execute permissions for user, group, and others.
    - Apply the updated permissions using `chmod`.
- **Output**:
    - The function does not return any value; it modifies the file permissions of the socket file based on the presence of attached sessions.


---
### server_accept<!-- {{#callable:server_accept}} -->
The `server_accept` function handles incoming client connections on a server socket, creating a new client instance for each accepted connection and managing access control.
- **Inputs**:
    - `fd`: The file descriptor of the server socket that is ready to accept a new connection.
    - `events`: A bitmask of event flags indicating the type of event that occurred, typically including EV_READ for read events.
    - `data`: Unused parameter, typically used for passing additional data to the callback function.
- **Control Flow**:
    - The function begins by calling `server_add_accept(0)` to set up the server to accept new connections.
    - It checks if the event is a read event (EV_READ); if not, it returns immediately.
    - The function attempts to accept a new connection using the `accept` system call, storing the new file descriptor in `newfd`.
    - If `accept` fails with certain errors (EAGAIN, EINTR, ECONNABORTED), it returns without taking further action.
    - If `accept` fails due to file descriptor limits (ENFILE, EMFILE), it calls `server_add_accept(1)` to delay further attempts and returns.
    - If `accept` fails for other reasons, it calls `fatal` to handle the error.
    - If the server is set to exit (`server_exit` is true), it closes the new file descriptor and returns.
    - A new client structure is created using `server_client_create(newfd)`.
    - The function checks if the client is allowed to join using `server_acl_join(c)`; if not, it sets an exit message and marks the client for exit.
- **Output**:
    - The function does not return a value; it manages client connections and modifies server state as needed.
- **Functions called**:
    - [`server_add_accept`](#server_add_accept)
    - [`server_client_create`](server-client.c.driver.md#server_client_create)
    - [`server_acl_join`](server-acl.c.driver.md#server_acl_join)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### server_add_accept<!-- {{#callable:server_add_accept}} -->
The `server_add_accept` function sets up an event to accept incoming connections on a server socket, with an optional timeout for backoff when file descriptors are low.
- **Inputs**:
    - `timeout`: An integer representing the timeout duration in seconds; if zero, the event is set for reading, otherwise for timeout.
- **Control Flow**:
    - Initialize a `timeval` structure `tv` with the given `timeout` and zero microseconds.
    - Check if `server_fd` is -1, and return immediately if true, indicating no server socket is available.
    - If the `server_ev_accept` event is already initialized, delete it to prevent duplicate events.
    - If `timeout` is zero, set the event `server_ev_accept` to trigger on read events (`EV_READ`) and add it without a timeout.
    - If `timeout` is non-zero, set the event `server_ev_accept` to trigger on timeout events (`EV_TIMEOUT`) and add it with the specified timeout `tv`.
- **Output**:
    - The function does not return a value; it sets up an event for accepting connections on the server socket.


---
### server_signal<!-- {{#callable:server_signal}} -->
The `server_signal` function handles various signals received by the server, executing specific actions based on the signal type.
- **Inputs**:
    - `sig`: An integer representing the signal number that the server has received.
- **Control Flow**:
    - Logs the received signal using `log_debug`.
    - Uses a switch statement to handle different signals:
    - For `SIGINT` and `SIGTERM`, sets `server_exit` to 1 and calls [`server_send_exit`](#server_send_exit) to initiate server shutdown.
    - For `SIGCHLD`, calls [`server_child_signal`](#server_child_signal) to handle child process signals.
    - For `SIGUSR1`, deletes the current accept event, creates a new server socket, updates the server file descriptor, and re-adds the accept event.
    - For `SIGUSR2`, toggles logging for the server process using [`proc_toggle_log`](proc.c.driver.md#proc_toggle_log).
- **Output**:
    - The function does not return a value; it performs actions based on the received signal.
- **Functions called**:
    - [`server_send_exit`](#server_send_exit)
    - [`server_child_signal`](#server_child_signal)
    - [`server_create_socket`](#server_create_socket)
    - [`server_update_socket`](#server_update_socket)
    - [`server_add_accept`](#server_add_accept)
    - [`proc_toggle_log`](proc.c.driver.md#proc_toggle_log)


---
### server_child_signal<!-- {{#callable:server_child_signal}} -->
The `server_child_signal` function handles the SIGCHLD signal by checking the status of child processes and taking appropriate actions based on whether they have stopped or exited.
- **Inputs**:
    - None
- **Control Flow**:
    - The function enters an infinite loop to continuously check for child process status changes.
    - It calls `waitpid` with `WAIT_ANY`, `&status`, and `WNOHANG|WUNTRACED` to check for any child process status changes without blocking.
    - If `waitpid` returns -1 and `errno` is `ECHILD`, it means there are no child processes to wait for, so the function returns.
    - If `waitpid` returns 0, it means there are no child processes that have changed state, so the function returns.
    - If a child process has stopped, indicated by `WIFSTOPPED(status)`, it calls [`server_child_stopped`](#server_child_stopped) with the child's PID and status.
    - If a child process has exited or was terminated by a signal, indicated by `WIFEXITED(status)` or `WIFSIGNALED(status)`, it calls [`server_child_exited`](#server_child_exited) with the child's PID and status.
- **Output**:
    - The function does not return any output; it performs actions based on the status of child processes.
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
    - Log the exit of the pane using `log_debug`.
    - Set the `PANE_EXITED` flag for the pane.
    - Check if the pane is ready to be destroyed using [`window_pane_destroy_ready`](window.c.driver.md#window_pane_destroy_ready).
    - If the pane is ready to be destroyed, call [`server_destroy_pane`](server-fn.c.driver.md#server_destroy_pane) to destroy it.
    - Break out of the loop once the matching pane is found and processed.
    - Call [`job_check_died`](job.c.driver.md#job_check_died) to handle any additional cleanup or checks for the exited job.
- **Output**:
    - The function does not return a value; it performs cleanup and updates the status of the window pane associated with the exited child process.
- **Functions called**:
    - [`window_pane_destroy_ready`](window.c.driver.md#window_pane_destroy_ready)
    - [`server_destroy_pane`](server-fn.c.driver.md#server_destroy_pane)
    - [`job_check_died`](job.c.driver.md#job_check_died)


---
### server_child_stopped<!-- {{#callable:server_child_stopped}} -->
The `server_child_stopped` function handles the event when a child process is stopped, specifically resuming the process if it was stopped by signals other than SIGTTIN or SIGTTOU.
- **Inputs**:
    - `pid`: The process ID of the child process that has stopped.
    - `status`: The status code returned by the waitpid function, indicating the reason for the child process's stop.
- **Control Flow**:
    - Check if the stop signal in the status is SIGTTIN or SIGTTOU; if so, return immediately without further action.
    - Iterate over all windows and their panes to find the pane with a matching process ID (pid).
    - If a matching pane is found, attempt to continue the process group of the stopped process using `killpg`; if that fails, use `kill` to send SIGCONT to the process.
    - Call [`job_check_died`](job.c.driver.md#job_check_died) to handle any additional cleanup or checks for the stopped process.
- **Output**:
    - The function does not return a value; it performs actions to resume a stopped child process and checks for any job-related cleanup.
- **Functions called**:
    - [`job_check_died`](job.c.driver.md#job_check_died)


---
### server_add_message<!-- {{#callable:server_add_message}} -->
The `server_add_message` function formats a message, logs it, and manages a message log with a specified limit.
- **Inputs**:
    - `fmt`: A format string for the message to be added.
    - `...`: A variable number of arguments to be formatted according to the format string.
- **Control Flow**:
    - Initialize a variable argument list and format the message string using [`xvasprintf`](xmalloc.c.driver.md#xvasprintf).
    - Log the formatted message using `log_debug`.
    - Allocate memory for a new `message_entry` structure and initialize it with the current time, a unique message number, and the formatted message string.
    - Insert the new message entry at the tail of the `message_log` queue.
    - Retrieve the message limit from global options using [`options_get_number`](options.c.driver.md#options_get_number).
    - Iterate over the `message_log` queue to remove and free messages that exceed the message limit, ensuring the log does not grow beyond the specified limit.
- **Output**:
    - The function does not return a value; it modifies the global `message_log` by adding a new message and potentially removing old ones.
- **Functions called**:
    - [`xvasprintf`](xmalloc.c.driver.md#xvasprintf)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`options_get_number`](options.c.driver.md#options_get_number)


