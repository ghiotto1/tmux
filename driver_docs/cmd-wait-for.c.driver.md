# Purpose
This C source code file is part of the tmux project, a terminal multiplexer, and it implements functionality for managing synchronization between clients using named wait channels. The primary purpose of this code is to provide mechanisms for clients to block or wake on specific channels, allowing for coordination and synchronization of tasks. The code defines a command entry for "wait-for" with options to signal, lock, or unlock a channel, and it manages a collection of wait channels using a red-black tree for efficient lookup and insertion.

The file includes several key components: the [`cmd_wait_for_exec`](#cmd_wait_for_exec) function, which executes the appropriate action based on command-line arguments; the `wait_channel` structure, which represents a named channel with associated waiters and lockers; and various functions to add, remove, and manipulate these channels. The code provides a narrow but essential functionality within the tmux ecosystem, focusing on inter-client communication and synchronization. It does not define a public API but rather integrates into the broader tmux command queue system, allowing clients to wait for or signal events on specific channels.
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
- **Description**: The `wait_channels` variable is a static instance of a red-black tree data structure, specifically designed to manage `wait_channel` structures. Each `wait_channel` represents a named synchronization point that can be locked, unlocked, or signaled, and can have multiple waiters or lockers associated with it.
- **Use**: This variable is used to store and manage all active wait channels, allowing operations such as adding, removing, locking, unlocking, and signaling wait channels.


---
### cmd_wait_for_add
- **Type**: `function`
- **Description**: The `cmd_wait_for_add` function is a static function that creates and initializes a new `wait_channel` structure with a given name. It allocates memory for the `wait_channel`, sets its name, initializes its `locked` and `woken` states to 0, and initializes its `waiters` and `lockers` queues.
- **Use**: This function is used to add a new wait channel to the global `wait_channels` red-black tree, allowing clients to wait on or signal the channel.


# Data Structures

---
### wait_item
- **Type**: `struct`
- **Members**:
    - `item`: A pointer to a cmdq_item structure, representing a command queue item associated with this wait item.
    - `entry`: A TAILQ_ENTRY macro for linking wait_item structures in a tail queue.
- **Description**: The `wait_item` structure is a component of a wait channel mechanism used to manage command queue items that are waiting for a specific event or condition to be met. It contains a pointer to a `cmdq_item`, which represents the command queue item that is waiting, and a `TAILQ_ENTRY` for linking multiple `wait_item` structures together in a tail queue, allowing for efficient insertion and removal operations.


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
- **Description**: The `wait_channel` structure is used to manage synchronization points in a system, allowing clients to block or wake on a named channel. It includes fields to track the channel's name, its lock and wake status, and lists of items waiting or locking the channel. The structure is integrated into a red-black tree for efficient management and lookup of channels.


# Functions

---
### wait_channel_cmp<!-- {{#callable:wait_channel_cmp}} -->
The `wait_channel_cmp` function compares two `wait_channel` structures based on their `name` fields using the `strcmp` function.
- **Inputs**:
    - `wc1`: A pointer to the first `wait_channel` structure to be compared.
    - `wc2`: A pointer to the second `wait_channel` structure to be compared.
- **Control Flow**:
    - The function takes two `wait_channel` pointers as input.
    - It uses the `strcmp` function to compare the `name` fields of the two `wait_channel` structures.
    - The result of the `strcmp` function is returned, which indicates the lexicographical order of the two names.
- **Output**:
    - The function returns an integer that is less than, equal to, or greater than zero if the `name` of `wc1` is found, respectively, to be less than, to match, or be greater than the `name` of `wc2`.


---
### cmd_wait_for_add<!-- {{#callable:cmd_wait_for_add}} -->
The `cmd_wait_for_add` function creates and initializes a new wait channel with a given name, adding it to a global collection of wait channels.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the wait channel to be created.
- **Control Flow**:
    - Allocate memory for a new `wait_channel` structure using [`xmalloc`](xmalloc.c.driver.md#xmalloc).
    - Duplicate the input name string using [`xstrdup`](xmalloc.c.driver.md#xstrdup) and assign it to the `name` field of the `wait_channel`.
    - Initialize the `locked` and `woken` fields of the `wait_channel` to 0.
    - Initialize the `waiters` and `lockers` queues using `TAILQ_INIT`.
    - Insert the new `wait_channel` into the global `wait_channels` red-black tree using `RB_INSERT`.
    - Log a debug message indicating the addition of the new wait channel.
    - Return a pointer to the newly created `wait_channel`.
- **Output**:
    - A pointer to the newly created and initialized `wait_channel` structure.
- **Functions called**:
    - [`xmalloc`](xmalloc.c.driver.md#xmalloc)
    - [`xstrdup`](xmalloc.c.driver.md#xstrdup)


---
### cmd_wait_for_remove<!-- {{#callable:cmd_wait_for_remove}} -->
The `cmd_wait_for_remove` function removes a wait channel from the global list if it is not locked, has no waiters, and is woken.
- **Inputs**:
    - `wc`: A pointer to a `struct wait_channel` representing the wait channel to be removed.
- **Control Flow**:
    - Check if the wait channel is locked; if so, return immediately without doing anything.
    - Check if the wait channel has any waiters or is not woken; if either condition is true, return immediately without doing anything.
    - Log a debug message indicating the removal of the wait channel.
    - Remove the wait channel from the global red-black tree of wait channels.
    - Free the memory allocated for the wait channel's name and the wait channel itself.
- **Output**:
    - The function does not return any value; it performs operations to remove and free a wait channel if certain conditions are met.


---
### cmd_wait_for_exec<!-- {{#callable:cmd_wait_for_exec}} -->
The `cmd_wait_for_exec` function determines the appropriate action to take on a wait channel based on command arguments, such as signaling, locking, unlocking, or waiting.
- **Inputs**:
    - `self`: A pointer to the `cmd` structure representing the command being executed.
    - `item`: A pointer to the `cmdq_item` structure representing the command queue item associated with the command.
- **Control Flow**:
    - Retrieve the command arguments using [`cmd_get_args`](cmd.c.driver.md#cmd_get_args) and extract the channel name from the arguments.
    - Initialize a `wait_channel` structure `find` with the channel name and search for an existing wait channel using `RB_FIND`.
    - Check if the command arguments include the 'S' flag; if so, call [`cmd_wait_for_signal`](#cmd_wait_for_signal) with the item, name, and wait channel.
    - Check if the command arguments include the 'L' flag; if so, call [`cmd_wait_for_lock`](#cmd_wait_for_lock) with the item, name, and wait channel.
    - Check if the command arguments include the 'U' flag; if so, call [`cmd_wait_for_unlock`](#cmd_wait_for_unlock) with the item, name, and wait channel.
    - If none of the above flags are present, call [`cmd_wait_for_wait`](#cmd_wait_for_wait) with the item, name, and wait channel.
- **Output**:
    - The function returns an `enum cmd_retval` indicating the result of the executed command, which could be normal, wait, or error.
- **Functions called**:
    - [`cmd_get_args`](cmd.c.driver.md#cmd_get_args)
    - [`args_string`](arguments.c.driver.md#args_string)
    - [`args_has`](arguments.c.driver.md#args_has)
    - [`cmd_wait_for_signal`](#cmd_wait_for_signal)
    - [`cmd_wait_for_lock`](#cmd_wait_for_lock)
    - [`cmd_wait_for_unlock`](#cmd_wait_for_unlock)
    - [`cmd_wait_for_wait`](#cmd_wait_for_wait)


---
### cmd_wait_for_signal<!-- {{#callable:cmd_wait_for_signal}} -->
The `cmd_wait_for_signal` function signals a wait channel, waking up any waiting items and removing the channel if no waiters remain.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which is unused in this function.
    - `name`: A constant character pointer representing the name of the wait channel to signal.
    - `wc`: A pointer to a `wait_channel` structure representing the wait channel to be signaled; if NULL, a new wait channel is added.
- **Control Flow**:
    - Check if the `wait_channel` pointer `wc` is NULL; if so, add a new wait channel with the given name using [`cmd_wait_for_add`](#cmd_wait_for_add).
    - If the wait channel has no waiters and is not already woken, log a debug message, set the `woken` flag to 1, and return `CMD_RETURN_NORMAL`.
    - Log a debug message indicating the wait channel has waiters.
    - Iterate over each `wait_item` in the wait channel's waiters list using `TAILQ_FOREACH_SAFE`.
    - For each `wait_item`, call [`cmdq_continue`](cmd-queue.c.driver.md#cmdq_continue) to continue the queued item, remove the `wait_item` from the waiters list, and free its memory.
    - Call [`cmd_wait_for_remove`](#cmd_wait_for_remove) to remove the wait channel if it is no longer needed.
    - Return `CMD_RETURN_NORMAL`.
- **Output**:
    - The function returns an `enum cmd_retval`, specifically `CMD_RETURN_NORMAL`, indicating the normal completion of the signal operation.
- **Functions called**:
    - [`cmd_wait_for_add`](#cmd_wait_for_add)
    - [`cmdq_continue`](cmd-queue.c.driver.md#cmdq_continue)
    - [`cmd_wait_for_remove`](#cmd_wait_for_remove)


---
### cmd_wait_for_wait<!-- {{#callable:cmd_wait_for_wait}} -->
The `cmd_wait_for_wait` function manages a wait operation on a named wait channel, adding the command queue item to the channel's waiters list if the channel is not already woken.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item that is waiting.
    - `name`: A string representing the name of the wait channel.
    - `wc`: A pointer to a `wait_channel` structure representing the wait channel, which can be NULL if the channel does not exist yet.
- **Control Flow**:
    - Retrieve the client associated with the command queue item using [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client).
    - If the client is NULL, log an error message and return `CMD_RETURN_ERROR`.
    - If the wait channel (`wc`) is NULL, create a new wait channel using [`cmd_wait_for_add`](#cmd_wait_for_add).
    - Check if the wait channel is already woken; if so, log a debug message, remove the wait channel using [`cmd_wait_for_remove`](#cmd_wait_for_remove), and return `CMD_RETURN_NORMAL`.
    - If the wait channel is not woken, log a debug message, allocate memory for a new `wait_item`, set its `item` to the provided `cmdq_item`, and insert it into the waiters list of the wait channel.
    - Return `CMD_RETURN_WAIT` to indicate that the item is waiting.
- **Output**:
    - The function returns an `enum cmd_retval` which can be `CMD_RETURN_ERROR`, `CMD_RETURN_NORMAL`, or `CMD_RETURN_WAIT`, indicating the result of the wait operation.
- **Functions called**:
    - [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client)
    - [`cmd_wait_for_add`](#cmd_wait_for_add)
    - [`cmd_wait_for_remove`](#cmd_wait_for_remove)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)


---
### cmd_wait_for_lock<!-- {{#callable:cmd_wait_for_lock}} -->
The `cmd_wait_for_lock` function attempts to lock a named wait channel for a command queue item, adding the item to a queue if the channel is already locked.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item that is attempting to lock the channel.
    - `name`: A constant character pointer representing the name of the wait channel to be locked.
    - `wc`: A pointer to a `wait_channel` structure representing the wait channel to be locked; if NULL, a new channel is created with the given name.
- **Control Flow**:
    - Check if the client associated with the `cmdq_item` is NULL; if so, log an error and return `CMD_RETURN_ERROR`.
    - If the `wait_channel` pointer `wc` is NULL, call [`cmd_wait_for_add`](#cmd_wait_for_add) to create a new wait channel with the given name.
    - Check if the wait channel is already locked; if so, allocate a new `wait_item`, associate it with the `cmdq_item`, and add it to the channel's `lockers` queue, then return `CMD_RETURN_WAIT`.
    - If the channel is not locked, set its `locked` attribute to 1 and return `CMD_RETURN_NORMAL`.
- **Output**:
    - The function returns an `enum cmd_retval` which can be `CMD_RETURN_ERROR`, `CMD_RETURN_WAIT`, or `CMD_RETURN_NORMAL`, indicating the result of the lock attempt.
- **Functions called**:
    - [`cmdq_get_client`](cmd-queue.c.driver.md#cmdq_get_client)
    - [`cmd_wait_for_add`](#cmd_wait_for_add)
    - [`xcalloc`](xmalloc.c.driver.md#xcalloc)


---
### cmd_wait_for_unlock<!-- {{#callable:cmd_wait_for_unlock}} -->
The `cmd_wait_for_unlock` function attempts to unlock a specified wait channel and continues any queued command items if the channel is locked.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item associated with the unlock operation.
    - `name`: A string representing the name of the wait channel to be unlocked.
    - `wc`: A pointer to a `wait_channel` structure representing the wait channel to be unlocked.
- **Control Flow**:
    - Check if the wait channel (`wc`) is NULL or not locked; if so, log an error message and return `CMD_RETURN_ERROR`.
    - Retrieve the first item from the `lockers` queue of the wait channel.
    - If there is a queued item, continue its execution, remove it from the queue, and free its memory.
    - If there are no queued items, set the `locked` status of the wait channel to 0 and call [`cmd_wait_for_remove`](#cmd_wait_for_remove) to potentially remove the channel.
    - Return `CMD_RETURN_NORMAL` to indicate successful completion.
- **Output**:
    - The function returns an `enum cmd_retval` value, which is either `CMD_RETURN_ERROR` if the channel is not locked or `CMD_RETURN_NORMAL` if the unlock operation is successful.
- **Functions called**:
    - [`cmdq_continue`](cmd-queue.c.driver.md#cmdq_continue)
    - [`cmd_wait_for_remove`](#cmd_wait_for_remove)


---
### cmd_wait_for_flush<!-- {{#callable:cmd_wait_for_flush}} -->
The `cmd_wait_for_flush` function iterates over all wait channels, continuing and removing all wait and lock items, marking channels as woken, and unlocking them before removing the channels if possible.
- **Inputs**:
    - None
- **Control Flow**:
    - Iterate over each wait channel in the `wait_channels` red-black tree using `RB_FOREACH_SAFE`.
    - For each wait channel, iterate over all wait items in the `waiters` queue using `TAILQ_FOREACH_SAFE`.
    - For each wait item, call [`cmdq_continue`](cmd-queue.c.driver.md#cmdq_continue) on the item, remove it from the queue, and free its memory.
    - Set the `woken` flag of the wait channel to 1.
    - Iterate over all lock items in the `lockers` queue using `TAILQ_FOREACH_SAFE`.
    - For each lock item, call [`cmdq_continue`](cmd-queue.c.driver.md#cmdq_continue) on the item, remove it from the queue, and free its memory.
    - Set the `locked` flag of the wait channel to 0.
    - Call [`cmd_wait_for_remove`](#cmd_wait_for_remove) to attempt to remove the wait channel if it is no longer needed.
- **Output**:
    - The function does not return any value; it modifies the state of wait channels and their associated items.
- **Functions called**:
    - [`cmdq_continue`](cmd-queue.c.driver.md#cmdq_continue)
    - [`cmd_wait_for_remove`](#cmd_wait_for_remove)


