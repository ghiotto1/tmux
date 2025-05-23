# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it provides functionality for managing inter-process communication and signal handling within the tmux server. The code defines two primary structures, `tmuxproc` and `tmuxpeer`, which represent a process and its communication peers, respectively. The `tmuxproc` structure manages signal events and maintains a list of `tmuxpeer` structures, which handle message passing between processes using the `imsg` library. The file includes functions for starting and managing the lifecycle of a `tmuxproc`, including setting up signal handlers, adding and removing peers, and sending and receiving messages.

The code is designed to be integrated into the tmux server, providing a robust mechanism for handling asynchronous events and inter-process communication. It uses the `libevent` library to manage event-driven programming, allowing the tmux server to respond to various signals and messages efficiently. The file does not define a public API but rather serves as an internal component of the tmux server, facilitating communication and process management. The use of macros and conditional compilation suggests that the code is designed to be portable across different systems, with optional support for libraries like `ncurses` and `utf8proc`.
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
- **Description**: The `tmuxproc` structure is a central component in the tmux process management system, encapsulating the process's name, exit status, signal handling callback, and a series of event structures for managing various signals. It also maintains a list of peers, represented by the `tmuxpeer` structure, which are connected to the process. This structure is crucial for handling inter-process communication and signal management within the tmux environment.


---
### tmuxpeer
- **Type**: `struct`
- **Members**:
    - `parent`: A pointer to the parent tmuxproc structure.
    - `ibuf`: An imsgbuf structure used for message buffering.
    - `event`: An event structure for handling events.
    - `uid`: The user ID associated with the peer.
    - `flags`: An integer representing the state flags of the peer, with PEER_BAD indicating a bad state.
    - `dispatchcb`: A function pointer for the dispatch callback handling messages.
    - `arg`: A void pointer to additional arguments for the dispatch callback.
    - `entry`: A TAILQ_ENTRY for linking tmuxpeer structures in a queue.
- **Description**: The `tmuxpeer` structure represents a peer connection in the tmux process communication system. It maintains a reference to its parent `tmuxproc`, handles message buffering and event management through `imsgbuf` and `event` structures, and stores a user ID for the peer. The structure includes a flags field to indicate the state of the peer, such as whether it is in a bad state. It also contains a dispatch callback function pointer and an argument pointer for handling incoming messages, and is linked into a queue of peers using a TAILQ_ENTRY.


# Functions

---
### proc_event_cb<!-- {{#callable:proc_event_cb}} -->
The `proc_event_cb` function handles read and write events for a `tmuxpeer` object, processing incoming messages and updating the event state accordingly.
- **Inputs**:
    - `fd`: An unused integer file descriptor, typically used in event callbacks.
    - `events`: A short integer representing the type of events (read or write) that have occurred.
    - `arg`: A pointer to a `tmuxpeer` structure, representing the peer associated with the event.
- **Control Flow**:
    - Check if the peer is not marked as bad and if a read event has occurred.
    - Attempt to read from the peer's message buffer; if unsuccessful, call the peer's dispatch callback with NULL and return.
    - Enter a loop to process messages from the peer's buffer using `imsg_get`; if an error occurs, call the dispatch callback with NULL and return.
    - For each message, log the message type, check the peer's version, and call the dispatch callback with the message if the version is valid.
    - Free the message after processing.
    - If a write event has occurred, attempt to write to the peer's message buffer; if unsuccessful, call the dispatch callback with NULL and return.
    - If the peer is marked as bad and the message buffer is empty, call the dispatch callback with NULL and return.
    - Update the event state for the peer using [`proc_update_event`](#proc_update_event).
- **Output**:
    - The function does not return a value; it performs actions based on the event type and peer state, such as calling the dispatch callback and updating the event state.
- **Functions called**:
    - [`peer_check_version`](#peer_check_version)
    - [`proc_update_event`](#proc_update_event)


---
### proc_signal_cb<!-- {{#callable:proc_signal_cb}} -->
The `proc_signal_cb` function is a signal handler callback that invokes a user-defined signal handling function for a given signal number.
- **Inputs**:
    - `signo`: An integer representing the signal number that triggered the callback.
    - `events`: A short integer representing the event type, which is unused in this function.
    - `arg`: A pointer to a `tmuxproc` structure, which contains the user-defined signal handling function.
- **Control Flow**:
    - The function casts the `arg` parameter to a `tmuxproc` structure pointer.
    - It then calls the `signalcb` function pointer within the `tmuxproc` structure, passing the `signo` as an argument.
- **Output**:
    - The function does not return any value; it is a void function.


---
### peer_check_version<!-- {{#callable:peer_check_version}} -->
The `peer_check_version` function verifies if a peer's protocol version matches the expected version and flags the peer as bad if it doesn't.
- **Inputs**:
    - `peer`: A pointer to a `tmuxpeer` structure representing the peer whose version is being checked.
    - `imsg`: A pointer to an `imsg` structure containing the message header with the peer's version information.
- **Control Flow**:
    - Extract the version from the `peerid` field of the `imsg` header using a bitwise AND operation with `0xff`.
    - Check if the message type is not `MSG_VERSION` and if the extracted version does not match `PROTOCOL_VERSION`.
    - If the version is incorrect, log a debug message indicating the bad version and peer address.
    - Send a `MSG_VERSION` message to the peer using [`proc_send`](#proc_send) to indicate the version mismatch.
    - Set the `PEER_BAD` flag on the peer to mark it as having a bad version.
    - Return `-1` to indicate a version mismatch.
    - Return `0` if the version is correct.
- **Output**:
    - Returns `0` if the peer's version is correct, otherwise returns `-1` if there is a version mismatch.
- **Functions called**:
    - [`proc_send`](#proc_send)


---
### proc_update_event<!-- {{#callable:proc_update_event}} -->
The `proc_update_event` function updates the event configuration for a given `tmuxpeer` by setting it to listen for read and optionally write events based on the message buffer state.
- **Inputs**:
    - `peer`: A pointer to a `tmuxpeer` structure representing the peer whose event configuration is to be updated.
- **Control Flow**:
    - The function begins by removing the current event associated with the peer using `event_del`.
    - It initializes the `events` variable to `EV_READ`, indicating that the event should listen for read operations.
    - The function checks if there are any messages queued in the peer's message buffer (`ibuf`) using `imsgbuf_queuelen`.
    - If there are messages queued, it adds `EV_WRITE` to the `events` variable, indicating that the event should also listen for write operations.
    - The function sets up the event for the peer using `event_set`, passing the file descriptor, the determined events, the callback function `proc_event_cb`, and the peer itself as arguments.
    - Finally, it adds the event to the event loop using `event_add`.
- **Output**:
    - The function does not return a value; it modifies the event configuration of the provided `tmuxpeer` in place.


---
### proc_send<!-- {{#callable:proc_send}} -->
The `proc_send` function sends a message to a specified peer using the imsg protocol, handling potential errors and updating the event state of the peer.
- **Inputs**:
    - `peer`: A pointer to a `tmuxpeer` structure representing the peer to which the message is to be sent.
    - `type`: An enumeration value of type `msgtype` indicating the type of message to be sent.
    - `fd`: An integer file descriptor associated with the message, or -1 if not applicable.
    - `buf`: A pointer to the buffer containing the message data to be sent.
    - `len`: A size_t value representing the length of the message data in bytes.
- **Control Flow**:
    - Check if the peer has the `PEER_BAD` flag set; if so, return -1 indicating failure.
    - Log a debug message indicating the message type, peer, and length of the message being sent.
    - Compose the message using `imsg_compose` with the provided parameters; if this fails (returns a value other than 1), return -1.
    - Call [`proc_update_event`](#proc_update_event) to update the event state of the peer.
    - Return 0 to indicate successful message sending.
- **Output**:
    - The function returns 0 on success, or -1 if the peer is marked as bad or if message composition fails.
- **Functions called**:
    - [`proc_update_event`](#proc_update_event)


---
### proc_start<!-- {{#callable:proc_start}} -->
The `proc_start` function initializes and starts a new tmux process with a given name, setting up logging and system information, and returns a pointer to the newly created `tmuxproc` structure.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the process to be started.
- **Control Flow**:
    - Open a log with the given process name using `log_open(name)`.
    - Set the process title using `setproctitle` with the process name and socket path.
    - Attempt to retrieve system information using `uname`; if it fails, zero out the `utsname` structure.
    - Log debug information about the process start, including process ID, version, socket path, protocol version, and system information.
    - Log additional debug information about the libraries being used, such as libevent, utf8proc, and ncurses, if available.
    - Allocate memory for a new `tmuxproc` structure and initialize it with zeroes using `xcalloc`.
    - Duplicate the process name string and assign it to the `name` field of the `tmuxproc` structure using `xstrdup`.
    - Initialize the `peers` queue in the `tmuxproc` structure using `TAILQ_INIT`.
    - Return the pointer to the newly created `tmuxproc` structure.
- **Output**:
    - A pointer to a newly allocated and initialized `tmuxproc` structure.


---
### proc_loop<!-- {{#callable:proc_loop}} -->
The `proc_loop` function manages the event loop for a `tmuxproc` structure, executing the loop until a specified exit condition is met.
- **Inputs**:
    - `tp`: A pointer to a `tmuxproc` structure, which contains information about the process, including its name and exit status.
    - `loopcb`: A pointer to a callback function that returns an integer, used to determine whether the loop should continue executing.
- **Control Flow**:
    - Log the entry into the loop using the process name from `tp`.
    - Enter a `do-while` loop that calls `event_loop(EVLOOP_ONCE)` to process events once per iteration.
    - Continue looping while the `exit` flag in `tp` is not set and the `loopcb` callback is either NULL or returns 0.
    - Log the exit from the loop using the process name from `tp`.
- **Output**:
    - The function does not return a value; it controls the flow of the event loop based on the `exit` flag and the `loopcb` callback.


---
### proc_exit<!-- {{#callable:proc_exit}} -->
The `proc_exit` function flushes all message buffers for each peer in a `tmuxproc` structure and sets the exit flag to indicate the process should terminate.
- **Inputs**:
    - `tp`: A pointer to a `tmuxproc` structure representing the process whose peers' message buffers need to be flushed and whose exit flag should be set.
- **Control Flow**:
    - Iterate over each `tmuxpeer` in the `peers` list of the `tmuxproc` structure `tp`.
    - For each `tmuxpeer`, call `imsgbuf_flush` on its `ibuf` to flush any pending messages.
    - Set the `exit` flag of the `tmuxproc` structure `tp` to 1, indicating that the process should exit.
- **Output**:
    - The function does not return any value.


---
### proc_set_signals<!-- {{#callable:proc_set_signals}} -->
The `proc_set_signals` function configures signal handling for a `tmuxproc` structure by ignoring certain signals and setting up custom handlers for others.
- **Inputs**:
    - `tp`: A pointer to a `tmuxproc` structure, which holds information about the process and its signal handling.
    - `signalcb`: A pointer to a callback function that takes an integer signal number as its argument, used to handle specific signals.
- **Control Flow**:
    - Assigns the provided `signalcb` function to the `signalcb` member of the `tmuxproc` structure `tp`.
    - Initializes a `sigaction` structure `sa` to zero and sets its mask to empty, flags to `SA_RESTART`, and handler to `SIG_IGN` to ignore certain signals.
    - Uses `sigaction` to ignore signals `SIGPIPE`, `SIGTSTP`, `SIGTTIN`, `SIGTTOU`, and `SIGQUIT`.
    - Sets up custom signal handlers for `SIGINT`, `SIGHUP`, `SIGCHLD`, `SIGCONT`, `SIGTERM`, `SIGUSR1`, `SIGUSR2`, and `SIGWINCH` using `signal_set` and `signal_add`, associating them with the `proc_signal_cb` callback and the `tmuxproc` structure `tp`.
- **Output**:
    - The function does not return a value; it modifies the signal handling behavior of the `tmuxproc` structure `tp`.


---
### proc_clear_signals<!-- {{#callable:proc_clear_signals}} -->
The `proc_clear_signals` function resets signal handlers to their default actions and removes signal event handlers for a given `tmuxproc` structure, with an option to reset additional signals if specified.
- **Inputs**:
    - `tp`: A pointer to a `tmuxproc` structure, which contains signal event handlers and other process-related information.
    - `defaults`: An integer flag indicating whether to reset additional signals to their default actions (non-zero value) or not (zero value).
- **Control Flow**:
    - Initialize a `sigaction` structure `sa` with zeroed memory and set its mask to empty, flags to `SA_RESTART`, and handler to `SIG_DFL` (default signal handler).
    - Set the default action for `SIGPIPE` and `SIGTSTP` signals using `sigaction`.
    - Remove signal event handlers for `SIGINT`, `SIGHUP`, `SIGCHLD`, `SIGCONT`, `SIGTERM`, `SIGUSR1`, `SIGUSR2`, and `SIGWINCH` using `signal_del`.
    - If `defaults` is non-zero, set the default action for additional signals (`SIGINT`, `SIGQUIT`, `SIGHUP`, `SIGCHLD`, `SIGCONT`, `SIGTERM`, `SIGUSR1`, `SIGUSR2`, `SIGWINCH`) using `sigaction`.
- **Output**:
    - The function does not return a value; it modifies the signal handling behavior of the process associated with the `tmuxproc` structure.


---
### proc_add_peer<!-- {{#callable:proc_add_peer}} -->
The `proc_add_peer` function initializes and adds a new peer to a tmux process, setting up necessary event handling and message buffering.
- **Inputs**:
    - `tp`: A pointer to a `tmuxproc` structure representing the tmux process to which the peer will be added.
    - `fd`: An integer file descriptor used for communication with the peer.
    - `dispatchcb`: A callback function pointer that will be called to handle incoming messages for the peer.
    - `arg`: A void pointer to user-defined data that will be passed to the dispatch callback function.
- **Control Flow**:
    - Allocate memory for a new `tmuxpeer` structure and initialize it with zeroes using `xcalloc`.
    - Set the `parent` field of the peer to the provided `tmuxproc` pointer `tp`.
    - Assign the `dispatchcb` and `arg` to the peer's corresponding fields.
    - Initialize the peer's message buffer `ibuf` with the provided file descriptor `fd` using `imsgbuf_init`.
    - Allow file descriptor passing in the message buffer with `imsgbuf_allow_fdpass`.
    - Set up an event for reading from the file descriptor using `event_set`, associating it with `proc_event_cb` as the callback function.
    - Attempt to retrieve the peer's effective user ID using `getpeereid`; if it fails, set the peer's UID to -1.
    - Log the addition of the peer for debugging purposes.
    - Insert the new peer into the `peers` list of the `tmuxproc` structure using `TAILQ_INSERT_TAIL`.
    - Call [`proc_update_event`](#proc_update_event) to update the event handling for the peer.
    - Return the pointer to the newly created `tmuxpeer` structure.
- **Output**:
    - Returns a pointer to the newly created and initialized `tmuxpeer` structure.
- **Functions called**:
    - [`proc_update_event`](#proc_update_event)


---
### proc_remove_peer<!-- {{#callable:proc_remove_peer}} -->
The `proc_remove_peer` function removes a `tmuxpeer` from its parent's peer list, cleans up associated resources, and deallocates memory.
- **Inputs**:
    - `peer`: A pointer to a `tmuxpeer` structure that represents the peer to be removed.
- **Control Flow**:
    - Remove the `peer` from its parent's peer list using `TAILQ_REMOVE`.
    - Log a debug message indicating the removal of the peer.
    - Delete the event associated with the `peer` using `event_del`.
    - Clear the message buffer of the `peer` using `imsgbuf_clear`.
    - Close the file descriptor associated with the `peer`'s message buffer.
    - Free the memory allocated for the `peer`.
- **Output**:
    - The function does not return any value; it performs cleanup and deallocation of resources associated with the specified `tmuxpeer`.


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
    - The function calls `imsgbuf_flush` with the `ibuf` member of the `tmuxpeer` structure pointed to by `peer`.
- **Output**:
    - The function does not return any value.


---
### proc_toggle_log<!-- {{#callable:proc_toggle_log}} -->
The `proc_toggle_log` function toggles the logging state for a given tmux process by its name.
- **Inputs**:
    - `tp`: A pointer to a `tmuxproc` structure, which contains information about a tmux process, including its name.
- **Control Flow**:
    - The function calls `log_toggle` with the `name` field of the `tmuxproc` structure pointed to by `tp`.
- **Output**:
    - The function does not return any value.


---
### proc_fork_and_daemon<!-- {{#callable:proc_fork_and_daemon}} -->
The `proc_fork_and_daemon` function creates a UNIX socket pair, forks the process, and turns the child process into a daemon, returning the process ID of the child to the parent and a file descriptor for inter-process communication.
- **Inputs**:
    - `fd`: A pointer to an integer where the function will store the file descriptor for the socket pair used for communication between the parent and child processes.
- **Control Flow**:
    - Create a UNIX socket pair using `socketpair` and store the file descriptors in `pair` array.
    - Fork the current process using `fork`.
    - In the child process (when `fork` returns 0), close the first socket in the pair, assign the second socket to `*fd`, and call `daemon` to run the process in the background as a daemon.
    - Return 0 from the child process after successfully becoming a daemon.
    - In the parent process (when `fork` returns a positive PID), close the second socket in the pair, assign the first socket to `*fd`, and return the child's PID.
- **Output**:
    - The function returns the process ID of the child process to the parent, and 0 to the child process. It also sets the file descriptor for inter-process communication in the provided `fd` pointer.


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


# Function Declarations (Public API)

---
### peer_check_version<!-- {{#callable_declaration:peer_check_version}} -->
Checks the version of a message from a peer.
- **Description**: This function verifies that a message received from a peer has the correct protocol version. It should be called whenever a message is received to ensure compatibility. If the message type is not MSG_VERSION and the version does not match the expected PROTOCOL_VERSION, the function marks the peer as having a bad version by setting the PEER_BAD flag and sends a MSG_VERSION message back to the peer. This function is essential for maintaining protocol consistency between peers.
- **Inputs**:
    - `peer`: A pointer to a tmuxpeer structure representing the peer from which the message was received. Must not be null.
    - `imsg`: A pointer to an imsg structure containing the message to be checked. Must not be null.
- **Output**: Returns 0 if the version is correct, or -1 if the version is incorrect and the peer is marked as bad.
- **See also**: [`peer_check_version`](#peer_check_version)  (Implementation)


---
### proc_update_event<!-- {{#callable_declaration:proc_update_event}} -->
Update the event configuration for a tmux peer.
- **Description**: This function updates the event configuration for a given tmux peer by modifying its event structure. It should be called whenever the state of the peer's message buffer changes, such as after sending or receiving messages. The function ensures that the appropriate read and write events are set based on the current state of the peer's message buffer. It is important to call this function to maintain correct event handling for the peer.
- **Inputs**:
    - `peer`: A pointer to a `tmuxpeer` structure representing the peer whose event configuration is to be updated. Must not be null. The caller retains ownership of the peer structure.
- **Output**: None
- **See also**: [`proc_update_event`](#proc_update_event)  (Implementation)


