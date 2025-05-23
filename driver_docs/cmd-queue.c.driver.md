# Purpose
The provided C source code is part of the `tmux` project, specifically handling the command queue system. This code is responsible for managing the execution of commands within `tmux`, a terminal multiplexer that allows users to switch between several programs in one terminal, detach them, and reattach them later. The command queue system is a critical component that ensures commands are processed in an orderly fashion, supporting both synchronous and asynchronous execution.

Key components of this code include the definition of structures such as `cmdq_item`, `cmdq_state`, and `cmdq_list`, which represent individual command queue items, the state of the command queue, and the queue itself, respectively. The code provides functions to create, manage, and execute these command queue items, including handling command callbacks, managing command states, and processing command execution results. The code also includes mechanisms for handling command hooks, error reporting, and client interactions, making it a comprehensive implementation for managing command execution within the `tmux` environment. This file is not a standalone executable but rather a part of the larger `tmux` codebase, intended to be compiled and linked with other components to form the complete application.
# Imports and Dependencies

---
- `sys/types.h`
- `ctype.h`
- `pwd.h`
- `stdlib.h`
- `string.h`
- `time.h`
- `unistd.h`
- `tmux.h`


# Data Structures

---
### cmdq_type
- **Type**: `enum`
- **Members**:
    - `CMDQ_COMMAND`: Represents a command type in the command queue.
    - `CMDQ_CALLBACK`: Represents a callback type in the command queue.
- **Description**: The `cmdq_type` is an enumeration that defines the types of items that can exist in a command queue within the tmux application. It includes two possible values: `CMDQ_COMMAND`, which indicates that the item is a command, and `CMDQ_CALLBACK`, which indicates that the item is a callback function. This enumeration is used to differentiate between different types of operations that can be queued and processed in the command queue system.


---
### cmdq_item
- **Type**: `struct`
- **Members**:
    - `name`: A pointer to a character array representing the name of the command queue item.
    - `queue`: A pointer to the command queue list to which this item belongs.
    - `next`: A pointer to the next command queue item in the list.
    - `client`: A pointer to the client associated with this command queue item.
    - `target_client`: A pointer to the target client for this command queue item.
    - `type`: An enumeration value indicating the type of the command queue item (command or callback).
    - `group`: An unsigned integer representing the group number of the command queue item.
    - `number`: An unsigned integer representing the sequence number of the command queue item.
    - `time`: A time_t value representing the time when the command queue item was created or fired.
    - `flags`: An integer representing various flags associated with the command queue item.
    - `state`: A pointer to the command queue state associated with this item.
    - `source`: A cmd_find_state structure representing the source state for the command.
    - `target`: A cmd_find_state structure representing the target state for the command.
    - `cmdlist`: A pointer to the list of commands associated with this item.
    - `cmd`: A pointer to the command associated with this item.
    - `cb`: A callback function pointer for the command queue item.
    - `data`: A pointer to any additional data associated with the command queue item.
    - `entry`: A TAILQ_ENTRY structure for linking this item in a tail queue.
- **Description**: The `cmdq_item` structure represents an item in a command queue, which is part of a system for managing and executing commands in a sequential manner. Each item in the queue can represent either a command or a callback, and it contains information about the command's execution context, such as the associated client, target client, and state. The structure also includes fields for managing the item's position in the queue, its execution time, and any additional data or callback functions needed for its execution. This structure is used within a larger framework to handle command execution in a controlled and organized way, allowing for complex command sequences and interactions with clients.


---
### cmdq_state
- **Type**: `struct`
- **Members**:
    - `references`: An integer representing the number of references to this state.
    - `flags`: An integer representing various flags associated with the command queue state.
    - `formats`: A pointer to a format_tree structure, which holds additional formats for commands.
    - `event`: A key_event structure representing the key event that triggered the command.
    - `current`: A cmd_find_state structure representing the current default target state.
- **Description**: The `cmdq_state` structure is used to maintain the context for commands in a command queue. It includes information about how commands were triggered, such as the key event and flags, as well as any additional formats that may be applied to the commands. The structure also keeps track of the current default target state, allowing multiple commands to share the same context and potentially update the default target. This structure is essential for managing the execution state and context of commands within the command queue system.


---
### cmdq_list
- **Type**: `struct`
- **Members**:
    - `item`: A pointer to a `cmdq_item` structure, representing a single command queue item.
    - `list`: A `cmdq_item_list` structure, which is a tail queue of `cmdq_item` structures.
- **Description**: The `cmdq_list` structure is used to represent a command queue in the system. It contains a pointer to a single command queue item (`cmdq_item`) and a list of such items (`cmdq_item_list`). This structure is part of a larger system for managing command execution, where commands are queued and processed in sequence. The `cmdq_list` acts as a container for managing and organizing these command items, allowing for operations such as appending, inserting, and removing command items from the queue.


# Functions

---
### cmdq_name<!-- {{#callable:cmdq_name}} -->
The `cmdq_name` function returns a string representation of a client's name or a pointer address if the name is not set.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client whose name or address is to be returned.
- **Control Flow**:
    - Declare a static character array `s` of size 256 to hold the resulting string.
    - Check if the input client pointer `c` is `NULL`. If it is, return the string `"<global>"`.
    - If the client's `name` field is not `NULL`, format the string `s` to include the client's name enclosed in angle brackets using `xsnprintf`.
    - If the client's `name` field is `NULL`, format the string `s` to include the client's pointer address enclosed in angle brackets using `xsnprintf`.
    - Return the formatted string `s`.
- **Output**:
    - A constant character pointer to a string representing the client's name or pointer address.


---
### cmdq_get<!-- {{#callable:cmdq_get}} -->
The `cmdq_get` function retrieves the command queue associated with a given client or returns a global queue if the client is NULL.
- **Inputs**:
    - `c`: A pointer to a `struct client`, representing the client for which the command queue is to be retrieved.
- **Control Flow**:
    - Check if the input client `c` is NULL.
    - If `c` is NULL, check if the static `global_queue` is NULL.
    - If `global_queue` is NULL, initialize it by calling `cmdq_new()`.
    - Return the `global_queue` if `c` is NULL.
    - If `c` is not NULL, return the `queue` member of the client `c`.
- **Output**:
    - Returns a pointer to a `struct cmdq_list`, which is the command queue associated with the client or the global queue if the client is NULL.
- **Functions called**:
    - [`cmdq_new`](#cmdq_new)


---
### cmdq_new<!-- {{#callable:cmdq_new}} -->
The `cmdq_new` function creates and initializes a new command queue list structure.
- **Inputs**:
    - None
- **Control Flow**:
    - Allocate memory for a new `cmdq_list` structure using `xcalloc`, which initializes the allocated memory to zero.
    - Initialize the `list` member of the `cmdq_list` structure using `TAILQ_INIT` to set up the tail queue.
    - Return the pointer to the newly created and initialized `cmdq_list` structure.
- **Output**:
    - A pointer to a newly allocated and initialized `cmdq_list` structure is returned.


---
### cmdq_free<!-- {{#callable:cmdq_free}} -->
The `cmdq_free` function deallocates a command queue structure after ensuring it is empty.
- **Inputs**:
    - `queue`: A pointer to a `cmdq_list` structure representing the command queue to be freed.
- **Control Flow**:
    - Check if the command queue list is not empty using `TAILQ_EMPTY`.
    - If the queue is not empty, call `fatalx` to terminate the program with an error message.
    - If the queue is empty, deallocate the memory for the `cmdq_list` structure using `free`.
- **Output**:
    - The function does not return any value; it performs a void operation.


---
### cmdq_get_name<!-- {{#callable:cmdq_get_name}} -->
The `cmdq_get_name` function retrieves the name of a command queue item.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, representing the command queue item whose name is to be retrieved.
- **Control Flow**:
    - The function takes a single argument, `item`, which is a pointer to a `cmdq_item` structure.
    - It directly returns the `name` field of the `cmdq_item` structure pointed to by `item`.
- **Output**:
    - The function returns a `const char *`, which is the name of the command queue item.


---
### cmdq_get_client<!-- {{#callable:cmdq_get_client}} -->
The `cmdq_get_client` function retrieves the client associated with a given command queue item.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure from which the client is to be retrieved.
- **Control Flow**:
    - The function takes a single argument, `item`, which is a pointer to a `cmdq_item` structure.
    - It directly returns the `client` member of the `cmdq_item` structure.
- **Output**:
    - A pointer to a `client` structure associated with the given `cmdq_item`.


---
### cmdq_get_target_client<!-- {{#callable:cmdq_get_target_client}} -->
The `cmdq_get_target_client` function retrieves the target client associated with a given command queue item.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which represents an item in the command queue.
- **Control Flow**:
    - The function directly returns the `target_client` member of the `cmdq_item` structure pointed to by `item`.
- **Output**:
    - A pointer to a `client` structure, representing the target client of the command queue item.


---
### cmdq_get_state<!-- {{#callable:cmdq_get_state}} -->
The `cmdq_get_state` function retrieves the state associated with a given command queue item.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure from which the state is to be retrieved.
- **Control Flow**:
    - The function directly returns the `state` member of the `cmdq_item` structure pointed to by the `item` argument.
- **Output**:
    - A pointer to a `cmdq_state` structure, which represents the state of the command queue item.


---
### cmdq_get_target<!-- {{#callable:cmdq_get_target}} -->
The `cmdq_get_target` function retrieves the target command find state from a given command queue item.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure from which the target command find state is to be retrieved.
- **Control Flow**:
    - The function takes a single argument, `item`, which is a pointer to a `cmdq_item` structure.
    - It returns the address of the `target` field within the `cmdq_item` structure.
- **Output**:
    - A pointer to a `cmd_find_state` structure representing the target state of the command queue item.


---
### cmdq_get_source<!-- {{#callable:cmdq_get_source}} -->
The `cmdq_get_source` function retrieves the source command find state from a given command queue item.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure from which the source command find state is to be retrieved.
- **Control Flow**:
    - The function takes a single argument, `item`, which is a pointer to a `cmdq_item` structure.
    - It returns the address of the `source` member within the `cmdq_item` structure.
- **Output**:
    - A pointer to the `cmd_find_state` structure representing the source of the command queue item.


---
### cmdq_get_event<!-- {{#callable:cmdq_get_event}} -->
The `cmdq_get_event` function retrieves the `key_event` structure from the state of a given command queue item.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, representing an item in the command queue.
- **Control Flow**:
    - The function accesses the `state` member of the `cmdq_item` structure pointed to by `item`.
    - It then returns the address of the `event` member within the `state` structure.
- **Output**:
    - A pointer to a `key_event` structure, which is part of the state of the given command queue item.


---
### cmdq_get_current<!-- {{#callable:cmdq_get_current}} -->
The `cmdq_get_current` function retrieves the current command find state from a given command queue item.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure from which the current command find state is to be retrieved.
- **Control Flow**:
    - The function accesses the `state` member of the `cmdq_item` structure pointed to by `item`.
    - It then returns a pointer to the `current` member of the `cmdq_state` structure, which is part of the `state`.
- **Output**:
    - A pointer to the `current` member of the `cmdq_state` structure associated with the given `cmdq_item`.


---
### cmdq_get_flags<!-- {{#callable:cmdq_get_flags}} -->
The `cmdq_get_flags` function retrieves the flags from the state of a given command queue item.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, representing an item in the command queue.
- **Control Flow**:
    - The function accesses the `state` member of the `cmdq_item` structure pointed to by `item`.
    - It then retrieves the `flags` member from the `state` structure.
- **Output**:
    - The function returns an integer representing the flags of the command queue item's state.


---
### cmdq_new_state<!-- {{#callable:cmdq_new_state}} -->
The `cmdq_new_state` function creates and initializes a new command queue state structure with specified event and flags, and optionally copies the current command find state.
- **Inputs**:
    - `current`: A pointer to a `cmd_find_state` structure representing the current command find state, which may be copied into the new state if valid.
    - `event`: A pointer to a `key_event` structure representing the event that triggered the command, which is copied into the new state if provided.
    - `flags`: An integer representing flags that are set in the new state.
- **Control Flow**:
    - Allocate memory for a new `cmdq_state` structure and initialize its `references` to 1 and `flags` to the provided flags.
    - If the `event` pointer is not NULL, copy the event data into the new state's `event` field; otherwise, set the event key to `KEYC_NONE`.
    - If the `current` pointer is not NULL and represents a valid state, copy it into the new state's `current` field; otherwise, clear the `current` state.
- **Output**:
    - Returns a pointer to the newly created and initialized `cmdq_state` structure.


---
### cmdq_link_state<!-- {{#callable:cmdq_link_state}} -->
The `cmdq_link_state` function increments the reference count of a given command queue state and returns the updated state.
- **Inputs**:
    - `state`: A pointer to a `cmdq_state` structure representing the command queue state whose reference count is to be incremented.
- **Control Flow**:
    - The function takes a pointer to a `cmdq_state` structure as input.
    - It increments the `references` field of the `cmdq_state` structure by one.
    - The function then returns the updated `cmdq_state` pointer.
- **Output**:
    - The function returns the same `cmdq_state` pointer that was passed as input, with its `references` field incremented.


---
### cmdq_copy_state<!-- {{#callable:cmdq_copy_state}} -->
The `cmdq_copy_state` function creates a new command queue state by copying the current state and optionally using a provided current find state.
- **Inputs**:
    - `state`: A pointer to a `cmdq_state` structure representing the current command queue state to be copied.
    - `current`: A pointer to a `cmd_find_state` structure representing the current find state, which can be used to override the current state in the new command queue state.
- **Control Flow**:
    - Check if the `current` argument is not NULL.
    - If `current` is not NULL, call [`cmdq_new_state`](#cmdq_new_state) with `current`, the event from `state`, and the flags from `state` to create a new state.
    - If `current` is NULL, call [`cmdq_new_state`](#cmdq_new_state) with the current find state from `state`, the event from `state`, and the flags from `state` to create a new state.
- **Output**:
    - Returns a pointer to a newly created `cmdq_state` structure that is a copy of the provided state, potentially using a different current find state.
- **Functions called**:
    - [`cmdq_new_state`](#cmdq_new_state)


---
### cmdq_free_state<!-- {{#callable:cmdq_free_state}} -->
The `cmdq_free_state` function decrements the reference count of a `cmdq_state` structure and frees its memory if the reference count reaches zero.
- **Inputs**:
    - `state`: A pointer to a `cmdq_state` structure that represents the state to be freed.
- **Control Flow**:
    - Decrement the `references` field of the `state` structure.
    - Check if the `references` field is not zero; if so, return immediately without freeing.
    - If the `formats` field of the `state` is not NULL, call `format_free` to free the formats.
    - Free the memory allocated for the `state` structure.
- **Output**:
    - The function does not return any value; it performs memory management by freeing the `cmdq_state` structure if its reference count reaches zero.


---
### cmdq_add_format<!-- {{#callable:cmdq_add_format}} -->
The `cmdq_add_format` function adds a formatted string to the command queue state's format tree using a specified key.
- **Inputs**:
    - `state`: A pointer to a `cmdq_state` structure representing the command queue state to which the format will be added.
    - `key`: A constant character pointer representing the key under which the formatted string will be stored.
    - `fmt`: A constant character pointer representing the format string, similar to printf-style formatting.
    - `...`: A variable argument list that provides the values to be formatted according to the format string.
- **Control Flow**:
    - Initialize a variable argument list `ap` using `va_start` with `fmt` as the last fixed argument.
    - Use `xvasprintf` to allocate and format a string `value` using the format string `fmt` and the variable arguments `ap`.
    - End the variable argument list processing with `va_end`.
    - Check if the `formats` field of the `state` is `NULL`; if so, initialize it using `format_create`.
    - Add the formatted string `value` to the `formats` tree of the `state` using the provided `key`.
    - Free the allocated memory for `value` using `free`.
- **Output**:
    - The function does not return a value; it modifies the `formats` field of the `cmdq_state` structure by adding a new formatted entry.


---
### cmdq_add_formats<!-- {{#callable:cmdq_add_formats}} -->
The `cmdq_add_formats` function adds a set of formats from a `format_tree` to a `cmdq_state` structure, initializing the formats if they are not already set.
- **Inputs**:
    - `state`: A pointer to a `cmdq_state` structure where the formats will be added.
    - `ft`: A pointer to a `format_tree` structure containing the formats to be added.
- **Control Flow**:
    - Check if the `formats` field in the `state` structure is `NULL`.
    - If `formats` is `NULL`, initialize it using `format_create` with default parameters.
    - Merge the formats from the `format_tree` `ft` into the `state->formats` using `format_merge`.
- **Output**:
    - The function does not return a value; it modifies the `formats` field of the `cmdq_state` structure in place.


---
### cmdq_merge_formats<!-- {{#callable:cmdq_merge_formats}} -->
The `cmdq_merge_formats` function merges command-related formats from a command queue item into a given format tree.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, representing a command queue item that may contain a command and associated formats.
    - `ft`: A pointer to a `format_tree` structure, which is the target format tree where formats from the command queue item will be merged.
- **Control Flow**:
    - Check if the `cmd` field of the `item` is not NULL, indicating that the item has an associated command.
    - If a command is present, retrieve its entry using `cmd_get_entry` and add the command's name to the format tree `ft` with the key 'command'.
    - Check if the `formats` field of the `item`'s state is not NULL, indicating that there are additional formats to merge.
    - If additional formats are present, merge them into the format tree `ft` using `format_merge`.
- **Output**:
    - The function does not return a value; it modifies the `format_tree` pointed to by `ft` by adding or merging formats from the `cmdq_item`.


---
### cmdq_append<!-- {{#callable:cmdq_append}} -->
The `cmdq_append` function appends a command queue item to a client's command queue, updating references and logging the action.
- **Inputs**:
    - `c`: A pointer to a `client` structure, representing the client whose command queue the item will be appended to.
    - `item`: A pointer to a `cmdq_item` structure, representing the command queue item to be appended.
- **Control Flow**:
    - Retrieve the command queue associated with the client `c` using [`cmdq_get`](#cmdq_get) function.
    - Iterate over the linked list of `cmdq_item` starting from `item`.
    - For each `cmdq_item`, set its `next` pointer to `NULL` to detach it from the list.
    - If the client `c` is not `NULL`, increment its reference count.
    - Assign the client `c` to the `client` field of the `cmdq_item`.
    - Assign the retrieved queue to the `queue` field of the `cmdq_item`.
    - Insert the `cmdq_item` at the tail of the queue's list using `TAILQ_INSERT_TAIL`.
    - Log the action using `log_debug`.
    - Continue to the next `cmdq_item` in the list until all items are processed.
- **Output**:
    - Returns a pointer to the last `cmdq_item` in the queue after appending.
- **Functions called**:
    - [`cmdq_get`](#cmdq_get)
    - [`cmdq_name`](#cmdq_name)


---
### cmdq_insert_after<!-- {{#callable:cmdq_insert_after}} -->
The `cmdq_insert_after` function inserts a command queue item after a specified item in a command queue, updating client references and queue pointers accordingly.
- **Inputs**:
    - `after`: A pointer to the `cmdq_item` structure after which the new item(s) will be inserted.
    - `item`: A pointer to the `cmdq_item` structure that is to be inserted after the `after` item.
- **Control Flow**:
    - Retrieve the client and queue from the `after` item.
    - Enter a loop that continues until all items in the linked list starting from `item` are processed.
    - In each iteration, store the next item in the list in `next`, update the `next` pointer of the current `item` to point to the item following `after`, and update `after`'s `next` pointer to point to the current `item`.
    - If the client `c` is not NULL, increment its reference count and assign it to the current `item`'s client field.
    - Assign the queue to the current `item`'s queue field.
    - Insert the current `item` into the queue list after the `after` item using `TAILQ_INSERT_AFTER`.
    - Log a debug message indicating the insertion of the item.
    - Update `after` to the current `item` and move to the next item in the list by setting `item` to `next`.
- **Output**:
    - Returns a pointer to the last `cmdq_item` that was inserted.
- **Functions called**:
    - [`cmdq_name`](#cmdq_name)


---
### cmdq_insert_hook<!-- {{#callable:cmdq_insert_hook}} -->
The `cmdq_insert_hook` function inserts a command hook into the command queue for a given session, item, and current state, using a formatted name to identify the hook.
- **Inputs**:
    - `s`: A pointer to a `struct session`, representing the session in which the hook is to be inserted; if NULL, global session options are used.
    - `item`: A pointer to a `struct cmdq_item`, representing the command queue item where the hook will be inserted.
    - `current`: A pointer to a `struct cmd_find_state`, representing the current command find state.
    - `fmt`: A format string used to create the name of the hook.
- **Control Flow**:
    - Check if the current item's state has the CMDQ_STATE_NOHOOKS flag set; if so, return immediately.
    - Determine the options to use based on whether the session `s` is NULL or not.
    - Use `va_start` and `va_end` to handle the variable arguments and create a formatted hook name using `xvasprintf`.
    - Retrieve the options entry for the hook name; if it doesn't exist, free the name and return.
    - Log the hook execution and create a new command queue state with CMDQ_STATE_NOHOOKS flag to prevent updates to the current target or formats.
    - Add the hook name and arguments to the new state's format tree.
    - Iterate over the arguments and flags of the command, adding each to the new state's format tree.
    - Iterate over the options array items for the hook, creating and inserting new command queue items for each command list found.
    - Free the new state and the hook name.
- **Output**:
    - The function does not return a value; it modifies the command queue by inserting hooks based on the provided session, item, and current state.
- **Functions called**:
    - [`cmdq_new_state`](#cmdq_new_state)
    - [`cmdq_add_format`](#cmdq_add_format)
    - [`cmdq_get_command`](#cmdq_get_command)
    - [`cmdq_insert_after`](#cmdq_insert_after)
    - [`cmdq_append`](#cmdq_append)
    - [`cmdq_free_state`](#cmdq_free_state)


---
### cmdq_continue<!-- {{#callable:cmdq_continue}} -->
The `cmdq_continue` function clears the CMDQ_WAITING flag from a command queue item, allowing it to continue processing.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item whose waiting status is to be cleared.
- **Control Flow**:
    - The function accesses the `flags` field of the `cmdq_item` structure pointed to by `item`.
    - It performs a bitwise AND operation with the negation of `CMDQ_WAITING` to clear the waiting flag.
- **Output**:
    - The function does not return any value; it modifies the `flags` field of the `cmdq_item` structure in place.


---
### cmdq_remove<!-- {{#callable:cmdq_remove}} -->
The `cmdq_remove` function removes a command queue item from its queue, cleans up associated resources, and frees the memory allocated for the item.
- **Inputs**:
    - `item`: A pointer to the `cmdq_item` structure that represents the command queue item to be removed.
- **Control Flow**:
    - Check if the `client` field of the `item` is not NULL, and if so, call `server_client_unref` to decrease the reference count of the client.
    - Check if the `cmdlist` field of the `item` is not NULL, and if so, call `cmd_list_free` to free the command list associated with the item.
    - Call [`cmdq_free_state`](#cmdq_free_state) to free the state associated with the item.
    - Remove the item from its queue using `TAILQ_REMOVE`.
    - Free the memory allocated for the `name` field of the item.
    - Free the memory allocated for the `item` itself.
- **Output**:
    - The function does not return any value; it performs cleanup and deallocation of resources associated with the command queue item.
- **Functions called**:
    - [`cmdq_free_state`](#cmdq_free_state)


---
### cmdq_remove_group<!-- {{#callable:cmdq_remove_group}} -->
The `cmdq_remove_group` function removes all subsequent command queue items that belong to the same group as the specified item.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item whose group is to be removed.
- **Control Flow**:
    - Check if the `group` attribute of the `item` is zero; if so, return immediately as there is no group to remove.
    - Initialize `this` to the next item in the queue after `item` using `TAILQ_NEXT`.
    - Iterate over the queue starting from `this`, checking each item's `group` attribute.
    - If an item's `group` matches the `group` of the initial `item`, call [`cmdq_remove`](#cmdq_remove) to remove it from the queue.
    - Continue iterating until the end of the queue is reached.
- **Output**:
    - The function does not return a value; it modifies the command queue by removing items.
- **Functions called**:
    - [`cmdq_remove`](#cmdq_remove)


---
### cmdq_empty_command<!-- {{#callable:cmdq_empty_command}} -->
The `cmdq_empty_command` function is a placeholder command callback that always returns a normal command return value.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which represents an item in the command queue; it is marked as unused in this function.
    - `data`: A pointer to any additional data required by the command; it is marked as unused in this function.
- **Control Flow**:
    - The function is defined as static, meaning it is only accessible within the file it is defined in.
    - The function takes two parameters, both of which are marked as unused, indicating they are not utilized within the function body.
    - The function immediately returns the constant `CMD_RETURN_NORMAL`, indicating a normal command execution result.
- **Output**:
    - The function returns an `enum cmd_retval` value, specifically `CMD_RETURN_NORMAL`, which signifies a normal, non-error return status for a command.


---
### cmdq_get_command<!-- {{#callable:cmdq_get_command}} -->
The `cmdq_get_command` function creates a linked list of command queue items from a given command list and state, initializing each item with relevant command and state information.
- **Inputs**:
    - `cmdlist`: A pointer to a `cmd_list` structure representing the list of commands to be processed.
    - `state`: A pointer to a `cmdq_state` structure representing the current state of the command queue; if NULL, a new state is created.
- **Control Flow**:
    - Check if the first command in `cmdlist` is NULL; if so, return a callback item for an empty command.
    - If `state` is NULL, create a new command queue state and set a flag indicating it was created.
    - Iterate over each command in `cmdlist`, performing the following steps for each command:
    - Retrieve the command entry for the current command.
    - Allocate memory for a new `cmdq_item` and initialize its fields, including name, type, group, state, command list, and command.
    - Increment the reference count of `cmdlist`.
    - Log debug information about the created item.
    - Link the newly created item to the list of items, updating `first` and `last` pointers as necessary.
    - Move to the next command in `cmdlist`.
    - If a new state was created, free it after processing all commands.
    - Return the first item in the linked list of command queue items.
- **Output**:
    - Returns a pointer to the first `cmdq_item` in the linked list of command queue items created from the command list.
- **Functions called**:
    - [`cmdq_new_state`](#cmdq_new_state)
    - [`cmdq_link_state`](#cmdq_link_state)
    - [`cmdq_free_state`](#cmdq_free_state)


---
### cmdq_find_flag<!-- {{#callable:cmdq_find_flag}} -->
The `cmdq_find_flag` function locates a target based on a specified flag within a command queue item and updates the command find state accordingly.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item being processed.
    - `fs`: A pointer to a `cmd_find_state` structure where the result of the target finding operation will be stored.
    - `flag`: A pointer to a `cmd_entry_flag` structure that specifies the flag to be used for finding the target.
- **Control Flow**:
    - Check if the flag's value is zero; if so, call `cmd_find_from_client` to set the find state from the target client and return `CMD_RETURN_NORMAL`.
    - Retrieve the argument value associated with the specified flag using `args_get`.
    - Attempt to find the target using `cmd_find_target` with the retrieved value and flag specifications.
    - If `cmd_find_target` fails, clear the find state using `cmd_find_clear_state` and return `CMD_RETURN_ERROR`.
    - If successful, return `CMD_RETURN_NORMAL`.
- **Output**:
    - The function returns an `enum cmd_retval`, which is either `CMD_RETURN_NORMAL` if the target is successfully found or `CMD_RETURN_ERROR` if an error occurs during the target finding process.


---
### cmdq_add_message<!-- {{#callable:cmdq_add_message}} -->
The `cmdq_add_message` function logs a command message with optional user and key information based on the client and command queue state.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item containing the client, state, and command information.
- **Control Flow**:
    - Retrieve the client and state from the `cmdq_item` structure.
    - Generate a string representation of the command using `cmd_print`.
    - If a client is associated with the item, retrieve the UID of the peer and compare it with the current UID to determine the user name.
    - If the client has an associated session and a key event is present, log a message including the client name, user, key, and command.
    - If no key event is present, log a message including the client name, user, and command.
    - If no client is associated, log a message with just the command.
    - Free any dynamically allocated memory for the user and command strings.
- **Output**:
    - The function does not return a value; it logs a message to the server based on the command and client information.


---
### cmdq_fire_command<!-- {{#callable:cmdq_fire_command}} -->
The `cmdq_fire_command` function executes a command from a command queue item, handling client targeting, command execution, and error or hook processing.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item to be executed.
- **Control Flow**:
    - Retrieve the command name and associated state, command, arguments, and entry from the `cmdq_item`.
    - If configuration is finished, add a message for the command; log the command if the log level is greater than 1.
    - Determine if the command is a control command and set the appropriate flags.
    - Find the client associated with the command if not already set.
    - Check command entry flags to determine if a client can fail or if specific client flags ('c' or 't') are required, and find the target client accordingly.
    - Set the target client in the `cmdq_item`.
    - Find and set the source and target flags for the command using [`cmdq_find_flag`](#cmdq_find_flag).
    - Execute the command using the entry's `exec` function and store the return value.
    - If the command has an `AFTERHOOK` flag, determine the valid state and insert an after-hook using [`cmdq_insert_hook`](#cmdq_insert_hook).
    - Restore the original client in the `cmdq_item`.
    - If the command execution resulted in an error, insert a command-error hook and guard the command with an error message; otherwise, guard the command with an end message.
    - Return the command execution result.
- **Output**:
    - Returns an `enum cmd_retval` indicating the result of the command execution, which can be normal, error, or wait.
- **Functions called**:
    - [`cmdq_name`](#cmdq_name)
    - [`cmdq_add_message`](#cmdq_add_message)
    - [`cmdq_guard`](#cmdq_guard)
    - [`cmdq_find_flag`](#cmdq_find_flag)
    - [`cmdq_insert_hook`](#cmdq_insert_hook)


---
### cmdq_get_callback1<!-- {{#callable:cmdq_get_callback1}} -->
The `cmdq_get_callback1` function creates and initializes a new command queue item of type callback with a specified name, callback function, and associated data.
- **Inputs**:
    - `name`: A constant character pointer representing the name of the callback item.
    - `cb`: A function pointer of type `cmdq_cb` representing the callback function to be associated with the item.
    - `data`: A void pointer to any data that should be associated with the callback item.
- **Control Flow**:
    - Allocate memory for a new `cmdq_item` structure using `xcalloc` and assign it to `item`.
    - Format the `name` of the item using `xasprintf` to include the provided `name` and the memory address of the item.
    - Set the `type` of the item to `CMDQ_CALLBACK`.
    - Initialize the `group` field of the item to 0.
    - Create a new command queue state using [`cmdq_new_state`](#cmdq_new_state) with `NULL` parameters and assign it to the `state` field of the item.
    - Assign the provided callback function `cb` to the `cb` field of the item.
    - Assign the provided data pointer `data` to the `data` field of the item.
    - Return the initialized `cmdq_item` structure.
- **Output**:
    - Returns a pointer to a newly created `cmdq_item` structure initialized as a callback item.
- **Functions called**:
    - [`cmdq_new_state`](#cmdq_new_state)


---
### cmdq_error_callback<!-- {{#callable:cmdq_error_callback}} -->
The `cmdq_error_callback` function logs an error message associated with a command queue item and then frees the allocated memory for the error message.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item associated with the error.
    - `data`: A pointer to a character string containing the error message to be logged.
- **Control Flow**:
    - The function retrieves the error message from the `data` pointer.
    - It calls [`cmdq_error`](#cmdq_error) to log the error message associated with the `cmdq_item`.
    - The function frees the memory allocated for the error message using `free`.
    - Finally, it returns `CMD_RETURN_NORMAL` to indicate normal completion.
- **Output**:
    - The function returns an `enum cmd_retval` value, specifically `CMD_RETURN_NORMAL`, indicating that the callback completed without any issues.
- **Functions called**:
    - [`cmdq_error`](#cmdq_error)


---
### cmdq_get_error<!-- {{#callable:cmdq_get_error}} -->
The `cmdq_get_error` function creates a command queue item that represents an error callback with a specified error message.
- **Inputs**:
    - `error`: A constant character pointer representing the error message to be used in the error callback.
- **Control Flow**:
    - The function calls `xstrdup` to duplicate the error message string, ensuring it is safely allocated.
    - It then calls `cmdq_get_callback` with `cmdq_error_callback` as the callback function and the duplicated error message as the data.
    - The `cmdq_get_callback` function returns a `cmdq_item` structure, which is then returned by `cmdq_get_error`.
- **Output**:
    - The function returns a pointer to a `cmdq_item` structure, which represents a command queue item configured to handle an error callback with the provided error message.


---
### cmdq_fire_callback<!-- {{#callable:cmdq_fire_callback}} -->
The `cmdq_fire_callback` function executes a callback function associated with a command queue item.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, which contains the callback function and its associated data to be executed.
- **Control Flow**:
    - The function retrieves the callback function (`cb`) and data (`data`) from the `cmdq_item` structure pointed to by `item`.
    - It then calls the callback function, passing the `cmdq_item` and its associated data as arguments.
    - The return value of the callback function is returned as the result of `cmdq_fire_callback`.
- **Output**:
    - The function returns an `enum cmd_retval`, which is the result of executing the callback function associated with the `cmdq_item`.


---
### cmdq_next<!-- {{#callable:cmdq_next}} -->
The `cmdq_next` function processes the next item in a command queue for a given client, executing commands or callbacks and handling their results.
- **Inputs**:
    - `c`: A pointer to a `struct client` representing the client whose command queue is to be processed.
- **Control Flow**:
    - Retrieve the command queue associated with the client using [`cmdq_get`](#cmdq_get) and get the queue's name using [`cmdq_name`](#cmdq_name).
    - Check if the queue is empty; if so, log a debug message and return 0.
    - Check if the first item in the queue has the `CMDQ_WAITING` flag set; if so, log a debug message and return 0.
    - Enter a loop to process each item in the queue until it is empty.
    - For each item, check if it has the `CMDQ_WAITING` flag set; if so, log a debug message and exit the loop.
    - If the item has not been fired (i.e., does not have the `CMDQ_FIRED` flag), set its time and number, then execute it based on its type (command or callback).
    - If executing a command returns an error, remove subsequent commands in the same group using [`cmdq_remove_group`](#cmdq_remove_group).
    - If executing a command returns `CMD_RETURN_WAIT`, set the `CMDQ_WAITING` flag and exit the loop.
    - Increment the count of processed items and remove the item from the queue.
    - Log a debug message indicating the queue is empty and return the count of processed items.
- **Output**:
    - The function returns the number of items processed from the command queue.
- **Functions called**:
    - [`cmdq_get`](#cmdq_get)
    - [`cmdq_name`](#cmdq_name)
    - [`cmdq_fire_command`](#cmdq_fire_command)
    - [`cmdq_remove_group`](#cmdq_remove_group)
    - [`cmdq_fire_callback`](#cmdq_fire_callback)
    - [`cmdq_remove`](#cmdq_remove)


---
### cmdq_running<!-- {{#callable:cmdq_running}} -->
The `cmdq_running` function retrieves the currently running command queue item for a given client, if it is not in a waiting state.
- **Inputs**:
    - `c`: A pointer to a `client` structure, representing the client for which the command queue is being queried.
- **Control Flow**:
    - Retrieve the command queue associated with the client using `cmdq_get(c)`.
    - Check if the current item in the queue is `NULL`; if so, return `NULL`.
    - Check if the current item's flags include `CMDQ_WAITING`; if so, return `NULL`.
    - If neither condition is met, return the current item in the queue.
- **Output**:
    - Returns a pointer to the currently running `cmdq_item` for the specified client, or `NULL` if there is no running item or if the item is waiting.
- **Functions called**:
    - [`cmdq_get`](#cmdq_get)


---
### cmdq_guard<!-- {{#callable:cmdq_guard}} -->
The `cmdq_guard` function writes a formatted control message to a client if the client is in control mode.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure, representing a command queue item that contains information about the client and command execution context.
    - `guard`: A constant character pointer representing the guard string to be included in the control message.
    - `flags`: An integer representing additional flags to be included in the control message.
- **Control Flow**:
    - Retrieve the client associated with the command queue item from the `item` structure.
    - Retrieve the time and number associated with the command queue item from the `item` structure.
    - Check if the client is not NULL and if the client is in control mode by checking the `CLIENT_CONTROL` flag.
    - If the client is in control mode, call `control_write` to send a formatted message including the guard string, time, number, and flags to the client.
- **Output**:
    - The function does not return a value; it performs an action by potentially writing a message to a client.


---
### cmdq_print_data<!-- {{#callable:cmdq_print_data}} -->
The `cmdq_print_data` function sends a message to a client using the provided event buffer.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item, which contains information about the client to which the message will be sent.
    - `evb`: A pointer to an `evbuffer` structure that contains the data to be printed to the client.
- **Control Flow**:
    - The function calls `server_client_print` with the client from the `cmdq_item`, a flag set to 1, and the provided `evbuffer` to print the data.
- **Output**:
    - The function does not return any value; it performs an action by printing data to a client.


---
### cmdq_print<!-- {{#callable:cmdq_print}} -->
The `cmdq_print` function formats a message and sends it to be printed by a command queue item.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item that will handle the printed message.
    - `fmt`: A format string similar to `printf` that specifies how the subsequent arguments are converted for output.
    - `...`: A variable number of arguments that are formatted according to the format string `fmt`.
- **Control Flow**:
    - Create a new `evbuffer` object to hold the formatted message.
    - Check if the `evbuffer` was successfully created; if not, terminate the program with an out-of-memory error.
    - Initialize a `va_list` to handle the variable arguments and use `evbuffer_add_vprintf` to format and add the message to the `evbuffer`.
    - End the use of the `va_list`.
    - Call [`cmdq_print_data`](#cmdq_print_data) to send the formatted message in the `evbuffer` to the specified command queue item for printing.
    - Free the `evbuffer` to release the allocated memory.
- **Output**:
    - The function does not return a value; it outputs the formatted message to the command queue item for printing.
- **Functions called**:
    - [`cmdq_print_data`](#cmdq_print_data)


---
### cmdq_error<!-- {{#callable:cmdq_error}} -->
The `cmdq_error` function logs and handles error messages for a command queue item, adjusting the output based on the client's state and session.
- **Inputs**:
    - `item`: A pointer to a `cmdq_item` structure representing the command queue item associated with the error.
    - `fmt`: A format string for the error message, similar to printf-style formatting.
    - `...`: A variable number of arguments to be formatted according to the format string `fmt`.
- **Control Flow**:
    - Initialize a variable argument list `ap` and format the error message into `msg` using `xvasprintf` with the provided format string and arguments.
    - Log the error message using `log_debug`.
    - Check if the client `c` associated with the `cmdq_item` is `NULL`.
    - If `c` is `NULL`, retrieve the source file and line number of the command and add the error message to the configuration causes using `cfg_add_cause`.
    - If the client `c` is not `NULL` and either has no session or is in control mode, add the message to the server and handle UTF-8 sanitization if necessary.
    - If the client is in control mode, write the message using `control_write`; otherwise, use `file_error` to display the message.
    - Set the client's return value to 1 if applicable.
    - If the client has a session, capitalize the first character of the message and set it as a status message using `status_message_set`.
    - Free the allocated memory for `msg`.
- **Output**:
    - The function does not return a value; it performs logging and error handling actions based on the state of the client and command.


