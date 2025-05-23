# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it implements functionality for managing synchronization between different clients using named wait channels. The primary purpose of this code is to provide mechanisms for clients to block or wake on specific channels, allowing for coordination and signaling between different parts of the application. The code defines a command entry for "wait-for" with options to signal, lock, or unlock a channel, and it manages a collection of wait channels using a red-black tree for efficient lookup and insertion.

The file includes several key components: the [`cmd_wait_for_exec`](#cmd_wait_for_exec) function, which serves as the main execution point for the "wait-for" command, and helper functions like [`cmd_wait_for_signal`](#cmd_wait_for_signal), [`cmd_wait_for_wait`](#cmd_wait_for_wait), [`cmd_wait_for_lock`](#cmd_wait_for_lock), and [`cmd_wait_for_unlock`](#cmd_wait_for_unlock), which handle specific actions on the wait channels. The `wait_channel` and `wait_item` structures are used to represent channels and the items waiting on them, respectively. The code also includes functions for adding and removing wait channels, ensuring that resources are managed correctly. This file is not a standalone executable but rather a part of the larger tmux codebase, providing specific synchronization functionality that can be invoked through the tmux command interface.
# Imports and Dependencies

---
- `sys/types.h`
- `stdlib.h`
- `string.h`
- `tmux.h`


# Global Variables

---
### cmd_wait_for_entry
- **Type**: `struct cmd_entry`
- **Description**: The `cmd_wait_for_entry` is a constant structure of type `cmd_entry` that defines the command 'wait-for' and its alias 'wait'. It specifies the arguments, usage, flags, and the execution function associated with this command.
- **Use**: This variable is used to register and define the behavior of the 'wait-for' command within the application, linking it to its execution logic.


---
### wait_channels
- **Type**: `struct wait_channels`
- **Description**: The `wait_channels` variable is a static instance of a red-black tree data structure, specifically designed to manage `wait_channel` structures. Each `wait_channel` represents a named synchronization point that can be locked, unlocked, or signaled, and contains lists of waiting and locking items. The red-black tree is initialized using the `RB_INITIALIZER` macro, which sets up the tree for subsequent operations like insertion and removal of `wait_channel` nodes.
- **Use**: This variable is used to store and manage multiple `wait_channel` instances, allowing for efficient synchronization operations across different named channels.


---
### cmd_wait_for_add
- **Type**: `function pointer`
- **Description**: `cmd_wait_for_add` is a static function that adds a new wait channel to the global `wait_channels` red-black tree. It allocates memory for a new `wait_channel` structure, initializes its fields, and inserts it into the tree.
- **Use**: This function is used to create and register a new wait channel by name, which can then be used for synchronization purposes in the program.


# Data Structures

---
### wait_item
- **Type**: `struct`
- **Members**:
    - `item`: A pointer to a cmdq_item structure, representing a command queue item associated with this wait item.
    - `entry`: A TAILQ_ENTRY macro for linking wait_item structures in a tail queue.
- **Description**: The `wait_item` structure is used to represent an individual item in a queue of wait operations within a command queue system. It contains a pointer to a `cmdq_item`, which is the command queue item associated with this wait operation, and a `TAILQ_ENTRY` for linking multiple `wait_item` structures together in a tail queue. This structure is part of a larger mechanism to manage blocking or waking clients on named wait channels, allowing for synchronization and coordination of command execution.


---
### wait_channel
- **Type**: `struct`
- **Members**:
    - `name`: A constant character pointer representing the name of the wait channel.
    - `locked`: An integer indicating whether the wait channel is currently locked.
    - `woken`: An integer indicating whether the wait channel has been woken.
    - `waiters`: A tail queue head for managing a list of wait items waiting on this channel.
    - `lockers`: A tail queue head for managing a list of wait items that have locked this channel.
    - `entry`: A red-black tree entry for managing the wait channel in a red-black tree.
- **Description**: The `wait_channel` structure is used to manage synchronization points in a system, allowing clients to block or wake on a named channel. It includes fields to track the channel's name, its lock and wake status, and lists of waiting and locking items. The structure is integrated into a red-black tree for efficient management and lookup of channels by name.


# Functions

---
### wait_channel_cmp<!-- {{#callable:wait_channel_cmp}} -->
The `wait_channel_cmp` function compares two `wait_channel` structures based on their `name` fields.
- **Inputs**:
    - `wc1`: A pointer to the first `wait_channel` structure to be compared.
    - `wc2`: A pointer to the second `wait_channel` structure to be compared.
- **Control Flow**:
    - The function uses the `strcmp` function to compare the `name` fields of the two `wait_channel` structures pointed to by `wc1` and `wc2`.
    - The result of the `strcmp` function is returned, which indicates the lexicographical order of the two names.
- **Output**:
    - The function returns an integer that is less than, equal to, or greater than zero if the `name` of `wc1` is found, respectively, to be less than, to match, or be greater than the `name` of `wc2`.


---
### cmd_wait_for_add<!-- {{#callable:cmd_wait_for_add}} -->
The `cmd_wait_for_add` function creates and initializes a new wait channel with a given name, adding it to a global collection of wait channels.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the wait channel to be created.
- **Control Flow**:
    - Allocate memory for a new `wait_channel` structure using `xmalloc`.
    - Duplicate the input `name` string using `xstrdup` and assign it to the `name` field of the `wait_channel`.
    - Initialize the `locked` and `woken` fields of the `wait_channel` to 0.
    - Initialize the `waiters` and `lockers` queues using `TAILQ_INIT`.
    - Insert the new `wait_channel` into the global `wait_channels` red-black tree using `RB_INSERT`.
    - Log a debug message indicating the addition of the wait channel.
    - Return the pointer to the newly created `wait_channel`.
- **Output**:
    - Returns a pointer to the newly created and initialized `wait_channel` structure.


---
### cmd_wait_for_remove<!-- {{#callable:cmd_wait_for_remove}} -->
The `cmd_wait_for_remove` function removes a wait channel from the global list if it is not locked, has no waiters, and is woken.
- **Inputs**:
    - `wc`: A pointer to a `wait_channel` structure that represents the wait channel to be removed.
- **Control Flow**:
    - Check if the wait channel is locked; if so, return immediately without doing anything.
    - Check if the wait channel has any waiters or is not woken; if either condition is true, return immediately without doing anything.
    - Log a debug message indicating the removal of the wait channel.
    - Remove the wait channel from the global red-black tree of wait channels.
    - Free the memory allocated for the wait channel's name and the wait channel itself.
- **Output**:
    - The function does not return any value; it performs operations to remove a wait channel from the system.


---
### cmd_wait_for_exec<!-- {{#callable:cmd_wait_for_exec}} -->
The `cmd_wait_for_exec` function executes a command to manage wait channels based on specified arguments, determining whether to signal, lock, unlock, or wait on a channel.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command to be executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item associated with the command.
- **Control Flow**:
    - Retrieve the command arguments using `cmd_get_args` and extract the channel name using `args_string`.
    - Initialize a `wait_channel` structure `find` with the channel name and search for it in the `wait_channels` red-black tree using `RB_FIND`.
    - Check if the 'S' argument is present using `args_has`; if so, call [`cmd_wait_for_signal`](#cmd_wait_for_signal) with the item, name, and wait channel, and return its result.
    - Check if the 'L' argument is present; if so, call [`cmd_wait_for_lock`](#cmd_wait_for_lock) with the item, name, and wait channel, and return its result.
    - Check if the 'U' argument is present; if so, call [`cmd_wait_for_unlock`](#cmd_wait_for_unlock) with the item, name, and wait channel, and return its result.
    - If none of the above arguments are present, call [`cmd_wait_for_wait`](#cmd_wait_for_wait) with the item, name, and wait channel, and return its result.
- **Output**:
    - The function returns an `enum cmd_retval` value indicating the result of the executed command, which could be normal, wait, or error.
- **Functions called**:
    - [`cmd_wait_for_signal`](#cmd_wait_for_signal)
    - [`cmd_wait_for_lock`](#cmd_wait_for_lock)
    - [`cmd_wait_for_unlock`](#cmd_wait_for_unlock)
    - [`cmd_wait_for_wait`](#cmd_wait_for_wait)


---
### cmd_wait_for_signal<!-- {{#callable:cmd_wait_for_signal}} -->
The `cmd_wait_for_signal` function signals a wait channel, waking up any waiting items and removing the channel if no waiters remain.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which is unused in this function.
    - `name`: A constant character pointer representing the name of the wait channel.
    - `wc`: A pointer to a `wait_channel` structure, which may be NULL if the channel does not exist.
- **Control Flow**:
    - Check if the `wait_channel` pointer `wc` is NULL; if so, create a new wait channel with the given name using [`cmd_wait_for_add`](#cmd_wait_for_add).
    - If the wait channel has no waiters and is not already woken, log a debug message, set the `woken` flag to 1, and return `CMD_RETURN_NORMAL`.
    - Log a debug message indicating the wait channel has waiters.
    - Iterate over each `wait_item` in the wait channel's waiters list using `TAILQ_FOREACH_SAFE`.
    - For each `wait_item`, call `cmdq_continue` to continue processing the item, remove it from the waiters list, and free its memory.
    - Call [`cmd_wait_for_remove`](#cmd_wait_for_remove) to remove the wait channel if it is no longer needed.
    - Return `CMD_RETURN_NORMAL` to indicate normal completion.
- **Output**:
    - The function returns an `enum cmd_retval` value, specifically `CMD_RETURN_NORMAL`, indicating the normal completion of the signal operation.
- **Functions called**:
    - [`cmd_wait_for_add`](#cmd_wait_for_add)
    - [`cmd_wait_for_remove`](#cmd_wait_for_remove)


---
### cmd_wait_for_wait<!-- {{#callable:cmd_wait_for_wait}} -->
The `cmd_wait_for_wait` function manages the waiting mechanism for a client on a specified wait channel, adding the client to the wait queue if the channel is not already woken.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item associated with the client.
    - `name`: A constant character pointer representing the name of the wait channel.
    - `wc`: A pointer to a `wait_channel` structure representing the wait channel; it can be NULL, in which case a new wait channel is created.
- **Control Flow**:
    - Retrieve the client associated with the `cmdq_item` using `cmdq_get_client` and store it in `c`.
    - Check if the client `c` is NULL; if so, log an error and return `CMD_RETURN_ERROR`.
    - If the `wait_channel` pointer `wc` is NULL, create a new wait channel using [`cmd_wait_for_add`](#cmd_wait_for_add).
    - Check if the wait channel `wc` is already woken; if so, log a debug message, remove the wait channel using [`cmd_wait_for_remove`](#cmd_wait_for_remove), and return `CMD_RETURN_NORMAL`.
    - Log a debug message indicating the wait channel is not woken.
    - Allocate memory for a new `wait_item` structure, set its `item` field to the provided `cmdq_item`, and insert it into the `waiters` queue of the wait channel `wc`.
    - Return `CMD_RETURN_WAIT` to indicate the client is now waiting on the channel.
- **Output**:
    - The function returns an `enum cmd_retval` which can be `CMD_RETURN_ERROR`, `CMD_RETURN_NORMAL`, or `CMD_RETURN_WAIT`, indicating the result of the wait operation.
- **Functions called**:
    - [`cmd_wait_for_add`](#cmd_wait_for_add)
    - [`cmd_wait_for_remove`](#cmd_wait_for_remove)


---
### cmd_wait_for_lock<!-- {{#callable:cmd_wait_for_lock}} -->
The `cmd_wait_for_lock` function attempts to lock a named wait channel for a command queue item, adding the item to a queue if the channel is already locked.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item that is attempting to lock the channel.
    - `name`: A string representing the name of the wait channel to be locked.
    - `wc`: A pointer to a `wait_channel` structure representing the wait channel to be locked; if NULL, a new channel is created with the given name.
- **Control Flow**:
    - Check if the client associated with the `cmdq_item` is NULL; if so, log an error and return `CMD_RETURN_ERROR`.
    - If the `wait_channel` pointer `wc` is NULL, call [`cmd_wait_for_add`](#cmd_wait_for_add) to create or retrieve a wait channel with the given name.
    - Check if the wait channel is already locked; if it is, allocate a new `wait_item`, associate it with the `cmdq_item`, and add it to the channel's `lockers` queue, then return `CMD_RETURN_WAIT`.
    - If the channel is not locked, set its `locked` status to 1 and return `CMD_RETURN_NORMAL`.
- **Output**:
    - The function returns an `enum cmd_retval` which can be `CMD_RETURN_ERROR`, `CMD_RETURN_WAIT`, or `CMD_RETURN_NORMAL`, indicating the result of the lock attempt.
- **Functions called**:
    - [`cmd_wait_for_add`](#cmd_wait_for_add)


---
### cmd_wait_for_unlock<!-- {{#callable:cmd_wait_for_unlock}} -->
The `cmd_wait_for_unlock` function attempts to unlock a specified wait channel and continues any queued command items if the channel is locked.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item associated with the unlock operation.
    - `name`: A constant character pointer representing the name of the wait channel to be unlocked.
    - `wc`: A pointer to a `wait_channel` structure representing the wait channel to be unlocked.
- **Control Flow**:
    - Check if the wait channel (`wc`) is NULL or not locked; if so, log an error message and return `CMD_RETURN_ERROR`.
    - If the wait channel is locked, check if there are any items in the `lockers` queue.
    - If there is an item in the `lockers` queue, continue the command queue item associated with it, remove it from the queue, and free its memory.
    - If there are no items in the `lockers` queue, set the `locked` status of the wait channel to 0 and call [`cmd_wait_for_remove`](#cmd_wait_for_remove) to potentially remove the wait channel.
    - Return `CMD_RETURN_NORMAL` to indicate successful completion.
- **Output**:
    - The function returns an `enum cmd_retval` which can be either `CMD_RETURN_ERROR` if the channel is not locked or `CMD_RETURN_NORMAL` if the unlock operation is successful.
- **Functions called**:
    - [`cmd_wait_for_remove`](#cmd_wait_for_remove)


---
### cmd_wait_for_flush<!-- {{#callable:cmd_wait_for_flush}} -->
The `cmd_wait_for_flush` function iterates over all wait channels, continuing and removing all wait and lock items, marking channels as woken, and unlocking them before removing the channels if possible.
- **Inputs**:
    - None
- **Control Flow**:
    - Iterate over each wait channel in the global `wait_channels` red-black tree using `RB_FOREACH_SAFE`.
    - For each wait channel, iterate over its `waiters` queue using `TAILQ_FOREACH_SAFE`.
    - For each wait item in the `waiters` queue, call `cmdq_continue` on the item, remove it from the queue, and free its memory.
    - Set the `woken` flag of the wait channel to 1.
    - Iterate over the `lockers` queue of the wait channel using `TAILQ_FOREACH_SAFE`.
    - For each lock item in the `lockers` queue, call `cmdq_continue` on the item, remove it from the queue, and free its memory.
    - Set the `locked` flag of the wait channel to 0.
    - Call [`cmd_wait_for_remove`](#cmd_wait_for_remove) to attempt to remove the wait channel if it is no longer needed.
- **Output**:
    - The function does not return any value; it modifies the state of wait channels and their associated items.
- **Functions called**:
    - [`cmd_wait_for_remove`](#cmd_wait_for_remove)


# Function Declarations (Public API)

---
### cmd_wait_for_exec<!-- {{#callable_declaration:cmd_wait_for_exec}} -->
Manage client blocking or waking on a named wait channel.
- **Description**: This function is used to control the blocking or waking of a client on a specified wait channel, based on the provided command arguments. It supports signaling, locking, unlocking, or waiting on a channel, depending on the flags passed. This function should be called with a valid command and command queue item, and it assumes that the command arguments have been properly initialized. The function handles different scenarios based on the presence of specific flags ('S', 'L', 'U') and performs the corresponding action on the wait channel.
- **Inputs**:
    - `self`: A pointer to a struct cmd representing the command to be executed. Must not be null and should be properly initialized with command arguments.
    - `item`: A pointer to a struct cmdq_item representing the command queue item. Must not be null and should be associated with a valid client context.
- **Output**: Returns an enum cmd_retval indicating the result of the operation, which can be normal, wait, or error based on the action performed and the state of the wait channel.
- **See also**: [`cmd_wait_for_exec`](#cmd_wait_for_exec)  (Implementation)


---
### wait_channel_cmp<!-- {{#callable_declaration:wait_channel_cmp}} -->
Compare two wait channels by their names.
- **Description**: Use this function to determine the lexicographical order of two wait channels based on their names. It is typically used in contexts where wait channels need to be sorted or organized in a specific order. The function assumes that both wait channels have valid, non-null name strings. It is important to ensure that the wait channels being compared are properly initialized and that their name fields are not null to avoid undefined behavior.
- **Inputs**:
    - `wc1`: A pointer to the first wait channel to compare. Must not be null and must have a valid name string.
    - `wc2`: A pointer to the second wait channel to compare. Must not be null and must have a valid name string.
- **Output**: Returns an integer less than, equal to, or greater than zero if the name of wc1 is found, respectively, to be less than, to match, or be greater than the name of wc2.
- **See also**: [`wait_channel_cmp`](#wait_channel_cmp)  (Implementation)


---
### cmd_wait_for_signal<!-- {{#callable_declaration:cmd_wait_for_signal}} -->
Signals a wait channel to wake up waiting clients.
- **Description**: Use this function to signal a named wait channel, potentially waking up any clients that are waiting on it. This function should be called when an event occurs that should wake up clients waiting on the specified channel. If the channel has no waiters and has not been woken, it will be marked as woken. If there are waiters, they will be continued and the channel will be removed. This function is typically used in scenarios where clients need to be synchronized or notified of certain events.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which is unused in this function. The caller retains ownership and it can be null.
    - `name`: A string representing the name of the wait channel to signal. It must not be null and should be a valid, existing channel name.
    - `wc`: A pointer to a `wait_channel` structure representing the wait channel to signal. If null, a new wait channel will be created with the given name.
- **Output**: Returns `CMD_RETURN_NORMAL` to indicate successful signaling of the wait channel.
- **See also**: [`cmd_wait_for_signal`](#cmd_wait_for_signal)  (Implementation)


---
### cmd_wait_for_wait<!-- {{#callable_declaration:cmd_wait_for_wait}} -->
Blocks a client on a named wait channel until it is woken.
- **Description**: This function is used to block a client on a specified wait channel, identified by a name, until the channel is woken. It should be called when a client needs to wait for a specific event or condition to be met. If the client is not available, the function will return an error. If the wait channel is already woken, the function will return immediately with a normal status. Otherwise, the client is added to the list of waiters for the channel. This function is typically used in scenarios where synchronization between different parts of a program is required.
- **Inputs**:
    - `item`: A pointer to a cmdq_item structure representing the client command queue item. Must not be null. If the client cannot be retrieved from this item, the function returns an error.
    - `name`: A string representing the name of the wait channel. This identifies the channel on which the client will wait. The caller retains ownership of this string.
    - `wc`: A pointer to a wait_channel structure representing the wait channel. Can be null, in which case a new wait channel is created with the given name. The function handles null values by creating a new channel if necessary.
- **Output**: Returns CMD_RETURN_ERROR if the client is not available, CMD_RETURN_NORMAL if the channel is already woken, and CMD_RETURN_WAIT if the client is successfully added to the waiters list.
- **See also**: [`cmd_wait_for_wait`](#cmd_wait_for_wait)  (Implementation)


---
### cmd_wait_for_lock<!-- {{#callable_declaration:cmd_wait_for_lock}} -->
Blocks or queues a command item on a named wait channel if it is locked.
- **Description**: This function is used to manage synchronization by blocking or queuing a command item on a specified wait channel identified by a name. It should be called when a command needs to wait for a resource or condition to be available. If the wait channel is already locked, the command item is queued, and the function returns a wait status. If the channel is not locked, it locks the channel and returns a normal status. The function requires a valid command queue item and handles cases where the client is unavailable by returning an error status.
- **Inputs**:
    - `item`: A pointer to a struct cmdq_item representing the command queue item to be blocked or queued. Must not be null, and the function will return an error if the associated client is unavailable.
    - `name`: A string representing the name of the wait channel. This identifies the channel on which the command item will be blocked or queued. Must not be null.
    - `wc`: A pointer to a struct wait_channel representing the wait channel. Can be null, in which case a new wait channel is created with the specified name.
- **Output**: Returns an enum cmd_retval indicating the result: CMD_RETURN_ERROR if the client is unavailable, CMD_RETURN_WAIT if the item is queued, or CMD_RETURN_NORMAL if the channel is successfully locked.
- **See also**: [`cmd_wait_for_lock`](#cmd_wait_for_lock)  (Implementation)


---
### cmd_wait_for_unlock<!-- {{#callable_declaration:cmd_wait_for_unlock}} -->
Unlocks a wait channel if it is locked.
- **Description**: This function is used to unlock a specified wait channel if it is currently locked. It should be called when a lock on a wait channel needs to be released, allowing any queued items to proceed. If the channel is not locked, an error is reported. This function is typically used in scenarios where synchronization between different command queue items is required. It is important to ensure that the wait channel is valid and locked before calling this function to avoid errors.
- **Inputs**:
    - `item`: A pointer to a struct cmdq_item representing the command queue item associated with the operation. Must not be null.
    - `name`: A string representing the name of the wait channel to be unlocked. The name is used for error reporting and must correspond to an existing channel.
    - `wc`: A pointer to a struct wait_channel representing the wait channel to be unlocked. Must not be null and must point to a locked channel; otherwise, an error is reported.
- **Output**: Returns CMD_RETURN_NORMAL if the channel is successfully unlocked or CMD_RETURN_ERROR if the channel is not locked.
- **See also**: [`cmd_wait_for_unlock`](#cmd_wait_for_unlock)  (Implementation)


---
### cmd_wait_for_add<!-- {{#callable_declaration:cmd_wait_for_add}} -->
Creates and registers a new wait channel with a specified name.
- **Description**: Use this function to create a new wait channel identified by a unique name. This function allocates memory for the wait channel and initializes its internal structures. It is typically used when a new synchronization point is needed for clients to wait on or be signaled. The function must be called with a valid, non-null name string, and the caller is responsible for ensuring the uniqueness of the name to avoid conflicts. The created wait channel is automatically registered in the global wait channel registry.
- **Inputs**:
    - `name`: A non-null string representing the name of the wait channel. The name must be unique among all wait channels to avoid conflicts. The caller retains ownership of the string, but the function duplicates it internally.
- **Output**: Returns a pointer to the newly created wait channel structure.
- **See also**: [`cmd_wait_for_add`](#cmd_wait_for_add)  (Implementation)


---
### cmd_wait_for_remove<!-- {{#callable_declaration:cmd_wait_for_remove}} -->
Removes a wait channel if it is not locked and has no active waiters.
- **Description**: This function is used to safely remove a wait channel from the system when it is no longer needed. It should be called when you want to clean up a wait channel that is not locked and has no active waiters. The function checks if the channel is locked or if there are any waiters; if either condition is true, the channel is not removed. This ensures that no active operations are disrupted. The function also handles the necessary cleanup of resources associated with the wait channel.
- **Inputs**:
    - `wc`: A pointer to a `struct wait_channel` representing the wait channel to be removed. The channel must not be locked, and it must have no active waiters for the removal to proceed. The caller retains ownership of the pointer, but the function will free the resources associated with the wait channel if it is removed.
- **Output**: None
- **See also**: [`cmd_wait_for_remove`](#cmd_wait_for_remove)  (Implementation)


