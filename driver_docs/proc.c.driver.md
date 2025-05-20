# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it provides functionality for managing inter-process communication and signal handling within the tmux server. The file defines two primary structures, `tmuxproc` and `tmuxpeer`, which represent a process and its communication peers, respectively. The `tmuxproc` structure manages signal events and a list of peers, while the `tmuxpeer` structure handles message buffering and dispatching for communication between processes. The code includes functions for starting and managing a tmux process, handling signals, sending and receiving messages, and managing peer connections.

The file is not an executable but rather a component of the tmux server, providing essential backend functionality for process management and communication. It uses the libevent library to handle asynchronous events and signals, ensuring that the tmux server can efficiently manage multiple client connections and respond to system signals. The code also includes mechanisms for version checking and logging, which are crucial for maintaining compatibility and debugging. This file is integral to the tmux server's ability to manage multiple terminal sessions and communicate with client processes, making it a critical part of the tmux infrastructure.
# Imports and Dependencies

---
- `sys/types.h`
- `sys/socket.h`
- `sys/uio.h`
- `sys/utsname.h`
- `errno.h`
- `signal.h`
- `stdlib.h`
- `string.h`
- `unistd.h`
- `ncurses.h`
- `tmux.h`


# Data Structures

---
### tmuxproc
- **Type**: `struct`
- **Members**:
    - `name`: A constant character pointer representing the name of the tmux process.
    - `exit`: An integer flag indicating whether the process should exit.
    - `signalcb`: A function pointer for handling signals.
    - `ev_sigint`: An event structure for handling SIGINT signals.
    - `ev_sighup`: An event structure for handling SIGHUP signals.
    - `ev_sigchld`: An event structure for handling SIGCHLD signals.
    - `ev_sigcont`: An event structure for handling SIGCONT signals.
    - `ev_sigterm`: An event structure for handling SIGTERM signals.
    - `ev_sigusr1`: An event structure for handling SIGUSR1 signals.
    - `ev_sigusr2`: An event structure for handling SIGUSR2 signals.
    - `ev_sigwinch`: An event structure for handling SIGWINCH signals.
    - `peers`: A tail queue head for managing a list of tmuxpeer structures.
- **Description**: The `tmuxproc` structure is designed to manage a tmux process, encapsulating its name, exit status, signal handling callback, and various signal event handlers. It also maintains a list of peers using a tail queue, allowing for efficient management of multiple peer connections. This structure is central to handling process-level events and signals within the tmux application, facilitating communication and control over the process lifecycle.


---
### tmuxpeer
- **Type**: `struct`
- **Members**:
    - `parent`: A pointer to the parent tmuxproc structure.
    - `ibuf`: An imsgbuf structure used for inter-process communication.
    - `event`: An event structure for handling events.
    - `uid`: The user ID associated with the peer.
    - `flags`: An integer representing the state flags of the peer, with PEER_BAD indicating a bad state.
    - `dispatchcb`: A function pointer for the dispatch callback function.
    - `arg`: A void pointer to additional arguments for the dispatch callback.
    - `entry`: A TAILQ_ENTRY for linking tmuxpeer structures in a queue.
- **Description**: The 'tmuxpeer' structure represents a peer in the tmux process communication system, encapsulating information about the peer's parent process, communication buffer, event handling, user identity, state flags, and callback mechanisms. It is designed to facilitate communication and event management between different processes in the tmux environment, using a linked list structure to manage multiple peers.


# Functions

---
### proc_event_cb<!-- {{#callable:proc_event_cb}} -->
The `proc_event_cb` function handles read and write events for a `tmuxpeer` object, processing incoming messages and updating the event state accordingly.
- **Inputs**:
    - `fd`: An unused integer file descriptor, typically required by the event callback signature.
    - `events`: A short integer representing the event type flags, such as EV_READ or EV_WRITE, that triggered the callback.
    - `arg`: A pointer to a `tmuxpeer` structure, which represents the peer connection being handled.
- **Control Flow**:
    - Check if the peer is not marked as bad and if the event is a read event (EV_READ).
    - Attempt to read from the peer's message buffer using [`imsgbuf_read`](compat/imsg.c.driver.md#imsgbuf_read); if unsuccessful, call the peer's dispatch callback with NULL and return.
    - Enter a loop to process messages from the peer's buffer using [`imsg_get`](compat/imsg.c.driver.md#imsg_get); if an error occurs, call the dispatch callback with NULL and return.
    - For each message, log the message type, check the version with [`peer_check_version`](#peer_check_version), and if valid, call the dispatch callback with the message.
    - Free the message after processing.
    - If the event is a write event (EV_WRITE), attempt to write to the peer's message buffer using [`imsgbuf_write`](compat/imsg.c.driver.md#imsgbuf_write); if unsuccessful, call the dispatch callback with NULL and return.
    - If the peer is marked as bad and the message buffer queue length is zero, call the dispatch callback with NULL and return.
    - Call [`proc_update_event`](#proc_update_event) to update the event state for the peer.
- **Output**:
    - The function does not return a value but may invoke the peer's dispatch callback with either a message or NULL, depending on the event handling outcome.
- **Functions called**:
    - [`imsgbuf_read`](compat/imsg.c.driver.md#imsgbuf_read)
    - [`imsg_get`](compat/imsg.c.driver.md#imsg_get)
    - [`peer_check_version`](#peer_check_version)
    - [`imsg_free`](compat/imsg.c.driver.md#imsg_free)
    - [`imsgbuf_write`](compat/imsg.c.driver.md#imsgbuf_write)
    - [`imsgbuf_queuelen`](compat/imsg.c.driver.md#imsgbuf_queuelen)
    - [`proc_update_event`](#proc_update_event)


---
### proc_signal_cb<!-- {{#callable:proc_signal_cb}} -->
The `proc_signal_cb` function is a callback that handles signals by invoking a signal handling function associated with a `tmuxproc` structure.
- **Inputs**:
    - `signo`: An integer representing the signal number that triggered the callback.
    - `events`: A short integer representing the event type, which is unused in this function.
    - `arg`: A pointer to a `tmuxproc` structure, which contains the signal handling function to be called.
- **Control Flow**:
    - The function casts the `arg` parameter to a `tmuxproc` structure pointer named `tp`.
    - It then calls the `signalcb` function pointer within the `tmuxproc` structure, passing the `signo` as an argument.
- **Output**:
    - The function does not return any value; it performs its operation by invoking a callback function.


---
### peer_check_version<!-- {{#callable:peer_check_version}} -->
The `peer_check_version` function verifies if a peer's protocol version matches the expected version and marks the peer as bad if it doesn't.
- **Inputs**:
    - `peer`: A pointer to a `tmuxpeer` structure representing the peer whose version is being checked.
    - `imsg`: A pointer to an `imsg` structure containing the message header with the peer's version information.
- **Control Flow**:
    - Extract the version from the `peerid` field of the `imsg` header using a bitwise AND operation with `0xff`.
    - Check if the message type is not `MSG_VERSION` and if the extracted version does not match `PROTOCOL_VERSION`.
    - If the version is incorrect, log a debug message indicating the bad version and peer address.
    - Send a `MSG_VERSION` message to the peer using [`proc_send`](#proc_send) to indicate a version mismatch.
    - Set the `PEER_BAD` flag in the peer's flags to mark it as having a bad version.
    - Return `-1` to indicate a version mismatch.
    - Return `0` if the version is correct.
- **Output**:
    - Returns `0` if the peer's version is correct, otherwise returns `-1` and marks the peer as having a bad version.
- **Functions called**:
    - [`proc_send`](#proc_send)


---
### proc_update_event<!-- {{#callable:proc_update_event}} -->
The `proc_update_event` function updates the event configuration for a given `tmuxpeer` by setting it to listen for read and optionally write events based on the message buffer state.
- **Inputs**:
    - `peer`: A pointer to a `tmuxpeer` structure representing the peer whose event configuration is to be updated.
- **Control Flow**:
    - The function begins by removing the current event associated with the `peer` using `event_del`.
    - It initializes the `events` variable to `EV_READ`, indicating that the event should listen for read operations.
    - The function checks if there are any messages queued in the peer's message buffer (`ibuf`) using [`imsgbuf_queuelen`](compat/imsg.c.driver.md#imsgbuf_queuelen).
    - If there are messages queued, it adds `EV_WRITE` to the `events` variable, indicating that the event should also listen for write operations.
    - The function sets up the event for the peer using `event_set`, passing the file descriptor, the determined events, the callback function `proc_event_cb`, and the peer itself as arguments.
    - Finally, the function adds the event to the event loop using `event_add`.
- **Output**:
    - The function does not return a value; it modifies the event configuration of the `tmuxpeer` in place.
- **Functions called**:
    - [`imsgbuf_queuelen`](compat/imsg.c.driver.md#imsgbuf_queuelen)


---
### proc_send<!-- {{#callable:proc_send}} -->
The `proc_send` function sends a message to a specified peer using the imsg protocol, handling errors and updating the event state.
- **Inputs**:
    - `peer`: A pointer to a `tmuxpeer` structure representing the peer to which the message is to be sent.
    - `type`: An enumeration value of type `msgtype` indicating the type of message to be sent.
    - `fd`: An integer file descriptor associated with the message, or -1 if not applicable.
    - `buf`: A pointer to the buffer containing the message data to be sent.
    - `len`: A size_t value representing the length of the message data in bytes.
- **Control Flow**:
    - Check if the peer's flags indicate a bad state (PEER_BAD); if so, return -1 to indicate failure.
    - Log a debug message indicating the message type, peer, and length of the message being sent.
    - Compose the message using [`imsg_compose`](compat/imsg.c.driver.md#imsg_compose) with the provided parameters; if this fails (returns not equal to 1), return -1.
    - Call [`proc_update_event`](#proc_update_event) to update the event state for the peer.
    - Return 0 to indicate success.
- **Output**:
    - The function returns 0 on success, or -1 if the peer is in a bad state or if message composition fails.
- **Functions called**:
    - [`imsg_compose`](compat/imsg.c.driver.md#imsg_compose)
    - [`proc_update_event`](#proc_update_event)


---
### proc_start<!-- {{#callable:proc_start}} -->
The `proc_start` function initializes and starts a new tmux process with a given name, setting up logging and system information, and returns a pointer to the newly created `tmuxproc` structure.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the process to be started.
- **Control Flow**:
    - Open a log with the given process name using `log_open(name)`.
    - Set the process title using `setproctitle` with the process name and socket path.
    - Attempt to retrieve system information using `uname(&u)`; if it fails, zero out the `utsname` structure `u`.
    - Log various debug information including process ID, version, socket path, protocol version, system information, and library versions.
    - Allocate memory for a new `tmuxproc` structure using [`xcalloc`](xmalloc.c.driver.md#xcalloc) and assign it to `tp`.
    - Duplicate the process name string and assign it to `tp->name` using [`xstrdup`](xmalloc.c.driver.md#xstrdup).
    - Initialize the `peers` queue in the `tmuxproc` structure using `TAILQ_INIT`.
    - Return the pointer to the newly created `tmuxproc` structure `tp`.
- **Output**:
    - Returns a pointer to a newly allocated and initialized `tmuxproc` structure.
- **Functions called**:
    - [`log_open`](log.c.driver.md#log_open)
    - [`getversion`](tmux.c.driver.md#getversion)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### proc_loop<!-- {{#callable:proc_loop}} -->
The `proc_loop` function manages the event loop for a `tmuxproc` structure, repeatedly executing the event loop until a specified exit condition is met.
- **Inputs**:
    - `tp`: A pointer to a `tmuxproc` structure, which contains information about the process, including its name and exit status.
    - `loopcb`: A pointer to a callback function that returns an integer, used to determine whether the loop should continue; it can be NULL.
- **Control Flow**:
    - Logs the entry into the loop using the process name from the `tmuxproc` structure.
    - Enters a `do-while` loop that calls `event_loop(EVLOOP_ONCE)` to process events once per iteration.
    - Continues looping while the `exit` flag in the `tmuxproc` structure is not set and the `loopcb` callback is either NULL or returns false (0).
    - Logs the exit from the loop using the process name from the `tmuxproc` structure.
- **Output**:
    - The function does not return a value; it operates in a loop until the exit condition is met.


---
### proc_exit<!-- {{#callable:proc_exit}} -->
The `proc_exit` function flushes all message buffers for each peer in a `tmuxproc` structure and sets the exit flag to indicate the process should terminate.
- **Inputs**:
    - `tp`: A pointer to a `tmuxproc` structure, which represents a process in the tmux application and contains a list of peers and an exit flag.
- **Control Flow**:
    - Iterate over each `tmuxpeer` in the `peers` list of the `tmuxproc` structure `tp`.
    - For each `tmuxpeer`, call [`imsgbuf_flush`](compat/imsg.c.driver.md#imsgbuf_flush) on its `ibuf` to flush any pending messages.
    - Set the `exit` flag of the `tmuxproc` structure `tp` to 1, indicating that the process should exit.
- **Output**:
    - The function does not return any value; it modifies the state of the `tmuxproc` structure by flushing message buffers and setting the exit flag.
- **Functions called**:
    - [`imsgbuf_flush`](compat/imsg.c.driver.md#imsgbuf_flush)


---
### proc_set_signals<!-- {{#callable:proc_set_signals}} -->
The `proc_set_signals` function configures signal handling for a `tmuxproc` structure by ignoring certain signals and setting up event handlers for others.
- **Inputs**:
    - `tp`: A pointer to a `tmuxproc` structure, which holds information about the process and its signal events.
    - `signalcb`: A pointer to a callback function that takes an integer signal number as its argument, used to handle signals.
- **Control Flow**:
    - Assigns the provided `signalcb` function to the `signalcb` member of the `tmuxproc` structure `tp`.
    - Initializes a `sigaction` structure `sa` to zero and sets its mask to empty, flags to `SA_RESTART`, and handler to `SIG_IGN` to ignore certain signals.
    - Uses `sigaction` to ignore the signals `SIGPIPE`, `SIGTSTP`, `SIGTTIN`, `SIGTTOU`, and `SIGQUIT`.
    - Sets up event handlers for the signals `SIGINT`, `SIGHUP`, `SIGCHLD`, `SIGCONT`, `SIGTERM`, `SIGUSR1`, `SIGUSR2`, and `SIGWINCH` using `signal_set` and `signal_add`, associating them with the `proc_signal_cb` callback and the `tmuxproc` structure `tp`.
- **Output**:
    - The function does not return a value; it sets up signal handling for the specified `tmuxproc` structure.


---
### proc_clear_signals<!-- {{#callable:proc_clear_signals}} -->
The `proc_clear_signals` function resets signal handlers to their default actions and removes signal events from a `tmuxproc` structure, with an option to reset additional signals if specified.
- **Inputs**:
    - `tp`: A pointer to a `tmuxproc` structure, which contains signal events and other process-related information.
    - `defaults`: An integer flag indicating whether to reset additional signals to their default actions.
- **Control Flow**:
    - Initialize a `sigaction` structure `sa` with zeroed memory and set its handler to `SIG_DFL` (default signal handling).
    - Set the `sa_flags` to `SA_RESTART` and clear the signal mask using `sigemptyset`.
    - Apply the default signal action to `SIGPIPE` and `SIGTSTP` using `sigaction`.
    - Remove signal events for `SIGINT`, `SIGHUP`, `SIGCHLD`, `SIGCONT`, `SIGTERM`, `SIGUSR1`, `SIGUSR2`, and `SIGWINCH` from the `tmuxproc` structure using `signal_del`.
    - If `defaults` is non-zero, apply the default signal action to additional signals: `SIGINT`, `SIGQUIT`, `SIGHUP`, `SIGCHLD`, `SIGCONT`, `SIGTERM`, `SIGUSR1`, `SIGUSR2`, and `SIGWINCH`.
- **Output**:
    - The function does not return a value; it modifies the signal handling state of the process associated with the `tmuxproc` structure.


---
### proc_add_peer<!-- {{#callable:proc_add_peer}} -->
The `proc_add_peer` function initializes and adds a new peer to a tmux process, setting up necessary event handling and message buffering.
- **Inputs**:
    - `tp`: A pointer to a `tmuxproc` structure representing the tmux process to which the peer will be added.
    - `fd`: An integer file descriptor used for communication with the peer.
    - `dispatchcb`: A callback function pointer that will be called to handle messages received from the peer.
    - `arg`: A void pointer to user-defined data that will be passed to the dispatch callback function.
- **Control Flow**:
    - Allocate memory for a new `tmuxpeer` structure and initialize it with zeroes.
    - Set the `parent` field of the peer to the provided `tmuxproc` pointer `tp`.
    - Assign the `dispatchcb` and `arg` to the peer's corresponding fields.
    - Initialize the peer's message buffer using [`imsgbuf_init`](compat/imsg.c.driver.md#imsgbuf_init) with the provided file descriptor `fd`.
    - Allow file descriptor passing in the message buffer using [`imsgbuf_allow_fdpass`](compat/imsg.c.driver.md#imsgbuf_allow_fdpass).
    - Set up an event for reading from the file descriptor using `event_set`, associating it with `proc_event_cb` as the callback function.
    - Attempt to retrieve the peer's effective user ID using [`getpeereid`](compat/getpeereid.c.driver.md#getpeereid); if it fails, set the peer's UID to -1.
    - Log a debug message indicating the addition of the peer.
    - Insert the peer into the `peers` list of the `tmuxproc` structure using `TAILQ_INSERT_TAIL`.
    - Call [`proc_update_event`](#proc_update_event) to update the event settings for the peer.
- **Output**:
    - Returns a pointer to the newly created and initialized `tmuxpeer` structure.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`imsgbuf_init`](compat/imsg.c.driver.md#imsgbuf_init)
    - [`imsgbuf_allow_fdpass`](compat/imsg.c.driver.md#imsgbuf_allow_fdpass)
    - [`getpeereid`](compat/getpeereid.c.driver.md#getpeereid)
    - [`proc_update_event`](#proc_update_event)


---
### proc_remove_peer<!-- {{#callable:proc_remove_peer}} -->
The `proc_remove_peer` function removes a `tmuxpeer` from its parent's peer list, cleans up associated resources, and deallocates memory.
- **Inputs**:
    - `peer`: A pointer to a `tmuxpeer` structure that represents the peer to be removed.
- **Control Flow**:
    - Remove the `peer` from its parent's peer list using `TAILQ_REMOVE`.
    - Log a debug message indicating the removal of the peer.
    - Delete the event associated with the peer using `event_del`.
    - Clear the message buffer of the peer using [`imsgbuf_clear`](compat/imsg.c.driver.md#imsgbuf_clear).
    - Close the file descriptor associated with the peer's message buffer.
    - Free the memory allocated for the `peer` structure.
- **Output**:
    - This function does not return any value; it performs cleanup and deallocation of resources associated with the specified peer.
- **Functions called**:
    - [`imsgbuf_clear`](compat/imsg.c.driver.md#imsgbuf_clear)


---
### proc_kill_peer<!-- {{#callable:proc_kill_peer}} -->
The `proc_kill_peer` function marks a `tmuxpeer` as bad by setting its `PEER_BAD` flag.
- **Inputs**:
    - `peer`: A pointer to a `tmuxpeer` structure representing the peer to be marked as bad.
- **Control Flow**:
    - The function accesses the `flags` field of the `tmuxpeer` structure pointed to by `peer`.
    - It uses the bitwise OR operator to set the `PEER_BAD` flag in the `flags` field.
- **Output**:
    - The function does not return any value; it modifies the `flags` field of the `tmuxpeer` structure in place.


---
### proc_flush_peer<!-- {{#callable:proc_flush_peer}} -->
The `proc_flush_peer` function flushes the message buffer of a given tmux peer.
- **Inputs**:
    - `peer`: A pointer to a `tmuxpeer` structure representing the peer whose message buffer is to be flushed.
- **Control Flow**:
    - The function calls [`imsgbuf_flush`](compat/imsg.c.driver.md#imsgbuf_flush) with the `ibuf` member of the `peer` structure to flush the message buffer.
- **Output**:
    - The function does not return any value.
- **Functions called**:
    - [`imsgbuf_flush`](compat/imsg.c.driver.md#imsgbuf_flush)


---
### proc_toggle_log<!-- {{#callable:proc_toggle_log}} -->
The `proc_toggle_log` function toggles the logging state for a given tmux process by its name.
- **Inputs**:
    - `tp`: A pointer to a `tmuxproc` structure, which contains information about a tmux process, including its name.
- **Control Flow**:
    - The function calls [`log_toggle`](log.c.driver.md#log_toggle) with the `name` field of the `tmuxproc` structure pointed to by `tp`.
- **Output**:
    - The function does not return any value.
- **Functions called**:
    - [`log_toggle`](log.c.driver.md#log_toggle)


---
### proc_fork_and_daemon<!-- {{#callable:proc_fork_and_daemon}} -->
The `proc_fork_and_daemon` function creates a UNIX socket pair, forks the process, and turns the child process into a daemon, returning the process ID of the child to the parent and a file descriptor for inter-process communication.
- **Inputs**:
    - `fd`: A pointer to an integer where the function will store the file descriptor for the socket pair used for communication between the parent and child processes.
- **Control Flow**:
    - Create a UNIX socket pair using `socketpair` for inter-process communication.
    - Fork the current process using `fork`.
    - In the child process (when `fork` returns 0):
    - - Close the parent's end of the socket pair.
    - - Assign the child's end of the socket pair to the provided file descriptor pointer `fd`.
    - - Call [`daemon`](compat/daemon.c.driver.md#daemon) to detach the child process from the terminal and run it in the background.
    - - Return 0 to indicate successful daemonization.
    - In the parent process (when `fork` returns the child's PID):
    - - Close the child's end of the socket pair.
    - - Assign the parent's end of the socket pair to the provided file descriptor pointer `fd`.
    - - Return the child's process ID.
- **Output**:
    - The function returns the process ID of the child process to the parent, and 0 to the child process after it has been daemonized.
- **Functions called**:
    - [`daemon`](compat/daemon.c.driver.md#daemon)


---
### proc_get_peer_uid<!-- {{#callable:proc_get_peer_uid}} -->
The function `proc_get_peer_uid` retrieves the user ID (UID) associated with a given `tmuxpeer` structure.
- **Inputs**:
    - `peer`: A pointer to a `tmuxpeer` structure from which the UID is to be retrieved.
- **Control Flow**:
    - The function directly accesses the `uid` field of the `tmuxpeer` structure pointed to by `peer`.
    - It returns the value of the `uid` field.
- **Output**:
    - The function returns the UID of the specified `tmuxpeer` as a value of type `uid_t`.


