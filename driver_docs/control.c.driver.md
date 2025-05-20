# Purpose
This C source code file is part of the tmux project, specifically handling the control mode functionality for tmux clients. The code is designed to manage the communication between tmux and its clients, allowing clients to receive updates and send commands to tmux. The file defines several data structures and functions to manage the output data blocks, control panes, and subscriptions for clients. It uses a combination of linked lists and red-black trees to efficiently manage and organize the data related to client interactions.

The primary components of this file include structures like `control_block`, `control_pane`, `control_sub`, and `control_state`, which are used to manage the state and data flow for each client. Functions are provided to handle reading and writing data to clients, managing subscriptions, and ensuring data is sent in the correct order. The code also includes mechanisms to handle client errors and manage the lifecycle of control mode sessions. This file is integral to the tmux control mode, providing a robust interface for clients to interact with tmux sessions programmatically.
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
- **Use**: This variable is used to store and identify the name of a subscription within the control client system.


---
### s
- **Type**: `char*`
- **Description**: The variable `s` is a pointer to a character array, typically used to store a string. In the context of the provided code, it is used to hold a formatted string that is logged for debugging purposes.
- **Use**: This variable is used to store a formatted string that is passed to the `log_debug` function for logging.


# Data Structures

---
### control_block
- **Type**: `struct`
- **Members**:
    - `size`: Represents the size of the data block.
    - `line`: Pointer to a character array holding the data line.
    - `t`: Timestamp or identifier for the block.
    - `entry`: Linked list entry for managing control blocks in a queue.
    - `all_entry`: Linked list entry for managing all control blocks in a global queue.
- **Description**: The `control_block` structure is used to manage blocks of data that are queued for output in a client-server architecture. Each block contains a size, a line of data, and a timestamp or identifier. The structure also includes two linked list entries, `entry` and `all_entry`, which allow the block to be part of two different queues: one specific to a client or pane, and another global queue for all blocks. This design facilitates the management and ordering of data output, ensuring that blocks are processed in the correct sequence.


---
### control_pane
- **Type**: `struct`
- **Members**:
    - `pane`: An unsigned integer representing the pane identifier.
    - `offset`: A structure representing the data written to the pane.
    - `queued`: A structure representing the data queued for the pane.
    - `flags`: An integer representing the state of the pane with defined flags for off and paused states.
    - `pending_flag`: An integer indicating if the pane is pending for processing.
    - `pending_entry`: A TAILQ entry for managing the pane in a pending list.
    - `blocks`: A TAILQ head for managing a list of control blocks associated with the pane.
    - `entry`: An RB entry for managing the pane in a red-black tree.
- **Description**: The `control_pane` structure is used to manage the state and data flow of a pane within a control client in the tmux application. It includes identifiers for the pane, offsets for data written and queued, flags to indicate the pane's state (such as off or paused), and structures for managing pending operations and associated data blocks. The structure is integrated into a red-black tree and a tail queue to facilitate efficient management and processing of multiple panes.


---
### control_sub_pane
- **Type**: `struct`
- **Members**:
    - `pane`: An unsigned integer representing the pane identifier.
    - `idx`: An unsigned integer representing the index of the pane.
    - `last`: A pointer to a character string, likely used to store the last state or value associated with the pane.
    - `entry`: A Red-Black tree entry used to link this structure into a Red-Black tree of control_sub_pane structures.
- **Description**: The `control_sub_pane` structure is used to represent a subscription to a specific pane within a control client context. It contains identifiers for the pane and its index, a string pointer for storing the last known state or value, and a Red-Black tree entry for organizing multiple subscriptions in a balanced tree structure. This allows efficient management and retrieval of pane subscriptions in the context of a control client, such as in a terminal multiplexer application.


---
### control_sub_window
- **Type**: `struct`
- **Members**:
    - `window`: An unsigned integer representing the window identifier.
    - `idx`: An unsigned integer representing the index of the window.
    - `last`: A pointer to a character string, likely used to store the last state or value associated with the window.
    - `entry`: A Red-Black tree entry used to link this structure into a Red-Black tree of control_sub_window structures.
- **Description**: The `control_sub_window` structure is used to represent a subscription to a window in a control client context, likely within a terminal multiplexer like tmux. It contains identifiers for the window and its index, a string pointer for storing the last known state or value, and a Red-Black tree entry for efficient management and retrieval of multiple window subscriptions.


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
    - `entry`: A red-black tree entry for managing the subscription in a tree.
- **Description**: The `control_sub` structure is used to represent a control client subscription in a system, such as a terminal multiplexer. It contains information about the subscription's name, format, type, and identifier, as well as the last known state. Additionally, it manages collections of subscription panes and windows using red-black trees, allowing for efficient insertion, deletion, and lookup operations. The structure is part of a larger system that handles client subscriptions and their updates.


---
### control_state
- **Type**: `struct`
- **Members**:
    - `panes`: A red-black tree of control_pane structures representing the panes managed by the control state.
    - `pending_list`: A tail queue of control_pane structures that are pending for processing.
    - `pending_count`: An unsigned integer representing the count of pending panes.
    - `all_blocks`: A tail queue of control_block structures representing all data blocks.
    - `read_event`: A pointer to a bufferevent structure for reading events.
    - `write_event`: A pointer to a bufferevent structure for writing events.
    - `subs`: A red-black tree of control_sub structures representing subscriptions.
    - `subs_timer`: An event structure for managing subscription timers.
- **Description**: The control_state structure is a comprehensive data structure used to manage the state of control clients in a system, particularly in the context of handling panes, data blocks, and subscriptions. It maintains a collection of panes and blocks using red-black trees and tail queues, respectively, to efficiently manage and process data. The structure also includes bufferevent pointers for handling read and write events, facilitating asynchronous I/O operations. Additionally, it manages subscriptions through a red-black tree and uses an event timer to periodically check and update subscription states.


# Functions

---
### control_pane_cmp<!-- {{#callable:control_pane_cmp}} -->
The `control_pane_cmp` function compares two `control_pane` structures based on their `pane` member values.
- **Inputs**:
    - `cp1`: A pointer to the first `control_pane` structure to be compared.
    - `cp2`: A pointer to the second `control_pane` structure to be compared.
- **Control Flow**:
    - The function first checks if the `pane` value of `cp1` is less than that of `cp2`; if so, it returns -1.
    - If the `pane` value of `cp1` is greater than that of `cp2`, it returns 1.
    - If neither condition is met, it returns 0, indicating the `pane` values are equal.
- **Output**:
    - The function returns an integer: -1 if `cp1`'s `pane` is less than `cp2`'s, 1 if greater, and 0 if they are equal.


---
### control_sub_cmp<!-- {{#callable:control_sub_cmp}} -->
The `control_sub_cmp` function compares two `control_sub` structures based on their `name` fields using the `strcmp` function.
- **Inputs**:
    - `csub1`: A pointer to the first `control_sub` structure to be compared.
    - `csub2`: A pointer to the second `control_sub` structure to be compared.
- **Control Flow**:
    - The function takes two pointers to `control_sub` structures as input.
    - It uses the `strcmp` function to compare the `name` fields of the two structures.
    - The result of the `strcmp` function is returned, which indicates the lexicographical order of the two names.
- **Output**:
    - The function returns an integer less than, equal to, or greater than zero if the `name` of `csub1` is found, respectively, to be less than, to match, or be greater than the `name` of `csub2`.


---
### control_sub_pane_cmp<!-- {{#callable:control_sub_pane_cmp}} -->
The `control_sub_pane_cmp` function compares two `control_sub_pane` structures based on their `pane` and `idx` fields to determine their ordering.
- **Inputs**:
    - `csp1`: A pointer to the first `control_sub_pane` structure to be compared.
    - `csp2`: A pointer to the second `control_sub_pane` structure to be compared.
- **Control Flow**:
    - The function first compares the `pane` field of `csp1` and `csp2`.
    - If `csp1->pane` is less than `csp2->pane`, the function returns -1, indicating `csp1` should come before `csp2`.
    - If `csp1->pane` is greater than `csp2->pane`, the function returns 1, indicating `csp1` should come after `csp2`.
    - If the `pane` fields are equal, the function then compares the `idx` fields of `csp1` and `csp2`.
    - If `csp1->idx` is less than `csp2->idx`, the function returns -1, indicating `csp1` should come before `csp2`.
    - If `csp1->idx` is greater than `csp2->idx`, the function returns 1, indicating `csp1` should come after `csp2`.
    - If both `pane` and `idx` fields are equal, the function returns 0, indicating `csp1` and `csp2` are equal in terms of ordering.
- **Output**:
    - The function returns an integer: -1 if `csp1` should come before `csp2`, 1 if `csp1` should come after `csp2`, and 0 if they are equal.


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
    - `cs`: A pointer to a `control_state` structure representing the current state of control clients.
    - `csub`: A pointer to a `control_sub` structure representing the subscription to be freed.
- **Control Flow**:
    - Iterate over each `control_sub_pane` in the `csub->panes` red-black tree using `RB_FOREACH_SAFE` to safely remove and free each pane.
    - Iterate over each `control_sub_window` in the `csub->windows` red-black tree using `RB_FOREACH_SAFE` to safely remove and free each window.
    - Free the `last` string in the `csub` structure.
    - Remove the `csub` from the `cs->subs` red-black tree.
    - Free the `name` and `format` strings in the `csub` structure.
    - Free the `csub` structure itself.
- **Output**:
    - The function does not return any value; it performs cleanup and deallocation of memory associated with a control subscription.


---
### control_free_block<!-- {{#callable:control_free_block}} -->
The `control_free_block` function deallocates memory associated with a `control_block` and removes it from a linked list of all blocks in a `control_state`.
- **Inputs**:
    - `cs`: A pointer to a `control_state` structure, which maintains the state of control clients, including a list of all control blocks.
    - `cb`: A pointer to a `control_block` structure, which represents a block of data to be output and is part of the control client's data management.
- **Control Flow**:
    - The function first frees the memory allocated for the `line` member of the `control_block` pointed to by `cb`.
    - It then removes the `control_block` from the `all_blocks` queue in the `control_state` using the `TAILQ_REMOVE` macro.
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
    - Use `RB_FIND` to search for and return the `control_pane` in the client's `control_state` that matches the temporary `control_pane` `cp`.
- **Output**:
    - Returns a pointer to the `control_pane` structure if found, otherwise returns `NULL`.


---
### control_add_pane<!-- {{#callable:control_add_pane}} -->
The `control_add_pane` function adds a new control pane to a client's control state or retrieves an existing one for a specified window pane.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the control pane is being added or retrieved.
    - `wp`: A pointer to a `window_pane` structure representing the window pane for which the control pane is being added or retrieved.
- **Control Flow**:
    - Retrieve the control state from the client structure `c`.
    - Call [`control_get_pane`](#control_get_pane) to check if a control pane for the given window pane `wp` already exists in the client's control state.
    - If the control pane exists, return it immediately.
    - If the control pane does not exist, allocate memory for a new `control_pane` structure using [`xcalloc`](xmalloc.c.driver.md#xcalloc).
    - Set the `pane` field of the new control pane to the ID of the window pane `wp`.
    - Insert the new control pane into the client's control state's red-black tree of panes using `RB_INSERT`.
    - Copy the `offset` and `queued` fields from the window pane `wp` to the new control pane using `memcpy`.
    - Initialize the `blocks` queue of the new control pane using `TAILQ_INIT`.
    - Return the newly created control pane.
- **Output**:
    - A pointer to the `control_pane` structure that was either retrieved or newly created.
- **Functions called**:
    - [`control_get_pane`](#control_get_pane)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)


---
### control_discard_pane<!-- {{#callable:control_discard_pane}} -->
The `control_discard_pane` function removes and frees all control blocks associated with a specific control pane for a client.
- **Inputs**:
    - `c`: A pointer to a `client` structure, representing the client whose control pane blocks are to be discarded.
    - `cp`: A pointer to a `control_pane` structure, representing the control pane whose blocks are to be removed and freed.
- **Control Flow**:
    - Retrieve the control state from the client structure `c`.
    - Iterate over each control block in the `blocks` queue of the control pane `cp` using `TAILQ_FOREACH_SAFE`.
    - For each block, remove it from the `blocks` queue of `cp` using `TAILQ_REMOVE`.
    - Free the block using the [`control_free_block`](#control_free_block) function, passing the control state and the block as arguments.
- **Output**:
    - The function does not return any value; it performs its operations directly on the data structures provided.
- **Functions called**:
    - [`control_free_block`](#control_free_block)


---
### control_window_pane<!-- {{#callable:control_window_pane}} -->
The `control_window_pane` function retrieves a `window_pane` structure for a given client and pane ID, ensuring the pane is part of the client's session.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the window pane is being retrieved.
    - `pane`: An unsigned integer representing the ID of the pane to be retrieved.
- **Control Flow**:
    - Check if the client's session is NULL; if so, return NULL.
    - Attempt to find the window pane by its ID using [`window_pane_find_by_id`](window.c.driver.md#window_pane_find_by_id); if not found, return NULL.
    - Check if the window associated with the found pane is part of the client's session using [`winlink_find_by_window`](window.c.driver.md#winlink_find_by_window); if not, return NULL.
    - If all checks pass, return the found `window_pane` structure.
- **Output**:
    - Returns a pointer to the `window_pane` structure if the pane is valid and part of the client's session, otherwise returns NULL.
- **Functions called**:
    - [`window_pane_find_by_id`](window.c.driver.md#window_pane_find_by_id)
    - [`winlink_find_by_window`](window.c.driver.md#winlink_find_by_window)


---
### control_reset_offsets<!-- {{#callable:control_reset_offsets}} -->
The `control_reset_offsets` function resets the control state of a client by clearing all pane offsets and pending operations.
- **Inputs**:
    - `c`: A pointer to a `struct client` which represents the client whose control state is to be reset.
- **Control Flow**:
    - Retrieve the control state (`cs`) from the client (`c`).
    - Iterate over each control pane (`cp`) in the control state's pane tree using `RB_FOREACH_SAFE`.
    - Remove each pane (`cp`) from the tree and free its memory.
    - Initialize the pending list of the control state to an empty state using `TAILQ_INIT`.
    - Set the pending count of the control state to zero.
- **Output**:
    - The function does not return any value; it modifies the control state of the client in place.


---
### control_pane_offset<!-- {{#callable:control_pane_offset}} -->
The `control_pane_offset` function retrieves the offset of a window pane for a client, considering various control flags and buffer conditions.
- **Inputs**:
    - `c`: A pointer to a `struct client`, representing the client for which the pane offset is being retrieved.
    - `wp`: A pointer to a `struct window_pane`, representing the window pane whose offset is being queried.
    - `off`: A pointer to an integer where the function will store a flag indicating the offset status.
- **Control Flow**:
    - Check if the client has the `CLIENT_CONTROL_NOOUTPUT` flag set; if so, set `*off` to 0 and return `NULL`.
    - Retrieve the control pane associated with the client and window pane using [`control_get_pane`](#control_get_pane).
    - If the control pane is `NULL` or has the `CONTROL_PANE_PAUSED` flag, set `*off` to 0 and return `NULL`.
    - If the control pane has the `CONTROL_PANE_OFF` flag, set `*off` to 1 and return `NULL`.
    - Set `*off` to 1 if the length of the output buffer in the client's control state is greater than or equal to `CONTROL_BUFFER_LOW`, otherwise set it to 0.
    - Return a pointer to the `offset` field of the control pane.
- **Output**:
    - Returns a pointer to the `struct window_pane_offset` of the control pane if conditions are met, otherwise returns `NULL`.
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
    - Check if the retrieved control pane `cp` is not NULL and if it has the `CONTROL_PANE_OFF` flag set.
    - If both conditions are true, clear the `CONTROL_PANE_OFF` flag from `cp->flags`.
    - Copy the `offset` and `queued` data from the window pane `wp` to the control pane `cp`.
- **Output**:
    - The function does not return a value; it modifies the state of the control pane associated with the client and window pane.
- **Functions called**:
    - [`control_get_pane`](#control_get_pane)


---
### control_set_pane_off<!-- {{#callable:control_set_pane_off}} -->
The `control_set_pane_off` function sets a specified pane to an 'off' state for a given client.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the pane state is being modified.
    - `wp`: A pointer to a `window_pane` structure representing the pane that is to be set to the 'off' state.
- **Control Flow**:
    - Call [`control_add_pane`](#control_add_pane) with the client `c` and window pane `wp` to ensure the pane is added to the client's control state and retrieve the corresponding `control_pane` structure.
    - Set the `CONTROL_PANE_OFF` flag in the `flags` field of the `control_pane` structure to mark the pane as 'off'.
- **Output**:
    - This function does not return a value; it modifies the state of the pane within the client's control state.
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
    - Check if the control pane `cp` is not NULL and if it is paused by checking the `CONTROL_PANE_PAUSED` flag.
    - If the pane is paused, clear the `CONTROL_PANE_PAUSED` flag to resume the pane.
    - Copy the `offset` and `queued` data from the window pane `wp` to the control pane `cp`.
    - Send a continue message to the client using [`control_write`](#control_write) with the window pane's ID.
- **Output**:
    - The function does not return a value; it modifies the state of the control pane and sends a message to the client.
- **Functions called**:
    - [`control_get_pane`](#control_get_pane)
    - [`control_write`](#control_write)


---
### control_pause_pane<!-- {{#callable:control_pause_pane}} -->
The `control_pause_pane` function pauses the output of a specified window pane for a given client.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the pane output is to be paused.
    - `wp`: A pointer to a `window_pane` structure representing the pane to be paused.
- **Control Flow**:
    - Retrieve or add a `control_pane` structure for the specified client and window pane using [`control_add_pane`](#control_add_pane).
    - Check if the `CONTROL_PANE_PAUSED` flag is not set in the `control_pane` structure's flags.
    - If the pane is not already paused, set the `CONTROL_PANE_PAUSED` flag in the `control_pane` structure's flags.
    - Call [`control_discard_pane`](#control_discard_pane) to discard any pending output for the pane.
    - Write a pause message to the client using [`control_write`](#control_write), indicating that the pane has been paused.
- **Output**:
    - The function does not return a value; it modifies the state of the `control_pane` associated with the client and window pane to indicate that the pane is paused.
- **Functions called**:
    - [`control_add_pane`](#control_add_pane)
    - [`control_discard_pane`](#control_discard_pane)
    - [`control_write`](#control_write)


---
### control_write<!-- {{#callable:control_write}} -->
The `control_write` function formats a string and either writes it directly to a client's output buffer or stores it in a queue for later writing, depending on the state of the client's control blocks.
- **Inputs**:
    - `c`: A pointer to a `struct client`, representing the client to which the formatted string will be written or queued.
    - `fmt`: A format string, similar to those used in `printf`, which specifies how the subsequent arguments are formatted.
    - `...`: A variable number of arguments that are formatted according to the `fmt` string.
- **Control Flow**:
    - Initialize a variable argument list `ap` using `va_start` with `fmt` as the last fixed argument.
    - Check if the `all_blocks` queue in the client's control state is empty.
    - If the queue is empty, call `control_vwrite` to write the formatted string directly to the client's output buffer and return.
    - If the queue is not empty, allocate memory for a new `control_block` and format the string into `cb->line` using [`xvasprintf`](xmalloc.c.driver.md#xvasprintf).
    - Insert the new `control_block` at the tail of the `all_blocks` queue.
    - Set the timestamp `cb->t` using `get_timer()`.
    - Log the action of storing the line for debugging purposes.
    - Enable the write event on the client's `write_event` buffer to ensure the data is eventually written out.
    - End the variable argument list using `va_end`.
- **Output**:
    - The function does not return a value; it writes or queues a formatted string for a client.
- **Functions called**:
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xvasprintf`](xmalloc.c.driver.md#xvasprintf)
    - [`get_timer`](tmux.c.driver.md#get_timer)


---
### control_check_age<!-- {{#callable:control_check_age}} -->
The `control_check_age` function checks the age of the first control block in a pane's queue and takes action based on the client's control flags and the block's age.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the age check is being performed.
    - `wp`: A pointer to a `window_pane` structure representing the window pane associated with the control pane.
    - `cp`: A pointer to a `control_pane` structure representing the control pane whose blocks are being checked for age.
- **Control Flow**:
    - Retrieve the first control block from the control pane's block queue.
    - If there is no control block, return 0 indicating no action is needed.
    - Get the current time using `get_timer()` and compare it with the timestamp of the control block.
    - If the block's timestamp is greater than or equal to the current time, return 0 indicating no action is needed.
    - Calculate the age of the block by subtracting the block's timestamp from the current time.
    - Log the age of the block for debugging purposes.
    - If the client has the `CLIENT_CONTROL_PAUSEAFTER` flag set, check if the block's age is less than the client's `pause_age`. If so, return 0 indicating no action is needed.
    - If the block's age is greater than or equal to the client's `pause_age`, set the control pane's `CONTROL_PANE_PAUSED` flag, discard the pane's blocks, and send a pause message to the client.
    - If the client does not have the `CLIENT_CONTROL_PAUSEAFTER` flag set, check if the block's age is less than `CONTROL_MAXIMUM_AGE`. If so, return 0 indicating no action is needed.
    - If the block's age is greater than or equal to `CONTROL_MAXIMUM_AGE`, set the client's exit message to "too far behind", set the `CLIENT_EXIT` flag, and discard the client's data.
    - Return 1 indicating that an action was taken.
- **Output**:
    - Returns an integer value: 0 if no action was taken, or 1 if the control pane was paused or the client was marked for exit.
- **Functions called**:
    - [`get_timer`](tmux.c.driver.md#get_timer)
    - [`control_discard_pane`](#control_discard_pane)
    - [`control_write`](#control_write)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)
    - [`control_discard`](#control_discard)


---
### control_write_output<!-- {{#callable:control_write_output}} -->
The `control_write_output` function manages the output data flow from a window pane to a client, handling conditions such as pane state and data queuing.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client to which the output is being written.
    - `wp`: A pointer to a `window_pane` structure representing the pane from which the output data is being retrieved.
- **Control Flow**:
    - Retrieve the control state from the client structure.
    - Check if the window pane is part of the client's session; if not, return immediately.
    - If the client has flags set to ignore output, attempt to retrieve the control pane; if found, log and ignore the pane, otherwise return.
    - Add the pane to the client's control state if not already present.
    - Check if the pane is marked as off or paused; if so, log and ignore the pane.
    - Check the age of the pane's data; if too old, return.
    - Retrieve new data from the window pane and update the queued data size; if no new data, return.
    - Allocate a new control block for the data, insert it into the client's all_blocks queue, and log the new block.
    - Insert the control block into the pane's blocks queue.
    - If the pane is not already pending, mark it as pending, add it to the pending list, and increment the pending count.
    - Enable the write event for the client's control state.
- **Output**:
    - The function does not return a value but modifies the client's control state by queuing new output data blocks and updating the pending list.
- **Functions called**:
    - [`winlink_find_by_window`](window.c.driver.md#winlink_find_by_window)
    - [`control_get_pane`](#control_get_pane)
    - [`control_add_pane`](#control_add_pane)
    - [`control_check_age`](#control_check_age)
    - [`window_pane_get_new_data`](window.c.driver.md#window_pane_get_new_data)
    - [`window_pane_update_used_data`](window.c.driver.md#window_pane_update_used_data)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`get_timer`](tmux.c.driver.md#get_timer)


---
### control_error<!-- {{#callable:control_error}} -->
The `control_error` function handles error reporting for a control client by logging a parse error message and cleaning up resources.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, representing the command queue item associated with the error.
    - `data`: A pointer to a character string containing the error message to be reported.
- **Control Flow**:
    - Retrieve the client associated with the command queue item using [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client).
    - Begin a guarded section with [`cmdq_guard`](cmd-queue.c.driver.md#cmdq_guard) using the label 'begin'.
    - Write the error message to the control client using [`control_write`](#control_write), formatted as 'parse error: %s'.
    - End the guarded section with [`cmdq_guard`](cmd-queue.c.driver.md#cmdq_guard) using the label 'error'.
    - Free the memory allocated for the error message string.
    - Return `CMD_RETURN_NORMAL` to indicate normal command completion.
- **Output**:
    - The function returns an `enum cmd_retval` value, specifically `CMD_RETURN_NORMAL`, indicating normal command completion.
- **Functions called**:
    - [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client)
    - [`cmdq_guard`](cmd-queue.c.driver.md#cmdq_guard)
    - [`control_write`](#control_write)


---
### control_error_callback<!-- {{#callable:control_error_callback}} -->
The `control_error_callback` function sets a client's exit flag when an error occurs in a bufferevent.
- **Inputs**:
    - `bufev`: A pointer to a bufferevent structure, which is unused in this function.
    - `what`: A short integer indicating the type of event that occurred, which is unused in this function.
    - `data`: A pointer to a client structure, which is used to access and modify the client's flags.
- **Control Flow**:
    - The function retrieves the client structure from the provided data pointer.
    - It sets the CLIENT_EXIT flag in the client's flags to indicate that the client should exit.
- **Output**:
    - The function does not return any value; it modifies the client's flags to signal an exit.


---
### control_read_callback<!-- {{#callable:control_read_callback}} -->
The `control_read_callback` function processes input lines from a buffer, executing commands or detaching the client based on the input.
- **Inputs**:
    - `bufev`: A pointer to a bufferevent structure, which is unused in this function.
    - `data`: A pointer to a client structure, representing the client associated with the control state.
- **Control Flow**:
    - The function enters an infinite loop to process lines from the buffer.
    - It reads a line from the buffer using `evbuffer_readln` and checks if the line is NULL to break the loop.
    - If the line is empty, it frees the line and sets the client flag to exit, then breaks the loop.
    - For non-empty lines, it creates a new command queue state and attempts to parse and append the command.
    - If a parse error occurs, it appends an error callback to the command queue.
    - The line is freed after processing.
- **Output**:
    - The function does not return a value; it modifies the client state and command queue based on the input lines.
- **Functions called**:
    - [`cmdq_new_state`](cmd-queue.c.driver.md#cmdq_new_state)
    - [`cmdq_append`](cmd-queue.c.driver.md#cmdq_append)
    - [`cmdq_free_state`](cmd-queue.c.driver.md#cmdq_free_state)


---
### control_all_done<!-- {{#callable:control_all_done}} -->
The `control_all_done` function checks if a control client has completed all its data output tasks.
- **Inputs**:
    - `c`: A pointer to a `client` structure, representing the control client whose output status is being checked.
- **Control Flow**:
    - Retrieve the `control_state` associated with the client `c`.
    - Check if the `all_blocks` queue in the `control_state` is not empty; if it is not empty, return 0 indicating that there are still blocks to be processed.
    - If the `all_blocks` queue is empty, check if the length of the output buffer in the `write_event` is zero; return the result of this check.
- **Output**:
    - Returns an integer: 0 if there are still blocks to be processed or if the output buffer is not empty, and 1 if all blocks are processed and the output buffer is empty.


---
### control_flush_all_blocks<!-- {{#callable:control_flush_all_blocks}} -->
The `control_flush_all_blocks` function flushes all control blocks with zero size from a client's control state, writing their contents to a buffer event and freeing the blocks.
- **Inputs**:
    - `c`: A pointer to a `struct client`, representing the client whose control blocks are to be flushed.
- **Control Flow**:
    - Retrieve the control state from the client structure.
    - Iterate over all control blocks in the client's control state using a safe traversal macro `TAILQ_FOREACH_SAFE`.
    - For each block, check if its size is zero; if not, break the loop.
    - If the block's size is zero, log a debug message indicating the line being flushed.
    - Write the block's line to the client's write event buffer, followed by a newline character.
    - Free the block using the [`control_free_block`](#control_free_block) function.
- **Output**:
    - The function does not return a value; it performs operations on the client's control state and write event buffer.
- **Functions called**:
    - [`control_free_block`](#control_free_block)


---
### control_append_data<!-- {{#callable:control_append_data}} -->
The `control_append_data` function appends new data from a window pane to an event buffer, encoding special characters as needed.
- **Inputs**:
    - `c`: A pointer to the client structure, which contains the state and flags of the client.
    - `cp`: A pointer to the control pane structure, which holds offsets and flags for the pane.
    - `age`: A 64-bit unsigned integer representing the age of the data block.
    - `message`: A pointer to an event buffer where the data will be appended; if NULL, a new buffer is created.
    - `wp`: A pointer to the window pane structure from which new data is retrieved.
    - `size`: The size of the data to be appended to the buffer.
- **Control Flow**:
    - Check if the `message` buffer is NULL; if so, create a new event buffer and add a formatted string to it based on the client's flags.
    - Retrieve new data from the window pane using [`window_pane_get_new_data`](window.c.driver.md#window_pane_get_new_data), updating the control pane's offset and getting the size of the new data.
    - Check if the new data size is less than the requested size; if so, terminate with an error.
    - Iterate over the new data, appending each byte to the `message` buffer, encoding bytes that are less than space or are backslashes.
    - Update the window pane's used data with the size of the data appended.
    - Return the `message` buffer.
- **Output**:
    - Returns a pointer to the event buffer containing the appended data.
- **Functions called**:
    - [`window_pane_get_new_data`](window.c.driver.md#window_pane_get_new_data)
    - [`window_pane_update_used_data`](window.c.driver.md#window_pane_update_used_data)


---
### control_write_data<!-- {{#callable:control_write_data}} -->
The `control_write_data` function writes a message to a client's control state write event buffer and then frees the message buffer.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client whose control state is being modified.
    - `message`: A pointer to an `evbuffer` containing the message data to be written to the client's control state.
- **Control Flow**:
    - Retrieve the control state from the client structure.
    - Log the message being written using the `log_debug` function.
    - Append a newline character to the message buffer using `evbuffer_add`.
    - Write the message buffer to the client's write event buffer using `bufferevent_write_buffer`.
    - Free the message buffer using `evbuffer_free`.
- **Output**:
    - The function does not return any value; it performs operations on the provided `evbuffer` and the client's control state.


---
### control_write_pending<!-- {{#callable:control_write_pending}} -->
The `control_write_pending` function manages the writing of pending data blocks from a control pane to a client, ensuring data is sent up to a specified limit and handling block removal and flushing as necessary.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client to which data is being written.
    - `cp`: A pointer to a `control_pane` structure representing the pane from which data blocks are being written.
    - `limit`: A `size_t` value specifying the maximum amount of data to write in this operation.
- **Control Flow**:
    - Retrieve the control state from the client and initialize variables for the window pane, message buffer, used size, and current time.
    - Attempt to get the actual window pane associated with the control pane; if not found or invalid, remove all blocks from the control pane and flush all blocks from the client, then return 0.
    - Enter a loop to process blocks while the used size is less than the limit and there are blocks in the control pane.
    - Check the age of the current block; if too old, free the message buffer and break the loop.
    - Calculate the age of the current block and log debug information.
    - Determine the size of data to write from the current block, ensuring it does not exceed the remaining limit, and update the used size.
    - Append data from the block to the message buffer, adjusting the block size accordingly.
    - If a block is fully written, remove it from the control pane and free it; if the first block in the client's all blocks queue is also empty, write the message buffer and flush all blocks.
    - After the loop, if there is a message buffer, write its data to the client.
    - Return whether there are remaining blocks in the control pane.
- **Output**:
    - Returns an integer indicating whether there are remaining blocks in the control pane (non-zero if there are, zero if not).
- **Functions called**:
    - [`get_timer`](tmux.c.driver.md#get_timer)
    - [`control_window_pane`](#control_window_pane)
    - [`control_free_block`](#control_free_block)
    - [`control_flush_all_blocks`](#control_flush_all_blocks)
    - [`control_check_age`](#control_check_age)
    - [`control_append_data`](#control_append_data)
    - [`control_write_data`](#control_write_data)


---
### control_write_callback<!-- {{#callable:control_write_callback}} -->
The `control_write_callback` function manages the writing of data to a client's buffer, ensuring that data is written in chunks while respecting buffer limits and pending data counts.
- **Inputs**:
    - `bufev`: A pointer to a bufferevent structure, which is unused in this function.
    - `data`: A pointer to a client structure, which contains the control state and other relevant data for the client.
- **Control Flow**:
    - The function begins by casting the `data` pointer to a `client` structure and retrieves the associated `control_state` structure.
    - It calls [`control_flush_all_blocks`](#control_flush_all_blocks) to flush any blocks of data that are ready to be written.
    - A loop is initiated to continue writing data as long as the buffer length is below `CONTROL_BUFFER_HIGH`.
    - Within the loop, it checks if there are any pending data blocks to write; if not, it breaks out of the loop.
    - It calculates the available space in the buffer and logs the available space and number of pending panes.
    - The limit for writing data is calculated based on the available space and the number of pending panes, ensuring a minimum write size of `CONTROL_WRITE_MINIMUM`.
    - It iterates over the pending list of control panes, attempting to write pending data for each pane up to the calculated limit.
    - If a pane's data is successfully written, it is removed from the pending list, and its pending flag is cleared.
    - If the buffer length reaches `CONTROL_BUFFER_HIGH`, the loop breaks.
    - Finally, if the buffer is empty, it disables the write event for the client's bufferevent.
- **Output**:
    - The function does not return a value; it operates by modifying the state of the client's buffer and control state.
- **Functions called**:
    - [`control_flush_all_blocks`](#control_flush_all_blocks)
    - [`control_write_pending`](#control_write_pending)


---
### control_start<!-- {{#callable:control_start}} -->
The `control_start` function initializes the control state for a client, setting up non-blocking I/O and event handling for control mode operations.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client for which the control state is being initialized.
- **Control Flow**:
    - Check if the client is in control mode by evaluating the CLIENT_CONTROLCONTROL flag in the client's flags.
    - If the client is in control mode, close the client's output file descriptor and set it to -1; otherwise, set the output file descriptor to non-blocking mode.
    - Set the client's main file descriptor to non-blocking mode.
    - Allocate memory for a new `control_state` structure and assign it to the client's `control_state` pointer.
    - Initialize the red-black tree and tail queue structures within the `control_state` for managing panes, pending lists, all blocks, and subscriptions.
    - Create a new buffered event for reading using the client's file descriptor and assign it to the `read_event` in the `control_state`.
    - If the `read_event` creation fails, terminate the program with an out-of-memory error.
    - If the client is in control mode, set the `write_event` to the same as `read_event`; otherwise, create a new buffered event for writing using the client's output file descriptor.
    - If the `write_event` creation fails, terminate the program with an out-of-memory error.
    - Set the low watermark for the `write_event` to CONTROL_BUFFER_LOW.
    - If the client is in control mode, write a specific control sequence to the `write_event` and enable writing.
- **Output**:
    - The function does not return a value; it initializes the control state for the client, setting up necessary structures and events for control mode.
- **Functions called**:
    - [`setblocking`](tmux.c.driver.md#setblocking)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)


---
### control_ready<!-- {{#callable:control_ready}} -->
The `control_ready` function enables the read event for a client's control state, allowing it to start processing input.
- **Inputs**:
    - `c`: A pointer to a `struct client`, representing the client whose control state read event is to be enabled.
- **Control Flow**:
    - The function calls `bufferevent_enable` with the client's control state's read event and the `EV_READ` flag.
    - This enables the read event, allowing the client to start processing incoming data.
- **Output**:
    - The function does not return any value; it modifies the state of the client's control state by enabling its read event.


---
### control_discard<!-- {{#callable:control_discard}} -->
The `control_discard` function discards all output for a given client by iterating over its control panes and disabling the read event.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client whose output is to be discarded.
- **Control Flow**:
    - Retrieve the control state associated with the client `c`.
    - Iterate over each control pane in the client's control state using the `RB_FOREACH` macro.
    - For each control pane, call [`control_discard_pane`](#control_discard_pane) to discard its output.
    - Disable the read event associated with the client's control state using `bufferevent_disable`.
- **Output**:
    - The function does not return a value; it performs operations to discard output and disable reading for the specified client.
- **Functions called**:
    - [`control_discard_pane`](#control_discard_pane)


---
### control_stop<!-- {{#callable:control_stop}} -->
The `control_stop` function terminates the control mode for a client by freeing associated resources and resetting states.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the control mode is being stopped.
- **Control Flow**:
    - Retrieve the `control_state` associated with the client `c`.
    - If the client is not in control mode (`CLIENT_CONTROLCONTROL` flag is not set), free the `write_event` buffer event.
    - Free the `read_event` buffer event regardless of the control mode.
    - Iterate over all control subscriptions (`control_subs`) and free each subscription using [`control_free_sub`](#control_free_sub).
    - If the subscription timer (`subs_timer`) is initialized, delete it using `evtimer_del`.
    - Iterate over all control blocks (`all_blocks`) and free each block using [`control_free_block`](#control_free_block).
    - Reset the control offsets for the client using [`control_reset_offsets`](#control_reset_offsets).
    - Free the `control_state` structure associated with the client.
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
    - `c`: A pointer to a `client` structure representing the client for which the session subscription is being checked.
    - `csub`: A pointer to a `control_sub` structure representing the subscription to be checked.
- **Control Flow**:
    - Retrieve the session associated with the client `c`.
    - Create a format tree `ft` with default values using the client `c` and session `s`.
    - Expand the format string from `csub` using the format tree `ft` to get the current value.
    - Free the format tree `ft`.
    - Check if the current value is the same as the last recorded value in `csub->last`.
    - If the values are the same, free the current value and return without doing anything further.
    - If the values are different, write a subscription change notification using [`control_write`](#control_write).
    - Free the last recorded value in `csub->last` and update it to the current value.
- **Output**:
    - The function does not return a value; it performs actions such as writing a notification and updating the subscription state.
- **Functions called**:
    - [`format_create_defaults`](format.c.driver.md#format_create_defaults)
    - [`format_expand`](format.c.driver.md#format_expand)
    - [`format_free`](format.c.driver.md#format_free)
    - [`control_write`](#control_write)


---
### control_check_subs_pane<!-- {{#callable:control_check_subs_pane}} -->
The `control_check_subs_pane` function checks for changes in a specific pane's subscription and updates the client if any changes are detected.
- **Inputs**:
    - `c`: A pointer to the client structure, representing the client for which the subscription is being checked.
    - `csub`: A pointer to the control_sub structure, representing the subscription details for a specific pane.
- **Control Flow**:
    - Retrieve the session associated with the client `c`.
    - Find the window pane using the ID from `csub`; if not found or invalid, exit the function.
    - Retrieve the window associated with the found pane.
    - Iterate over each winlink in the window's winlinks list.
    - For each winlink, check if it belongs to the client's session; if not, continue to the next winlink.
    - Create a format tree with default values using the client, session, winlink, and window pane.
    - Expand the format string from `csub` using the format tree and store the result in `value`.
    - Free the format tree after use.
    - Create a `control_sub_pane` structure `find` with the pane ID and winlink index.
    - Search for an existing `control_sub_pane` in `csub->panes` using `find`.
    - If not found, allocate a new `control_sub_pane`, initialize it, and insert it into `csub->panes`.
    - If the last value in the found or newly created `control_sub_pane` is the same as `value`, free `value` and continue to the next winlink.
    - If the value has changed, write a subscription change message to the client and update the last value in the `control_sub_pane`.
- **Output**:
    - The function does not return a value; it performs operations to check and update pane subscriptions, writing changes to the client if necessary.
- **Functions called**:
    - [`window_pane_find_by_id`](window.c.driver.md#window_pane_find_by_id)
    - [`format_create_defaults`](format.c.driver.md#format_create_defaults)
    - [`format_expand`](format.c.driver.md#format_expand)
    - [`format_free`](format.c.driver.md#format_free)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`control_write`](#control_write)


---
### control_check_subs_all_panes<!-- {{#callable:control_check_subs_all_panes}} -->
The function `control_check_subs_all_panes` checks all panes in a session for changes in subscription values and updates the client if any changes are detected.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the subscription checks are being performed.
    - `csub`: A pointer to a `control_sub` structure representing the subscription details, including the format string to be evaluated for each pane.
- **Control Flow**:
    - Retrieve the session associated with the client `c`.
    - Iterate over each window link (`wl`) in the session's windows.
    - For each window (`w`), iterate over its panes (`wp`).
    - Create a format tree (`ft`) with default values for the current client, session, window link, and pane.
    - Expand the format string from `csub` using the format tree to get the current value for the pane.
    - Free the format tree after use.
    - Create a `control_sub_pane` structure `find` with the current pane ID and window link index.
    - Search for an existing `control_sub_pane` in `csub->panes` that matches `find`.
    - If no matching `control_sub_pane` is found, allocate a new one, initialize it with the current pane ID and index, and insert it into `csub->panes`.
    - If the last recorded value for the pane (`csp->last`) is not NULL and matches the current value, free the current value and continue to the next pane.
    - If the value has changed, write a subscription change message to the client `c`, free the last recorded value, and update it with the current value.
- **Output**:
    - The function does not return a value; it performs operations to update the client's subscription state and writes messages to the client if changes are detected.
- **Functions called**:
    - [`format_create_defaults`](format.c.driver.md#format_create_defaults)
    - [`format_expand`](format.c.driver.md#format_expand)
    - [`format_free`](format.c.driver.md#format_free)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`control_write`](#control_write)


---
### control_check_subs_window<!-- {{#callable:control_check_subs_window}} -->
The `control_check_subs_window` function checks for changes in a window's subscription state and updates the client if any changes are detected.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the subscription check is being performed.
    - `csub`: A pointer to a `control_sub` structure representing the subscription details for the window being checked.
- **Control Flow**:
    - Retrieve the session associated with the client `c` and store it in `s`.
    - Find the window by its ID using `csub->id`; if not found, return immediately.
    - Iterate over each `winlink` in the window's `winlinks` list.
    - For each `winlink`, check if its session matches the client's session `s`; if not, continue to the next `winlink`.
    - Create a format tree `ft` with default values using the client, session, and `winlink`.
    - Expand the format string `csub->format` using the format tree and store the result in `value`.
    - Free the format tree `ft`.
    - Initialize a `control_sub_window` structure `find` with the window ID and `winlink` index.
    - Search for an existing `control_sub_window` in `csub->windows` using `find`; if not found, allocate a new one, initialize it, and insert it into the tree.
    - If the `last` value of the found or newly created `control_sub_window` matches `value`, free `value` and continue to the next `winlink`.
    - If the `last` value does not match `value`, write a subscription change message to the client using [`control_write`](#control_write), update `csw->last` with `value`, and free the old `last` value.
- **Output**:
    - The function does not return a value; it performs operations on the client and subscription structures, updating them as necessary.
- **Functions called**:
    - [`window_find_by_id`](window.c.driver.md#window_find_by_id)
    - [`format_create_defaults`](format.c.driver.md#format_create_defaults)
    - [`format_expand`](format.c.driver.md#format_expand)
    - [`format_free`](format.c.driver.md#format_free)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`control_write`](#control_write)


---
### control_check_subs_all_windows<!-- {{#callable:control_check_subs_all_windows}} -->
The function `control_check_subs_all_windows` checks all windows in a session for changes based on a subscription format and notifies the client if any changes are detected.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the subscription changes are being checked.
    - `csub`: A pointer to a `control_sub` structure representing the subscription details, including the format to be checked against each window.
- **Control Flow**:
    - Retrieve the session associated with the client `c`.
    - Iterate over each window link (`wl`) in the session's windows.
    - For each window link, retrieve the associated window `w`.
    - Create a format tree `ft` with default values using the client, session, and window link.
    - Expand the subscription format using the format tree to get the current value.
    - Free the format tree after use.
    - Create a `control_sub_window` structure `find` with the current window ID and index.
    - Search for an existing `control_sub_window` in the subscription's windows tree that matches `find`.
    - If no existing `control_sub_window` is found, allocate a new one, initialize it with the current window ID and index, and insert it into the subscription's windows tree.
    - If the last recorded value for the window is the same as the current value, free the current value and continue to the next window.
    - If the value has changed, write a subscription change notification to the client, update the last recorded value for the window, and free the previous last value.
- **Output**:
    - The function does not return a value; it performs operations to check for changes and notify the client if necessary.
- **Functions called**:
    - [`format_create_defaults`](format.c.driver.md#format_create_defaults)
    - [`format_expand`](format.c.driver.md#format_expand)
    - [`format_free`](format.c.driver.md#format_free)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`control_write`](#control_write)


---
### control_check_subs_timer<!-- {{#callable:control_check_subs_timer}} -->
The `control_check_subs_timer` function is responsible for periodically checking and updating the state of client subscriptions based on their type.
- **Inputs**:
    - `fd`: An unused integer file descriptor parameter.
    - `events`: An unused short integer representing event flags.
    - `data`: A pointer to a `client` structure, which contains the control state and subscriptions to be checked.
- **Control Flow**:
    - Log a debug message indicating the timer has fired.
    - Re-add the subscription timer to trigger again after one second.
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
The `control_add_sub` function manages the addition of a subscription to a client's control state, ensuring that any existing subscription with the same name is replaced and a timer is set for subscription checks.
- **Inputs**:
    - `c`: A pointer to a `client` structure representing the client for which the subscription is being added.
    - `name`: A constant character pointer representing the name of the subscription.
    - `type`: An enumeration value of type `control_sub_type` indicating the type of subscription.
    - `id`: An integer representing the identifier for the subscription, which could be a pane or window ID depending on the subscription type.
    - `format`: A constant character pointer representing the format string for the subscription.
- **Control Flow**:
    - Retrieve the control state from the client structure.
    - Create a `control_sub` structure named `find` with the provided `name`.
    - Check if a subscription with the same name already exists in the client's control state using `RB_FIND`.
    - If an existing subscription is found, free it using [`control_free_sub`](#control_free_sub).
    - Allocate memory for a new `control_sub` structure and initialize its fields with the provided parameters.
    - Insert the new subscription into the client's control state using `RB_INSERT`.
    - Initialize the `panes` and `windows` red-black trees for the new subscription.
    - Check if the subscription timer is initialized; if not, set it up with `evtimer_set`.
    - Ensure the subscription timer is active by adding it with `evtimer_add` if it is not already pending.
- **Output**:
    - The function does not return a value; it modifies the client's control state by adding or replacing a subscription and ensuring the subscription timer is active.
- **Functions called**:
    - [`control_free_sub`](#control_free_sub)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### control_remove_sub<!-- {{#callable:control_remove_sub}} -->
The `control_remove_sub` function removes a subscription from a client's control state and stops the subscription timer if no subscriptions remain.
- **Inputs**:
    - `c`: A pointer to a `struct client`, representing the client whose subscription is to be removed.
    - `name`: A constant character pointer representing the name of the subscription to be removed.
- **Control Flow**:
    - Retrieve the control state from the client structure.
    - Create a temporary `control_sub` structure `find` with the name to search for.
    - Use `RB_FIND` to locate the subscription in the client's subscription tree using the `find` structure.
    - If the subscription is found, call [`control_free_sub`](#control_free_sub) to free the subscription resources.
    - Check if the subscription tree is empty using `RB_EMPTY`.
    - If the subscription tree is empty, delete the subscription timer using `evtimer_del`.
- **Output**:
    - The function does not return a value; it performs operations on the client's control state to remove a subscription.
- **Functions called**:
    - [`control_free_sub`](#control_free_sub)


