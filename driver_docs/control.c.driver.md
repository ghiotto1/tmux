# Purpose
The provided C source code is part of the tmux project, specifically handling the control mode functionality for tmux clients. This code is responsible for managing the communication between the tmux server and its clients when operating in control mode. It defines several data structures and functions to handle the queuing, writing, and reading of data blocks to and from clients, ensuring that data is sent in the correct order and managing the state of client connections.

Key components of this code include structures like `control_block`, `control_pane`, `control_sub`, and `control_state`, which are used to manage data blocks, pane states, subscriptions, and overall client state, respectively. The code implements functions to add, remove, and manage these structures, ensuring that data is processed efficiently and in the correct sequence. It also includes mechanisms for handling client subscriptions to various tmux entities, such as panes and windows, allowing clients to receive updates about changes in the tmux environment. The code is designed to be integrated into the tmux server, providing a robust interface for clients to interact with tmux sessions programmatically.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `time.h`
- `unistd.h`
- `tmux.h`


# Global Variables

---
### name
- **Type**: `char*`
- **Description**: The `name` variable is a pointer to a character string that represents the name of a control client subscription. It is part of the `control_sub` structure, which is used to manage subscriptions in the control client state.
- **Use**: This variable is used to identify and manage subscriptions for control clients, allowing the system to track and respond to changes in the subscribed entities.


---
### s
- **Type**: `char*`
- **Description**: The variable `s` is a pointer to a character array (string) that is used to store a formatted string. It is dynamically allocated using `xvasprintf` in the `control_vwrite` function.
- **Use**: This variable is used to hold a formatted string that is logged and then written to a buffer event for output.


# Data Structures

---
### control_block
- **Type**: `struct`
- **Members**:
    - `size`: Represents the size of the data block.
    - `line`: Pointer to a character array holding the data line.
    - `t`: Timestamp or identifier for the block.
    - `entry`: Links the control_block into a queue for pane-specific blocks.
    - `all_entry`: Links the control_block into a queue for all blocks.
- **Description**: The `control_block` structure is used to manage blocks of data that are queued for output in a client-server architecture, specifically within the context of a terminal multiplexer like tmux. Each block contains a size, a line of data, and a timestamp, and is linked into two separate queues: one for pane-specific data and another for all data. This structure helps in managing the order and flow of data to ensure that output is correctly synchronized and delivered to clients in the correct sequence.


---
### control_pane
- **Type**: `struct`
- **Members**:
    - `pane`: An unsigned integer representing the pane identifier.
    - `offset`: A structure representing the data that has been written to the pane.
    - `queued`: A structure representing the data that has been queued for the pane.
    - `flags`: An integer representing the state of the pane with defined flags for off and paused states.
    - `pending_flag`: An integer indicating if the pane is pending for some operation.
    - `pending_entry`: A TAILQ entry for managing the pane in a pending list.
    - `blocks`: A TAILQ head for managing a list of control blocks associated with the pane.
    - `entry`: An RB entry for managing the pane in a red-black tree.
- **Description**: The `control_pane` structure is used to manage the state and data flow of a pane in a control client. It includes identifiers for the pane, offsets for written and queued data, flags to indicate the pane's state, and structures for managing pending operations and associated data blocks. The structure is integrated into a red-black tree and a tail queue to facilitate efficient management and access within the control system.


---
### control_sub_pane
- **Type**: `struct`
- **Members**:
    - `pane`: An unsigned integer representing the pane identifier.
    - `idx`: An unsigned integer representing the index of the pane.
    - `last`: A pointer to a character string, likely used to store the last state or value associated with the pane.
    - `entry`: A macro for Red-Black tree entry, used to link this structure into a Red-Black tree.
- **Description**: The `control_sub_pane` structure is used to represent a subscription to a specific pane in a control client context. It contains identifiers for the pane and its index, a string pointer for storing the last known state or value, and a Red-Black tree entry for efficient management and lookup within a collection of such structures. This structure is part of a larger system for managing client subscriptions to pane updates in a terminal multiplexer environment.


---
### control_sub_window
- **Type**: `struct`
- **Members**:
    - `window`: An unsigned integer representing the window identifier.
    - `idx`: An unsigned integer representing the index of the window.
    - `last`: A pointer to a character string, likely used to store the last state or value associated with the window.
    - `entry`: A Red-Black tree entry used to link this structure into a Red-Black tree of control_sub_window structures.
- **Description**: The `control_sub_window` structure is used to represent a subscription to a window in a control client context, likely within a terminal multiplexer like tmux. It contains identifiers for the window and its index, a pointer to a string for storing the last known state or value, and a Red-Black tree entry for organizing multiple subscriptions in a balanced tree structure. This allows efficient management and retrieval of window subscriptions.


---
### control_sub
- **Type**: `struct`
- **Members**:
    - `name`: A pointer to a character string representing the name of the subscription.
    - `format`: A pointer to a character string representing the format of the subscription.
    - `type`: An enumeration value indicating the type of the subscription.
    - `id`: An unsigned integer representing the identifier of the subscription.
    - `last`: A pointer to a character string representing the last known state or value of the subscription.
    - `panes`: A red-black tree structure containing subscription panes.
    - `windows`: A red-black tree structure containing subscription windows.
    - `entry`: A red-black tree entry for the control_sub structure.
- **Description**: The `control_sub` structure is used to represent a control client subscription in a system, such as a terminal multiplexer. It contains information about the subscription's name, format, type, and identifier, as well as the last known state or value. Additionally, it maintains red-black tree structures for managing associated panes and windows, allowing for efficient insertion, deletion, and lookup operations. The `entry` member is used to link this structure into a red-black tree, facilitating organized storage and retrieval of multiple subscriptions.


---
### control_state
- **Type**: `struct`
- **Members**:
    - `panes`: A red-black tree of control_pane structures representing the panes managed by the control state.
    - `pending_list`: A tail queue of control_pane structures that are pending for processing.
    - `pending_count`: An unsigned integer representing the count of pending panes in the pending_list.
    - `all_blocks`: A tail queue of control_block structures representing all data blocks managed by the control state.
    - `read_event`: A pointer to a bufferevent structure for handling read events.
    - `write_event`: A pointer to a bufferevent structure for handling write events.
    - `subs`: A red-black tree of control_sub structures representing the subscriptions managed by the control state.
    - `subs_timer`: An event structure used for managing subscription timers.
- **Description**: The `control_state` structure is a comprehensive data structure used to manage the state of control clients in a system, such as a terminal multiplexer. It maintains a collection of panes and data blocks, handles read and write events through bufferevents, and manages subscriptions with associated timers. The structure uses red-black trees and tail queues to efficiently organize and process panes and subscriptions, ensuring that data is correctly queued and processed for each client.


# Functions

---
### control_pane_cmp<!-- {{#callable:control_pane_cmp}} -->
The `control_pane_cmp` function compares two `control_pane` structures based on their `pane` member values.
- **Inputs**:
    - `cp1`: A pointer to the first `control_pane` structure to be compared.
    - `cp2`: A pointer to the second `control_pane` structure to be compared.
- **Control Flow**:
    - Check if the `pane` value of `cp1` is less than that of `cp2`; if true, return -1.
    - Check if the `pane` value of `cp1` is greater than that of `cp2`; if true, return 1.
    - If neither condition is met, return 0, indicating the `pane` values are equal.
- **Output**:
    - Returns an integer: -1 if `cp1`'s pane is less than `cp2`'s, 1 if greater, and 0 if they are equal.


---
### control_sub_cmp<!-- {{#callable:control_sub_cmp}} -->
The `control_sub_cmp` function compares two `control_sub` structures based on their `name` fields using the `strcmp` function.
- **Inputs**:
    - `csub1`: A pointer to the first `control_sub` structure to be compared.
    - `csub2`: A pointer to the second `control_sub` structure to be compared.
- **Control Flow**:
    - The function calls `strcmp` with the `name` fields of `csub1` and `csub2` as arguments.
    - The result of `strcmp` is returned directly, which indicates the lexicographical order of the two names.
- **Output**:
    - The function returns an integer less than, equal to, or greater than zero if the `name` of `csub1` is found, respectively, to be less than, to match, or be greater than the `name` of `csub2`.


---
### control_sub_pane_cmp<!-- {{#callable:control_sub_pane_cmp}} -->
The `control_sub_pane_cmp` function compares two `control_sub_pane` structures based on their `pane` and `idx` fields to determine their order.
- **Inputs**:
    - `csp1`: A pointer to the first `control_sub_pane` structure to be compared.
    - `csp2`: A pointer to the second `control_sub_pane` structure to be compared.
- **Control Flow**:
    - Check if the `pane` field of `csp1` is less than that of `csp2`; if true, return -1.
    - Check if the `pane` field of `csp1` is greater than that of `csp2`; if true, return 1.
    - If the `pane` fields are equal, check if the `idx` field of `csp1` is less than that of `csp2`; if true, return -1.
    - Check if the `idx` field of `csp1` is greater than that of `csp2`; if true, return 1.
    - If both `pane` and `idx` fields are equal, return 0.
- **Output**:
    - An integer indicating the order of the two `control_sub_pane` structures: -1 if `csp1` is less than `csp2`, 1 if `csp1` is greater than `csp2`, and 0 if they are equal.


---
### control_sub_window_cmp<!-- {{#callable:control_sub_window_cmp}} -->
The `control_sub_window_cmp` function compares two `control_sub_window` structures based on their `window` and `idx` fields to determine their ordering.
- **Inputs**:
    - `csw1`: A pointer to the first `control_sub_window` structure to be compared.
    - `csw2`: A pointer to the second `control_sub_window` structure to be compared.
- **Control Flow**:
    - Check if the `window` field of `csw1` is less than that of `csw2`; if true, return -1.
    - Check if the `window` field of `csw1` is greater than that of `csw2`; if true, return 1.
    - If the `window` fields are equal, check if the `idx` field of `csw1` is less than that of `csw2`; if true, return -1.
    - Check if the `idx` field of `csw1` is greater than that of `csw2`; if true, return 1.
    - If both `window` and `idx` fields are equal, return 0.
- **Output**:
    - An integer indicating the relative order of the two `control_sub_window` structures: -1 if `csw1` is less than `csw2`, 1 if `csw1` is greater than `csw2`, and 0 if they are equal.


---
### control_free_sub<!-- {{#callable:control_free_sub}} -->
The `control_free_sub` function deallocates and removes a control subscription and its associated panes and windows from the control state.
- **Inputs**:
    - `cs`: A pointer to the `control_state` structure, representing the current state of control clients.
    - `csub`: A pointer to the `control_sub` structure, representing the subscription to be freed.
- **Control Flow**:
    - Iterate over each `control_sub_pane` in the `csub->panes` red-black tree using `RB_FOREACH_SAFE` to safely remove and free each pane.
    - Iterate over each `control_sub_window` in the `csub->windows` red-black tree using `RB_FOREACH_SAFE` to safely remove and free each window.
    - Free the `last` string in the `csub` structure.
    - Remove the `csub` from the `cs->subs` red-black tree.
    - Free the `name` and `format` strings in the `csub` structure.
    - Free the `csub` structure itself.
- **Output**:
    - The function does not return any value; it performs cleanup by freeing memory and removing elements from data structures.


---
### control_free_block<!-- {{#callable:control_free_block}} -->
The `control_free_block` function deallocates memory associated with a `control_block` and removes it from a linked list of all blocks in a `control_state`.
- **Inputs**:
    - `cs`: A pointer to a `control_state` structure, which manages the state of control clients and their associated data blocks.
    - `cb`: A pointer to a `control_block` structure, representing a block of data that needs to be freed and removed from the control state's list of all blocks.
- **Control Flow**:
    - The function first frees the memory allocated for the `line` member of the `control_block` pointed to by `cb`.
    - It then removes the `control_block` from the `all_blocks` queue in the `control_state` pointed to by `cs` using the `TAILQ_REMOVE` macro.
    - Finally, it frees the memory allocated for the `control_block` itself.
- **Output**:
    - The function does not return any value; it performs memory deallocation and list removal operations.


---
### control_get_pane<!-- {{#callable:control_get_pane}} -->
The `control_get_pane` function retrieves a `control_pane` structure associated with a specific `window_pane` for a given client.
- **Inputs**:
    - `c`: A pointer to a `client` structure, representing the client for which the pane is being retrieved.
    - `wp`: A pointer to a `window_pane` structure, representing the window pane whose associated control pane is being retrieved.
- **Control Flow**:
    - Retrieve the `control_state` from the client `c`.
    - Create a temporary `control_pane` structure `cp` with the `pane` field set to the ID of the `window_pane` `wp`.
    - Use `RB_FIND` to search for a `control_pane` in the client's `control_state` that matches the temporary `control_pane` `cp`.
    - Return the found `control_pane` or `NULL` if no match is found.
- **Output**:
    - Returns a pointer to the `control_pane` structure associated with the specified `window_pane`, or `NULL` if no such pane exists.


---
### control_add_pane<!-- {{#callable:control_add_pane}} -->
The `control_add_pane` function adds a new control pane to a client's control state if it doesn't already exist, initializing its properties and inserting it into the control pane tree.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the control pane is being added.
    - `wp`: A pointer to a `window_pane` structure representing the window pane to be added to the client's control state.
- **Control Flow**:
    - Retrieve the control state from the client structure `c`.
    - Call [`control_get_pane`](#control_get_pane) to check if the pane `wp` already exists in the client's control state.
    - If the pane exists, return the existing control pane.
    - If the pane does not exist, allocate memory for a new `control_pane` structure using `xcalloc`.
    - Set the `pane` field of the new control pane to the ID of the window pane `wp`.
    - Insert the new control pane into the client's control pane tree using `RB_INSERT`.
    - Copy the `offset` and `queued` fields from the window pane `wp` to the new control pane.
    - Initialize the `blocks` queue of the new control pane using `TAILQ_INIT`.
    - Return the newly created control pane.
- **Output**:
    - Returns a pointer to the `control_pane` structure that was either found or newly created.
- **Functions called**:
    - [`control_get_pane`](#control_get_pane)


---
### control_discard_pane<!-- {{#callable:control_discard_pane}} -->
The `control_discard_pane` function removes and frees all control blocks associated with a specific control pane for a client.
- **Inputs**:
    - `c`: A pointer to a `client` structure, representing the client whose control pane blocks are to be discarded.
    - `cp`: A pointer to a `control_pane` structure, representing the control pane whose blocks are to be discarded.
- **Control Flow**:
    - Retrieve the `control_state` from the client `c`.
    - Iterate over each `control_block` in the `blocks` queue of the `control_pane` `cp` using `TAILQ_FOREACH_SAFE` to safely remove elements during iteration.
    - For each `control_block`, remove it from the `blocks` queue of `cp` using `TAILQ_REMOVE`.
    - Free the memory associated with each `control_block` by calling [`control_free_block`](#control_free_block).
- **Output**:
    - The function does not return any value; it performs its operations directly on the provided data structures.
- **Functions called**:
    - [`control_free_block`](#control_free_block)


---
### control_window_pane<!-- {{#callable:control_window_pane}} -->
The `control_window_pane` function retrieves a `window_pane` associated with a given client and pane ID, ensuring the pane is part of the client's session.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the window pane is being retrieved.
    - `pane`: An unsigned integer representing the ID of the pane to be retrieved.
- **Control Flow**:
    - Check if the client's session is NULL; if so, return NULL.
    - Attempt to find the window pane by its ID using `window_pane_find_by_id`; if not found, return NULL.
    - Check if the window associated with the found pane is part of the client's session using `winlink_find_by_window`; if not, return NULL.
    - If all checks pass, return the found `window_pane`.
- **Output**:
    - Returns a pointer to the `window_pane` if it is valid and part of the client's session, otherwise returns NULL.


---
### control_reset_offsets<!-- {{#callable:control_reset_offsets}} -->
The `control_reset_offsets` function resets the control state of a client by clearing all pane offsets and resetting the pending list.
- **Inputs**:
    - `c`: A pointer to a `struct client` which contains the control state to be reset.
- **Control Flow**:
    - Retrieve the control state (`cs`) from the client (`c`).
    - Iterate over each `control_pane` in the `cs->panes` red-black tree using `RB_FOREACH_SAFE`.
    - Remove each `control_pane` from the tree and free its memory.
    - Initialize the `cs->pending_list` as an empty tail queue.
    - Set `cs->pending_count` to 0.
- **Output**:
    - The function does not return any value; it modifies the control state of the client in place.


---
### control_pane_offset<!-- {{#callable:control_pane_offset}} -->
The `control_pane_offset` function retrieves the offset of a window pane for a client, considering various control flags and buffer conditions.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the pane offset is being retrieved.
    - `wp`: A pointer to a `window_pane` structure representing the specific window pane for which the offset is being retrieved.
    - `off`: A pointer to an integer where the function will store the offset status (0 or 1).
- **Control Flow**:
    - Check if the client has the `CLIENT_CONTROL_NOOUTPUT` flag set; if so, set `*off` to 0 and return `NULL`.
    - Retrieve the control pane associated with the client and window pane using [`control_get_pane`](#control_get_pane).
    - If the control pane is `NULL` or has the `CONTROL_PANE_PAUSED` flag set, set `*off` to 0 and return `NULL`.
    - If the control pane has the `CONTROL_PANE_OFF` flag set, set `*off` to 1 and return `NULL`.
    - Calculate the offset status by checking if the length of the output buffer in the client's control state is greater than or equal to `CONTROL_BUFFER_LOW`, and store the result in `*off`.
    - Return a pointer to the `offset` field of the control pane.
- **Output**:
    - A pointer to the `window_pane_offset` structure of the control pane, or `NULL` if certain conditions are met.
- **Functions called**:
    - [`control_get_pane`](#control_get_pane)


---
### control_set_pane_on<!-- {{#callable:control_set_pane_on}} -->
The `control_set_pane_on` function enables a control pane for a client by clearing the CONTROL_PANE_OFF flag and updating its offset and queued data from the corresponding window pane.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the pane is being controlled.
    - `wp`: A pointer to a `window_pane` structure representing the window pane to be set on for the client.
- **Control Flow**:
    - Retrieve the control pane associated with the client `c` and window pane `wp` using [`control_get_pane`](#control_get_pane) function.
    - Check if the retrieved control pane `cp` is not NULL and has the `CONTROL_PANE_OFF` flag set.
    - If both conditions are true, clear the `CONTROL_PANE_OFF` flag from `cp->flags`.
    - Copy the `offset` and `queued` data from the window pane `wp` to the control pane `cp`.
- **Output**:
    - The function does not return any value; it modifies the state of the control pane associated with the client and window pane.
- **Functions called**:
    - [`control_get_pane`](#control_get_pane)


---
### control_set_pane_off<!-- {{#callable:control_set_pane_off}} -->
The `control_set_pane_off` function sets a specified pane to an 'off' state for a given client.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the pane is being set to 'off'.
    - `wp`: A pointer to a `window_pane` structure representing the pane that is to be set to 'off'.
- **Control Flow**:
    - Call [`control_add_pane`](#control_add_pane) with the client and window pane to ensure the pane is added to the client's control state.
    - Set the `CONTROL_PANE_OFF` flag on the returned `control_pane` structure to mark the pane as 'off'.
- **Output**:
    - The function does not return a value; it modifies the state of the pane within the client's control state.
- **Functions called**:
    - [`control_add_pane`](#control_add_pane)


---
### control_continue_pane<!-- {{#callable:control_continue_pane}} -->
The `control_continue_pane` function resumes a paused control pane for a given client and window pane, updating its offsets and sending a continue message.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client associated with the control pane.
    - `wp`: A pointer to a `window_pane` structure representing the window pane to be continued.
- **Control Flow**:
    - Retrieve the control pane associated with the client `c` and window pane `wp` using [`control_get_pane`](#control_get_pane) function.
    - Check if the control pane exists and is currently paused by evaluating the `CONTROL_PANE_PAUSED` flag.
    - If the pane is paused, clear the `CONTROL_PANE_PAUSED` flag to resume the pane.
    - Copy the current offset and queued data from the window pane `wp` to the control pane `cp`.
    - Send a continue message to the client using [`control_write`](#control_write) function, indicating the pane ID.
- **Output**:
    - The function does not return a value; it performs operations to resume a paused control pane and sends a message to the client.
- **Functions called**:
    - [`control_get_pane`](#control_get_pane)
    - [`control_write`](#control_write)


---
### control_pause_pane<!-- {{#callable:control_pause_pane}} -->
The `control_pause_pane` function pauses a specific pane for a client by setting a flag and discarding its output.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the pane is to be paused.
    - `wp`: A pointer to a `window_pane` structure representing the pane to be paused.
- **Control Flow**:
    - Retrieve or add a `control_pane` structure for the given client and window pane using [`control_add_pane`](#control_add_pane).
    - Check if the `CONTROL_PANE_PAUSED` flag is not set in the `control_pane` structure's flags.
    - If the pane is not already paused, set the `CONTROL_PANE_PAUSED` flag in the `control_pane` structure's flags.
    - Discard any output for the pane using [`control_discard_pane`](#control_discard_pane).
    - Write a pause message to the client using [`control_write`](#control_write).
- **Output**:
    - The function does not return a value; it modifies the state of the pane and client by pausing the pane and discarding its output.
- **Functions called**:
    - [`control_add_pane`](#control_add_pane)
    - [`control_discard_pane`](#control_discard_pane)
    - [`control_write`](#control_write)


---
### control_write<!-- {{#callable:control_write}} -->
The `control_write` function formats a message and either writes it directly to a client's output buffer or stores it in a queue for later writing, depending on the state of the client's control blocks.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client to which the message is to be written.
    - `fmt`: A format string similar to `printf` that specifies how the subsequent arguments are formatted.
    - `...`: A variable number of arguments that are formatted according to the `fmt` string.
- **Control Flow**:
    - Initialize a variable argument list `ap` using `va_start` with the format string `fmt`.
    - Check if the `all_blocks` queue in the client's control state is empty.
    - If the queue is empty, call `control_vwrite` to write the formatted message directly to the client's output buffer and return.
    - If the queue is not empty, allocate memory for a new `control_block` and format the message into `cb->line` using `xvasprintf`.
    - Insert the new `control_block` at the tail of the `all_blocks` queue.
    - Set the timestamp `cb->t` for the new block using `get_timer()`.
    - Log the action of storing the line for debugging purposes.
    - Enable the write event on the client's `write_event` buffer to ensure the message is eventually written out.
    - End the variable argument list using `va_end`.
- **Output**:
    - The function does not return a value; it writes a formatted message to a client's output buffer or stores it in a queue for later writing.


---
### control_check_age<!-- {{#callable:control_check_age}} -->
The `control_check_age` function checks the age of the first control block in a pane's queue and takes action based on the client's control flags and the block's age.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client for which the age check is being performed.
    - `wp`: A pointer to a `struct window_pane` representing the window pane associated with the control pane.
    - `cp`: A pointer to a `struct control_pane` representing the control pane whose blocks are being checked for age.
- **Control Flow**:
    - Retrieve the first control block from the control pane's block queue.
    - If the block is NULL, return 0 indicating no action is needed.
    - Get the current time using `get_timer()` and compare it with the block's timestamp.
    - If the block's timestamp is greater than or equal to the current time, return 0 indicating no action is needed.
    - Calculate the age of the block by subtracting the block's timestamp from the current time.
    - Log the age of the block for debugging purposes.
    - If the client has the `CLIENT_CONTROL_PAUSEAFTER` flag set, check if the block's age is less than the client's pause age; if so, return 0.
    - If the block's age is greater than or equal to the client's pause age, set the control pane's paused flag, discard the pane's blocks, and write a pause message to the client.
    - If the client does not have the `CLIENT_CONTROL_PAUSEAFTER` flag set, check if the block's age is less than `CONTROL_MAXIMUM_AGE`; if so, return 0.
    - If the block's age is greater than or equal to `CONTROL_MAXIMUM_AGE`, set the client's exit message to "too far behind", set the client's exit flag, and discard the client's control state.
    - Return 1 indicating that an action was taken.
- **Output**:
    - Returns an integer indicating whether an action was taken (1) or not (0).
- **Functions called**:
    - [`control_discard_pane`](#control_discard_pane)
    - [`control_write`](#control_write)
    - [`control_discard`](#control_discard)


---
### control_write_output<!-- {{#callable:control_write_output}} -->
The `control_write_output` function manages the output data flow from a window pane to a client, handling conditions such as pane state and data queuing.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client to which the output is being written.
    - `wp`: A pointer to a `window_pane` structure representing the pane from which the output data is being retrieved.
- **Control Flow**:
    - Check if the window pane is part of the client's session; if not, return immediately.
    - If the client has flags set to ignore output, check if the pane is already being ignored and return if so.
    - Add the pane to the client's control state if not already present.
    - Check if the pane is marked as off or paused; if so, log and ignore the pane.
    - Check if the pane's data is too old; if so, return without processing.
    - Retrieve new data from the window pane and update the queued data size; if no new data, return.
    - Allocate a new control block for the data, insert it into the client's all_blocks queue, and log the new block.
    - If the pane is not already pending, mark it as pending, add it to the pending list, and enable the write event for the client.
- **Output**:
    - The function does not return a value but modifies the client's control state by queuing new data blocks for output and managing pane states.
- **Functions called**:
    - [`control_get_pane`](#control_get_pane)
    - [`control_add_pane`](#control_add_pane)
    - [`control_check_age`](#control_check_age)


---
### control_error<!-- {{#callable:control_error}} -->
The `control_error` function handles error reporting for a control client by logging a parse error message and cleaning up resources.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, representing the command queue item associated with the error.
    - `data`: A pointer to a character string containing the error message to be reported.
- **Control Flow**:
    - Retrieve the client associated with the command queue item using `cmdq_get_client`.
    - Begin a guarded section with `cmdq_guard` using the label 'begin'.
    - Write the error message to the control client using [`control_write`](#control_write), formatted as 'parse error: %s'.
    - End the guarded section with `cmdq_guard` using the label 'error'.
    - Free the memory allocated for the error message string.
    - Return `CMD_RETURN_NORMAL` to indicate normal command completion.
- **Output**:
    - The function returns an `enum cmd_retval` value, specifically `CMD_RETURN_NORMAL`, indicating that the command completed normally.
- **Functions called**:
    - [`control_write`](#control_write)


---
### control_error_callback<!-- {{#callable:control_error_callback}} -->
The `control_error_callback` function sets a client's exit flag when an error occurs in a bufferevent.
- **Inputs**:
    - `bufev`: A pointer to a bufferevent structure, which is unused in this function.
    - `what`: A short integer indicating the type of event that occurred, which is unused in this function.
    - `data`: A pointer to a client structure, which is used to access and modify the client's flags.
- **Control Flow**:
    - The function retrieves the client structure from the data pointer.
    - It sets the CLIENT_EXIT flag in the client's flags to indicate that the client should exit.
- **Output**:
    - The function does not return any value; it modifies the client's flags to signal an exit.


---
### control_read_callback<!-- {{#callable:control_read_callback}} -->
The `control_read_callback` function processes input lines from a buffer, executing commands or detaching the client if an empty line is encountered.
- **Inputs**:
    - `bufev`: A pointer to a `bufferevent` structure, which is unused in this function.
    - `data`: A pointer to a `client` structure, representing the client associated with the control state.
- **Control Flow**:
    - The function enters an infinite loop to process lines from the buffer.
    - It reads a line from the buffer using `evbuffer_readln` and checks if the line is `NULL`, breaking the loop if so.
    - If the line is empty, it frees the line, sets the `CLIENT_EXIT` flag on the client, and breaks the loop.
    - For non-empty lines, it creates a new command queue state and attempts to parse and append the command using `cmd_parse_and_append`.
    - If a parse error occurs, it appends an error callback to the command queue.
    - The line is freed after processing.
- **Output**:
    - The function does not return a value; it modifies the client's state and command queue based on the input lines.


---
### control_all_done<!-- {{#callable:control_all_done}} -->
The `control_all_done` function checks if a control client has completed all its data output tasks.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the control client whose output status is being checked.
- **Control Flow**:
    - Retrieve the `control_state` associated with the client `c`.
    - Check if the `all_blocks` queue in the `control_state` is not empty; if it is not empty, return 0 indicating that there are still blocks to be processed.
    - If the `all_blocks` queue is empty, check if the length of the output buffer in the `write_event` is zero; return the result of this check.
- **Output**:
    - Returns an integer: 0 if there are still blocks to be processed or if the output buffer is not empty, otherwise returns 1 indicating all output tasks are done.


---
### control_flush_all_blocks<!-- {{#callable:control_flush_all_blocks}} -->
The `control_flush_all_blocks` function flushes all control blocks with zero size from a client's control state, writing their contents to a buffer event and freeing the blocks.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client whose control blocks are to be flushed.
- **Control Flow**:
    - Retrieve the control state from the client structure.
    - Iterate over all control blocks in the control state's `all_blocks` queue using `TAILQ_FOREACH_SAFE`.
    - For each block, check if its size is non-zero; if so, break the loop.
    - If the block's size is zero, log a debug message indicating the line being flushed.
    - Write the block's line to the control state's write event buffer, followed by a newline character.
    - Free the control block using [`control_free_block`](#control_free_block).
- **Output**:
    - The function does not return a value; it performs operations on the client's control state and write event buffer.
- **Functions called**:
    - [`control_free_block`](#control_free_block)


---
### control_append_data<!-- {{#callable:control_append_data}} -->
The `control_append_data` function appends new data from a window pane to an event buffer, formatting it for control output.
- **Inputs**:
    - `c`: A pointer to the client structure, representing the client for which data is being appended.
    - `cp`: A pointer to the control pane structure, which contains information about the pane's data offsets.
    - `age`: A 64-bit unsigned integer representing the age of the data block being processed.
    - `message`: A pointer to an evbuffer structure where the data will be appended; if NULL, a new evbuffer is created.
    - `wp`: A pointer to the window pane structure from which new data is retrieved.
    - `size`: The size of the data to be appended, in bytes.
- **Control Flow**:
    - Check if the `message` buffer is NULL; if so, create a new evbuffer and add a formatted header based on the client's flags.
    - Retrieve new data from the window pane using `window_pane_get_new_data`, updating the control pane's offset and getting the size of the new data.
    - Verify that the new data size is at least as large as the requested size; if not, terminate with an error.
    - Iterate over the new data, appending each byte to the `message` buffer, escaping non-printable characters and backslashes.
    - Update the window pane's used data offset with the size of the data appended.
    - Return the `message` buffer containing the appended data.
- **Output**:
    - Returns a pointer to the evbuffer containing the appended data.


---
### control_write_data<!-- {{#callable:control_write_data}} -->
The `control_write_data` function appends a newline to a message buffer, writes it to a client's write event buffer, and then frees the message buffer.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client to which the message is being written.
    - `message`: A pointer to an `evbuffer` structure containing the message data to be written to the client.
- **Control Flow**:
    - Retrieve the control state from the client structure.
    - Log the message being written using the `log_debug` function.
    - Append a newline character to the message buffer using `evbuffer_add`.
    - Write the message buffer to the client's write event buffer using `bufferevent_write_buffer`.
    - Free the message buffer using `evbuffer_free`.
- **Output**:
    - The function does not return a value; it performs operations on the provided message buffer and client structure.


---
### control_write_pending<!-- {{#callable:control_write_pending}} -->
The `control_write_pending` function manages the writing of pending data blocks from a control pane to a client, ensuring data is written up to a specified limit and handling block removal and flushing as necessary.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client to which data is being written.
    - `cp`: A pointer to a `control_pane` structure representing the pane from which data blocks are being written.
    - `limit`: A `size_t` value specifying the maximum amount of data to write in this operation.
- **Control Flow**:
    - Retrieve the control state from the client and initialize necessary variables.
    - Check if the window pane associated with the control pane is valid; if not, remove all blocks from the pane and flush all blocks, then return 0.
    - Iterate over the blocks in the control pane while the used size is less than the limit and there are blocks remaining.
    - For each block, check its age and decide whether to continue processing or break out of the loop if the age check fails.
    - Calculate the size of data to write from the current block, ensuring it does not exceed the remaining limit.
    - Append the data to the message buffer and update the used size and block size accordingly.
    - If a block is fully written, remove it from the queue and free it; if the first block in the all_blocks queue is also empty, write the message and flush all blocks.
    - After processing, if there is a message buffer, write its data to the client.
    - Return whether there are still blocks remaining in the control pane.
- **Output**:
    - Returns an integer indicating whether there are still blocks remaining in the control pane (non-zero if there are, zero if not).
- **Functions called**:
    - [`control_window_pane`](#control_window_pane)
    - [`control_free_block`](#control_free_block)
    - [`control_flush_all_blocks`](#control_flush_all_blocks)
    - [`control_check_age`](#control_check_age)
    - [`control_append_data`](#control_append_data)
    - [`control_write_data`](#control_write_data)


---
### control_write_callback<!-- {{#callable:control_write_callback}} -->
The `control_write_callback` function manages the writing of data to a client's buffer, ensuring that data is written in chunks while respecting buffer limits and pending data constraints.
- **Inputs**:
    - `bufev`: A pointer to a `bufferevent` structure, which is unused in this function.
    - `data`: A pointer to a `client` structure, representing the client for which data is being written.
- **Control Flow**:
    - The function begins by flushing all blocks of data that are ready to be written using `control_flush_all_blocks(c)`.
    - It enters a loop that continues as long as the length of the event buffer (`evb`) is less than `CONTROL_BUFFER_HIGH`.
    - Within the loop, it checks if there are any pending data blocks (`cs->pending_count == 0`), and breaks the loop if none are pending.
    - Calculates the available space in the buffer (`space`) and determines a `limit` for how much data can be written per pane, ensuring it is not less than `CONTROL_WRITE_MINIMUM`.
    - Iterates over the pending list of control panes (`cs->pending_list`) using `TAILQ_FOREACH_SAFE`, attempting to write pending data for each pane using `control_write_pending(c, cp, limit)`.
    - If a pane's data is fully written, it is removed from the pending list, and its pending flag is cleared.
    - If the event buffer is empty after processing, the write event is disabled using `bufferevent_disable(cs->write_event, EV_WRITE)`.
- **Output**:
    - The function does not return a value; it operates by modifying the state of the client's control structures and the event buffer.
- **Functions called**:
    - [`control_flush_all_blocks`](#control_flush_all_blocks)
    - [`control_write_pending`](#control_write_pending)


---
### control_start<!-- {{#callable:control_start}} -->
The `control_start` function initializes the control state for a client, setting up non-blocking I/O and event handling for control mode operations.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the control state is being initialized.
- **Control Flow**:
    - Check if the client is in control mode by evaluating the `CLIENT_CONTROLCONTROL` flag in `c->flags`.
    - If the client is in control mode, close the `out_fd` file descriptor and set it to -1; otherwise, set `out_fd` to non-blocking mode.
    - Set the main file descriptor `fd` to non-blocking mode.
    - Allocate memory for a new `control_state` structure and assign it to `c->control_state`.
    - Initialize the `panes`, `pending_list`, `all_blocks`, and `subs` data structures within the `control_state`.
    - Create a new `bufferevent` for reading using `c->fd` and assign it to `cs->read_event`. If allocation fails, terminate the program with an error message.
    - If the client is in control mode, set `cs->write_event` to `cs->read_event`; otherwise, create a new `bufferevent` for writing using `c->out_fd` and assign it to `cs->write_event`. If allocation fails, terminate the program with an error message.
    - Set the low watermark for the write event to `CONTROL_BUFFER_LOW`.
    - If the client is in control mode, write a specific control sequence to `cs->write_event` and enable the write event.
- **Output**:
    - The function does not return a value; it initializes the control state for the client, setting up necessary events and configurations for control mode.


---
### control_ready<!-- {{#callable:control_ready}} -->
The `control_ready` function enables the read event for a client's control state, allowing it to start processing incoming data.
- **Inputs**:
    - `c`: A pointer to a `client` structure, representing the client whose control state read event is to be enabled.
- **Control Flow**:
    - The function takes a `client` structure pointer as an argument.
    - It accesses the `control_state` member of the `client` structure to retrieve the `read_event`.
    - The `bufferevent_enable` function is called with the `read_event` and `EV_READ` flag to enable reading.
- **Output**:
    - The function does not return any value; it modifies the state of the client's control event to enable reading.


---
### control_discard<!-- {{#callable:control_discard}} -->
The `control_discard` function discards all output for a given client by iterating over its control panes and disabling the read event.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client whose output is to be discarded.
- **Control Flow**:
    - Retrieve the control state associated with the client `c`.
    - Iterate over each control pane in the client's control state using the `RB_FOREACH` macro.
    - For each control pane, call [`control_discard_pane`](#control_discard_pane) to discard its output.
    - Disable the read event associated with the client's control state.
- **Output**:
    - The function does not return a value; it performs operations to discard output and disable reading for the specified client.
- **Functions called**:
    - [`control_discard_pane`](#control_discard_pane)


---
### control_stop<!-- {{#callable:control_stop}} -->
The `control_stop` function terminates the control mode for a client by freeing associated resources and resetting states.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which control mode is being stopped.
- **Control Flow**:
    - Retrieve the `control_state` associated with the client `c`.
    - If the client is not in control-control mode, free the `write_event` buffer event.
    - Free the `read_event` buffer event.
    - Iterate over all control subscriptions in `cs->subs` and free each using [`control_free_sub`](#control_free_sub).
    - If the subscription timer is initialized, delete it using `evtimer_del`.
    - Iterate over all control blocks in `cs->all_blocks` and free each using [`control_free_block`](#control_free_block).
    - Reset the control offsets for the client using [`control_reset_offsets`](#control_reset_offsets).
    - Free the `control_state` structure `cs`.
- **Output**:
    - The function does not return any value; it performs cleanup operations on the client's control state.
- **Functions called**:
    - [`control_free_sub`](#control_free_sub)
    - [`control_free_block`](#control_free_block)
    - [`control_reset_offsets`](#control_reset_offsets)


---
### control_check_subs_session<!-- {{#callable:control_check_subs_session}} -->
The `control_check_subs_session` function checks if the session subscription format has changed and writes a notification if it has.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the subscription is being checked.
    - `csub`: A pointer to a `control_sub` structure representing the subscription to be checked.
- **Control Flow**:
    - Retrieve the session associated with the client `c`.
    - Create a format tree with default settings using the client and session information.
    - Expand the subscription format using the format tree to get the current value.
    - Free the format tree after use.
    - Check if the current value is the same as the last recorded value for the subscription.
    - If the values are the same, free the current value and return without further action.
    - If the values differ, write a subscription change notification to the client.
    - Free the last recorded value and update it with the current value.
- **Output**:
    - The function does not return a value but writes a notification to the client if the subscription format has changed.
- **Functions called**:
    - [`control_write`](#control_write)


---
### control_check_subs_pane<!-- {{#callable:control_check_subs_pane}} -->
The `control_check_subs_pane` function checks for changes in a specific pane's subscription and updates the client if any changes are detected.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the subscription is being checked.
    - `csub`: A pointer to a `control_sub` structure representing the subscription details for the pane being checked.
- **Control Flow**:
    - Retrieve the session associated with the client `c` and find the window pane using `csub->id`.
    - If the window pane is not found or its file descriptor is invalid, return immediately.
    - Iterate over each winlink in the window's winlinks list.
    - For each winlink, check if it belongs to the same session as the client; if not, continue to the next winlink.
    - Create a format tree with default values and expand the format string from `csub` using this format tree.
    - Search for an existing `control_sub_pane` entry in `csub->panes` using the pane ID and winlink index.
    - If no entry is found, allocate a new `control_sub_pane` structure, initialize it, and insert it into the `csub->panes` tree.
    - Compare the newly expanded format value with the last stored value in the `control_sub_pane` structure.
    - If the values are different, write a subscription change message to the client and update the last stored value.
- **Output**:
    - The function does not return a value; it writes a subscription change message to the client if a change is detected.
- **Functions called**:
    - [`control_write`](#control_write)


---
### control_check_subs_all_panes<!-- {{#callable:control_check_subs_all_panes}} -->
The function `control_check_subs_all_panes` checks all panes in a client's session for changes in subscription values and updates the client if any changes are detected.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client whose subscriptions are being checked.
    - `csub`: A pointer to a `control_sub` structure representing the subscription details, including the format string to be evaluated for each pane.
- **Control Flow**:
    - Iterate over all window links (`winlink`) in the client's session's windows.
    - For each window link, iterate over all panes (`window_pane`) in the window.
    - Create a format tree with default values for the current client, session, window link, and pane.
    - Expand the format string from the subscription using the format tree to get the current value.
    - Free the format tree after use.
    - Create a `control_sub_pane` structure to find or insert in the subscription's pane tree, using the pane ID and window link index.
    - Check if a `control_sub_pane` entry already exists in the subscription's pane tree for the current pane and window link index.
    - If no entry exists, allocate a new `control_sub_pane` structure, set its pane ID and index, and insert it into the subscription's pane tree.
    - Compare the current value with the last recorded value for the pane; if they are the same, free the current value and continue to the next pane.
    - If the values differ, write a subscription change message to the client, update the last recorded value for the pane, and free the previous last value.
- **Output**:
    - The function does not return a value; it performs operations to update the client's subscription state and sends messages to the client if changes are detected.
- **Functions called**:
    - [`control_write`](#control_write)


---
### control_check_subs_window<!-- {{#callable:control_check_subs_window}} -->
The `control_check_subs_window` function checks for changes in a specific window's subscription and updates the client if any changes are detected.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the subscription is being checked.
    - `csub`: A pointer to a `control_sub` structure representing the subscription details for the window being checked.
- **Control Flow**:
    - Retrieve the session associated with the client `c`.
    - Find the window by its ID using `csub->id`; if not found, return immediately.
    - Iterate over each `winlink` in the window's `winlinks` list.
    - For each `winlink`, check if it belongs to the client's session; if not, continue to the next `winlink`.
    - Create a format tree with default values and expand the format string from `csub->format`.
    - Search for an existing `control_sub_window` entry in `csub->windows` using the window ID and index.
    - If no entry is found, allocate a new `control_sub_window`, initialize it, and insert it into the `csub->windows` tree.
    - Compare the newly expanded format value with the last stored value in the `control_sub_window`.
    - If the values differ, write a subscription change message to the client and update the last stored value.
- **Output**:
    - The function does not return a value but writes a subscription change message to the client if a change is detected.
- **Functions called**:
    - [`control_write`](#control_write)


---
### control_check_subs_all_windows<!-- {{#callable:control_check_subs_all_windows}} -->
The function `control_check_subs_all_windows` checks all windows in a client's session for changes based on a subscription format and updates the subscription state if changes are detected.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client whose session's windows are being checked.
    - `csub`: A pointer to a `control_sub` structure representing the subscription that contains the format to be checked against each window.
- **Control Flow**:
    - Iterate over each window link (`wl`) in the client's session's windows using `RB_FOREACH`.
    - For each window link, retrieve the associated window (`w`).
    - Create a format tree (`ft`) with default settings using `format_create_defaults` and expand the subscription format (`csub->format`) into a string (`value`).
    - Free the format tree (`ft`) after use.
    - Create a `control_sub_window` structure (`find`) with the current window's ID and index.
    - Search for an existing `control_sub_window` in the subscription's windows tree using `RB_FIND`.
    - If no existing `control_sub_window` is found, allocate a new one, set its window ID and index, and insert it into the subscription's windows tree using `RB_INSERT`.
    - If the last stored value in the `control_sub_window` (`csw->last`) is not NULL and matches the newly expanded value, free the new value and continue to the next window.
    - If the values differ, write a subscription change message using [`control_write`](#control_write), free the old value, and update `csw->last` with the new value.
- **Output**:
    - The function does not return a value; it updates the subscription state and writes messages to the client if changes are detected.
- **Functions called**:
    - [`control_write`](#control_write)


---
### control_check_subs_timer<!-- {{#callable:control_check_subs_timer}} -->
The `control_check_subs_timer` function is a timer callback that periodically checks and updates the state of client subscriptions in a control system.
- **Inputs**:
    - `fd`: An unused integer file descriptor parameter.
    - `events`: An unused short integer representing event flags.
    - `data`: A pointer to a `client` structure, which contains the control state and subscriptions to be checked.
- **Control Flow**:
    - Log a debug message indicating the timer has fired.
    - Re-add the timer event to trigger again after 1 second.
    - Iterate over each subscription in the client's control state using a safe iteration macro.
    - For each subscription, check its type and call the corresponding function to handle updates for that subscription type (session, pane, all panes, window, or all windows).
- **Output**:
    - The function does not return a value; it operates by side effects, updating the state of subscriptions and scheduling the next timer event.
- **Functions called**:
    - [`control_check_subs_session`](#control_check_subs_session)
    - [`control_check_subs_pane`](#control_check_subs_pane)
    - [`control_check_subs_all_panes`](#control_check_subs_all_panes)
    - [`control_check_subs_window`](#control_check_subs_window)
    - [`control_check_subs_all_windows`](#control_check_subs_all_windows)


---
### control_add_sub<!-- {{#callable:control_add_sub}} -->
The `control_add_sub` function manages the addition of a subscription to a client's control state, ensuring that any existing subscription with the same name is replaced and that the subscription timer is set up.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the subscription is being added.
    - `name`: A constant character pointer representing the name of the subscription.
    - `type`: An enumeration value of type `control_sub_type` indicating the type of subscription.
    - `id`: An integer representing the identifier for the subscription, which could be a pane or window ID depending on the subscription type.
    - `format`: A constant character pointer representing the format string for the subscription.
- **Control Flow**:
    - Retrieve the control state from the client structure `c`.
    - Create a `control_sub` structure `find` and set its `name` field to the provided `name` argument.
    - Check if a subscription with the same name already exists in the client's control state using `RB_FIND`.
    - If an existing subscription is found, free it using [`control_free_sub`](#control_free_sub).
    - Allocate memory for a new `control_sub` structure and initialize its fields with the provided arguments.
    - Insert the new subscription into the client's control state using `RB_INSERT`.
    - Initialize the `panes` and `windows` red-black trees for the new subscription.
    - Check if the subscription timer is initialized; if not, set it up with `evtimer_set`.
    - Ensure the subscription timer is active by adding it with `evtimer_add` if it is not already pending.
- **Output**:
    - The function does not return a value; it modifies the client's control state by adding or replacing a subscription and ensuring the subscription timer is active.
- **Functions called**:
    - [`control_free_sub`](#control_free_sub)


---
### control_remove_sub<!-- {{#callable:control_remove_sub}} -->
The `control_remove_sub` function removes a subscription from a client's control state and stops the subscription timer if no subscriptions remain.
- **Inputs**:
    - `c`: A pointer to a `struct client`, representing the client whose subscription is to be removed.
    - `name`: A constant character pointer representing the name of the subscription to be removed.
- **Control Flow**:
    - Retrieve the control state from the client structure.
    - Create a `control_sub` structure `find` with the `name` to search for the subscription.
    - Use `RB_FIND` to locate the subscription in the client's subscription tree using the `find` structure.
    - If the subscription is found, call [`control_free_sub`](#control_free_sub) to free the subscription resources.
    - Check if the subscription tree is empty using `RB_EMPTY`, and if so, delete the subscription timer using `evtimer_del`.
- **Output**:
    - The function does not return a value; it performs operations to remove a subscription and manage the subscription timer.
- **Functions called**:
    - [`control_free_sub`](#control_free_sub)


